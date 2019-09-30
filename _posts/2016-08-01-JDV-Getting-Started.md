---
layout: default
title: JDV Getting Started
---

## JDV Getting Started

The purpose of this document is to help new developers/projects get familiar with the JDV environment.

### Introduction
If you are new to Data Virtualization, you can get started by learning from the articles and videos available online at http://www.jboss.org/products/datavirt/get-started/#!project=datavirt. Note that this is a link to the community version, known as `Teiid`. With open-source software from JBoss, there are community supported versions, and Red Hat supported versions. Both use the same code, but the Red Hat supported version is a tested version that is compatible with their current Middleware offerings. The community version may be bleeding edge, but has not gone through certification by Red Hat. With that said, the design patterns and best practices are universal no matter which version you pick.

### Software
If you have access to https://access.redhat.com, then you can download the installers and customize it to your liking. This is the best way to install the software since the installer will tailor it to your system.  The ZIP installer works fine, and may be sharable with other users, but I usually get into situations where my local paths are baked into the ZIP.


### Additional Software
JDV uses XML files for its underlying data structure, so GIT can be used to save those artifacts.

Deployments of VDBs can be done using any deployment tool, so something like Jenkins can help automate that entire process.  Anytool can/will work though.


### Documentation
All the official Red Hat documentation is available online. Please log into https://access.redhat.com in order to get the most up-to-date documentation.


### Practice
The following project on Github is a good way to get started with Data Virtualization. It walks a developer through the process of creating layers with traditional RDBMS as well as web services. This is not a Red Hat hosted Github project, but the example they use is very close to what you would learn in on-site training.

https://github.com/DataVirtualizationByExample/DVWorkshop


### Deployments
VDBs can be deployed two ways. Manual deployment via the Admin Console, or automated using Jenkins (or any other tool).
