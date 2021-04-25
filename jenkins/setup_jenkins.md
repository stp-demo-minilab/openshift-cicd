[TOC]

# Set Up Jenkins



### Set Up Jenkins via OpenShift Template



```bash
# create project
oc new-project "$GUID-jenkins" --display-name="$GUID Shared Jenkins"

# deploy Jenkins with built-in jenkins persistent template
oc new-app jenkins-persistent \
  --param=ENABLE_OAUTH=true \
  --param=VOLUME_CAPACITY=4Gi \
  --param=MEMORY_LIMIT=2Gi \
  --param=DISABLE_ADMINISTRATIVE_MONITORS=true \
  --as-deployment-config=true
  
# set resources quota
oc set resources dc/jenkins --requests=cpu=500m,memory=1Gi --limits=cpu=2,memory=2Gi

# verify dc
oc get dc

# watch pod status
watch oc get pods

# verify route
oc get route

# open jenkins route url in browser
# click "Login with OpenShift", and then choose "Allow selected permissions"

```

