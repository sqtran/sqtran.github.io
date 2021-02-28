---
layout: single
title: OpenShift Image Management
date: 2019-06-13
tags: image ocp
---

In the containerized world, we no longer just move build artifacts (EARs, WARs, JARs, etc.) between servers.  We package those build artifacts along with runtime configurations and dependent libraries into self-contained immutable images, so that we have control over the entire application.  Everything we need is packaged in the image, so that running our containers is repeatable, to a certain degree.

## Builds

We build our images using `Docker`, `Buildah`, or whatever tool you like, and we need a place to publish our images for others to use.  These places are known as container image registries, and they come in many flavors.  We won't be discussing the various flavors in this article, but most people are probably familiar with `Docker`, and their publicly available registry at [Docker Hub](https://hub.docker.com/).  If you're coming from the Software Development world, you may or may not be surprised that `Artifactory` and `Nexus` can also be used as container image registries.

Moving these images from one registry to another becomes a new challenge.  Technically, if using `Docker`, you can `docker export` and `docker import` but that requires having an intermediary machine with a `Docker` daemon running.  There's a new movement to get away from the `Docker` daemon, because it requires running as root, but we'll save that discussion for another day.

## Tools
There's a tool called `skopeo` that can be used to manipulate images, which includes copying them between registries.   It's a pretty robust tool that does a lot.  You can check out their [Github project](https://github.com/containers/skopeo) for more information.  Using their tool requires having their tool installed on whatever box you're running from.  I don't have a problem with this, but I find it redundant if you're already on Openshift.

With Openshift 3.11 *(or newer)*, the command `oc image mirror` can steam images directly from one registry to another, without the need of `skopeo`, `Docker`, or any other external tools.  Although I haven't fully pushed its limits yet, my transfer rates have been pretty quick.  I can also do this without having `Docker` installed, or at least stopped.  Refer to the official [documentation](https://docs.openshift.com/container-platform/3.11/dev_guide/managing_images.html) for more details.
