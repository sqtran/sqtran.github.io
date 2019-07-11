---
layout: post
title: Active Choices Plugin + Image Registry
---

## Jenkins Active Choices Plugin + Image Registry

I recently needed to retrieve a list of `images` from a container registry, so that it could be used as a parameter in a Jenkins Pipeline.  The list of images needed to be dynamic, based on other input fields the user can use.  The registry can be anything, but the examples in this post is for the Openshift internal registry and JFrog Artifactory.

Jenkins has a plugin built specifically for this.  It is reactive based on other parameters, which is the `Active Choices Plugin`.  The only hiccup was the integration with the `REST` API, and some gymnastics required to reuse Jenkins' Credentials.


### Scoping

The scope in which the reactive parameter script is being run in is not the same as if the pipeline were actually executing (doing a build).  This means DSL methods such as `withCredentials` are not in-scope, so they won't work.  This causes a slight issue because in order to access the container registries via their `REST` API, authentication tokens are required.  All my credentials are already stored as `Jenkins Credentials` on the Jenkins server, so I didn't want to create another place to store this token.  It also creates more headache later when the token needs to be changed.


A forum showed me how to programatically instantiate a `Jenkins Credential`.  Once instantiated, it behaves in similar fashion with the `withCredentials` DSL method.

```Groovy
import jenkins.model.*
def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(com.cloudbees.plugins.credentials.Credentials.class, Jenkins.instance, null, null)
def token = creds.find {it.id == 'my-secret-in-jenkins'}
println "${token.getSecret().getPlainText()}"
```


### REST API
The second half of the solution is actually pulling the data from the respective registries.  Good ole `curl` to the rescue to interact with the `REST` API.


#### Openshift
```bash
curl -k https://<OCP_REGISTRY_ROUTE>/v2/<OCP_NAMESPACE>/<PROJECT_NAME>/tags/list -u any:<token>
```
The **any** username is **arbitrary**. I believe any String can be used.  The only part that matters is the OCP auth token you plug in as the password.


Note the variables you use need to be global Jenkins variables, or references to other parameters.  Although you can't tell in my example, OCP_REGISTRY_ROUTE is a global variable, and the other two are parameters on the same page that must be selected before getting to this parameter.

The Active Choices parameter requires a list to be returned, which is as easy as transforming the resulting `JSON` into something easy to manipulate.  

```Groovy
import groovy.json.JsonSlurper

// if you need to filter your results, you can use any of the Groovy collections closures
// return new JsonSlurper().parseText(curl.execute().text).tags.findAll { it.contains("some filter value") }.sort()   
return new JsonSlurper().parseText(curl.execute().text).tags.sort()
```

#### Artifactory
To access the Artifactory's registry, you just need to post the Artifactory API token as a header.

```bash
curl -H 'X-JFrog-Art-Api:<my-frog-art-token>' <artifactory_url>/artifactory/api/docker/docker-repo/v2/mysubfolder/<image_name>/tags/list
````

My curl command parameterized the artifactory_url and image_name.  The path "mysubfolder" is the name of the registry inside of Artifactory, which will vary between use-cases.

### Example
The entire `script` looks like this, although some of the variable substitution may not work in your case.  The `ENV` variable referenced comes from a parameter on the pipeline.

```Groovy
import groovy.json.JsonSlurper
import jenkins.model.*

try {
  def creds = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(com.cloudbees.plugins.credentials.Credentials.class, Jenkins.instance, null, null);
  def token
  def curl

  if (ENVS.equals("env1")) {
    token = creds.find {it.id == 'my-secret-in-jenkins'}
    curl = [ 'bash', '-c', "curl -k https://$OCP_REGISTRY_ROUTE/v2/$OCP_NAMESPACE/$PROJECT_NAME/tags/list -u any:\${token.getSecret().getPlainText()}"]
  }
  else if (ENVS.equals("env2")) {
    token = creds.find {it.id == 'my-secret-jfrog-api-in-jenkins'}
    curl = [ 'bash', '-c', "curl -H 'X-JFrog-Art-Api:${token.getSecret().getPlainText()}' '$ARTIFACTORY_URL/artifactory/api/docker/docker-repo/v2/mysubfolder/$PROJECT_NAME/tags/list' "]
  }

  return new JsonSlurper().parseText(curl.execute().text).tags.sort()
```

References
1. https://wiki.jenkins.io/display/JENKINS/Active+Choices+Plugin
