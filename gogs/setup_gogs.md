[TOC]

# Set Up Gogs on OpenShift



## Set Up Gogs via OpenShift Template



### Create Project

```bash
oc new-project "$GUID-gogs" --display-name="$GUID Shared Gogs"
```



### Import Gogs OpenShift Template

**Summary:**

- Gogs OpenShift Template:
  - https://raw.githubusercontent.com/OpenShiftDemos/gogs-openshift-docker/master/openshift/gogs-persistent-template.yaml

- PostgreSQL:

  - ImageStreamTag: `postgresql:${DATABASE_VERSION}`

    ```bash
    oc get imagestreamtag -n openshift | grep -i postgre
    ```

- Gogs: 

  - Docker Image: `docker.io/openshiftdemos/gogs:${GOGS_VERSION}`

    

```bash
# includes objects:
# ServiceAccount: gogs
# Service: gogs-postgresql, gogs
# DeploymentConfig: gogs-postgresql, gogs
# Route: gogs
# ImageStream: gogs (docker.io/openshiftdemos/gogs:${GOGS_VERSION})
# PVC: gogs-data, gogs-postgres-data
# ConfigMap: gogs-config

oc apply -f gogs/template/gogs-persistent-template.yaml
```



### Deploy Gogs via OpenShift Template

On OpenShift web console, switch to "Developer" view, and click Add, and choose "From Catalog", and search "gogs".

You can Instantiate Template on OpenShift web console, or run by below commands.

```bash
# deploy gogs
# HOSTNAME
# <service>-<namespace>.apps.*
# route example: `oc whoami --show-console`
# DATABASE_VERSION: oc get imagestreamtag -n openshift | grep -i postgre
# GOGS_VERSION: https://hub.docker.com/r/openshiftdemos/gogs/tags
oc new-app gogs-persistent \
  -p DATABASE_VERSION='9.6-el8' \
  -p GOGS_VERSION='0.11.34' \
  -p HOSTNAME='<HOSTNAME>'


# verify deployment config
oc get dc
oc rollout status dc gogs

# watch pod status
watch oc get pods

```



### Verify Installation

```bash
# verify route
oc get route

# open gogs in browser
# register an account
# login gogs
# create a repository and set it as private

```



### References

- https://github.com/OpenShiftDemos/gogs-openshift-docker
- https://github.com/wkulhanek/docker-openshift-gogs (not work in openshift 4.6)
- [Gogs Downloads](https://dl.gogs.io/)
- https://github.com/gogs/gogs



## ~~Set Up Gogs via OpenShift Template (v2 )~~



**Summary:**

- Gogs OpenShift Template:
  - https://raw.githubusercontent.com/redhat-cop/containers-quickstarts/master/gogs/.openshift/templates/gogs-persistent-template.yaml

- PostgreSQL:

  - ImageStreamTag: `postgresql:9.5`

    ```bash
    oc get imagestreamtag -n openshift | grep -i postgre
    ```

- Gogs: 
  - Docker Image: `docker.io/gogs/gogs:${GOGS_VERSION}`



```bash
oc new-project "$GUID-gogs2" --display-name="$GUID Shared Gogs2"

wget -O gogs/template/gogs.yaml https://raw.githubusercontent.com/redhat-cop/containers-quickstarts/master/gogs/.openshift/templates/gogs-persistent-template.yaml

oc process -f gogs/template/gogs.yaml -p NAMESPACE="$(oc project)" | oc apply -f -
  
```



错误：

- Error from server (Forbidden): securitycontextconstraints.security.openshift.io "gogs-anyuid" is forbidden: User "lsirui-redhat.com" cannot get resource "securitycontextconstraints" in API group "security.openshift.io" at the cluster scope



 Gogs container will run under `anyuid` SCC. Ensure you are logged into the Cluster (step 4) with an user with privileges to modify existing SecurityContextConstraints



So can not run by regular user ?



