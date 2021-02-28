---
layout: single
title: JBoss EAP7 Perimeter Authentication
date: 2019-02-16
#categories: jboss eap authn
---

According to Oracle's Documentation, Weblogic Server has this feature called [Perimeter Authentication](https://docs.oracle.com/cd/E13212_01/wles/docs42/secintro/services.html#1056409) [5], which all requests must pass through before accessing any content being served by EAR/WARs.  This allows the apps to be configured with BASIC (clear text) auth, as defined in their web.xml with <auth-method>BASIC/auth-method>, but use SSO so domain users don't need to log in again.

JBoss doesn't have the same Perimeter Authentication functionality that Weblogic does.  People have emulated similar functionality with [JBoss Valves](https://tomcat.apache.org/tomcat-5.5-doc/catalina/docs/api/org/apache/catalina/authenticator/SingleSignOn.html) [3], which are essentially request interceptors that you inject into the request/response pipeline (think of CXF interceptors).  This was the norm in the EAP 6.1.X series.  In EAP 7, the web subsystem was completely replaced with Undertow, and `Valves` were replaced with `Handlers`.  Undertow (EAP7) now comes built-in with [SPNEGO capability](https://github.com/wildfly-security/jboss-negotiation/blob/3.0.3.Final/jboss-negotiation-common/src/main/java/org/jboss/security/negotiation/NegotiationMechanism.java) [2], so we don't need the emulation with JBoss `Valve` like we did in EAP 6.  The old JBoss Valve behaved like Weblogic in the sense that it intercepted all requests, regardless of what was defined in the web.xml settings.  With EAP 7, that's not the documented solution anymore.

To handle SPNEGO, Wildfly (what EAP7 is based on) built in an authentication mechanism (auth-method) to handle [SPNEGO](https://github.com/wildfly/wildfly/blob/10.1.0.Final/undertow/src/main/java/org/wildfly/extension/undertow/ServletContainerAdd.java#L121) [1].  This is a JBoss-specific value that is being used in the JEE-standard web.xml file.  I disagree with this approach because I would have preferred vendor-specific code in jboss-web.xml, but this is Red Hat's documented [solution](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html-single/how_to_set_up_sso_with_kerberos/index#spnego
) [4].  They want us to set  <auth-method>SPNEGO</auth-method> in the web.xml file, so anything else would mean doing something custom.  This is only a concern if you want to maintain RH Support.


## References  
1. [https://github.com/wildfly/wildfly/blob/10.1.0.Final/undertow/src/main/java/org/wildfly/extension/undertow/ServletContainerAdd.java#L121](https://github.com/wildfly/wildfly/blob/10.1.0.Final/undertow/src/main/java/org/wildfly/extension/undertow/ServletContainerAdd.java#L121)
2. [https://github.com/wildfly-security/jboss-negotiation/blob/3.0.3.Final/jboss-negotiation-common/src/main/java/org/jboss/security/negotiation/NegotiationMechanism.java](https://github.com/wildfly-security/jboss-negotiation/blob/3.0.3.Final/jboss-negotiation-common/src/main/java/org/jboss/security/negotiation/NegotiationMechanism.java)
3. [https://tomcat.apache.org/tomcat-5.5-doc/catalina/docs/api/org/apache/catalina/authenticator/SingleSignOn.html](https://tomcat.apache.org/tomcat-5.5-doc/catalina/docs/api/org/apache/catalina/authenticator/SingleSignOn.html)
4. [https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html-single/how_to_set_up_sso_with_kerberos/index#spnego](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html-single/how_to_set_up_sso_with_kerberos/index#spnego)
5. [https://docs.oracle.com/cd/E13212_01/wles/docs42/secintro/services.html#1056409](https://docs.oracle.com/cd/E13212_01/wles/docs42/secintro/services.html#1056409)
