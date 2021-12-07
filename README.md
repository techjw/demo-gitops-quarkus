# demo-gitops-quarkus
Kubernetes manifests for deploying the Quarkus insult generator application via Argo CD

### Quarkus web application
See code at: https://github.com/sa-ne/InsultGenerator

### MySQL database (persistent storage)
* Quarkus web application will initiate stub data into the tables on startup
* Persistent storage configured via PVC
    * Change storage class as-needed

### Kubernetes manifests

You will find the Kubernetes manifest files that deploy either MySQL and the Quarkus web app at `k8s/manifests` folder.

This repository has following structure:

```
.
├── k8s                                 ---> All K8s manifests and ArgoCD app yamls.
│   ├── arogcd-app-prod.yaml            ---> ArgoCD App to deploy the app in the Production namespace
│   ├── arogcd-app-qa.yaml              ---> ArgoCD App to deploy the app in the QA namespace
│   ├── environments                    ---> kustomization files that deploys the entire solution (MySQL + App) 
│   │   ├── prod                        ---> To deploy MySQL + App with production settings run it using Kustomize, e.g.: oc apply -k ./k8s/environments/prod/
│   │   │   ├── kustomization.yaml
│   │   │   └── namespace.yaml
│   │   └── qa                          ---> Same for QA, e.g.: oc apply -k ./k8s/environments/qa/
│   │       ├── kustomization.yaml
│   │       └── namespace.yaml
│   └── manifests                       ---> K8s deployment manifest files using Kustomize
│       ├── insultgenerator             ---> K8s deployment manifest files for insultgenerator quarkus web app
│       │   ├── base                    ---> contains base files to deploy it using standard parameter values
│       │   │   ├── deployment.yaml
│       │   │   ├── kustomization.yaml
│       │   │   ├── route.yaml
│       │   │   └── service.yaml
│       │   └── overlays                ---> Contains yaml files for what differs in each environment only (patches)
│       │       ├── prod                ---> If you need to deploy only the web app run from this folder, e.g.: oc apply -k ./k8s/manifests/insultgenerator/overlays/prod/
│       │       │   ├── deployment-patch.yaml
│       │       │   └── kustomization.yaml
│       │       └── qa                  ---> If you need to deploy only the web app run from this folder, e.g.: oc apply -k ./k8s/manifests/insultgenerator/overlays/qa/
│       │           ├── deployment-patch.yaml
│       │           └── kustomization.yaml
│       └── mysql                       ---> The same as before, but for MySQL
│           ├── base
│           │   ├── configmap.yaml
│           │   ├── deployment.yaml
│           │   ├── kustomization.yaml
│           │   ├── pvc.yaml
│           │   ├── secret.yaml
│           │   └── service.yaml
│           └── overlays
│               ├── prod                ---> If you need to deploy only MySQL run from this folder, e.g.: oc apply -k ./k8s/manifests/mysql/overlays/prod/
│               │   ├── kustomization.yaml
│               │   └── secret.yaml
│               └── qa                  ---> If you need to deploy only MySQL run from this folder, e.g.: oc apply -k ./k8s/manifests/mysql/overlays/prod/
│                   ├── kustomization.yaml
│                   └── secret.yaml
```

### Argo CD Application

In this repository you will find two ArgoCD applications, one to deploy the Quarkus insult generator application in a QA namespace and another for Production namespace.

# Deployment procedure

This application is deployed using GitOps (ArgoCD) practices. Follow the instructions below to deploy it.

You may deploy it directly or using ArgoCD, see below the steps for each case.

## Deploying without ArgoCD

To deploy directly on OpenShift (without using ArgoCD) run the following commands, that uses `Kustomize` to generate all manifests:

**For Production** 

```
oc apply -k ./k8s/environments/prod/
```

**For QA**

```
oc apply -k ./k8s/environments/qa/
```

---
**NOTE**

This procedure has been tested using the following `oc` version: `v4.9.8`. It many not work smoothly in older versions due to the `Kustomize` version of `oc`. If you get errors I recommend you to update your `oc` cli version or, alternatively, run it using kustomize directly, example:

```
kustomize build ./k8s/environments/prod/ | oc apply -f -
```
---

## Deploying with ArgoCD

**NOTE**

If you have not already, create your own fork of this repository, then use a separate branch for your demo.
1. Create a new branch for your demo, for example: `git checkout -b demo`
1. Update both the deployment files `k8s/argocd-app-prod.yaml` and `k8s/argocd-app-qa.yaml` to use your new repo/branch
```
...
spec:
  project: default
  source:
    repoURL: 'https://github.com/YOUR_GH_USER/demo-gitops-quarkus.git'
    path: k8s/environments/prod
    targetRevision: YOUR_GH_BRANCH_NAME
...
```

### Pre-requisites

1. You should have ArgoCD running on namespace `openshift-gitops`
2. Set the proper permission to ArgoCD's ServiceAccount to allow it to create namespaces and others tasks:
```
oc adm policy add-cluster-role-to-user cluster-admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller
```

### Deployment

Run the following command (using your branched file version) to create the ArgoCD Application, which will deploy the application:

**For Production** 

```
oc apply -f ./k8s/argocd-app-prod.yaml
```

**For QA** 

```
oc apply -f ./k8s/argocd-app-qa.yaml
```
