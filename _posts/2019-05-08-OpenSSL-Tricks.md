---
layout: default
title: OpenSSL Tricks
---

## OpenSSL Commands

Here's a handy command to download a `SSL/TLS` certificate via command line.  I end up using this a lot, but for whatever reason, can't seem to memorize the exact syntax.

```bash
openssl s_client -connect <host>:<port> < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > the_certificate.cer
```

Note that we only need the host name without specifying the protocol (`https`).  

The typical port for `https` is `443`, but YMMV if you're running your webservice on a different port (i.e. `8443` for `JBoss`).


If you need to convert a certificate from binary (der) to base64, use this command.

```bash
openssl x509 -in binaryCert.certx -inform der -text -out mycert.pem
```
The file extension doesn't actually have to match the encoding of the file since users can name it whatever they want.  What's important is knowing the `-inform` of the file.  The `der` format is a binary representation of the certificate.  The `pem` format is a base64-encoded ASCII file.

The `-text` flag just gives us some verbosity when running the command, and is optional.
