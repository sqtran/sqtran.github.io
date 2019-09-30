---
layout: default
title: JBoss EAP7 Role Mapping
---

# JBoss EAP Role Mapping

Without Elytron, the EAP legacy security domain had a mapping-module that let you map between your domain-specific roles/groups to your application-specific roles.  This is necessary when your application has different role names that are loosely based on what your domain calls them.


### Server Side
This is the configuration you need in the legacy security domain.  Note the `<mapping>` element.  This goes in your `standalone.xml` or `domain.xml` file.

```xml
<security-domain name="my-example" cache-type="default">
    <authentication>
        <login-module code="LdapExtended" flag="required">
            ...
        </login-module>
    </authentication>
    <mapping>
        <mapping-module code="org.jboss.security.mapping.providers.DeploymentRoleToRolesMappingProvider" type="role"/>
    </mapping>
</security-domain>
```


### Application Side

In your application, you'll need to make changes in two files.

Here are the `web.xml` changes.
```xml
<security-constraint>
  <web-resource-collection>
    <web-resource-name>Restricted</web-resource-name>
    <url-pattern>/Secured/*</url-pattern>
  </web-resource-collection>
  <auth-constraint>
    <role-name>steve_role</role-name>
  </auth-constraint>
  <user-data-constraint>
    <transport-guarantee>CONFIDENTIAL</transport-guarantee>  <!-- Stops clear-text credentials on the wire.  Requires SSL to be configured.  Set to BASIC if you don't care -->
  </user-data-constraint>
</security-constraint>

<security-role>
  <role-name>steve_role</role-name> <!-- A list of roles the servlet knows about - this looks optional, but put it in here for good measure -->
</security-role>
```

Here are the `jboss-web.xml` changes.  Documentation says `jboss-app.xml` will also work too.
```xml
<security-domain>my-example</security-domain>
<security-role>
    <role-name>AD_ROLE_A</role-name>             <!-- AD Group -->
    <principal-name>steve_role</principal-name>  <!-- Application-specific role -->
</security-role>
```

If all your configuration is correct, but you're still running into authentication issues, set your logging settings to `TRACE` and look for the following.

```text
11:09:35,374 TRACE [org.jboss.security] (default task-10) PBOX00331: Roles before mapping: Roles(AD_ROLE_A,AD_ROLE_B,AD_ROLE_C,AD_ROLE_D,AD_ROLE_E,)
11:09:35,374 DEBUG [org.jboss.security] (default task-10) PBOX00322: Mapping provider options [principal: null, principal to roles map: {}, subject principals: [usersteve, Roles(members:AD_ROLE_A,AD_ROLE_B,AD_ROLE_C,AD_ROLE_D,AD_ROLE_E,)
11:09:35,374 DEBUG [org.jboss.security] (default task-1), CallerPrincipal(members:usersteve)]]
11:09:35,374 TRACE [org.jboss.security] (default task-10) PBOX00332: Roles after mapping: Roles(steve_role,AD_ROLE_B,AD_ROLE_C,AD_ROLE_D,AD_ROLE_E,)
```


References
1. https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html-single/login_module_reference/#deploymentroletorolesmappingprovider
