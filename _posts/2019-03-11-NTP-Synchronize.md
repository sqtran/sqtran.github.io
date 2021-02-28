---
title: NTP Synchronize Command
date: 2019-03-11
tags: ntp
---

Figured I'd create a quick paragraph blog entry that describes how to synchronize your system clock with The Atomic Clock.

My RHEL 7.5 instance didn't typically auto-update for whatever reason, so I had to manually.  I have noticed that this is the same problem when I move between timezones too...

## tl;dr
```bash
sudo ntpdate time.nist.gov
```