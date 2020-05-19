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

dn: ou=groups,ou=intranet,dc=example,dc=com
ou: groups
description: generic groups branch
objectclass: organizationalunit

## SECOND Level hierarchy
## ADD a single entry under FIRST (people) level
# this is an ENTRY sequence and is preceded by a BLANK line
# the ou: Human Resources is the department name

dn: cn=Homer Simpson,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Homer Simpson
sn: Simpson
uid: hsimpson
userpassword: 123456
homephone: 555-555-5555
mail: homer@simpson.com
description: test account

dn: cn=Marge Simpson,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Marge Simpson
sn: Simpson
uid: mbsimpson
userpassword: 123456
homephone: 555-555-5555
mail: marge@simpson.com
description: test account

dn: cn=Lisa Simpson,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Lisa Simpson
sn: Simpson
uid: lsimpson
userpassword: 123456
homephone: 555-555-5555
mail: lisa@simpson.com
description: test account

dn: cn=Bart Simpson,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Bart Simpson
sn: Simpson
uid: bsimpson
userpassword: 123456
homephone: 555-555-5555
mail: bart@simpson.com
description: test account

dn: cn=Maggie Simpson,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Maggie Simpson
sn: Simpson
uid: msimpson
userpassword: 123456
homephone: 555-555-5555
mail: maggie@simpson.com
description: test account

dn: cn=Kirt Van Houten,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Kirt Van Houten
sn: Van Houten
uid: kvanhouten
userpassword: 123456
homephone: 555-555-5555
mail: kirk@vanhouten.com
description: test account

dn: cn=Luann Van Houten,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Luann Van Houten
sn: Van Houten
uid: lvanhouten
userpassword: 123456
homephone: 555-555-5555
mail: luann@vanhouten.com
description: test account

dn: cn=Milhouse Van Houten,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Milhouse Van Houten
sn: Van Houten
uid: mvanhouten
userpassword: 123456
homephone: 555-555-5555
mail: milhouse@vanhouten.com
description: test account


dn: cn=Clancy Wiggum,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Clancy Wiggum
sn: Wiggum
uid: cwiggum
userpassword: 123456
homephone: 555-555-5555
mail: clancy@wiggum.com
description: test account

dn: cn=Sarah Wiggum,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Sarah Wiggum
sn: Wiggum
uid: swiggum
userpassword: 123456
homephone: 555-555-5555
mail: sarah@wiggum.com
description: test account

dn: cn=Ralph Wiggum,ou=people,ou=intranet,dc=example,dc=com
objectclass: inetOrgPerson
cn: Ralph Wiggum
sn: Wiggum
uid: rwiggum
userpassword: 123456
homephone: 555-555-5555
mail: ralph@wiggum.com
description: test account

dn: cn=Simpsons,ou=groups,ou=intranet,dc=example,dc=com
cn: Simpsons
description: The Simpsons group
objectclass: groupOfNames
member: cn=Homer Simpson,ou=people,ou=intranet,dc=example,dc=com
member: cn=Marge Simpson,ou=people,ou=intranet,dc=example,dc=com
member: cn=Bart Simpson,ou=people,ou=intranet,dc=example,dc=com
member: cn=Lisa Simpson,ou=people,ou=intranet,dc=example,dc=com
member: cn=Maggie Simpson,ou=people,ou=intranet,dc=example,dc=com

dn: cn=Van Houtens,ou=groups,ou=intranet,dc=example,dc=com
cn: Van Houtens
description: The Van Houtens group
objectclass: groupOfNames
member: cn=Kirk Van Houten,ou=people,ou=intranet,dc=example,dc=com
member: cn=Luann Van Houten,ou=people,ou=intranet,dc=example,dc=com
member: cn=Milhouse Van Houten,ou=people,ou=intranet,dc=example,dc=com

dn: cn=Wiggums,ou=groups,ou=intranet,dc=example,dc=com
cn: Wiggums
description: The Wiggums group
objectclass: groupOfNames
member: cn=Clancy Wiggum,ou=people,ou=intranet,dc=example,dc=com
member: cn=Sarah Wiggum,ou=people,ou=intranet,dc=example,dc=com
member: cn=Ralph Wiggum,ou=people,ou=intranet,dc=example,dc=com

```

Note that the image honors some environment variables to change the default values, such as changing the administrative bind account name.  See the Github page https://github.com/openshift/openldap for a full list of all the variables.


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
