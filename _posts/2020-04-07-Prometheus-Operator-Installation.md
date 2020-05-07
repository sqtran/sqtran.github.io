---
layout: default
title: Prometheus Operator Configuration
---

## Prometheus Operator Configuration

As of OCP 4.3, application metrics via the internal Prometheus instance is a Tech Preview Feature.  That means enabling this Tech Preview puts your cluster into an unsupported mode, which is not ideal for production systems.

We can install the Prometheus operator from the Operator Marketplace, and use it specifically to scrape application metrics, while maintaining supportability from Red Hat.  Configuring Prometheus is pretty simple via the GUI, but here are the steps and Kubernetes manifest objects needed to automate the process.


### HAProxy Configurations

#### Prerequisites
Three (3) VIPs and DNS entries are needed.  VIPs are not required if using 3 different load balancers, but that seems like a waste of resources.  In this configuration, a single external HAProxy that is configured with 3 VIPs.  The \*.apps certificate that is used in OCP will also be reused to decrypt/encrypt traffic.



#### VIP 1
The first VIP is for the `oc cli` tool.  We need an endpoint for 6443 traffic, which will point to the OCP masters.  The DNS entry associates api.<your_subdomain\> to this VIP.

```
frontend fe-ocp-api-servers
    bind x.x.x.1:6443
    default_backend be-ocp-api-servers
    mode tcp

backend be-ocp-api-servers
    balance source
    mode tcp
    server master1 master1.<your_subdomain>:6443 check
    server master2 master2.<your_subdomain>:6443 check
    server master3 master3.<your_subdomain>:6443 check
```

#### VIP 2
The second VIP is for OAuth.  For whatever reason, OCP did not like it when we tried to reencrypt traffic, so as a workaround, a VIP was created just to handle OAuth authentication traffic.  Any traffic heading to the OAuth route would just be forwarded along, as usual.  The DNS entry associates oauth-openshift.apps.<your_subdomain\> to this VIP.

```
frontend fe-oauth-route
    bind x.x.x.2:443
    default_backend be-oauth-route
    mode tcp

backend be-oauth-route
    balance source
    mode tcp
    server infra1 infra1.<your_subdomain>:443 check
    server infra2 infra2.<your_subdomain>:443 check
    server infra3 infra3.<your_subdomain>:443 check
```

#### VIP 3
The third is for everything else.  Any traffic heading to \*.apps.<your_subdomain\> will be decrypted, x-forwarded-for headers added, and then reencrypted.

The configuration also drops any existing `X-Forwarded-For` headers, in case a client tries to spoof its source IP.  Not required, but good practice.

```
frontend fe-route-http
    bind x.x.x.3:80
    default_backend be-route-http
    mode http
    option forwardfor except 127.0.0.0/8
    reqidel ^X-Forwarded-For:.*

backend be-route-http
    balance source
    mode http
    server infra1 infra1.<your_subdomain>:80 check
    server infra2 infra2.<your_subdomain>:80 check
    server infra3 infra3.<your_subdomain>:80 check

frontend fe-route-https
    bind x.x.x.3:443 ssl crt <path_to_cert>.crt
    default_backend be-route-https
    mode http
    option forwardfor except 127.0.0.0/8
    reqidel ^X-Forwarded-For:.*

backend be-route-https
    balance source
    mode http
    server infra1 infra1.<your_subdomain>:443 check ssl ca-file <path_to_cert>.crt
    server infra2 infra2.<your_subdomain>:443 check ssl ca-file <path_to_cert>.crt
    server infra3 infra3.<your_subdomain>:443 check ssl ca-file <path_to_cert>.crt
```

### Reference

http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt
