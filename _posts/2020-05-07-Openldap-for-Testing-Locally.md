---
layout: default
title: OpenLDAP for Testing Locally
---

## OpenLDAP for Testing Locally

Enterprise middleware components usually requires integration with enterprise identity providers - such as Windows Active Directory.  There are many vendors in this space, but luckily there is a common protocol that can talk to most of these in a standard way.  We have LDAP - Lightweight Directory Access Protocol and the opensource tool, OpenLDAP.  

If you've ever had to integrate with these components in the past, you'd understand that everyone sets up their directory trees differently.  There's probably best practices on how to organize the tree, and where/how to store certain attributes, but everyone customizes it for their specific organization.  That can cause quite a headache when it comes time for integration, such as for Authentication and Authorization.

Standing up your own containerized OpenLDAP server is quick and painless way to test integration points, especially working in a disconnected environment.

### Image

The image being used is this https://hub.docker.com/r/openshift/openldap-2441-centos7/.  Even though this may not be the latest version of OpenLDAP, and it has a bold disclaimer that this is only meant to be used internally for testing their Openshift project, it still provides enough functionality for local testing.

One thing to note is the certificate it uses is expired, so the `ldaps://` protocol won't be able to establish a secure connection.  It probably doesn't matter for testing purposes though.


### Configuration

An example `LDIF` is provided here for convenience.  Customize this for your specific use-case, or use your own `LDIF` file.


```bash
sh-4.2$ cat sample.ldif

## DEFINE DIT ROOT/BASE/SUFFIX ####
## uses RFC 2377 format
## replace example and com as necessary below
## or for experimentation leave as is

## dcObject is an AUXILLIARY objectclass and MUST
## have a STRUCTURAL objectclass (organization in this case)
# this is an ENTRY sequence and is preceded by a BLANK line

## FIRST Level hierarchy - people
## uses mixed upper and lower case for objectclass
# this is an ENTRY sequence and is preceded by a BLANK line

dn: ou=intranet,dc=example,dc=com
ou: intranet
objectclass: organizationalunit

dn: ou=people,ou=intranet,dc=example,dc=com
ou: people
description: All people in organisation
objectclass: organizationalunit

dn: ou=Test Engineering,ou=people,ou=intranet,dc=example,dc=com
ou: Test Engineering
objectClass: organizationalUnit


## SECOND Level hierarchy
## ADD a single entry under FIRST (people) level
# this is an ENTRY sequence and is preceded by a BLANK line
# the ou: Human Resources is the department name

dn: cn=Steve Tester,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Steve Tester
sn: Tester
uid: stester
userpassword: 123456
homephone: 555-555-5555
mail: stester@example.com
description: testing account
ou: Test Engineering

```

Note that the image honors some environment variables to change the default values, such as changing the administrative bind account name.  See the Github page for a full list of all the variables.


### Deployment

#### OCP

It's fairly easy to just click on the deploy image button within the Openshift UI.  Even the most underpowered systems will deploy this in less than a minute.  The real bottleneck is network bandwidth though, as the image weighs in at 355MB.  

#### Docker

Running this locally also works, assuming you have Docker installed.

```bash
docker run -it --rm openshift/openldap-2441-centos7
```

#### Podman
This container image unfortunately runs as root, so there is a little bit of work to get it to run correctly.  Instructions are TODO.


### Useful Commands

If you visit the Github site, you'll see what the default username and passwords are.  Most of the commands require you to bind to an administrative account.  Here are some useful commands for interacting with OpenLDAP.

#### Searching
```bash
ldapsearch -h <host> -w admin -D "cn=Manager,dc=example,dc=com" -b "dc=example,dc=com"
```

Response
```bash
# extended LDIF
#
# LDAPv3
# base <dc=example,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# example.com
dn: dc=example,dc=com
objectClass: top
objectClass: dcObject
objectClass: organization
dc: example
o: example

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```


#### Adding
```bash
ldapadd -h <host> -w admin -D "cn=Manager,dc=example,dc=com" -f sample.ldif
```

Response
```bash
adding new entry "ou=intranet,dc=example,dc=com"

adding new entry "ou=people,ou=intranet,dc=example,dc=com"

adding new entry "ou=Test Engineering,ou=people,ou=intranet,dc=example,dc=com"

adding new entry "cn=Steve Tester,ou=people,ou=intranet,dc=example,dc=com"
```

#### Deleting
```bash
ldapdelete -h <host> -w admin -D "cn=Manager,dc=example,dc=com" "cn=Steve Tester,ou=people,ou=intranet,dc=example,dc=com"
ldapdelete -h <host> -w admin -D "cn=Manager,dc=example,dc=com" "ou=Test Engineering,ou=people,ou=intranet,dc=example,dc=com"
ldapdelete -h <host> -w admin -D "cn=Manager,dc=example,dc=com" "ou=people,ou=intranet,dc=example,dc=com"
ldapdelete -h <host> -w admin -D "cn=Manager,dc=example,dc=com" "ou=intranet,dc=example,dc=com" -h <host>
```
