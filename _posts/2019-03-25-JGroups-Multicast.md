---
layout: post
title: JGroups Multicast
---

## JGroups Multicast

These are some basic JGroups Multicast debugging steps.

JBoss Wildfly/EAP has built-in High Availability capability.  It uses `JGroups` for intra-cluster communication such as replicating state information.  That's great out of the box in sandbox environments, but it's another story when you move this into a customer's environment.  By default, JBoss is configured for `UDP` multicasting, and a lot of customer environments don't like that; especially when you have servers in different network zones.  Routers/switches don't like to propagate that "noise" throughout their network, so they'll drop those packets.

If it's not the router/switch, it could also be how the server is configured.  I like to run `netstat -tuplen` to see what sockets are listening on TCP and UDP, along with the program name and user that is running it.  Also check to see if `iptables` or `firewalld` is blocking communication.  Out-of-the-box, HA wants to be listening on **23364/udp** and **45688/udp**

At least on EAP 7.1, you have to configure `-Djboss.bind.address.private=<real_ip_address>` where the real_ip_address isn't 0.0.0.0.  I prefer to use `$(hostname -i)` so that IPs aren't hardcoded

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
