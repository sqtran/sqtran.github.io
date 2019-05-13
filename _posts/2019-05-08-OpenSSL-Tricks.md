---
layout: post
title: OpenSSL Tricks
---

## OpenSSL Commands

Here's a handy command to download a `SSL/TLS` certificate via command line.  I end up using this a lot, but for whatever reason, can't seem to memorize the exact syntax.

```bash
openssl s_client -connect <host>:<port> < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > the_certificate.cer
```

Note that we only need the host name without specifying the protocol (`https`).  

The typical port for `https` is `443`, but YMMV if you're running your webservice on a different port (i.e. `8443` for `JBoss`).
