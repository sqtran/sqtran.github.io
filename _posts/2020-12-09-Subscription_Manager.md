---
title: Subscription Manager
date: 2020-12-09
tags: subscription-manager yum
---


The Red Hat Enterprise Subscription Manager does what its name implies, which is manages your subscription to Red Hat Supported software.  This isn't what I normally do day-to-day, so I'm constantly looking up what the commands are to interact with it.  This post is more of a note to myself on the common things that I keep forgetting how to do.


## Check subscription status
```bash
subscription-manager status
```

## View Pool ID
This is useful if you need to save the existing Pool ID before attempting to re/attach it to a different pool.

```bash
subscription-manager list --consumed
```

## View enabled repos
Note that 'sudo' may be required if they say they're all disabled
```bash
yum repolist all
```

## Manage repos
```bash
subscription-manager --disable <repoid>
subscription-manager --enable <repoid>
```

## Refresh packages
This is useful after manipulating the available repos through subscription manager.
```bash
yum clean all -v
yum makecache
```