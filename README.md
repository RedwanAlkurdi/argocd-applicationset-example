## Argo CD ApplicationSet per-customer example

- One ApplicationSet per customer (`I00001`..`I00017`).
- Each ApplicationSet lives alongside its customer data in `data/Ixxxxx/applicationset.yaml` and scans that customer’s `data/Ixxxxx/**/config.json` files to map keys to Helm values.
- Deploys a minimal Helm chart into the in-cluster cluster (`https://kubernetes.default.svc`).

### Structure

- `chart/simple-demo`: minimal Helm chart with Deployment and Service
- `data/I0000x/Cluster-y/config.json`: per-cluster JSON with flattened keys
- `data/I0000x/applicationset.yaml`: the per-customer ApplicationSet

### Configure

- Replace `REPO_URL_HERE` in each `data/Ixxxxx/applicationset.yaml` with your Git repo URL.
- If your default branch is not `main`, update `revision`/`targetRevision`.

### Apply

- Apply one customer:
```bash
kubectl apply -n argocd -f "ArgoCD Appset example/data/I00001/applicationset.yaml"
```
- Or apply all customers:
```bash
kubectl apply -n argocd -f "ArgoCD Appset example/data/"
```

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
