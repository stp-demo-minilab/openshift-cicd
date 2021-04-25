[TOC]

# Set Up Nexus



## How to

You can set up Nexus on OpenShift by:

- Command & Yaml
- OpenShift Template
- Operator
- Helm
- Ansbile



## Set Up Nexus by OpenShift Template



### Create Project

```bash
# create project
oc new-project "$GUID-nexus" --display-name="$GUID Shared Nexus"
```



### Import Nexus OpenShift Tempalte

```bash


# https://github.com/monodot/openshift-nexus
# import template
# includes objects:
# - ImageStream
# - DeploymentConfig
# - Service
# - Route
# - PersistenVolumeClaim
# parameters:
# - SERVICE_NAME (default value: nexus)
# - NEXUS_VERSION (default value: 3.30.0)
# - VOLUME_CAPACITY (10Gi)

oc apply -f nexus/template/nexus3-persistent-template.yaml
```



### Deploy Nexus via OpenShift Template

On OpenShift web console, switch to "Developer" view, and click Add, and choose "From Catalog", and search "nexus".



You can Instantiate Template on OpenShift web console, or run by below commands.

Deploy Nexus 3:

```bash
# Deploy Nexus 3
# https://hub.docker.com/r/sonatype/nexus3/
# Deploy Nexus 3 Specific Version
# Nexus may failed start if no engouh volume capacity
oc new-app nexus3-persistent \
  -p SERVICE_NAME=nexus \
  -p NEXUS_VERSION=3.30.0 \
  -p VOLUME_CAPACITY=10Gi
  

# watch pod status
watch oc get pods

```



### Verify Installation

Verify installation:

```bash
# Verify PVC on OpenShift web console: Admin / Storage / Persistent Volume Claims
oc get pvc

# verify route
oc get routes

# login nexus
# get nexus password
# username: admin
# reset password after login
# choose "Disable anonymous access" for security reason
oc rsh $(oc get pods | grep nexus | awk '/Running/ {print $1}') cat /nexus-data/admin.password


```



### Create a Docker registry in Nexus



```bash
# Login Nexus with admin
# Create docker hosted repository in Nexus
# Name: Docker
# HTTP: 5000

oc patch dc nexus -p '{"spec":{"template":{"spec":{"containers":[{"name":"nexus","ports":[{"containerPort": 5000,"protocol":"TCP","name":"docker"}]}]}}}}'


# watch pod status when re-deployment
watch oc get pods
```



After patch, a new port "5000" named "docker" will be added into the deployment config.



Expose service:

```bash
# expose as service
oc expose dc nexus --name=nexus-registry --port=5000

# verify services
oc get svc

```



Create a secure (TLS) route to the registry:

```bash
# temination: edge
oc create route edge nexus-registry --service=nexus-registry
```



References:

- https://docs.openshift.com/container-platform/4.6/networking/routes/secured-routes.html#nw-ingress-creating-an-edge-route-with-a-custom-certificate_secured-routes



TODO: Use Nexus Docker registry outside or inside OpenShift Cluster.





### References

References:

- https://github.com/monodot/openshift-nexus

- https://tomd.xyz/openshift-nexus-docker-registry/ 

- https://github.com/OpenShiftDemos/nexus

- https://github.com/eformat/openshift-nexus

- https://support.sonatype.com/hc/en-us/articles/360045220393-Scripting-Nexus-Repository-Manager-3

- https://blog.sonatype.com/nexus-as-a-container-registry

  

  

