---
layout: single
title: JBoss Vault Cheat Sheet
date: 2019-05-06
#categories: jboss eap vault
---

This is a quick and dirty cheat-sheet for manipulating a JBoss Vault.  I tend to forget and relearn these commands frequently, so this should at least make it easier to start back up.

## Create/Update
```bash
~/jboss-eap-7.1/bin/vault.sh --keystore vault/myvault/vault.keystore --keystore-password mysecretpassword --alias vault --vault-block myvaultblock --attribute password --sec-attr "my_secret_password_value" --enc-dir vault/myvault
```

## Read
```bash
~/jboss-eap-7.1/bin/vault.sh --keystore vault/myvault/vault.keystore --keystore-password mysecretpassword --alias vault --check-sec-attr --vault-block myvaultblock --attribute password --enc-dir vault/myvault --iteration 23 --salt 12345678
```

## Delete
```bash
~/jboss-eap-7.1/bin/vault.sh --keystore vault/myvault/vault.keystore --keystore-password mysecretpassword --alias vault  --vault-block myvaultblock -r password --enc-dir vault/myvault/
```
