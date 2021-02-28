---
layout: single
title: Windows Batch Fun
date: 2020-09-16
#categories: windows batch
---

## Why?

Most of my time is spent on either Red Hat Enterprise Linux for work, or Fedora for play.  But more times than not, my customers require that I remote into their environment.  It's usually a Virtual Desktop Interface, and most likely a Windows environment, so I get to explore the Wild Wild World of Windows.

This time around, for whatever reason - my putty sessions were not being persisted between Citrix Remote Sessions.  This was a pain because I was administering a ton of servers, and due to how the remote session worked, I couldn't cut and paste from my host machine into the virtual desktop.  Plus, all the server names did not follow a naming convention, so two servers in a DEV environment weren't necessarily closely named (like having a number or letter incremented).

## Batch Tips
So my ~~lazy~~ efficient developer brain created a bunch of Windows Batch files to would organize my SSH sessions. I present to you a neat trick that probably took me an hour to work through, but will save me far more time in the future.

First, I organized all my servers by name.  I prefixed all the server names with their respective environment.  In particular, I kept it to a 4 character prefix: `dev-`, `qat-`, `int-`, or `prd-`.  

Then I created a batch file with the following content.
```bat
set name=%~n0
start putty.exe -ssh username@%name:~4%
```

I didn't want to open up each file and add the server name to it, since the file name already indicated the server.  So, what the `.bat` file above does is take its own file name as a variable and strips off the first 4 characters.   I didn't know Windows Batch files were so flexible.  

So now I'm left with a folder with several dozens of these batch files.  An added benefit is adding new servers to my inventory/folder is quite easy.  All I gotta do is just copy any one of these, and rename it with the proper server name.

```bat
dev-server1234abc.bat
dev-server4123abd.bat
int-server3258afz.bat
int-server3258jgz.bat
prd-server99935d3.bat
prd-server0923ssd.bat
```

## Really?
Was this really necessary?  Maybe not, but I learned something new along the way.  What I should have probably done was figure out what's up with the Citrix Remote Session...
