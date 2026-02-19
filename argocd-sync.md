# Sync Waves

Now you'll deploy the App of Apps application to Argo. This is a deployment method used to deploy entire sets of applications.

Notice in each application, there is an annotation for *argocd.argoproj.io/sync-wave: "-3"* as noted below. Sync-waves allow for the ordering applications and components inside your manifest.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-ingress
  namespace: argocd
  annotations:
      argocd.argoproj.io/sync-wave: "-3"
```

Sync-waves are integers in quotes and processed in ascending order. 

Now that your apps are deployed, you can now manage you applications via Git. You can navigate to the *argocd-repo* and change images, scale etc. Argo will sync these changes over, but can take up to 5 minutes. You can accelerate this by going to ArgoCD UI, click sync and force the git sync.

Once you completed, delete all applications before starting the next section on Application Sets.

# Application Sets

Please deploy the *nginx-ingress* application prior to starting this section. 

CNCF [video](https://www.youtube.com/watch?v=9RvDczVIiS0)

Application Sets allow you to build templates to deploy multiple applications. 

The example built below is using the *Git* generator as outlined below.

[Generators](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators/)

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: appset-demo
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  syncPolicy:
    applicationsSync: create-update
  generators:
  - git:
      repoURL: git@github.com:SalesAmerSP/argocd-repo.git
      revision: HEAD
      directories:
      - path: manifests/*
  template:
    metadata:
      name: '{{.path.basename}}'
    spec:
      project: default
      source:
        repoURL: git@github.com:SalesAmerSP/argocd-repo.git
        targetRevision: HEAD
        path: '{{.path.path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{.path.basename}}'
      syncPolicy:
        automated:
          enabled: true
        syncOptions:
        - CreateNamespace=true
```