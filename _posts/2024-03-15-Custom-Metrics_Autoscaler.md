---
title: Custom Metrics Autoscaling with KEDA
date: 2024-03-15
tags: quarkus ocp keda
---

## Background
The default Kubernetes Pod Autoscaler allows users to monitor and horizontally scale out pods based on memory and CPU usage.  CPU and Memory is more of a proxy metric, and what developers, or sophisticated applications, need are more application-centric metrics that trigger scaling events.  The upstream KEDA project is an extension of the Horizontal Pod Autoscaler that does just that.  KEDA allows Kubernetes to use any custom-exposed metric to make determinations on when to scale.



## Installation

### Platform Requirements

OpenShift provides the Custom Metric Autoscaler Operator, which is based upon the upstream KEDA project. This is a platform/cluster level operation that requires administrator permissions to perform.

The Operator simplifies the installation, configuration, and management of KEDA-related resources within a Kubernetes cluster. It leverages the Operator pattern to streamline the deployment process and handle the necessary interactions with the Kubernetes API server.

Installing the Operator allows you to instantiate a Kubernetes CustomResource for a `KedaController`.

Once the `KedaController` is created, additional configurations are required:

- ServiceAccount to allow the Metrics Server access to the application’s namespace to scrape metrics
- TriggerAuthentication that links the ServiceAccount token with the ScaledJob that the application team will create (see below)
- Roles that allows the Prometheus Scraping process to read/access the application team’s namespace
- ServiceMonitor specifies what workloads are providing metrics and what endpoint those metrics are exposed on
- ConfigMap that enables userWorkLoads to be scrape for metrics



### Application Requirements

KEDA can connect to other types of custom metrics, such as a Kafka Topic, but our demo focuses on consuming prometheus-formatted metrics from the application itself.  With Java applications, there already exist common libraries that specialize in this. One set of specifications, known as MicroProfile, was created specifically for building efficient and resilient microservices. One particular library that implements the Microprofile Metrics specification is called `smallrye-metrics`, but the latest and greatest library recommended is `micrometer-registry-prometheus`.

After the prometheus-formatted metrics are available from the application, another resource is required.

- ScaledObject defines the rules for scaling a deployment.

The ScaledObject uses PromQL (Prometheus Query Language) to specify the metric to capture, and the threshold to trigger on.

