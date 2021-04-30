[TOC]

# Set Up SonarQube



## Set Up SonarQube via OpenShift Template



### Create Project

```bash
oc new-project "$GUID-sonarqube" --display-name="$GUID Shared Sonarqube"
```



### Import SonarQube Template



**Summary:**

- OpenShift Template:
  - https://github.com/OpenShiftDemos/sonarqube-openshift-docker/blob/master/sonarqube-postgresql-template.yaml
- SonarQube: 
  - Docker Image:  `quay.io/gpte-devops-automation/sonarqube:${SONARQUBE_VERSION}`
- PostgreSQL:
  - ImageStreamTag:  `postgresql:${DATABASE_VERSION}`

```bash
# includes objects:
# Service: sonarqube, postgresql-sonarqube
# Route: sonarqube
# ImageStream: sonarqube (quay.io/gpte-devops-automation/sonarqube:${SONARQUBE_VERSION})
# DeploymentConfig: sonarqube, postgresql-sonarqube
# PVC: sonarqube-data, postgresql-sonarqube-data

oc apply -f sonarqube/template/sonarqube-persistent-template.yaml
```



### Deploy SonarQube via OpenShift Template

On OpenShift web console, switch to "Developer" view, and click Add, and choose "From Catalog", and search "sonarqube".

You can Instantiate Template on OpenShift web console, or run by below commands.

```bash
# deploy sonarqube
# DATABASE_VERSION: oc get imagestreamtag -n openshift | grep -i postgre
# SONARQUBE_VERSION: https://quay.io/repository/gpte-devops-automation/sonarqube
oc new-app sonarqube-persistent \
  -p DATABASE_VERSION='9.6-el8' \
  -p SONARQUBE_VERSION='7.9.1'


# verify deployment config
oc get dc

# watch pod status
watch oc get pods

```



### Verify Installation

```bash
# get route
oc get route

# open sonarqube route url in browser
# default username/password: admin / amdin
# open account seetings / security, and reset password

# security settings
# https://docs.sonarqube.org/latest/instance-administration/security/
# Administration > Configuration > General Settings > Security, and then turn on "Force user authentication"
# Administration > Projects > Management, and then change "Default visibility of new projects" to "private"
# Administration > Security > Global Permissions, and then revoke all permissions of "Anyone"
# Administration > Security > Permission Templates, and then revoke all permissions of "Creators"

```





## Set Up SonarQube via OpenShift Template (v2)



TODO



