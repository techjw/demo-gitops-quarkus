# demo-gitops-quarkus
Kubernetes manifests for deploying the Quarkus insult generator application via Argo CD

### Quarkus web application
See code at: https://github.com/sa-ne/quarkify_webapp

### MySQL database (persistent storage)
* Quarkus web application will initiate stub data into the tables on startup
* Persistent storage configured via PVC
    * Change storage class as-needed

### Argo CD Application
* Define quarkus/mysql application for deployment/management via Argo CD

> **_NOTE:_** To use the manifests here with ArgoCD run the command below to set permission ArgoCD's required permissions:

```
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```
