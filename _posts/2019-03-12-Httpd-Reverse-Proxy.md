---
layout: single
title: Httpd Reverse Proxy
date: 2019-03-12
tags: apache httpd
---

This is how you set up a reverse proxy with Apache Httpd on RHEL 7.5

## Background

Httpd is a super simple web server, but even better if you're looking for a reverse proxy.  I had a use-case where I wanted incoming http traffic to pseudo-randomly access my local back-end servers.  I was running several instances of EAP locally, with different port offsets, and I wanted a single vanity URL.  With Apache Httpd, I could specify a virtual host that listens to all traffic coming in on a specific port (80 is the default web port) and redirect it to my back-end servers.


## Install and Configure
Installing httpd is very easy through `yum`, on RHEL (Fedora)
```bash
sudo yum install -y httpd
```

And then make sure it's started, or enabled if you want it to start on bootup.  Since this is my local development environment, I like to explicitly start services.
```bash
sudo systemctl start httpd
```

## Test
Test that it's working with a good ole curl command.
```bash
curl http://localhost
```

## Troubleshoot
If you don't get a message that says "It works", then you probably need to configure your `SELinux` rules.

```bash
sudo /usr/sbin/setsebool -P httpd_can_network_connect 1
```

The problem is very obvious if you `tail` /var/log/messages

```bash
Mar 12 14:02:04 stran setroubleshoot: SELinux is preventing /usr/sbin/httpd from name_connect access on the tcp_socket port 8080. For complete SELinux messages run: sealert -l cb0169a0-52d7-4a30-8896-6ade1d8ca4ce
Mar 12 14:02:04 stran python: SELinux is preventing /usr/sbin/httpd from name_connect access on the tcp_socket port 8080.#012#012*****  Plugin catchall_boolean (47.5 confidence) suggests   ******************#012#012If you want to allow httpd to can network connect#012Then you must tell SELinux about this by enabling the 'httpd_can_network_connect' boolean.#012#012Do#012setsebool -P httpd_can_network_connect 1#012#012*****  Plugin catchall_boolean (47.5 confidence) suggests   ******************#012#012If you want to allow httpd to can network relay#012Then you must tell SELinux about this by enabling the 'httpd_can_network_relay' boolean.#012#012Do#012setsebool -P httpd_can_network_relay 1#012#012*****  Plugin catchall (6.38 confidence) suggests   **************************#012#012If you believe that httpd should be allowed name_connect access on the port 8080 tcp_socket by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -i my-httpd.pp#012
Mar 12 14:02:07 stran setroubleshoot: SELinux is preventing httpd from name_connect access on the tcp_socket port 8180. For complete SELinux messages run: sealert -l eb15900b-2714-4f95-adf5-9ecf54b63c7a
Mar 12 14:02:07 stran python: SELinux is preventing httpd from name_connect access on the tcp_socket port 8180.#012#012*****  Plugin connect_ports (85.9 confidence) suggests   *********************#012#012If you want to allow httpd to connect to network port 8180#012Then you need to modify the port type.#012Do#012# semanage port -a -t PORT_TYPE -p tcp 8180#012    where PORT_TYPE is one of the following: dns_port_t, dnssec_port_t, http_port_t, kerberos_port_t, ocsp_port_t.#012#012*****  Plugin catchall_boolean (7.33 confidence) suggests   ******************#012#012If you want to allow httpd to can network connect#012Then you must tell SELinux about this by enabling the 'httpd_can_network_connect' boolean.#012#012Do#012setsebool -P httpd_can_network_connect 1#012#012*****  Plugin catchall_boolean (7.33 confidence) suggests   ******************#012#012If you want to allow nis to enabled#012Then you must tell SELinux about this by enabling the 'nis_enabled' boolean.#012#012Do#012setsebool -P nis_enabled 1#012#012*****  Plugin catchall (1.35 confidence) suggests   **************************#012#012If you believe that httpd should be allowed name_connect access on the port 8180 tcp_socket by default.#012Then you should report this as a bug.#012You can generate a local policy module to allow this access.#012Do#012allow this access for now by executing:#012# ausearch -c 'httpd' --raw | audit2allow -M my-httpd#012# semodule -i my-httpd.pp#012
```

## Configure
To configure the reverse proxy with a balancer so that it cycles through your backend hosts, create the following file `/etc/httpd/conf.d/default-site.conf`.  I think this file path is specific to how yum installs httpd on RHEL.  It's a little different when I tried it on the httpd container image, and the Alpine Linux image too.

```text
<VirtualHost *:80>
<Proxy balancer://mycluster>
    BalancerMember http://192.168.122.1:8080
    BalancerMember http://192.168.122.1:8180
    BalancerMember http://192.168.122.1:8280
</Proxy>

   ProxyPreserveHost On

   ProxyPass / balancer://mycluster/
   ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```

Reload httpd and test it with a `curl` again
```bash
sudo systemctl restart httpd
curl http://localhost
```
