---
layout: default
title: Openshift Image Mirror to Artifactory
---

---
layout: single
title: OpenShift Image Mirror to Artifactory
date: 2019-06-19
#categories: artifactory oc-mirror ocp
---

## OpenShift Image Mirror to Artifactory

I introduced the `oc image mirror` command that came out with the 3.11 release of the OC Client, in my previous post.  This article goes over the steps needed to get it integrated with Artifactory.


The `oc image mirror` command uses the Docker-formatted `~/.docker/config.json` file to store authentication information, even though it technically doesn't require Docker installed.  Unless you're already familiar with how that file is laid out, you can always `docker login` to each of the servers so the file is automatically generated/updated for you.  You can also specify the location of your configuration file via `--registry-config=/path/to/custom-config.json`.

It's not difficult to hand-craft your own though, and heads up, there's an error in the snippet below that will be explained later on.
```json
{
	"auths": {
		"docker-registry-default.steve.com": {
			"auth": "bm90bXk6cmVhbHBhc3N3b3JkCg=="
		},
		"docker-repo.build-repo.steve.com": {
			"auth": "bm90bXk6cmVhbHBhc3N3b3JkCg==",
			"email": "email@steve.com"
		},
		"https://index.docker.io/v1/": {
			"auth": "bm90bXk6cmVhbHBhc3N3b3JkCg=="
		}
	}
}
```


If you're trying to `docker login` to a registry that uses self-signed certificates, and you don't have the root/intermediary CAs installed, you'll run into x509 certificate issues.  This is very common and it's described in great details all over the internet.  I update the `insecure-registries` stanza in the `/etc/docker/daemon.json` file.


If you're wondering how to log into the OpenShift Registry, just grab the external facing route and do the docker login.  **Keep in mind that if you're not using a service account, this password will expire and you'll need a way to refresh your saved configs.**

```bash
$ oc login
$ docker login -u ocpsteve -p $(oc whoami -t) docker-registry default.steve.com
```


I was running into authorization issues whenever I ran the command, so I built my own version of the OC Client in order to add debugging messages to figure out what was happening behind the scenes.  The `ocsteve` and `oc` commands are equivalent, minus my debugging output.

```bash
$ ocsteve image mirror docker-registry-default.steve.com/steve-test3/mock-service:1.0 docker-repo.build-repo.steve.com/steve/mock-service:1.0 --insecure

client.go Creating Registry @ https://docker-registry-default.steve.com and repoName of steve-test3/mock-service
Pinging url https://docker-registry-default.steve.com
BasicFromKeyring being called for https://docker-registry-default.steve.com/openshift/token
Username ocpsteve password g9DHBAL8KrFCm-o6P9nwlnHSGCNBZEoY_XiNXO-HaC0 url attempted = https://docker-registry-default.steve.com/openshift/token

client.go Creating Registry @ https://docker-repo.build-repo.steve.com and repoName of steve/mock-service
Pinging url https://docker-repo.build-repo.steve.com
BasicFromKeyring being called for https://build-repo.steve.com:443/artifactory/api/docker/docker-repo/v2/token

docker-repo.build-repo.steve.com/
  steve/mock-service
    manifests:
      sha256:68f818970bdbf241b562f92417cad0b958e49c2ed6708a553b764b28d39309c5 -> 1.0
  stats: shared=0 unique=0 size=0B

phase 0:
  docker-repo.build-repo.steve.com steve/mock-service blobs=0 mounts=0 manifests=1 shared=0

info: Planning completed in 320ms
error: unable to push manifest to docker-repo.build-repo.steve.com/steve/mock-service:1.0: unauthorized: The client does not have permission to push to the repository.
info: Mirroring completed in 0s (0B/s)
error: one or more errors occurred while uploading images
```

Checking the access.log in Artifactory showed a ton of messages saying anonymous logged in.

```
2019-06-17 22:01:39,942 [ACCEPTED LOGIN]   for client : anonymous / 10.250.36.161.
2019-06-17 22:01:39,974 [ACCEPTED LOGIN]   for client : anonymous / 10.250.36.161.
2019-06-17 22:01:39,989 [ACCEPTED LOGIN]   for client : anonymous / 10.250.36.161.
2019-06-17 22:01:40,020 [ACCEPTED LOGIN]   for client : anonymous / 10.250.36.161.
2019-06-17 22:01:40,130 [ACCEPTED LOGIN]   for client : anonymous / 10.250.36.161.
```

My `ocsteve` printed out that the OC client was trying to find credentials for  `build-repo.steve.com:443/artifactory/api/docker/docker-repo/v2/token`, or more specifically, for the `build-repo.steve.com` domain.  I thought I could just `docker login` into that URL, but that gave me other issues.


```bash
$ docker login build-repo.steve.com
Username (steve):
Password:
Error response from daemon: Login: <!doctype html><html lang="en"><head><title>HTTP Status 404 – Not Found</title><style type="text/css">h1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;} h2 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:16px;} h3 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:14px;} body {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;} b {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;} p {font-family:Tahoma,Arial,sans-serif;background:white;color:black;font-size:12px;} a {color:black;} a.name {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class="line" /><p><b>Type</b> Status Report</p><p><b>Message</b> &#47;v1&#47;users&#47;</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class="line" /><h3>Apache Tomcat/8.5.32</h3></body></html> (Code: 404; Headers: map[Date:[Wed, 19 Jun 2019 21:41:30 GMT] Server:[Apache/2.4.29 (Win64) OpenSSL/1.1.0g] Content-Type:[text/html;charset=utf-8] Content-Language:[en] Content-Length:[1091]])
```

So it is still a mystery to me as to why that happens, but to get around that, I just manually modified the config.json file.

The working configuration file looks like this.

```json
{
    "auths": {
        "docker-registry-default.steve.com": {
            "auth": "bm90bXk6cmVhbHBhc3N3b3JkCg=="
        },
        "docker-repo.build-repo.steve.com": {
            "auth": "bm90bXk6cmVhbHBhc3N3b3JkCg==",
            "email": "email@steve.com"
        },
        "build-repo.steve.com": {
            "auth": "bm90bXk6cmVhbHBhc3N3b3JkCg==",
            "email": "email@steve.com"
        },
        "https://index.docker.io/v1/": {
            "auth": "bm90bXk6cmVhbHBhc3N3b3JkCg=="
        }
    }
}
```

Now I can use the unmodified OC client to do the mirror successfully.
```bash
$ oc image mirror docker-registry-default.steve.com/steve-test3/mock-service:1.0 docker-repo.build-repo.steve.com/steve/mock-service:1.0 --insecure
docker-repo.build-repo.steve.com/
  steve/mock-service
    manifests:
      sha256:68f818970bdbf241b562f92417cad0b958e49c2ed6708a553b764b28d39309c5 -> 1.0
  stats: shared=0 unique=0 size=0B

phase 0:
  docker-repo.build-repo.steve.com steve/mock-service blobs=0 mounts=0 manifests=1 shared=0

info: Planning completed in 260ms
sha256:68f818970bdbf241b562f92417cad0b958e49c2ed6708a553b764b28d39309c5 docker-repo.build-repo.steve.com/steve/mock-service:1.0
info: Mirroring completed in 40ms (0B/s)
```
