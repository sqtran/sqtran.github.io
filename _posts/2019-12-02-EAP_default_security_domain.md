---
layout: single
title: JBoss EAP Default Security Domain
date: 2019-12-02
tags: jboss eap security
---

A default security-domain can be configured in the Undertow subsystem of EAP, so that it is applied to all deployments.  This gives applications the ability to *not* specify a particular security-domain, which is useful if all applications should use a common domain, which follows the `DRY` principle of not duplicating code.

The default security domain can be overridden when needed, which is configured via the `jboss-web.xml` file.

This page explains how setting a default security-domain affects deployments on EAP.  It references a security-domain named SPNEGO, which must be installed before setting any of these settings.  Your default security doesn't have to be SPNEGO though, it can be anything really.  Note that SPNEGO is only the __name__ of the security domain, and security-domains can have any name.


## Server Settings
In your standalone.xml or domain.xml (make sure you're using the right profile when in domain mode), set the `default-security-domain` attribute on the Undertow subsystem.  It should have a value of "SPNEGO" to follow the example in this article, and is most likely configured as "other" by default.

domain.xml
```xml
<subsystem instance-id="${jboss.instance.id}" xmlns="urn:jboss:domain:undertow:7.0" default-server="default-server" default-virtual-host="default-host" default-servlet-container="default" default-security-domain="SPNEGO">
```

standalone.xml
```xml
<subsystem xmlns="urn:jboss:domain:undertow:7.0" default-server="default-server" default-virtual-host="default-host" default-servlet-container="default" default-security-domain="SPNEGO">
```

Domain mode has an additional attribute for `instance-id`, otherwise the configurations are identical.



## Application Settings
If you have a jboss-web.xml in your application, a security-domain defined there will override the default-security-domain in EAP's Undertow subsystem.  Inversely, if you don't have this file, or if you don't have a security-domain defined in this file, it will use the default.

```xml
<jboss-web xmlns="http://www.jboss.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.jboss.com/xml/ns/javaee http://www.jboss.org/j2ee/schema/jboss-web_7_0.xsd"
    version="7.0">

    <!-- Uncomment to override EAP's default -->
    <!-- <security-domain>override-with-this-security-domain-here</security-domain> -->

</jboss-web>
```

And that's it!  This is a common task to do when working with an enterprise as it eliminates the need for each application/team to specify the security-domain they need to use.



## References
1. [https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/how_to_configure_identity_management/application_configuration](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/how_to_configure_identity_management/application_configuration
)
