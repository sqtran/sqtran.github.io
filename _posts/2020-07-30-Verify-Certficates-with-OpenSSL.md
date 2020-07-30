---
layout: default
title: Verify Certificates with OpenSSL
---

## Verify Certificates with OpenSSL

Working with certificates isn't as complicated as most developers seem to think.  There are plenty of guides and examples online to follow.  `OpenSSL` is probably the most powerful tool out there.

A common task is to verify the SSL/TLS capabilities of a server.  If you need to interrogate a server to determine what protocols are allowed, use `OpenSSL` to connect to it.  The output may look arcane, but it's not really that bad.

```bash
openssl s_client -connect the_server_host:the_port_number
```

```bash
CONNECTED(00000003)
depth=0 CN = aa61f9423ec3
verify error:num=18:self signed certificate
verify return:1
depth=0 CN = aa61f9423ec3
verify error:num=10:certificate has expired
notAfter=Oct  8 17:16:43 2016 GMT
verify return:1
depth=0 CN = aa61f9423ec3
notAfter=Oct  8 17:16:43 2016 GMT
verify return:1
---
Certificate chain
 0 s:/CN=aa61f9423ec3
   i:/CN=aa61f9423ec3
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIB5TCCAU6gAwIBAgIFAKQzYN0wDQYJKoZIhvcNAQELBQAwFzEVMBMGA1UEAxMM
YWE2MWY5NDIzZWMzMB4XDTE1MTAwODE3MTY0M1oXDTE2MTAwODE3MTY0M1owFzEV
MBMGA1UEAxMMYWE2MWY5NDIzZWMzMIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKB
gQC1bqKzq66lwXCmCnB5OqnICmdEKS3KI7c1FpsXR+etvMheq+4GSCENM5NqxSw4
VxlbxmD+MrNqpjrF03OMr1vxWjHKtfDKANUdM4WtG+8F55FKpFzH1gme1q/PseaK
E45XjZljxXhaU4frfwu0FaIRV5JxWZt2wKr3HbXKG0P5iQIDAQABoz0wOzA5BgNV
HREEMjAwggxhYTYxZjk0MjNlYzOCCWxvY2FsaG9zdIIVbG9jYWxob3N0LmxvY2Fs
ZG9tYWluMA0GCSqGSIb3DQEBCwUAA4GBAG8eZI1dVv646f1mzVItFw55t4SxD6xk
mvglJmXfJsgPlhLvvLJEtp4m/xZS4M69oBkWOD/Jc1959OaNc+SZQuAzCX8XTJZx
rZjnXOKqtnbCj8dAfqhDXKfxwVq7kq8Nz0I0faIigE9lpIkTOeoUt4suTsI+ZdiW
fby5DLqQH8uV
-----END CERTIFICATE-----
subject=/CN=aa61f9423ec3
issuer=/CN=aa61f9423ec3
---
No client certificate CA names sent
---
SSL handshake has read 802 bytes and written 479 bytes
---
New, TLSv1/SSLv3, Cipher is AES256-GCM-SHA384
Server public key is 1024 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : AES256-GCM-SHA384
    Session-ID: BFB26BA5E0750C03AB94C11FCEC73608519FD6117864507E76E062003B6AEA41
    Session-ID-ctx:
    Master-Key: 4F1D55625578A18AE67F70204A9E33C95AD36B8461C3181790A65731765AE96AB25E0AB7D1CC82C364535A0C5F07DAC5
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    TLS session ticket lifetime hint: 300 (seconds)
    TLS session ticket:
    0000 - 2a 32 21 fc 56 f8 bf d4-8b 5a 33 63 6e 61 15 57   *2!.V....Z3cna.W
    0010 - 64 dd 3c c8 b3 cc e3 90-53 e5 cc ef ea 6e 63 03   d.<.....S....nc.
    0020 - 75 47 62 73 92 38 d1 1b-1e f0 20 6b 26 49 af 32   uGbs.8.... k&I.2
    0030 - 53 cf c7 41 20 eb 10 01-59 a1 4b 43 7b d3 dc ae   S..A ...Y.KC{...
    0040 - 72 2d d7 00 53 e2 0a 20-b4 cb a3 34 ce 84 3b f8   r-..S.. ...4..;.
    0050 - a8 ff e5 b7 a5 72 86 48-8d 49 00 63 97 2d 0d 79   .....r.H.I.c.-.y
    0060 - c8 86 7f 8f 2a 70 1e 0b-38 92 af 67 94 c0 e2 79   ....*p..8..g...y
    0070 - 4e 27 95 c6 ee c4 19 60-38 e4 44 01 22 b4 6e e4   N!.....!8.D.!.n.
    0080 - 74 98 3a 13 97 88 fa 1f-49 9d b9 55 3a 6a 62 bd   t.:.....I..U:jb.
    0090 - 87 85 f5 c7 58 07 e7 46-b6 73 97 49 79 b3 7b b8   ....X..F.s.Iy.{.

    Start Time: 1590697740
    Timeout   : 300 (sec)TLS 1.1
    Verify return code: 10 (certificate has expired)
---
```

It's an outdated, and insecure practice if the server is accepting `TLS 1.1` or lower.  To check a specific TLS version, just pass in the following flags, rerun the command with any of the following flags. `-TLS1_2`, `-TLS1_1`, `-SSL3`
OpenSSL

Download the certificate by following this example [here](2019-05-08-OpenSSL-Tricks.md).  If the certificate is already present on the machine, just use this command.

```bash
openssl x509 -in the_certificate_here.crt -text -noout
```