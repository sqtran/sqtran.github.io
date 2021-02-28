---
layout: single
title: Spring Boot No Main Manifest
date: 2019-10-08
tags: springboot java
---


## Problem

Seeing "no main manifest attribute" in your error message may seem obvious to some, but I see this stump a lot of developers I consult for.  You deploy your Spring Boot application to your Kubernetes cluster (Openshift in my example), but the deployment never rolls out.  Looking into the pod logs you see the following.

```java
Starting the Java application using /opt/run-java/run-java.sh ...
exec java -javaagent:/opt/jolokia/jolokia.jar=config=/opt/jolokia/etc/jolokia.properties -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:+UseParallelOldGC -XX:MinHeapFreeRatio=10 -XX:MaxHeapFreeRatio=20 -XX:GCTimeRatio=4 -XX:AdaptiveSizePolicyWeight=90 -XX:MaxMetaspaceSize=100m -XX:+ExitOnOutOfMemoryError -cp . -jar /deployments/your-app.jar
no main manifest attribute, in /deployments/your-app.jar
```

Before deploying to Openshift, running this locally would have given you clue that things might not work.  Executing a `mvn clean package spring-boot:run` would have quickly identified the runtime issue.

## Solution

The solution is to include the following in your `pom.xml` file.

```xml
<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
</plugins>
```