A sample image is available for you at [hello-quarkus](https://quay.io/stran/hello-quarkus).  Source code is available as well in this Git [repo](https://github.com/sqtran/hello-quarkus).



## YAMLs
Here are all the YAML files needed mentioned above.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: thanos-metrics-reader
  namespace: steve-keda
rules:
  - verbs:
      - get
    apiGroups:
      - ''
    resources:
      - pods
  - verbs:
      - get
      - list
      - watch
    apiGroups:
      - metrics.k8s.io
    resources:
      - pods
      - nodes
---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: thanos-metrics-reader
  namespace: steve-keda
subjects:
  - kind: ServiceAccount
    name: thanos
    namespace: steve-keda
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: thanos-metrics-reader
---

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: steve-keda
  namespace: steve-keda
spec:
  endpoints:
    - interval: 5s
      path: /q/metrics
      port: 8080-tcp
      scheme: http
  namespaceSelector: {}
  selector:
    matchLabels:
      app: hello-quarkus
---

apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: prometheus-scaledobject
  namespace: steve-keda
  finalizers:
    - finalizer.keda.sh
  labels:
    scaledobject.keda.sh/name: prometheus-scaledobject
spec:
  maxReplicaCount: 10
  minReplicaCount: 1
  pollingInterval: 5
  scaleTargetRef:
    kind: deployment
    name: hello-quarkus
  triggers:
    - authenticationRef:
        name: keda-trigger-auth-prometheus
      metadata:
        authModes: bearer
        metricName: http_server_active_requests
        namespace: steve-keda
        query: sum(http_server_active_requests{job="optionally-filter-on-your-deployment"})
        serverAddress: 'https://thanos-querier.openshift-monitoring.svc.cluster.local:9092'
        threshold: '5'
      type: prometheus
---

kind: KedaController
apiVersion: keda.sh/v1alpha1
metadata:
  name: keda
  namespace: openshift-keda
spec:
  admissionWebhooks:
    logEncoder: console
    logLevel: info
  metricsServer:
    logLevel: '0'
  operator:
    logEncoder: console
    logLevel: info
  serviceAccount: null
  watchNamespace: ''
```



This last piece requires the creation of a Service Account.


```bash
oc create serviceaccount thanos -n steve-keda
oc adm policy add-role-to-user thanos-metrics-reader -z thanos --role-namespace=steve-keda

oc describe sa thanos -n steve-keda
```

Grab the service account token id and plug it into the follow `TriggerAuthentication` yaml.


```yaml
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  finalizers:
    - finalizer.keda.sh
  name: keda-trigger-auth-prometheus
  namespace: steve-keda
spec:
  secretTargetRef:
    - key: token
      name: thanos-token-$TOKEN_ID
      parameter: bearerToken
    - key: ca.crt
      name: thanos-token-$TOKEN_ID
      parameter: ca
```

Note - make sure you grab the "token" and not the "dockercfg" ID. They look similar but are not the same.


## Test it out

You'll need a load test tool to generate enough HTTP traffic to test your autoscaling.  I use the follow script that spins up another container with the `work2` application.  You can use any tool you want though.

```bash
#!/bin/bash

HOSTNAME=hello-quarkus-steve-keda.apps.OCP_DOMAIN
ENDPOINT=/varsleep?min=1000\&max=2000
# -R = TPS
# -t = threads
# -c = TCP connections to keep open
# -d = duration (recommend min 30s)

podman run --rm cylab/wrk2 -R 500 -t 4 -c 20 -d 30s https://$HOSTNAME$ENDPOINT
```


## Permissions for non-clusteradmins

The creation of ServiceMonitors requires additional permissions that regular users probably don't have.  You'll need to create a new role and binding that grants users access to these objects.


```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: manage-servicemonitors
  namespace: myapplication
rules:
  - verbs:
      - '*'
    apiGroups:
      - monitoring.coreos.com
    resources:
      - servicemonitors

```

## TLS Configurations

 Some applications may not be configured to listen on the standard HTTP port, so you may need to configure your ServiceMonitor for HTTPS.  That's easily done by setting the scheme to "https" and specifying a targetPort (if not 443).

```yaml
  - interval: 5s
    path: /q/metrics
    port: 8443-TCP
    scheme: https
```


 Another noteworthy scenario is when your application is doing client certificate validation, so your ServiceMonitor needs to send a certificate to your service to grab the /metrics endpoint.  In this case, you can configure settings in the "tlsConfig" stanza of the ServiceMonitor.  Here's an example.


```yaml
- interval: 5s
  path: /q/metrics
  targetPort: 10443
  scheme: https
  tlsConfig:
    cert:
      secret:
        key: tls.crt
        name: clientcert
    keySecret:
      key: tls.key
      name: clientcert
```

It references a Secret named "clientcert" - which contains a certificate and key in **PEM** format.

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: clientcert
data:
  tls.crt: <certificate_material_here>
  tls.key: <key_material_here>
type: kubernetes.io/tls
```


## Bonus Automation of Operator Installation

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-keda
---

apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/openshift-custom-metrics-autoscaler-operator.openshift-keda: ''
  name: openshift-custom-metrics-autoscaler-operator
  namespace: openshift-keda
spec:
  channel: stable
  installPlanApproval: Automatic
  name: openshift-custom-metrics-autoscaler-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
  startingCSV: custom-metrics-autoscaler.v2.11.2-322
---

apiVersion: operators.coreos.com/v1alpha2
kind: OperatorGroup
metadata:
  generateName: openshift-keda-
  annotations:
    olm.providedAPIs: 'ClusterTriggerAuthentication.v1alpha1.keda.sh,KedaController.v1alpha1.keda.sh,ScaledJob.v1alpha1.keda.sh,ScaledObject.v1alpha1.keda.sh,TriggerAuthentication.v1alpha1.keda.sh'
  name: openshift-keda
  namespace: openshift-keda
spec: {}
```

If you want to see an example of this configured in OpenShift GitOps (ArgoCD), visit https://github.com/sqtran/argocd-demo


## Tips and Tricks

If the auto-scaler doesn't appear to be working, check the Horizontal Pod Autoscaler Tab and verify that the PromQL query in your ScaledObject is correct.  The active count should fluctuate under load, during the specified interval (make sure you're putting it under load for at least that scrape interval).

If unsure about query syntax, run the PromQL query in the Metrics Tab.  Any ad-hoc query can be entered there, to verify the syntax or metric name is correct.

Make sure the filter is set `{job="your_deployment"}` to avoid metric name collision when aggregating results.  This is important when multiple applications are producing the same metric in the same namespace.

Pause the ScaledObject if you want to test the application's behavior without auto-scaling.  There are two annotations to set on the ScaledObject to either pause all autoscaling, or to statically set the number of replicas to a specific number.  They are  and `autoscaling.keda.sh/paused: "true"`, and `autoscaling.keda.sh/paused-replicas: "n"`, respectively, where n is the number of replicas to hard-code to.


## Kubernetes Update

Kubernetes recently changed how ServiceAccount are created, so the ServiceAccount Secrets are no longer generated automatically.  They will need to be created manually with the following commands.

```bash
oc create token thanos --duration=31536000s # 2 year expiration in seconds
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: yoursecret
  namespace: steve-keda
type: Opaque
data:
  bearerToken: <your token here>
```

```yaml
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  finalizers:
    - finalizer.keda.sh
  name: keda-trigger-auth-prometheus
  namespace: steve-keda
spec:
  secretTargetRef:
    - key: bearerToken
      name: yoursecret
      parameter: bearerToken
```