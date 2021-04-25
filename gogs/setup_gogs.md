[TOC]

# Set Up Gogs on OpenShift



## Set Up Gogs via OpenShift Template



### Create Project

```bash
oc new-project "$GUID-gogs" --display-name="$GUID Shared Gogs"
```



### Import Gogs OpenShift Template

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

