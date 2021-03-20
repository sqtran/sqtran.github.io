---
title: OCP and SpringBoot
date: 2021-03-20
tags: openshift springboot
---

## Background
I've helped about dozens of customers adopt Kubernetes and OpenShift in the past few years, and a recurring theme I keep hearing is how they believe rewriting their monolithic applications with `spring-boot` is their modernization strategy.  While I do agree that `spring-boot` is a great tool, most of the architects and developers I worked with seem to care more about the `boot` part than the `spring`.

While the `spring-boot` technology is newer, `Spring` has been around for a long time.  The `spring-boot` project makes it easy for developers to adopt the `Spring` Framework.  They allow developers to quickly bootstrap applications, and do things the `Spring` way via conventions over configurations.

These `spring-boot` applications are easily deployed to OpenShift as regular Java applications, and this post today is more of a note-to-self on how to configure Java applications, including `spring-boot`.


### Deployments

All applications need to be declaratively deployed in Kubernetes somehow.  It doesn't matter if it's a `Deployment`, `DeploymentConfig`, `StatefulSet`, etc., this is just part of Kubernetes.

An application should externalize configurational settings so the image being built can be deployed into any environment, which allows environment-specific properties to be loaded at runtime.  This follows the build once, deploy everywhere paradigm.  This approach allows an application image to be fully tested in lower environments, with development-centric properties/configurations (logging levels, test databases, feature flags, etc.).  Once an image is fully tested, it can be promoted to a higher environment with confidence that the code works.  Any issue found will most likely be a misconfiguration, or due to dirty data based on the nature of working with real, production data.


### Spring Boot Configurations

A `spring-boot` application reads properties from `src/main/resources/application.properties` by default.  Some deployment strategies include creating environment specific `application.properties` files every environment, and checking them all into the repository.  Another strategy would be to create a `ConfigMap` or `Secret`, and project this file into the running container.  We're going to do the latter today as an example, as the former is pretty straightforward.


### ConfigMap
Creating a `ConfigMap` in Kubernetes is straight forward.  If you have an existing properties file, you can use `oc create cm propfile --from-file=<file/path>`.  If not, you can just create a new ConfigMap and add the properties in yourself.  Just make sure that the key is the name of the file, and use the `|-` character to add values inline.

It should look something like this.

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
    name: propfile
data:
    application.properties: |-
        server.port=8080
        # more properties here
```

Once the `ConfigMap` is available, project it into the Deployment as a volume.  It can be anywhere, as long as it doesn't override anything important.  A safe place is `/deployments/config`


### Environment Variables

When deploying a `spring-boot` application onto OpenShift using the Red Hat OpenJDK base image, all there's left to do is set the `JAVA_ARGS` to point to the new location of where the `application.properties` file is.

**Note: `JAVA_ARGS` gets appended to the entrypoint script when executing the `java` command, so users can pass in any additional Java flags as needed.**

Set the ENV variable `JAVA_ARGS` value to `--spring.config.location=file://deployments/config/application.properties`


When the pod restarts, it will now be reading from the newly projected `ConfigMap`!