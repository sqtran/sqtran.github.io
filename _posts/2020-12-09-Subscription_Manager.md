---
layout: default
title: Subscription Manager
---

## Subscription Manager

The Red Hat Enterprise Subscription Manager does what its name implies, which is manages your subscription to Red Hat Supported software.  This isn't what I normally do day-to-day, so I'm constantly looking up what the commands are to interact with it.  This post is more of a note to myself on the common things that I keep forgetting how to do.


To check subscription status

```bash
subscription-manager status
```

To view the Pool ID - useful if you need to save the existing Pool ID before attempting to re/attach it to a different pool.
```bash
subscription-manager list --consumed
```

To view which repos are enabled - might have to 'sudo' if they say they're all disabled
```bash
yum repolist all
```

Managing repos
```bash
subscription-manager --disable <repoid>
subscription-manager --enable <repoid>
```

Refreshing the packages - useful after manipulating the available repos through subscription manager.
```bash
yum clean all -v
yum makecache
```