## Argo CD ApplicationSet example (per-customer)

This repo shows one ApplicationSet per customer that scans JSON files and deploys the Bitnami NGINX chart with a minimal set of values.

Prerequisite
- A brand-new Minikube cluster (empty):
```bash
minikube start
```

### Install Argo CD and ApplicationSet via Helm
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

kubectl create ns argocd || true

# Argo CD
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd \
  --set controller.replicas=1 \
  --set server.service.type=ClusterIP

# ApplicationSet controller
helm upgrade --install argocd-applicationset argo/argocd-applicationset \
  --namespace argocd
```

### Apply the ApplicationSet(s)
- Apply customer I00001 only:
```bash
kubectl apply -n argocd -f "ArgoCD_Appset_example/deployment/I00001/applicationset.yaml"
```
- Or apply everything under `deployment/` (I00001 and I00017):
```bash
kubectl apply -n argocd -f "ArgoCD_Appset_example/deployment/"
```

What it does
- Each `applicationset.yaml` uses the git-files generator against this GitHub repo
  (`https://github.com/RedwanAlkurdi/argocd-applicationset-example.git`) and
  scans `deployment/Ixxxxx/**/config.json` per customer.
- Each discovered JSON turns into an Argo CD Application that deploys Bitnami NGINX with values from the JSON.

### Repository layout
- `deployment/I0000x/Cluster-y/config.json`: per-cluster JSON
- `deployment/I0000x/applicationset.yaml`: ApplicationSet for that customer

### JSON example
```json
{
  "name": "demo-I00001-c1",
  "namespace": "default",
  "image_repository": "nginx",
  "image_tag": "1.27",
  "replicaCount": 1,
  "service_type": "ClusterIP",
  "service_port": 80
}
```

### Notes
- The chart used is Bitnami NGINX from `https://charts.bitnami.com/bitnami`.
- You can modify JSON files to change only the listed values; more can be added if needed.

### Architecture (Mermaid)
```mermaid
flowchart TD
  subgraph Repo[Git Repository]
    A1[deployment/I00001/applicationset.yaml]
    A2[deployment/I00001/Cluster-1/config.json]
    A3[deployment/I00001/Cluster-2/config.json]
    A4[deployment/I00001/Cluster-3/config.json]
  end

  subgraph ArgoCD[Argo CD in cluster - namespace argocd]
    AS[ApplicationSet Controller]
    AP1[Rendered Application I00001 c1]
    AP2[Rendered Application I00001 c2]
    AP3[Rendered Application I00001 c3]
  end

  subgraph Cluster[Kubernetes Cluster]
    NS1[(Namespace from JSON default)]
    NS2[(Namespace from JSON ...)]
    NS3[(Namespace from JSON ...)]

    D1[NGINX Deployment Bitnami chart]
    S1[Service]
    D2[NGINX Deployment Bitnami chart]
    S2[Service]
    D3[NGINX Deployment Bitnami chart]
    S3[Service]
  end

  A1 -- git files generator scans --> A2
  A1 -- git files generator scans --> A3
  A1 -- git files generator scans --> A4

  A1 -->|spec.generators.git.files| AS
  A2 -->|valuesObject from JSON| AS
  A3 -->|valuesObject from JSON| AS
  A4 -->|valuesObject from JSON| AS

  AS -->|template creates| AP1
  AS -->|template creates| AP2
  AS -->|template creates| AP3

  AP1 -->|source Bitnami nginx chart| D1
  AP1 --> S1
  AP2 -->|source Bitnami nginx chart| D2
  AP2 --> S2
  AP3 -->|source Bitnami nginx chart| D3
  AP3 --> S3

  D1 --- NS1
  S1 --- NS1
  D2 --- NS2
  S2 --- NS2
  D3 --- NS3
  S3 --- NS3
```
