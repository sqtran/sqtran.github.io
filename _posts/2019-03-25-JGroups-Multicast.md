---
layout: post
title: JGroups Multicast
---

## JGroups Multicast

These are some basic JGroups Multicast debugging steps.

JBoss Wildfly/EAP has built-in High Availability capability.  It uses `JGroups` for intra-cluster communication such as replicating state information.  That's great out of the box in sandbox environments, but it's another story when you move this into a customer's environment.  By default, JBoss is configured for `UDP` multicasting, and a lot of customer environments don't like that; especially when you have servers in different network zones.  Routers/switches don't like to propagate that "noise" throughout their network, so they'll drop those packets.

If it's not the router/switch, it could also be how the server is configured.  I like to run `netstat -tuplen` to see what sockets are listening on TCP and UDP, along with the program name and user that is running it.  Also check to see if `iptables` or `firewalld` is blocking communication.  Out-of-the-box, I needed to open up **23364/udp**, **45688/udp**, **55200/udp**, and **45700/udp**.

At least on EAP 7.1, you have to configure `-Djboss.bind.address.private=<real_ip_address>` where the real_ip_address isn't 0.0.0.0.  I prefer to use `$(hostname -i)` so that IPs aren't hardcoded

The failure detection socket configuration also changed in EAP 7.1, so if you see messages in your logs like the following, you'll need to configure additional ports.

```
00:06:05,394 WARN  [org.jgroups.protocols.FD_SOCK] (FD_SOCK pinger,ejb,clusterA03.steve.com:server-one) clusterA03.steve.com:server-one: creating the client socket to 10.101.128.11:42503 failed: No route to host (Host unreachable)
00:06:05,429 WARN  [org.jgroups.protocols.FD_SOCK] (FD_SOCK pinger,ejb,clusterA03.steve.com:server-one) clusterA03.steve.com:server-one: creating the client socket to 10.101.128.10:35787 failed: No route to host (Host unreachable)
```

```xml
<stack name="udp">
  <transport type="UDP" socket-binding="jgroups-udp"/>
  <protocol type="PING"/>
  <protocol type="MERGE3"/>
  <protocol type="FD_SOCK">
    <property name="start_port">50000</property>
    <property name="port_range">0</property>
  </protocol>
  ...
</stack>
```

The `start_port` is the port that you want to listen to, while the `port_range` is how many additional ports you want to check.  EAP 7.1 automatically selects a random port, but configuring it this way lets you specify explicit firewall rules.  You will need to add **50000/tcp** as a rule, and note that this is **TCP** too.

If that doesn't work, then we can use a simple test program to verify that Multicasting is even configured correctly.  If you're using EAP6.1+, a test jar is already provided.  It's located at `$EAP_HOME/bin/client/jboss-client.jar`  You could always go straight to the source and compile it yourself if you're using JGroups but aren't using JBoss [1].

From one machine, run the Receiver.

```bash
java -cp $EAP_HOME/bin/client/jboss-client.jar org.jgroups.tests.McastReceiverTest -mcast_addr 230.11.11.11 -port 23364
```


And from the other machine, run the Sender
```bash
java -cp $EAP_HOME/bin/client/jboss-client.jar org.jgroups.tests.McastSenderTest -mcast_addr 230.11.11.11 -port 23364
```

You should see the messages entered on the sender appear on the receiver's terminal.  If all else fails, you can always configure JGroups with TCP.

References
1. https://github.com/belaban/JGroups/blob/master/tests/other/org/jgroups/tests/McastSenderTest.java
