---
layout: single
title: Containerized Apache Httpd
date: 2019-03-13
tags: containers apache httpd
---

This is an expansion of what I wrote about yesterday.  I took the Httpd reverse proxy setup and containerized it so that I wouldn't need to install Httpd locally, nor open up SELinux rules that I don't want to keep open permanently.

Take a look at my [containered_httpd](https://github.com/sqtran/containered_httpd) project on Github.  It's essentially a minimal `Dockerfile` to use `httpd:2-alpine` and overwrite it's default configuration file.

Configuration of Httpd on Alpine is a little different than the RHEL example I posted.  I'm basically laying down my own copy of /usr/local/apache2/conf/httpd.conf into the container so that it picks up the configurations I want.

If you look at the httpd-custom.conf file, you'll see that I've enabled a bunch of modules related to `proxy_*`, and configured my `VirtualHost` there.

I had to do it this way because I ran into permission issues when trying to volume mount this file directly into a running container.  This is something I should look into some more later.


## Build

Build the container and see for yourself.

```bash
sudo systemctl start docker
docker build . -t sqtran/httpd:0.1-beta
docker run -it --rm sqtran/httpd:0.1-beta
```

## Test
To test this, you'll need to curl your container.  If you don't know what IP your container is running on, follow these steps.

```bash
docker inspect $(docker ps | grep "sqtran/httpd:0.1-beta" | awk '{print $1}' ) | grep IPAddress
```
Then curl the address you just found.  You should get a "It works" message if you haven't customized httpd.conf, or you should get the home page of the web server that you redirected to.


## Housekeeping
When you're done with everything, you should prune to eliminate any dangling resources.  Warning, this will destroy your stopped pods too, including log messages if you need them for later.

```bash
docker system prune
```
