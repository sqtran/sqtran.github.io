---
layout: default
title: Artifactory REST Examples
---

## Artifactory REST Examples

Artifactory has a pretty robust REST interface, but their documentation does not have the best examples.  I find myself constantly digging through my notes and old work in order to remember the syntax.

Most of API calls I commonly use are in the context of CICD, and usually originate from Jenkins.  These are the most forgotten image manipulation commands I use.  


###  Deleting Images

The target URI is in the from /artifactory_url/{repository}/{image_name}[/{tag}]


Delete an individual Tag
```bash
curl -k -u username:password -X DELETE <artifactory_url>/artifactory/docker-release-local/cicd/spring-boot-example/1.0
```

Delete an entire image
```bash
curl -k -u username:password -X DELETE <artifactory_url>/artifactory/docker-release-local/cicd/spring-boot-example
```

### Copying images (Promoting)

``` bash
curl -k -X POST '<artifactory_url>/artifactory/api/docker/docker-release-local/v2/promote' \
 -H 'cache-control: no-cache' \
 -H 'content-type: application/json' \
 -H 'x-jfrog-art-api: $APIKEY' \
 -d '{ "targetRepo" : "docker-release-local",
       "dockerRepository" : "cicd/$img",
       "tag": "$tag",
       "targetTag": "$artifactVersion",
       "copy" : true}'
```


### Listing images

 ```bash
 curl -H 'X-JFrog-Art-Api:$APIKEY' '<artifactory_url>/artifactory/api/docker/docker-repo/v2/mysubfolder/$PROJECT_NAME/tags/list' "]
 ```
