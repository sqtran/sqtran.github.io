---
title: Image Import Sync
date: 2024-03-19
tags: ocp images
---

## Background
If you're deploying images from an external registry into OpenShift, and you created ImageStreams/Tags that mirror your external registry, you'll eventually need to sync the two whenever there are changes to that external registry.  I keep forgetting the command to do that, so here it is (for me) when I eventually forget again.


## Solution
It's a really easy command, but if you don't do it often, you'll forget how to do it.  This command will sync all image tags, but you can specify specifically which tags you want too.

```bash
# omit the --all flag if you want to pin it to a specific tag
oc import-image <imagestream name> --from=quay.io/<repo/image> --confirm --all
```