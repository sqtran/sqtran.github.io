---
title: OpenShift Image Mirror to Artifactory Revisited
date: 2022-12-07
tags: artifactory oc-mirror ocp
---

## Background
This is revisiting a [previous post](2019-06-19-Openshift-Image-Mirror-Artifactory.md) about mirroring images to external registries.  I'm introducing a few new commands that will help if your OpenShift registry is not publicly exposed.


### Prerequisites:

You're going to need access to the `oc` command line tool, and you'll need an account with significant permissions in order to run the `oc debug` command.

You'll also need credentials to be able to push to your external registry.


### Steps

Since the OpenShift internal registry is not exposed, the only way to access it is within the cluster.  In order to do that, we'll run a `debug` pod from within the cluster.


```bash
oc get nodes
oc debug nodes/<worker_node_name>
```

You'll get a helpful message about running `chroot /host` in order to run certain binaries.  This wasn't necessary in my cluster, but it may vary for you.

The `podman` tool is also available by default (at least in my 4.10 test cluster).  You'll need to log into both the openshift registry and the external registry.

```bash
podman login <internal registry>
podman login <external registry>
```

Once you have the credentials set up, you would run the `oc image mirror <src_image>:<tag> <dest_image>:<tag>` command to move the images over.

```bash
oc image mirror image-registry.openshift-image-registry.svc:5000/steve-nonprod/myapp:latest artifactory-url.example.com/demos/myapp:1.0
```

Tack on the `--insecure` flag if you're trying to connect via SSL to a non-publicly trusted certificate.

That's it!  It seems a little bit complicated but all the tools you need were available from within your cluster.