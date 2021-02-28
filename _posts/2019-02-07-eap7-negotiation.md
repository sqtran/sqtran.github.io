---
title: EAP7 SPNEGO Classloading
date: 2019-02-07
tags: jboss eap classloading spnego
---

I recently encountered an issue that I believe is a bug in Red Hat EAP 7.1.5, specifically in the classloading of SPNEGO classes in EAR files.  I'm not sure if this affects all subdeployments and dependent modules, but the symptoms I've seen would lead me to believe it *should*.  There isn't anything special about SPNEGO classes that would force the Classloader to behave differently with other dependent modules.

I was working on integrating Single Sign On with my web application.  It required me to configure SPNEGO/Kerberos, with a fallback mechanism set to use another security-domain configured for Active Directory, as demonstrated [here](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/how_to_set_up_sso_with_kerberos) [1].  To make debugging easier, I used the jboss-[negotiation-toolkit](https://github.com/wildfly-security/jboss-negotiation/tree/master/jboss-negotiation-toolkit) [3] application, and validated that my configurations were correct.  I only saw an issue with SPNEGO classes when trying to deploy an EAR file.

## Symptoms

The symptoms appeared as a login prompt that seemed to be stuck in a loop.  Strange that the Kerberos configuration didn't work and it was falling back to AD, but we put that on hold and focused why we were stuck in this login loop.  We initially thought the username/password was not being passed to AD for authentication correctly.  What actually happened was that the SPNEGOLoginModule class was not being found by the classloader, so ClassNotFoundExceptions were printed in the logs.  I think this classloader error was fatal enough to break the SPNEGO security-domain, despite only being printed at the DEBUG level.  Since it broke the entire security-domain, it was never actually getting to the configuration that specified it would use the AD fallback mechanism.  I think it was actually defaulting to whatever the server uses when nothing is configured.  It wasn't the users.properties file though, cause that didn't seem to work either.

The logs printed out that the classloader wasn't looking at all the jars defined in the module.xml file.  See `jboss-eap-7.1/modules/system/layers/base/org/jboss/security/negotiation/main`.  So to "assist" the classloader, I explicitly declared the JBoss module jar (`jboss-negotiation-spnego-3.0.4.Final-redhat-1.jar`) that contains the necessary classes to be available for all deployments, which is just a matter of setting it up the [Global Modules](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/configuration_guide/overview_of_class_loading_and_modules#add_a_global_module) [2].  After doing this, the embedded WAR file could now see and load the SPNEGO security-domain.  The logs also confirmed that the classloader found what it needed.

```
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "deployment.directory-assistance-ear-2.0.0-200181221.200554-2-steve.ear.directory-assistance-ui-2.0.0-SNAPSHOT.war" from Service Module Loader
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "deployment.directory-assistance-ear-2.0.0-200181221.200554-2-steve.ear.directory-assistance-ui-2.0.0-SNAPSHOT.war" from Service Module Loader
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "org.picketbox" from local module loader @340f438e (finder: local module finder @30c7da1e (roots: /opt/rh/eap7/root/usr/share/wildfly/modules,/opt/rh/eap7/root/usr/share/wildfly/modules/system/layers/base))
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Finding class org.jboss.security.negotiation.spnego.SPNEGOLoginModule from Module "deployment.directory-assistance-ear-2.0.0-200181221.200554-2-steve.ear.directory-assistance-ui-2.0.0-SNAPSHOT.war" from Service Module Loader
2019-02-01 15:32:12,655 TRACE [org.jboss.modules] (default task-10) Class org.jboss.security.negotiation.spnego.SPNEGOLoginModule not found from Module "deployment.directory-assistance-ear-2.0.0-200181221.200554-2-steve.ear.directory-assistance-ui-2.0.0-SNAPSHOT.war" from Service Module Loader
2019-02-01 15:32:12,655 DEBUG [org.jboss.security] (default task-10) PBOX00206: Login failure: javax.security.auth.login.LoginException: unable to find LoginModule class: org.jboss.security.negotiation.spnego.SPNEGOLoginModule
        at javax.security.auth.login.LoginContext.invoke(LoginContext.java:794)
        at javax.security.auth.login.LoginContext.access$000(LoginContext.java:195)
        at javax.security.auth.login.LoginContext$4.run(LoginContext.java:682)
        at javax.security.auth.login.LoginContext$4.run(LoginContext.java:680)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.login.LoginContext.invokePriv(LoginContext.java:680)
        at javax.security.auth.login.LoginContext.login(LoginContext.java:587)
        at org.jboss.security.authentication.JBossCachedAuthenticationManager.defaultLogin(JBossCachedAuthenticationManager.java:406)
        at org.jboss.security.authentication.JBossCachedAuthenticationManager.proceedWithJaasLogin(JBossCachedAuthenticationManager.java:345)
        at org.jboss.security.authentication.JBossCachedAuthenticationManager.authenticate(JBossCachedAuthenticationManager.java:323)
        at org.jboss.security.authentication.JBossCachedAuthenticationManager.isValid(JBossCachedAuthenticationManager.java:146)
        at org.wildfly.extension.undertow.security.JAASIdentityManagerImpl.verifyCredential(JAASIdentityManagerImpl.java:123)
        at org.wildfly.extension.undertow.security.JAASIdentityManagerImpl.verify(JAASIdentityManagerImpl.java:94)
        at io.undertow.security.impl.BasicAuthenticationMechanism.authenticate(BasicAuthenticationMechanism.java:167)
        at io.undertow.security.impl.SecurityContextImpl$AuthAttempter.transition(SecurityContextImpl.java:245)
        at io.undertow.security.impl.SecurityContextImpl$AuthAttempter.transition(SecurityContextImpl.java:263)
        at io.undertow.security.impl.SecurityContextImpl$AuthAttempter.transition(SecurityContextImpl.java:263)
        at io.undertow.security.impl.SecurityContextImpl$AuthAttempter.access$100(SecurityContextImpl.java:231)
        at io.undertow.security.impl.SecurityContextImpl.attemptAuthentication(SecurityContextImpl.java:125)
        at io.undertow.security.impl.SecurityContextImpl.authTransition(SecurityContextImpl.java:99)
        at io.undertow.security.impl.SecurityContextImpl.authenticate(SecurityContextImpl.java:92)
        at io.undertow.servlet.handlers.security.ServletAuthenticationCallHandler.handleRequest(ServletAuthenticationCallHandler.java:55)
        at io.undertow.server.handlers.DisableCacheHandler.handleRequest(DisableCacheHandler.java:33)
        at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)
        at io.undertow.security.handlers.AuthenticationConstraintHandler.handleRequest(AuthenticationConstraintHandler.java:53)
        at io.undertow.security.handlers.AbstractConfidentialityHandler.handleRequest(AbstractConfidentialityHandler.java:46)
        at io.undertow.servlet.handlers.security.ServletConfidentialityConstraintHandler.handleRequest(ServletConfidentialityConstraintHandler.java:64)
        at io.undertow.servlet.handlers.security.ServletSecurityConstraintHandler.handleRequest(ServletSecurityConstraintHandler.java:59)
        at io.undertow.security.handlers.AuthenticationMechanismsHandler.handleRequest(AuthenticationMechanismsHandler.java:60)
        at io.undertow.servlet.handlers.security.CachedAuthenticatedSessionHandler.handleRequest(CachedAuthenticatedSessionHandler.java:77)
        at io.undertow.security.handlers.NotificationReceiverHandler.handleRequest(NotificationReceiverHandler.java:50)
        at io.undertow.security.handlers.AbstractSecurityContextAssociationHandler.handleRequest(AbstractSecurityContextAssociationHandler.java:43)
        at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)
        at org.wildfly.extension.undertow.security.jacc.JACCContextIdHandler.handleRequest(JACCContextIdHandler.java:61)
        at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)
        at org.wildfly.extension.undertow.deployment.GlobalRequestControllerHandler.handleRequest(GlobalRequestControllerHandler.java:68)
        at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43)
        at io.undertow.servlet.handlers.ServletInitialHandler.handleFirstRequest(ServletInitialHandler.java:292)
        at io.undertow.servlet.handlers.ServletInitialHandler.access$100(ServletInitialHandler.java:81)
        at io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:138)
        at io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:135)
        at io.undertow.servlet.core.ServletRequestContextThreadSetupAction$1.call(ServletRequestContextThreadSetupAction.java:48)
        at io.undertow.servlet.core.ContextClassLoaderSetupAction$1.call(ContextClassLoaderSetupAction.java:43)
        at org.wildfly.extension.undertow.security.SecurityContextThreadSetupAction.lambda$create$0(SecurityContextThreadSetupAction.java:105)
        at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1501)
        at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1501)
        at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1501)
        at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1501)
        at org.wildfly.extension.undertow.deployment.UndertowDeploymentInfoService$UndertowThreadSetupAction.lambda$create$0(UndertowDeploymentInfoService.java:1501)
        at io.undertow.servlet.handlers.ServletInitialHandler.dispatchRequest(ServletInitialHandler.java:272)
        at io.undertow.servlet.handlers.ServletInitialHandler.access$000(ServletInitialHandler.java:81)
        at io.undertow.servlet.handlers.ServletInitialHandler$1.handleRequest(ServletInitialHandler.java:104)
        at io.undertow.server.Connectors.executeRootHandler(Connectors.java:330)
        at io.undertow.server.HttpServerExchange$1.run(HttpServerExchange.java:812)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)

2019-02-01 15:32:12,656 TRACE [org.jboss.security] (default task-10) PBOX00201: End isValid, result = false

2019-02-01 15:32:12,656 DEBUG [io.undertow.request.security] (default task-10) Authentication failed with message UT000038: Authentication failed, requested user name 'steve' and mechanism BASIC for HttpServerExchange{ GET /mywebapp/ request {accept=[text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8], accept-language=[en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7], accept-encoding=[gzip, deflate, br], dnt=[1], user-agent=[Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.67 Safari/537.36], authorization=[Basic dmVucmhlMTY6V2VsY29tZTFvZGZs], upgrade-insecure-requests=[1], Host=[dc1-d-a-common01.dev.odcloud.odfl.com:8543]} response {Expires=[0], Cache-Control=[no-cache, no-store, must-revalidate], X-Powered-By=[Undertow/1], Server=[JBoss-EAP/7], Pragma=[no-cache]}}
2019-02-01 15:32:12,656 DEBUG [io.undertow.request.security] (default task-10) Authentication outcome was NOT_AUTHENTICATED with method io.undertow.security.impl.BasicAuthenticationMechanism@19f339e5 for HttpServerExchange{ GET /mywebapp/ request {accept=[text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8], accept-language=[en-US,en;q=0.9,vi-VN;q=0.8,vi;q=0.7], accept-encoding=[gzip, deflate, br], dnt=[1], user-agent=[Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.67 Safari/537.36], authorization=[Basic dmVucmhlMTY6V2VsY29tZTFvZGZs], upgrade-insecure-requests=[1], Host=[dc1-d-a-common01.dev.odcloud.odfl.com:8543]} response {Expires=[0], Cache-Control=[no-cache, no-store, must-revalidate], X-Powered-By=[Undertow/1], Server=[JBoss-EAP/7], Pragma=[no-cache]}}
```

## Solution

So I added the following XML stanza into my `standalone.xml` or `domain.xml`, which seems like it increased its order of precedence.
```xml
<subsystem xmlns="urn:jboss:domain:ee:4.0">
  <global-modules>
    <module name="org.jboss.security.negotiation" slot="main" />
  </global-modules>
  ...
</subsystem>
```

## References

1. [https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/how_to_set_up_sso_with_kerberos/how_to_set_up_sso_for_jboss_eap_with_kerberos#configure_legacy_security_subsystem](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/how_to_set_up_sso_with_kerberos/how_to_set_up_sso_for_jboss_eap_with_kerberos#configure_legacy_security_subsystem)
2. [https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/configuration_guide/overview_of_class_loading_and_modules#add_a_global_module](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.1/html/configuration_guide/overview_of_class_loading_and_modules#add_a_global_module)
3. [https://github.com/wildfly-security/jboss-negotiation/tree/master/jboss-negotiation-toolkit](https://github.com/wildfly-security/jboss-negotiation/tree/master/jboss-negotiation-toolkit)
