---
layout: single
title: Openconnect on RHEL7
date: 2018-12-03
#categories: openconnect
---

## Background

Customers who let you do remote work will most likely not have internet-accessible servers (probably for the best).  If you're lucky, you'll be able to VPN into their network, or at least onto a bastion host to get to other machines.  Otherwise, the worst case scenario is you'll have to use some sort of virtual remote desktop, or a Citrix web-client.

I've found that using the open source openconnect client is probably the most hassle-free approach for connecting to VPN on RHEL/Centos/Fedora (many distros).  It saves you the headache of all the CNTLM hackery, installing Cisco Anyconnect, or whatever magic you need to get it to work.

## Installation
First, you'll need to make sure openconnect is installed, or install it if it's not...

```bash
sudo yum install -y openconnect
```

You then connect with the following.  The actual flags required will be different depending on what the VPN server requires.  The good-ole-man pages is still the best place to find out more.  
```bash
sudo openconnect -v vpn_host_address -u username
```

Unless the VPN server has publicly issued certificates, you'll probably get a warning about connecting to the server for the first time.  

You can pass in this flag if you want to suppress that warning `--servercert sha256:XXXXX`

Other than that, it's super easy to fire up a terminal and start a VPN connection.  You can take it one step further and create a script that does all the legwork for you.  It might not be 100% script-able if you need to enter a PIN+OTP, but at least it's less commands to type.