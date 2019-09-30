---
layout: default
title: Grafana AD Integration
---

## Grafana with AD Integration Gotcha
I ran into a rather odd issue with Grafana and AD integration.  It took about a day to pinpoint the issue, and hopefully this post will save someone some valuable hours in the future.  The solution is simple, and maybe it's something obvious that everyone already knows to do.  The error in the logs wasn't very clear, but luckily the solution is very quick to implement.


The symptoms was I could log into Grafana with an AD backed user account, but any subsequent users login attempts would not authenticate.  Initial assumption was that the Service
Account that it was binding with didn't have access to the particular subtree in AD - that was quickly proven wrong by as our AD accounts were getting locked after X unsuccessful login attempts.  Time to turn on Grafana debugging to gather more clues.

```ini
# Either "debug", "info", "warn", "error", "critical", default is "info"
level = debug

# optional settings to set different levels for specific loggers. Ex filters = sqlstore:debug
filters = ldap:debug
```

This is one of the debug messages in the logs, which is semi-helpful, but doesn't clearly indicate what is going on.

```
t=2019-09-17T17:56:52+0000 lvl=dbug msg="Ldap Search For User Request" logger=ldap info="(ldap.SearchRequest) {\n BaseDN: (string) (len=22) \"DC=corp,DC=steve,DC=com\",\n Scope: (int) 2,\n DerefAliases: (int) 0,\n SizeLimit: (int) 0,\n TimeLimit: (int) 0,\n TypesOnly: (bool) false,\n Filter: (string) (len=24) \"(sAMAccountName=stransolution)\",\n Attributes: ([]string) {\n },\n Controls: ([]ldap.Control) <nil>\n}\n"
t=2019-09-17T17:56:52+0000 lvl=dbug msg="Ldap User found" logger=ldap info="(*ldap.UserInfo)(0xc000442b60)({\n DN: (string) (len=98) \"CN=Steve Tran,OU=Users,OU=Networking,OU=Information Systems & Technology,DC=corp,DC=steve,DC=com\",\n FirstName: (string) \"\",\n LastName: (string) \"\",\n Username: (string) \"\",\n Email: (string) \"\",\n MemberOf: ([]string) {\n }\n})\n"
t=2019-09-17T17:56:52+0000 lvl=eror msg="Error while trying to authenticate user" logger=context userId=0 orgId=0 uname= error="UNIQUE constraint failed: user.email"
```

After logging into Grafana with a local Administrator account, I was able to see the accounts being created after each successful **first** login.  It was creating an entry with a blank username and blank email.  Seeing the empty entry on the GUI, along with the `error="UNIQUE constraint failed: user.email"` message in the log was enough for me to correlate the two together.  

What was happening was a new entry was being created in Grafana based on the information pulled from AD, via LDAP.  Due to a configuration oversight, there were several **required** attributes that needed to be configured.  So back in the `grafana.ini` file, we need to specify which element in the AD schema holds the email field.  For consistency's sake, I mapped the other fields of the user's Grafana profile too, so that it pulls back the correct information.   

So long story short, this is what I needed in my configuration file.
```ini
# Specify names of the ldap attributes your ldap uses
[servers.attributes]
name = "givenName"
surname = "sn"
username = "sAMAccountName"
member_of = "memberOf"
email = "mail"
```

I don't know why the default grafana configurations don't assume some sort of sensible default.  Time to look into the `grafana/grafana` Docker image and perhaps create a feature request.
