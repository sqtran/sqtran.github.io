---
title: HAProxy Docker Setup
date: 2020-01-28
tags: haproxy docker
---

When doing local development/testing, I find myself constantly referring to online examples on setting up a local `HAProxy`, for various `HTTP Request` manipulations.  Instead of feeding Google the same searches ever other week, I finally decided to create a post in order to reference it myself, but also hoping that it will reinforce my understanding of it so that I don't need to look it up everytime.

I'll be using `Docker` in this example, but `Podman` is definitely on the radar for next time.  Since most of us have probably already used `Docker` already, this should be a really quick post.

## Steps
First, pull the image.  I prefer to use the `Alpine` image since it's small, and quick to pull, but use whatever you need to use.  See `HAProxy`'s [Docker Hub](https://hub.docker.com/_/haproxy) page for a list of all versions.

```bash
docker pull haproxy:1.8-alpine
```

Now run it.
```bash
docker run --rm docker.io/haproxy:1.8-alpine
```
But not so fast.  You'll get an error message that says the following.

```bash
[ALERT] 028/063713 (1) : Cannot open configuration file/directory /usr/local/etc/haproxy/haproxy.cfg : No such file or directory
```

The README didn't say that it required it, but I guess you should assume it needs some sort of configuration.


So let's try this.

```bash
docker run -v /tmp/haproxy.config:/usr/local/etc/haproxy/haproxy.cfg --rm docker.io/haproxy:1.8-alpine
```

But I was getting this error message now.

```bash
[ALERT] 028/064711 (1) : Could not open configuration file /usr/local/etc/haproxy/haproxy.cfg : Permission denied
```

The solution is to add the `:Z` modifier to the volume mount.  After reading the documentation, the Z flag is something to do with SELinux, but that's all I really care to know when running my simple, test containers locally.  This would be a different story if real work was being done on a production server somewhere.

**The final solution is this.**

```bash
docker run -v /tmp/haproxy.config:/usr/local/etc/haproxy/haproxy.cfg:Z --rm docker.io/haproxy:1.8-alpine
```


## Sample HAProxy Configuration
Here's a sample HAProxy config file too.  It does some weird things that's only useful for myself, so your config will be completely different.
```
global
  debug                       # uncomment to enable debug mode for HAProxy

defaults
  mode                    http
  option                  httplog
  option                  dontlognull
  option http-server-close
  option forwardfor       except 127.0.0.0/8
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000

  timeout connect 5000ms       # max time to wait for a connection attempt to a server to succeed
  timeout client 50000ms       # max inactivity time on the client side
  timeout server 50000ms       # max inactivity time on the server side
###############

frontend http-apps             # define what port to listed to for HAProxy
  mode tcp#http
  bind *:980
  default_backend legacy-http

backend legacy-http            # define a group of backend servers to handle legacy requests
  mode tcp#http
  server spring-boot-8080 192.168.122.224:80 send-proxy  # add a server to this backend
  reqidel ^X-Forwarded-For:.*

###############

frontend https-apps            # define what port to listed to for HAProxy
  mode tcp
  bind *:9443
  default_backend legacy-https

backend legacy-https           # define a group of backend servers to handle legacy requests
  mode tcp
  server spring-boot-8443 192.168.122.224:443 send-proxy  # add a server to this backend
##############
```
