[TOC]

# Set Up GitLab CE







## Deploy GitLab CE via Yaml



GitLab CE:

- GitLab CE
- PostgreSQL
- Redis



### Export resources

```bash
# get resource names
oc get all -o name
oc get pvc -o name
oc get sa -o name

# project specific resources
oc get deploymentconfig.apps.openshift.io/gitlab-ce -o yaml > deploymentconfig-gitlab-ce.yaml
oc get deploymentconfig.apps.openshift.io/gitlab-ce-postgresql -o yaml > deploymentconfig-gitlab-ce-postgresql.yaml
oc get deploymentconfig.apps.openshift.io/gitlab-ce-redis -o yaml > deploymentconfig-gitlab-ce-redis.yaml

oc get service/gitlab-ce -o yaml > service-gitlab-ce.yaml
oc get service/gitlab-ce-postgresql -o yaml > service-gitlab-ce-postgresql.yaml
oc get service/gitlab-ce-redis -o yaml > service-gitlab-ce-redis.yaml

oc get imagestream.image.openshift.io/gitlab-ce -o yaml > imagestream-gitlab-ce.yaml
oc get imagestream.image.openshift.io/gitlab-ce-redis -o yaml > imagestream-gitlab-ce-redis.yaml
oc get imagestream.image.openshift.io/postgresql -o yaml > imagestream-postgresql.yaml

oc get route.route.openshift.io/gitlab-ce -o yaml > route-gitlab-ce.yaml > route-gitlab-ce.yaml

# pvc
oc get persistentvolumeclaim/gitlab-ce-data -o yaml > pvc-gitlab-ce-data.yaml
oc get persistentvolumeclaim/gitlab-ce-etc -o yaml > pvc-gitlab-ce-etc.yaml
oc get persistentvolumeclaim/gitlab-ce-postgresql -o yaml > pvc-gitlab-ce-postgresql.yaml
oc get persistentvolumeclaim/gitlab-ce-redis-data -o yaml > pvc-gitlab-ce-redis-data.yaml

# serviceaccount
oc get serviceaccount/gitlab-ce-user -o yaml  > serviceaccount-gitlab-ce-user.yaml

```



### Install GitLab CE via Yaml

不建议用Yaml方式来部署GitLab。

GitLab官网建议用Helm方式来部署。



## Deploy GitLab CE via Helm



### Helm command completion

```bash
helm completion zsh > /path/to/_helm

vim ~/.zshrc
source /path/to/_helm

source ~/.zshrc
```



### Install GitLab CE Charts

References:

- https://docs.gitlab.com/charts/
- https://docs.gitlab.com/charts/quickstart/index.html (non-production deployment)
- https://docs.gitlab.com/charts/installation/ (production deployment)



```bash
# search charts
helm search repo gitlab

# download charts
# See README.md for detailed instruction
helm pull stable/gitlab-ce --untar=true

# install charts
helm repo add gitlab http://charts.gitlab.io/

# only for example
helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600s \
  --set global.hosts.domain=example.com \
  --set global.hosts.externalIP=10.10.10.10 \
  --set certmanager-issuer.email=me@example.com
```



### Run as any user

Some of the containers that we will use need to run as root (GitLab) or any other user (Postgres has hardcoded user 26 but this will be fixed soon).
Hence, it is required to used the project’s service account (here we created a project called gitlab) to the SCC named anyuid.



```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:openshift:scc:anyuid
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:openshift:scc:anyuid
subjects:
- kind: ServiceAccount
  name: gitlab-ce-user
```



Anyuid SCC:

- anyuid provides all features of the restricted SCC but allows users to run with any UID and any GID.
- In platforms such as kubernetes and OpenShift this will be the equivalent as allowing UID 0, or root user, both inside and outside the container. That will be discussed in further blog posts. SELinux plays an important role here adding a layer of protection and it's a good idea to use seccomp to filter non desired system calls as well.



Grant `anyuid` permissions to specific service account:

```bash
oc adm policy add-scc-to-user anyuid -z <service-account> -n <namespace>
```



Why need `anyuid` permissions:

- Run as port <= 1024
- Run as root user (uid=0)
- ...



References:

- https://www.openshift.com/blog/deploy-gitlab-openshift
- https://www.openshift.com/blog/managing-sccs-in-openshift
- https://www.openshift.com/blog/a-guide-to-openshift-and-uids
- https://docs.openshift.com/container-platform/4.6/authentication/managing-security-context-constraints.html



## References

- https://github.com/redhat-cop/containers-quickstarts/tree/master/gitlab-ce