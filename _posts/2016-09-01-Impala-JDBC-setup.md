---
layout: post
title: JDV Cloudera/Impala Module
---

## Impala JDBC Driver Setup in EAP

JBoss EAP modules allows you to create a shared library that can be referenced by the platform, by any deployed application.  This is a good way to reduce code duplication if you have multiple applications bringing their own copy of a library.  This is also a good way to make sure every deployment is using a consistent version of a library.  A good use-case is when working with databases, and all your applications need a JDBC Driver.

It's typically pretty easy to set up.  You just add a folder to $JBOSS_HOME/modules, create a module.xml file, drop in your jar(s) and call it a day.  Doing so for Cloudera's Impala drivers took a little more work than usual though.  This is what I had to do to get it to work on JDV 6.4, which should work on JBoss EAP as well.

First, create your folder structure to in $JBOSS_HOME/modules.  I like to stay out of the "systems" directory as that folder's what comes with EAP out-of-the box.  It's just my preference to keep any of my customization in a separate area so that I don't accidentally break anything.   

I created this file `$JBOSS_HOME/modules/org.apache.hadoop.impala/module.xml` with the following content.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<module xmlns="urn:jboss:module:1.0" name="org.apache.hadoop.impala">
    <resources>
      <resource-root path="ImpalaJDBC41.jar"/>
      <resource-root path="hive_metastore.jar"/>
      <resource-root path="hive_service.jar"/>
      <resource-root path="libfb303-0.9.0.jar"/>
      <resource-root path="libthrift-0.9.0.jar"/>
      <resource-root path="TCLIServiceClient.jar"/>
    </resources>
 
    <dependencies>
        <module name="org.apache.log4j"/>
        <module name="org.slf4j"/>
        <module name="org.apache.commons.logging"/>
        <module name="javax.api"/>
        <module name="javax.resource.api"/>        
    </dependencies>
</module>
```

The Impala JDBC driver had a lot of dependencies, as you can see above.  I found most of this out by read through their documentation, and a lot of trial and error.

Once your module is setup, you'll need to add this driver into the "drivers" section of your configuration file.  This is either `standalone.xml` or `domain.xml` - based on what mode you're running.  This is pretty standard JBoss configuration at this point, so you should be able to take it from here.

```xml
<datasources>
  ...
  <drivers>
    <driver name="impala" module="org.apache.hadoop.impala">
        <driver-class>com.cloudera.impala.jdbc41.Driver</driver-class>
    </driver>
  </drivers>
</datasources>
```
