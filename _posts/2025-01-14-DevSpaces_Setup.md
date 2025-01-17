---
title: OpenShift Dev Spaces
date: 2025-01-14
tags: ocp devspaces
---

## Configuring OAuth with OpenShift Dev Spaces and GitLab

This is more of a lesson learned in the configuring of OAuth in OpenShift Dev Spaces for GitLab.  The instructions for setting up an OAuth2 token is straight forward.  It requires creating an Application OAuth Token in GitLab by clicking on your user avatar, adding a new application with the following details.

- Name : {Something Recognizable as Dev Spaces}
- Redirect URL: https://{devspaces_at_domain}/api/oauth/callback
- Confidental: box is checked
- Scopes: api, write_repository, openid

After filling out those fields, an ID and Secret will be presented.  Save those values because the Secret will not be viewable after closing the popup.

The trick here is to plug those values into a Kubernetes Secret but if you use the OpenShift web console, it automatically base64 encodes the values.  What Devspaces needs are raw values instead.  Create a Secret like the example below and do an `oc apply -f <file>` to get it to apply correctly.

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: gitlab-oauth-config
  namespace: devspaces
  labels:
    app.kubernetes.io/component: oauth-scm-configuration
    app.kubernetes.io/part-of: che.eclipse.org
  annotations:
    che.eclipse.org/oauth-scm-server: gitlab
    che.eclipse.org/scm-server-endpoint: "https://your.gitlab.hostname"
stringData:
  id: <Oauth ID here>
  secret: <OAuth Secret here>

```

## Configuring an encrypted Maven settings.xml file

Setting up a maven settings.xml file is pretty simple with a few special Dev Spaces annotations on a Secret or ConfigMap.  We'll use a Secret in this example because sometimes a Maven settings.xml file contains unencrypted passwords (to servers, repos, etc.).  It's easy to encrypt these passwords though, so if you aren't using encrypte passwords, you should convert to them now.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mvn-settings-secret
  labels:
    controller.devfile.io/mount-to-devworkspace: 'true'
    controller.devfile.io/watch-secret: 'true'
  annotations:
    controller.devfile.io/mount-path: '/home/user/.m2'
    controller.devfile.io/mount-as: subpath
data:
  settings.xml: <your_file>
  settings-security.xml: <your_file>
```

Note: the `controller.devfile.io/mount-as: subpath` annotation is really important or else the ./m2 folder loses its permissions and will not be writable when Maven tries to create a repository subfolder to save artifacts.