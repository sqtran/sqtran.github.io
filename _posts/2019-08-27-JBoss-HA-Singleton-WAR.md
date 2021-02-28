---
layout: single
title: JBoss EAP HA Singleton Web Applications
date: 2019-08-27
tags: jboss eap ha singleton
---


This is another entry about JBoss EAP, configured with HA, but now we're adding `singleton` web applications into the mix.

The `jboss-developer` community has a project on Github that has a ton of ready-to-go examples. See [jboss-eap-quickstarts](https://github.com/jboss-developer/jboss-eap-quickstarts/) to see all their different quickstarts.

I'm basing my work off of the [ha-single-deployment](https://github.com/jboss-developer/jboss-eap-quickstarts/tree/7.2.0.GA/ha-singleton-deployment) project, in the 7.2.0.GA branch.

Running `mvn clean package` produces a completely deployable jar file, but we're interested in creating a WAR file.  Luckily, it only takes a two modifications.

## Steps

###  Modify pom.xml
You'll need to change the packaging format from EJB to WAR.

```xml
<artifactId>ha-singleton-deployment</artifactId>
<!--packaging>ejb</packaging-->     <!-- original packaging -->
<packaging>war</packaging>          <!-- new packaging -->
<name>Quickstart: HA Singleton Deployment</name>
```

### Move singleton-deployment.xml

The sample project already has the singleton-deployment.xml file, so we just need to move it so that it's at the top level of the generated WAR.  Maven's convention is to look at the `webapp` folder, so just create that folder and move the META-INF folder there.

```bash
src/main
├── java
│   └── org
│       └── jboss
│           └── as
│               └── quickstarts
│                   └── ha
│                       └── singleton
│                           └── SingletonTimer.java
├── resources
└── webapp
    └── META-INF
        └── singleton-deployment.xml
```

That's it!  There are options for how you control the singleton deployment, but that's a topic for another day.
