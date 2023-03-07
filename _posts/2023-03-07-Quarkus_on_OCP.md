---
title: Quarkus on OpenShift
date: 2023-03-07
tags: quarkus ocp
---

## Background
Something changed since the last time I build and deployed a Quarkus app on OpenShift, because the default build-type is now a `fast-jar`.  You can run your application locally if you run the following command.

```bash
java -jar target/quarkus-app/quarkus-run.jar
```
When I tried to deploy this in OCP, via an S2I build, it kept giving me a missing manifest error.

```bash
Starting the Java application using /opt/jboss/container/java/run/run-java.sh ...
INFO exec  java -javaagent:/usr/share/java/jolokia-jvm-agent/jolokia-jvm.jar=config=/opt/jboss/container/jolokia/etc/jolokia.properties -XX:+UseParallelGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:+ExitOnOutOfMemoryError -cp "." -jar /deployments/hello-quarkus-1.0.0-SNAPSHOT.jar
no main manifest attribute, in /deployments/hello-quarkus-1.0.0-SNAPSHOT.jar
```

### Quick Fix

You can build an `uber-jar` aka fat jar by specifying it as a property in your `application.properties`, or setting it in your `pom.xml` under the properties section.  I'm leaning towards having it in the `pom.xml` so I can leave application specific configurations in the `application.properties`, but that's just a matter of preference.

After you package the app again, a file with similar pattern `hello-quarkus-1.0-SNAPSHOT-runner.jar` will be produced, which you can run anywhere now.