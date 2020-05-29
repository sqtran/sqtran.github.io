---
layout: default
title: Podman on Fedora 32
---

## Podman on Fedora 32

Running rootless `podman` on Fedora requires just a little customization.  There are lots of resources out there to help troubleshoot issues, but this is just a little reminder to myself on how I resolved it.  It might help someone else out there too.


Assuming `podman` is already installed via `yum` or `dnf`, the next step is to test logging into a registry.  This example uses `docker.io` but it can be anything.  You could also just build your own image too, and bypass this test all together.

```bash
podman login docker.io
```

Test pulling an image from the registry.

```bash
podman pull docker.io/sqtran/spring-boot:latest
```

Pulling the image is fine, but when it tries to save the image, a message like the following could appear:
```
Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 192:192 for /run/systemd/netif): lchown /run/systemd/netif: invalid argument
Error: error pulling image "docker.io/sqtran/spring-boot:latest": unable to pull docker.io/sqtran/spring-boot:latest: unable to pull image: Error committing the finished image: error adding layer with blob "sha256:00f17e0b37b0515380a4aece3cb72086c0356fc780ef4526f75476bea36a2c8b": Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 192:192 for /run/systemd/netif): lchown /run/systemd/netif: invalid argument
```

To fix that, we need to configure subids and subgids.  A lot of examples used 65535 or some variant of a 2^32, but I think that's a bit much.  I'm using 9999 and it seems to work okay.  It probably won't hurt using 65k though if this is a personal machine.

```bash
cat /etc/subuid
<user1>:10000:9999
<user2>:11000:9999
```

```bash
cat /etc/subgid
<user1>:10000:9999
<user2>:11000:9999
```

After making the changes to `/etc/subuid` and `/etc/subgid`, run the following command.

```bash
podman system migrate
```

And that's it!  Pull the image again and run it.  

I wasted a lot of time troubleshooting this before I found the `podman system migrate` command.
