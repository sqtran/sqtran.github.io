---
layout: single
title: Ansible Dynamic Groups Trick
date: 2019-05-28
tags: ansible tips
---

Here is a handy Ansible task to dynamically create a list of hosts, based on items in an inventory.  This was created specifically for a playbook dealing with `JBoss EAP` `TCP_PING` configurations, but the technique can be applied to various other applications.

## Steps
First set an Ansible Fact that will hold the list.
{% raw %}
```yml
- name: Collect IPs for the entire cluster
  set_fact:
    list_of_ips: "{{ groups['cluster'] | map('extract', hostvars, ['ansible_default_ipv4', 'address']) | join('[7600],') }}[7600]"
```
{% endraw %}

Note that we are also pulling out the `ansible_default_ipv4` field out of `hostvars`.  We're appending "[7600]" because that's what JBoss expects the default port to be, but that is not pertinent to what we are doing with Ansible here.


Now that the variable has been set, you can use it anywhere.
```yml
- name: Set TCPPING Protocol Configurations
  xml:
    path: "{{ config_path }}"
    namespaces:
      a: urn:jboss:domain:5.0
      b: urn:jboss:domain:jgroups:5.0
    xpath: "/a:domain/a:profiles/a:profile[@name='{{eap_profile}}']/b:subsystem/b:stacks/b:stack[@name='tcp']/b:protocol[@type='org.jgroups.protocols.TCPPING']/b:property[@name='{{item.name}}']"
    pretty_print: true
    value: "{{item.value}}"
  loop:
    - { name: "initial_hosts", value: "{{ list_of_ips }}"}
    - { name: "port_range", value: "0"}
    - { name: "timeout", value: "300000"}
    - { name: "num_initial_members", value: "{{groups['cluster']|length}}"}
```

### Bonus
There's a bonus tip here - you can do call `length` on the map to get the number of hosts in that particular Ansible group.

The only issue with this task is it expects there to be at least 1 host in the inventory file, which is probably a safe assumption.  You could make this more robust and be more defensive.
