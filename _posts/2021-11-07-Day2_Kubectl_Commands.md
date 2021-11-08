---
title: Day2 Kubectl Commands
date: 2021-11-07
tags: ocp
---

## Background
Here's a note to self about the various `oc` or `kubectl` commands that I keep forgetting about.  I call them my Day2 commands because these are more useful once you have actual workloads in your cluster, and you need finer grained navigation of the kubernetes objects.

### Finding and Cleaning up Pods

Running an `oc get pods` command returns a list of all pods, in all various lifecycle phases.  It's sort of confusing to see a pod with status `Completed`, but kubernetes references it as being in a `Succeeded` state.  You might need to read up the documentation if the command you're running doesn't exactly line up with the statuses being presented.  https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/

Nonetheless, the command to find these is straightforward.

```bash
# This will get you all the pods that are "done" and won't be restarted
oc get pods --field-selector=status.phase==Succeeded

# If you want to delete them, just run it with the delete command
oc delete pods --field-selector=status.phase==Succeeded

# Sometimes pods can be reported as `Completed` but its status is `Failed.
oc get pods --field-selector=status.phase==Failed
```


### Finding and Cleaning up PipelineRuns

When using Tekton Pipelines, a lot of failed/completed `PipelineRuns` will be collected over the next few days, especially at the beginning when first developing and testing the pipeline itself.  There might be a more elegant way to do this, but just in case there isn't, there's always the good ole brute force way using bash-fu.

```bash
oc get pipelinerun | grep "Failed" | awk '{print $1}' | xargs oc delete pipelinerun
```

This could get really fancy with timestamp checks, so pipelineruns older than a week (or whatever useful duration you decide) can be pruned.