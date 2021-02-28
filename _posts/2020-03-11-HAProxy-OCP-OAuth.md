---
layout: single
title: OCP Layer 7 Reencryption
date: 2020-03-11
tags: ocp haproxy proxy_protocol layer7
---

With OCP 4.x, we lost functionality to configure the OCP ingress routers, which _should_ be fixed in a future release.

Specifically, we lost the `PROXY_PROTOCOL` Boolean flag.  When set, it configures the OCP routers (which is an instance of HAProxy) to accept the `PROXY_PROTOCOL` protocol.

This protocol configuration is set on both sides.  The sender sends traffic with `PROXY_PROTOCOL` enabled, and the receiver (HAProxy) knows to accept it.  It configures both sides to communicate additional information about the traffic being sent, specifically the `Source IP` address is forwarded along to the receiver.  The source IP address is typically not available due to how the network is set up.

The OCP routers are usually not sitting at the edge of the network, so traffic is routed through multiple intermediaries before it reaches the cluster.  Every hop along the route, the source is set to the previous hop's IP, so by the time it reaches the actual application, all it knows about is the previous proxy.  The application typically doesn't know, or care, about how the network is configured, so if it was relying on the client's source IP address for anything, it won't have the correct IP.  This is what the `PROXY_PROTOCOL` is for.


So until we have the PROXY_PROTOCOL in a future update of OCP4.x, we have to do it the old fashioned way and manipulate X-Forwarded-For headers.  Manipulating the headers is easy when only working with HTTP traffic, but is impossible with HTTPS.  To get around this, we will need to decrypt HTTPS traffic, modify the headers, and then re-encrypt it.  This is a commonly implemented solution, not just for Openshift.

Note that switching from Layer 4 to Layer 7 will incur some performance penalties.  That's due to the additional overhead of decrypting TLS and reencrypting it.


## HAProxy Configurations

### Prerequisites
Three (3) VIPs and DNS entries are needed.  VIPs are not required if using 3 different load balancers, but that seems like a waste of resources.  In this configuration, a single external HAProxy that is configured with 3 VIPs.  The \*.apps certificate that is used in OCP will also be reused to decrypt/encrypt traffic.



### VIP 1
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

### VIP 2
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

### VIP 3
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

## Reference

[http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt](http://www.haproxy.org/download/1.8/doc/proxy-protocol.txt
)
