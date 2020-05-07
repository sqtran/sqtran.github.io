---
layout: default
title: jboss-cli on OCP
---

## jboss-cli on OCP

If your application runs on Red Hat JBoss Enterprise Application Platform, or uses JBoss EAP as a base (such as BRMS, Decision Manager, Data Virtualization, etc.), you can quickly modify the behavior of the platform by using the `jboss-cli.sh` utility.  Although not tested, I don't see why it wouldn't work for the community Wildfly platforms too.

The only prerequisite is that the `oc` client is installed on your host machine.  The `jboss-cli.sh` utility is running within the container, so it is not needed locally.  You can run pretty much any command, and most operations take effect immediately, but if not, issuing a `:reload` command works too.

Here's an example to change the log level to `DEBUG` mode.  The `--` after the `exec` command allows you to pass in flags to the command you are trying execute in the container.

```bash
oc exec $(oc get pods -o name | grep your_pod_name) -- /opt/eap/bin/jboss-cli.sh --connect --command=/subsystem=logging/root-logger=ROOT:change-root-log-level\(level=DEBUG\)
```

Note that these changes are lost if the pod is restarted, unless a Persistent Volume is being used to save the `standalone.xml` configuration file.
