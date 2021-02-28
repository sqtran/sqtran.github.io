---
layout: single
title: OSSM Setup
date: 2020-10-28
tags: ossm istio ocp openshift
---

Red Hat's Istio offering is known as OpenShift Service Mesh, or `OSSM` for short.  Just like all their project vs product offerings, Red Hat expands upon the open source community by adding additional features to make OSSM a more complete offering.  Back in February, I gave a talk at Red Hat Microservices Day in Atlanta, which was recorded but I'll have to find a link for that later.  Part of the talk was a really great demo about how easy it was to instrument existing applications.  I used the Google Microservices Demo, which is a mock Hipster Shop.

Almost 6 months later, I wanted to revisit that demo but found myself unable to without the assistance of my notes.  I didn't bother writing instructions at the time because I thought it was "just too easy", and nobody would really need help.  Well, I spent a little bit of time now to document that process, because I'm pretty sure I will forget how to do this again.


## Prerequisites

I am using an OCP 4 cluster, which I have full cluster administrative permissions to.  This is required because I need to install 3 Operators from the Operator Hub.

- Red Hat OpenShift Service Mesh
- Red Hat OpenShift Jaeger
- Kiali Operator


## Configuration

### Namespace
The default namespace is assumed to be `istio-system`, so that's a safe place to start.  Otherwise, you'll need to change the default locations in dependent service configuration files.  There aren't too many different places, but I prefer to customize only when necessary, so the default is good enough for now.

### Control Plane
The Control Plane defaults with a name of `basic-install` which is good enough for now.  This one is safe to customize, but I just clicked to continue with all the defaults.

### Member Roll
The ServiceMesh MemberRoll defines who is in your mesh.  It defaults with 2 sample namespaces, but you can immediately delete those and add the namespace of projects you want in your mesh.  It's okay if you don't know yet, as this can be modified at anytime.


## Demo Application
The Google Demo is located on Github, but here's the K8S manifest that contains all the pieces we'll need to deploy.  Take a look at the different Services and Deployments before blindly running any commands.
https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml



```bash
oc new-project shopdemo

oc apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/microservices-demo/master/release/kubernetes-manifests.yaml

for i in $(oc get deployment -o name); do
  oc patch $i -p '{"spec": {"template": {"metadata": {"annotations": {"sidecar.istio.io/inject": "true"}}}}}';
done;

oc delete pod --all
```

I looped through all the deployments to add an annotation, which is specific to OpenShift so that the istio-proxy sidecar gets automatically injected.

Give it a minute to bounce all the pods and then you're done ! .... almost


### Routing

Internally, everything should be up and running, but there's no way to access this service externally.  Because this is now inside the Mesh, you'll need to create Istio specific objects in order to provide ingress into the Mesh.


A `Gateway` and a `VirtualService` is needed, and an example is provided below.
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: hipstershop
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: hipstershop
spec:
  hosts:
  - "*"
  gateways:
  - hipstershop
  http:
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: frontend-external
```

Lastly, you'll need to build a route, or use the existing route.  The existing route can be found by running the following.

```bash
oc get routes -n istio-system  | grep istio-ingressgateway
```

If you want a friendlier route name, you can always create your own.  This is just an example though, and the `VirutalService` and the `Gateway` is accepting all traffic, so this wouldn't work if you have realistic Mesh configurations.

Access the Route and see the newly deployed Hipster Shop demo!
