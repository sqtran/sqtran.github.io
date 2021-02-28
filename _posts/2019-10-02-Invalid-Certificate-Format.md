---
title: Invalid certificate in Openshift Secret
date: 2019-10-02
tags: openshift secret certificate
---

I saw this error when I passed in the wrong certificate format in my Secret.  It was the binary encoded DER format, but Openshift requires it to be base64 encoded.

It's certainly not obvious from the error message you see in the logs.

## Error

```ruby
oci runtime error: process_linux.go:295: setting oom score for ready process caused "write /proc/461/oom_score_adj: invalid argument"
```

## Solution
You can use the `openssl` tool to convert between formats.

```bash
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
```

Now replace the secret in Openshift and watch your pod come up.
