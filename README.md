# gitops-argocd-demo

A small GitOps example: a web application described with Kustomize (one base, two overlays) and deployed to Kubernetes by ArgoCD. Git is the single source of truth, and a pipeline validates every manifest before it can be merged.

![Validate manifests](https://github.com/PedroCastro0507/gitops-argocd-demo/actions/workflows/validate-manifests.yml/badge.svg)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?logo=kubernetes&logoColor=white)
![ArgoCD](https://img.shields.io/badge/ArgoCD-EF7B4D?logo=argo&logoColor=white)
![Kustomize](https://img.shields.io/badge/Kustomize-326CE5?logo=kubernetes&logoColor=white)

## Layout

```text
apps/web/base/            Deployment and Service shared by every environment
apps/web/overlays/dev/    namespace web-dev, 2 replicas
apps/web/overlays/prod/   namespace web-prod, 3 replicas
argocd/applications/      ArgoCD Application resources for dev and prod
.github/workflows/        CI that renders and validates the manifests
```

## How it works

1. The base defines a hardened nginx Deployment (non-root, dropped capabilities, resource requests and limits, readiness and liveness probes) and a Service.
2. Each overlay sets its own namespace and labels. The prod overlay raises the replica count to three.
3. Each ArgoCD Application points at one overlay, with automated sync, pruning and self-heal enabled, and creates its namespace if needed.
4. Changing a manifest in Git is the only way to change the cluster: ArgoCD detects the new commit and reconciles.

## Try it on a local cluster

```bash
# 1. Start a local cluster, for example with minikube
minikube start

# 2. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Register the applications
kubectl apply -f argocd/applications/

# 4. Watch ArgoCD sync them
kubectl get applications -n argocd
```

## Continuous integration

The workflow [validate-manifests.yml](.github/workflows/validate-manifests.yml) renders each overlay with `kubectl kustomize` and checks the output against the Kubernetes schemas with kubeconform. It also validates the ArgoCD Application files, skipping schemas that are not published for custom resources.

## Scope and limits

The CI proves that the manifests render and are schema-valid. A deployment on a live cluster is not part of this repository, so the steps in the section above are instructions rather than recorded results.

## Roadmap

- Add an ApplicationSet to generate one Application per environment.
- Add a Helm chart variant for comparison.
- Add progressive delivery with Argo Rollouts.
