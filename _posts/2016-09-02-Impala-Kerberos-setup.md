---
layout: single
title: Impala Datasource with Kerberos
date: 2016-09-02
#categories: jboss jdv impala kerberos datasource
---

These are instructions on setting up an Impala EAP Datasource with Kerberos authentication on Windows.  Make sure you install your Impala drivers as a module too.  See previous post on how to do that if you haven't already done so.

## Configuration

### System Properties
Add the following system-properties in your standalone.xml (or domain.xml) configuration.  Note that `<system-properties>` are at the same level as `<extensions>`.


```xml
<system-properties>
    <property name="sun.security.krb5.debug" value="true"/> <!--Don't set this true in production -->
    <property name="java.security.krb5.conf" value="/path/to/krb5.conf"/> <!--Only if you want to override your system defaults -->
</system-properties>
```


### Cache Container
You'll also need to add a new cache-container configuration into the `Infinispan` subsystem.

```xml
<subsystem xmlns="urn:jboss:domain:infinispan:1.5">
    ...
    <cache-container name="security" default-cache="auth-cache">
      <local-cache name="impalaKerberos">
        <eviction strategy="LRU" max-entries="1000"/>
        <expiration max-idle="3540000" lifespan="3540000"/>
      </local-cache>
    </cache-container>
</subsystem>
```

#### Cache Container Details
In the example above, we set:
- **lifespan** : 59 minutes (in milliseconds) as being just below the 1 hour ticket lifespan as was used in our tests.
  Out of the box, Windows AD will issue tickets with a lifespan for 10h so you would for example set the lifespan to for example 9 hours.
  You just need to make sure it's lower then the actual lifespan. Don't set it to low, or there will be to many requests send to the KDC server.
- **max-idle**: 59 minutes (in milliseconds) : this is not very critical, it just means when a (still valid) ticket will be removed after it has not been used.
- **max-entries**: the maximum number of (copies of) the kerberos ticket you want to keep in the cache. This is a one-to-one with the maximum number of configured connections in your datasource.

See [https://access.redhat.com/solutions/218863](https://access.redhat.com/solutions/218863) for some additional information about how the `Infinispan` cache-container is configured.


### Security Domain
Next, you'll need to add a security domain.

```xml
<security-domain name="impalaKerberos" cache-type="infinispan">
  <authentication>
    <login-module name="Kerberos-Module" code="org.jboss.security.negotiation.KerberosLoginModule" module="org.jboss.security.negotiation" flag="required" >
      <module-option name="storeKey" value="false"/>
      <module-option name="useKeyTab" value="true"/>
      <module-option name="doNotPrompt" value="true"/>
      <module-option name="debug" value="true"/>
      <module-option name="keyTab" value="/path/to/eap.keytab"/>
      <module-option name="principal" value="someone@EXAMPLE.COM"/>
      <module-option name="isInitiator" value="true"/>
      <module-option name="refreshKrb5Config" value="true"/>
      <module-option name="addGSSCredential" value="true"/>
      <module-option name="wrapGSSCredential" value="true"/>
      <module-option name="credentialLifetime" value="-1"/>
    </login-module>
  </authentication>
</security-domain>
```

### Datasource
Finally, add your datasources like you normally would.

```xml
<datasources>
  ...
  <datasource jndi-name="java:/impala-ds" pool-name="01ImpalaDS" enabled="true" use-java-context="true">
      <connection-url>jdbc:impala://HOST:21051;AuthMech=1;KrbRealm=EXAMPLE.COM;KrbHostFQDN=server01.example.com;KrbServiceName=impala;SSL=1;CAIssuedCertNamesMismatch=1</connection-url>
      <driver>impala</driver>
      <security>
          <security-domain>impalaKerberos</security-domain>
      </security>
  </datasource>
  <drivers>
    ...
    <driver name="impala" module="org.apache.hadoop.impala">
        <driver-class>com.cloudera.impala.jdbc41.Driver</driver-class>
    </driver>
  </drivers>
</datasources>
```

### Datasource Details
Some important notes about the configuration above:
- **CAIssuedCertNamesMismatch** - If your Impala server's host name does not match the certificate (perhaps you used something self-signed without a SAN), you'll need to set this to 1
- **SSL** - defaults to 0, set to 1 to enable SSL
- **AuthMech** - The authorization mechanism to use.
  - 0 for No Authentication.
  - 1 for Kerberos.
  - 2 for User Name.
  - 3 for User Name And Password.
  - XX possibly others that I don't know about
