# k8s-experiments - Experimental Kubernetes deployments

# How to use

Each directory in `experiments` represents an experiment. Experiments are structured as follows:

```
experiments
└── <experiment-name>
    └── clusters
        ├── <cluster-name>
        │   └── apps
        │       ├── <app-manifest>
        │       └── <app-manifest>
        └── <cluster-name>
            └── apps
                ├── <app-manifest>
                └── <app-manifest>
```

`<experiment-name>` represents a named experiment, such as `rook-external`. `<cluster-name>` represents a cluster logical name (not necessarily a Kubernetes context). `<app-manifest>` represents a single ArgoCD Application resource.

To deploy an experiment to a cluster, use an ApplicationSet:

```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-apps
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - git:
      repoURL: https://github.com/jnschaeffer/k8s-experiments.git
      revision: HEAD
      directories:
      - path: experiments/hello-world/clusters/cluster1/apps/*
  template:
    metadata:
      name: '{{.path.basename}}'
    spec:
      source:
        repoURL: https://github.com/jnschaeffer/k8s-experiments.git
        targetRevision: HEAD
        path: '{{.path.path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{.path.basename}}'
      syncPolicy:
        syncOptions:
        - CreateNamespace=true
```
