---
title: Internal Image Registry
date: 2023-07-06
tags: ocp, image-registry
---

## Background
I've been writing a lot of code lately, and to shorten the developer inner loop, I've been building locally too.  That means I've been building images with Podman but I need a place to put them so I can deploy and test them.  Quay.io is free and open to all, but I don't want to clutter that space with my dev builds.  What I like to do is use the OpenShift internal registry for development purposes, so I'll demonstrate how to do that here.


## Instructions
The first thing we need to do is expose the image registry by creating a Route.  The Image Registry is managed by an Operator, so the only way to do that is to patch the ImageRegistry object's configurations.

```bash
oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
```

Once that change has taken affect, a `default-route` will be created.  You can use this snippet to log into the registry with Podman.  This assumes you're already logged in.

```bash
podman login $(oc get route default-route -o jsonpath='{.spec.host}' -n openshift-image-registry) -u $(oc whoami) -p $(oc whoami -t)
```

Note: If you're planning on doing this for a non-human process, such as a pipeline - you'll need to create a service account token

```bash
# This will give you a token that's valid for 1 year, but the default is 1 hour
oc create token builder --duration 525600m
```


Use this to grab the route's hostname.
```bash
oc get route default-route -o jsonpath='{.spec.host}' -n openshift-image-registry
```

Assuming you've already built the image, use that hostname in the script above to retag your image.

```bash
podman tag <existing tag> <new tag>
```

Note that <new tag> will looklike the following:
```text
<route_output>/<namespace>/<image-name>:<version>
```

Now all you gotta do is podman push it!