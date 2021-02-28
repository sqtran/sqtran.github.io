---
layout: single
title: Application Signal Handling
date: 2018-04-11
tags: appdev containers ocp springboot
---

## Signal Handling

Signals are how the operating system (or other applications) interact with running processes.  Whether it's running on bare metal, virtual machines, containers, or all of the above on a PaaS platform, the onus is on the application to interpret these signals and respond appropriately.  This general topic is known as signal handling, but we're taking a narrower subset of these signals and only focusing on two signals - `SIGINT` and `SIGTERM`

In the web based world, we run our web-applications inside Servlet Containers.  There's plenty of different flavors of Servlet Containers to choose from, but for this example, we're only dealing with Tomcat and Springboot.  Both have their advantages and disadvantages, but this post isn't to discuss which one you should use.  Regardless of the Servlet Container you use, there is default behavior built-in to handle a `SIGINT` and `SIGTERM` signal.  It's up to you as a developer to determine how to handle these operations.

Part of handling a termination signal (`SIGTERM`) could be performing a graceful shutdown, depending on what your application actually does. A graceful shutdown can involve any number of things.  It could be making sure in-flight transactions (requests that have started but has not yet sent a response) are given a chance to complete before terminating, or making sure resources such as file handlers or connection pools are released, or executing a database commit to ensure database-transactions are atomic.  There is not a one-size-fits-all solution, and a default behavior will not apply to all situations.

## Considerations
Gracefully shutting down your application is not a guarantee that requests and/or resources will be properly handled.  A `SIGTERM` can be immediate if the OS deems it fatal enough.  The application architect needs to design their software with this in mind, and implement semaphores and sentinels as appropriate to make their application as durable as possible.  Also keep in mind that there are various sources of signals, especially when you move your application onto a PaaS platform because it introduces new layers of hardware/software abstractions.  A `SIGTERM` can now be sent by OCP when it needs to scale down.

Also ask yourself if you even need to gracefully shutdown your application.  Applications that interact with databases should be creating atomic transactions, and those automatically get rolled back if the unit fails (or committed when the entire transaction is complete).  If your application absolutely needs to have a large transaction window, there may be other design patterns to keep your service as state-free as possible.

Lastly, you could run into zombie threads if you force the JVM to hang while waiting for shutdown code to complete.  This could happen if you force a long-executing database write to finish before closing, or waiting indefinitely for an external system to give you a confirmation.

## Examples
We've set up two examples of how you can continue to process in-flight transactions before actually terminating.  Your application may not need to continue processing in-flight transactions, but the method of how we are trapping these signals can be used as a bootstrap.

DISCLAIMER : This is not intended to be your definitive guide of what you *MUST* do.  This is how you *COULD* go about building your signal handling.

### Spring Boot

The Springboot lifecycle is a little different than the typical tomcat Servlet lifecycle.  Springboot creates a different type of [Application Context] (https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications) [1] that isn't the same as the Servlet Context used by base tomcat.  What this means for you as the developer is that you need to implement your trapping method a little differently.

What we want to do is create a method with a signature that has arguments for `(ContextClosedEvent cce)`.  Springboot does a class introspection to determine which EventListener to call based on the Event argument type.
In our code example, you'll see that we implemented the method to wait for any Threads still we recorded as not finished.  The method gets called before the ServletContext gets closed, which is different than how tomcat behaves.  If you don't intercept the `SIGTERM` at this point in the handling chain, it will be too late to send a response back to whoever initiated the http request.  If you don't need to send a response back, it would be perfectly acceptable to trap this further down the chain.

We also synchronized the method to ensure that multiple Threads aren't running the shutdown method.  If your business logic inside the shutdown method is already thread-safe, then the synchronized keyword can be removed.

If your application doesn't need to send http responses back, you could also register a shutdown hook with the JVM.   The following will gets executed right before the JVM is shutdown, so it executes late in the lifecycle.  This is pretty much the last chance before the JVM is terminated.  Be aware that if multiple shutdown hooks are added, there is no guaranteed order they will be executed in.  If you do choose to go down the shutdown hook route, it is assumed that it will be added early in the life cycle, preferably in an init method.

```java
Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
  public void run() {
    // Your code here
  }
}));
```

### Tomcat

With Tomcat, you only need create a class that implements the javax.servlet.ServletContextListener interface.  One of the interface methods is `contextDestroyed(ServletContextEvent sce)`.  This method gets called when the Servlet Context is about to be destroyed, which is one of the last few methods in the shutdown chain.  Note that `@WebListener` requires Servlet 3.0, so if you're at an older version and can't upgrade, you'll need to register your "YourGracefulShutdownClass" manually in your web.xml file.

```java
​// Define a class that Implements the ServletContextListener Interface, as required by the @WebListener annotation
@WebListener
public class YourGracefulShutdownClass implements ServletContextListener {
	@Override
	public void contextDestroyed(ServletContextEvent servletContextEvent) {
        // Your code here
	}
}
```

Add this to your pom.xml to import the ServletContextListener interface and the `@WebListener` annotation.
```xml
​<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<version>3.0.1</version>   <!-- Pick a version appropriate for your application -->
	<scope>provided</scope>
</dependency>
```

The following is only needed if you don't have the the `@WebListener` annotation available in your servlet-api version (pre 3.x)
```xml
<listener>
	<listener-class>com.example.YourGracefulShutdownClass</listener-class>
</listener>
```

### Thread Tracking

Whether using Springboot or Tomcat, your application needs to know how much work it currently has outstanding.  You'll need to build your application so that it is aware of its current state.  In the two examples we've provided, there is a thread-safe Queue that keeps a reference to every thread that is spun up.  This takes advantage of the fact that we know Servlets are inherently multi-threaded.  That is, every request a servlet receives is spawned as a new thread from the Servlet Container.  We also know our test application is simple and doesn't spawn other worker threads, nor does it do anything with file handles, streaming input/output, make asynchronous calls, or connect to databases.  You'll need to define what it means to be able to gracefully shutdown, and handle those resources on a case-by-case basis.  This might require you to do some refactoring if you aren't already handling it properly.

## Testing
You should test your application independently before moving it into OpenShift.  This is a critical step in software development because how your WAR or JAR behaves in your local test environment should be how your application behaves when deployed as an image in OpenShift.  You can send `SIGINT` and `SIGTERM` from the terminal by executing the "kill " command on Linux.  Also, sending a CTRL-C will send a non -9 `SIGTERM` as well to Apache Tomcat and Springboot.

## References
1. [https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-web-applications)
