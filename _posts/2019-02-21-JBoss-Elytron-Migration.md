---
title: JBoss EAP7 Elytron Migration Gotchas
date: 2019-02-21
tags: jboss eap elytron migration
---

I recently upgraded the JBoss EAP7's security subsystem to the newer Elytron security subsystem, and it was not easy.  The Red Hat documentation was good, but it didn't explain why we were doing certain things, and it also assumed that I had a full understanding of all the components in play.  While it's probably my own fault for not understanding the prerequisite knowledge of how EAP works, all I cared about was moving from the legacy model that worked exactly how I needed it to, to the newer model which I ran into all these gotchas.

## Gotchas
- login-module Role Mapper not available
- SASL authentication-factory is REQUIRED for jboss-cli (docs sounded like it was optional, or maybe I didn't understand how SASL was being used)
- credential_store not replicated
- multiple places for credential store definition (host.xml and domain.xml)?
- needing to use the legacy SSL while in Domain Mode because server 2 and server 3 didn't work.  Server 1 worked just fine though...


## TODO
There's a ton of stuff that wasn't captured, so if you're going through the same exercise and need help, send me an email!