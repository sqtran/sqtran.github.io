---
layout: single
title: Storage Benchmarking
date: 2019-12-15
#categories: benchmark bonnie++
---

## Motivation

There are various tools for benchmarking disk IO.  I recently created a [container toolbox project](https://github.com/sqtran/container-toolbox) for doing so, which is based on a handy tool called `bonnie++`, which I added to an `mainline-alpine nginx` base image.  It's a quick and dirty way to run some IO benchmarks to test different storage types, which is useful if you want to see real numbers on how your cloud-based disks.


## Usage
Essentially, you need to create a `PV` and `PVC` and mount it into your pod.  Shell into your running pod/container and run the following example commands.

```bash
cd /tmp
bonnie++ -d /tmp/nfs-mnt/ -x 10 -n 10 -m nfs-mnt > nfs-mnt.txt &

bonnie++ -d /tmp/thin-mnt/ -x 10 -n 10 -m thin-mnt > thin-mnt.txt &
```

Above, I am testing a VMWare VMDK with thin provision, and a NetApp NFS export.  It doesn't matter what you call the actual folders, but it would make more sense to name them something based on what type of storage it actually is.  

After the tests are completed, run the following to convert the CSV into HTML
```bash
/tmp/convertResults.sh
```

Now just go to <server>:8080 to see a listing of all the generated results.
