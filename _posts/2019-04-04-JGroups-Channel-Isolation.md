---
layout: default
title: JGroups Channel Isolation
---

## JGroups Channel Isolation

This is a continuation of my previous post about JGroups.  See previous post for more background.

After standing up two HA clusters with UDP multicast, our users started seeing errors with their deployments.  The logs showed cross-pollination of UDP messages.  Messages from "clusterA" was joining "clusterB" as indicated by messages like the following in the `server.log` file.

```
...
2019-04-04 12:36:07,438 INFO  [org.infinispan.remoting.transport.jgroups.JGroupsTransport] (thread-2) ISPN000094: Received new cluster view for channel ejb: [clusterA02.steve.com:server-one|46] (5) [clusterA02.steve.com:server-one, clusterA03.steve.com:server-one, master:server-one, clusterB03.steve.com:server-one, clusterB02.steve.com:server-one]
2019-04-04 12:36:07,438 INFO  [org.infinispan.remoting.transport.jgroups.JGroupsTransport] (thread-2) ISPN000094: Received new cluster view for channel ejb: [clusterA02.steve.com:server-one|46] (5) [clusterA02.steve.com:server-one, clusterA03.steve.com:server-one, master:server-one, clusterB03.steve.com:server-one, clusterB02.steve.com:server-one]
2019-04-04 12:36:07,439 INFO  [org.infinispan.remoting.transport.jgroups.JGroupsTransport] (thread-2) ISPN000094: Received new cluster view for channel ejb: [clusterA02.steve.com:server-one|46] (5) [clusterA02.steve.com:server-one, clusterA03.steve.com:server-one, master:server-one, clusterB03.steve.com:server-one, clusterB02.steve.com:server-one]
2019-04-04 12:36:07,439 INFO  [org.infinispan.remoting.transport.jgroups.JGroupsTransport] (thread-2) ISPN000094: Received new cluster view for channel ejb: [clusterA02.steve.com:server-one|46] (5) [clusterA02.steve.com:server-one, clusterA03.steve.com:server-one, master:server-one, clusterB03.steve.com:server-one, clusterB02.steve.com:server-one]
...
```

Searching the internet didn't yield any immediate results, so I took a look at the `domain.xml` file directly to see if anything stood out.  JBoss EAP comes with one `channel` configured by default, when you're using the `ha` or `full-ha` EAP profiles.

```xml
<subsystem xmlns="urn:jboss:domain:jgroups:5.0">
  <channels default="ee">
      <channel name="ee" stack="udp" cluster="ejb"/>
  </channels>
  <stacks>
  ...
</subsystem>
```

I'm guessing we could add additional channels, and update our configuration file to specify a particular channel to use.  The quick and dirty way to fix this was to change the cluster name from "ejb" to something unique for this cluster.  That way all the underlying plumbing wouldn't need to change because the default channel would still be `ee`.  

I changed the cluster name to "ejb4steve" as an example, so the configuration looked like the following.  When moving this to a real environment, just make sure cluster is a unique name for that network.
```xml
<subsystem xmlns="urn:jboss:domain:jgroups:5.0">
  <channels default="ee">
      <channel name="ee" stack="udp" cluster="ejb4steve"/>
  </channels>
  <stacks>
  ...
</subsystem>
```

After making our changes, the logs indicated that we had successfully filtered out any noise from our noisy neighbors.

```
2019-04-04 18:28:57,226 WARN  [org.jboss.as.clustering.jgroups.protocol.UDP] (thread-3) JGRP000012: discarded message from different cluster ejb (our cluster is ejb4steve). Sender was 51736315-56be-03fe-f0a6-cd6e6e1952e0
2019-04-04 18:29:40,365 WARN  [org.jboss.as.clustering.jgroups.protocol.UDP] (thread-1) JGRP000012: discarded message from different cluster ejb (our cluster is ejb4steve). Sender was 4615b360-d3cb-234d-2f3c-5d0d4fda8409 (received 25 identical messages from 4615b360-d3cb-234d-2f3c-5d0d4fda8409 in the last 60001 ms)
2019-04-04 18:29:43,745 WARN  [org.jboss.as.clustering.jgroups.protocol.UDP] (thread-1) JGRP000012: discarded message from different cluster ejb (our cluster is ejb4steve). Sender was e9061bb1-2056-e3f5-7a1d-24b4a934d698 (received 7 identical messages from e9061bb1-2056-e3f5-7a1d-24b4a934d698 in the last 60001 ms)
2019-04-04 18:29:57,552 WARN  [org.jboss.as.clustering.jgroups.protocol.UDP] (thread-1) JGRP000012: discarded message from different cluster ejb (our cluster is ejb4steve). Sender was 51736315-56be-03fe-f0a6-cd6e6e1952e0 (received 31 identical messages from 51736315-56be-03fe-f0a6-cd6e6e1952e0 in the last 60326 ms)
```

Since we were still multicasting, any new cluster on our network will contribute more WARNs to our logs.  So for good measure, you can easily set a log category to filter that out like the following.

```xml
<subsystem xmlns="urn:jboss:domain:logging:3.0">
  ...
  <logger category="org.jboss.as.clustering.jgroups.protocol.UDP">
     <level name="ERROR"/>
  </logger>
  ...
</subsystem>
```
