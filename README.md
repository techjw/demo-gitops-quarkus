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

# Deployment procedure

This application is deployed using GitOps (ArgoCD) practices. Follow the instructions below to deploy it.

## Pre-requisites

1. You should have ArgoCD running on namespace `openshift-gitops`
2. Set the permission below ArgoCD's ServiceAccount:
```
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```

## Deployment

Run the following command to create the ArgoCD Application, which will deploy the application:

```
oc apply -f k8s/arogcd-app.yaml
```
