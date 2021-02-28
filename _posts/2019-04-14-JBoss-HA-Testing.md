---
layout: single
title: JBoss EAP HA Testing
date: 2019-04-14
#categories: jboss eap ha testing
---

This is another continuation of my previous post about JGroups.  By now, you've probably already configured your EAP servers in HA mode, and you'll need a way to verify that this is even working.  We should use a sample test program before attempting to deploy a real application, just to eliminate any variables that may interrupt our clustering tests.

## Sample Application
Fortunately, Red Hat created an example, which I have access to, but I could not find where it was publicly hosted on Github, so I posted a copy to my personal repository.  [session-counter](https://github.com/sqtran/session-counter)

It used to be an `ant` project, but I converted it to Maven just because most people prefer Maven over Ant.

## Configuration
The `jboss-web.xml` is the only place where you'll need any configuration (if necessary).  The test application requires the use of a cache, which must be predefined on EAP ahead of time.  Out of the box, EAP has one named "web" which is what the example uses.  You can override this value with different caches for testing various configurations.

## Test
After the application is deployed, you can open up your browser and go to http://localhost:8080/counter.  Every time the page is reloaded, a session counter is incremented.

A sample curl command is displayed on the page, which you can paste into a terminal.  The `JSessionID` is how your request knows which session counter to update.  It should increment regardless if your request comes from the browser, or from the terminal, just as long as the `JSessionID` is the same.

```bash
curl -v http://www.example.com:8080/counter --header "Cookie: JSESSIONID=ueRgXKhBFRjRubmyzF5z2LjZZIdJzi_nG2zUA-2y"
```

Note: configure your `host` file so that example.com will map to localhost, so that you can mimic having two servers.