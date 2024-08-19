---
title: EAP Boot Time Configuration
date: 2024-08-19
tags: EAP
---

## Configure EAP Configurations at Boot Time

This example demonstrates how to dynamically configure properties inside an EAP configuration file.  Normally this is trivial if the properties can be defined ahead of time (such as in a properties file or as ENV variables), but this set up allows the properties to be retrieved/defined at runtime.  This provides the ability to store the properties in an external system or tool.  For example, adjust the logging level based on a configuration stored in another tool, or retrieve sensitive strings stored in an external Secrets Management system.


## Example

Let's configure the logging level as an example, but it can be adapted for any other property within the XML file.

Your standalone-openshift.xml file has been parameterized with the following ENV named `CUSTOM_LOG_LEVEL`.  Note we're deploying this on OpenShift, but the same set up could be done on a non-OpenShift deployment as well.


```xml
  <root-logger>
    <level name="${CUSTOM_LOG_LEVEL}"/>
      <handlers>
          <handler name="CONSOLE"/>
      </handlers>
  </root-logger>
```

Only two files are needed to configure this - A JBoss CLI script that interacts with EAP, and a shell script that collects your dynamic variables.

With this example being on OpenShift, they are embedded in a `ConfigMap` that will get projected into the running container.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postconfigure-bash
  namespace: eap-demo
data:
  extensions.cli: |

    ## This sets up only one variable, but could be rewritten to loop through multiple variables

    embed-server --std-out=echo  --server-config=standalone-openshift.xml
    /system-property=CUSTOM_LOG_LEVEL:add(value="${MOUNTED_LOG_LEVEL}")
    quit

  postconfigure.sh: >
    #!/usr/bin/env bash

    set -x

    echo "Executing postconfigure.sh"

    ## Could use a bash loop to programmatically collect all the variables in this directory and export them as ENVs
    export MOUNTED_LOG_LEVEL=$(cat /tmp/mock-azure-mount/CUSTOM_LOG_LEVEL)

    ## Environment variables are not resolvable in the jboss-cli by default,
    ## Enable it following this example - https://mirocupak.com/using-environment-variables-in-jboss-cli/
    sed -i "s/<resolve-parameter-values>false<\/resolve-parameter-values>/<resolve-parameter-values>true<\/resolve-parameter-values>/" $JBOSS_HOME/bin/jboss-cli.xml

    ## Must write to tmp b/c most of the filesystem is read only
    printenv > /tmp/env.properties

    $JBOSS_HOME/bin/jboss-cli.sh --file=$JBOSS_HOME/extensions/extensions.cli --properties=/tmp/env.properties

```

This ConfigMap is a stand-in for an external system that would hold the real dynamic values.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-projection
  namespace: eap-demo
data:
  CUSTOM_LOG_LEVEL: TRACE
binaryData: {}
immutable: false
```

And finally, the `Deployment` (yes, this is a `DeploymentConfig` because this is an older example).

```yaml
kind: DeploymentConfig
apiVersion: apps.openshift.io/v1
metadata:
  name: eap-app
  namespace: eap-demo
  labels:
    application: eap-app
    template: eap74-basic-s2i
    xpaas: 7.4.0
spec:
  strategy:
    type: Recreate
    recreateParams:
      timeoutSeconds: 600
    resources: {}
    activeDeadlineSeconds: 21600
  triggers:
    - type: ImageChange
      imageChangeParams:
        automatic: true
        containerNames:
          - eap-app
        from:
          kind: ImageStreamTag
          namespace: eap-demo
          name: 'eap-app:latest'
        lastTriggeredImage: >-
          image-registry.openshift-image-registry.svc:5000/eap-demo/eap-app@sha256:5adc05d56a06fa00d607ab0fa9c2acd81d7549f05e15ef8d79d663fb8c6631a4
    - type: ConfigChange
  replicas: 1
  revisionHistoryLimit: 10
  test: false
  selector:
    deploymentConfig: eap-app
  template:
    metadata:
      name: eap-app
      creationTimestamp: null
      labels:
        application: eap-app
        com.company: Red_Hat
        com.redhat.component-name: EAP
        com.redhat.component-type: application
        com.redhat.component-version: '7.4'
        com.redhat.product-name: Red_Hat_Runtimes
        com.redhat.product-version: 2021-Q2
        deploymentConfig: eap-app
    spec:
      volumes:
        - name: jboss-cli
          configMap:
            name: postconfigure-bash
            defaultMode: 493
        - name: myenv
          configMap:
            name: test-projection
            defaultMode: 493
      containers:
        - resources:
            limits:
              memory: 1Gi
          readinessProbe:
            exec:
              command:
                - /bin/bash
                - '-c'
                - /opt/eap/bin/readinessProbe.sh
            initialDelaySeconds: 10
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          terminationMessagePath: /dev/termination-log
          name: eap-app
          livenessProbe:
            exec:
              command:
                - /bin/bash
                - '-c'
                - /opt/eap/bin/livenessProbe.sh
            initialDelaySeconds: 60
            timeoutSeconds: 1
            periodSeconds: 10
            successThreshold: 1
            failureThreshold: 3
          env:
            - name: JGROUPS_PING_PROTOCOL
              value: dns.DNS_PING
            - name: OPENSHIFT_DNS_PING_SERVICE_NAME
              value: eap-app-ping
            - name: OPENSHIFT_DNS_PING_SERVICE_PORT
              value: '8888'
            - name: MQ_CLUSTER_PASSWORD
              value: dkVnQYPD
            - name: MQ_QUEUES
            - name: MQ_TOPICS
            - name: JGROUPS_CLUSTER_PASSWORD
              value: RvDNIDnQ
            - name: AUTO_DEPLOY_EXPLODED
              value: 'false'
            - name: ENABLE_GENERATE_DEFAULT_DATASOURCE
              value: 'false'
          ports:
            - name: jolokia
              containerPort: 8778
              protocol: TCP
            - name: http
              containerPort: 8080
              protocol: TCP
            - name: ping
              containerPort: 8888
              protocol: TCP
          imagePullPolicy: Always
          volumeMounts:
            - name: jboss-cli
              mountPath: /opt/eap/extensions
            - name: myenv
              mountPath: /tmp/mock-azure-mount
          terminationMessagePolicy: File
          image: >-
            image-registry.openshift-image-registry.svc:5000/eap-demo/eap-app@sha256:5adc05d56a06fa00d607ab0fa9c2acd81d7549f05e15ef8d79d663fb8c6631a4
      restartPolicy: Always
      terminationGracePeriodSeconds: 75
      dnsPolicy: ClusterFirst
      securityContext: {}
      schedulerName: default-scheduler
```