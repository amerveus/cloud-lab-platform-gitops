# Bootstrapping a cluster

Argo CD is installed once by hand; after that, adding an application means adding a file.

## Dev

```bash
helm repo add argo https://argoproj.github.io/argo-helm && helm repo update argo
helm install argocd argo/argo-cd --kube-context cloud-lab-dev -n argocd --create-namespace \
  --version 10.9.6 --set dex.enabled=false --set notifications.enabled=false --wait --timeout 10m
kubectl --context cloud-lab-dev apply -f argocd/root.yaml
kubectl --context cloud-lab-dev -n argocd get applications
```

`root` watches `argocd/apps` (non-recursive) and creates the six dev Applications. Expected: seven applications,
all `Synced` and `Healthy`.

## Prod

Prod is a separate cluster with its own Argo CD, so the two never see each other's applications:

```bash
helm install argocd argo/argo-cd --kube-context cloud-lab-prod -n argocd --create-namespace \
  --version 10.9.6 --set dex.enabled=false --set notifications.enabled=false --wait --timeout 10m
kubectl --context cloud-lab-prod apply -f argocd/prod/root.yaml
```

## Before you bootstrap after a rebuild

1. Set the secret values in Secrets Manager (chaos token, Grafana admin) so External Secrets can sync.
2. Refresh the three hard-coded values listed in the README.
3. Expect `lab-api-dev` to need one manual Sync if it races the Prometheus CRDs.

The full procedure, with the infrastructure steps, is in the infra repo:
`docs/runbooks/rebuild-dev.md`.

## Admin access to the UI

The Argo CD server is a ClusterIP service. Use a port-forward and never expose it:

```bash
kubectl --context cloud-lab-dev -n argocd port-forward svc/argocd-server 8443:443
```

Delete the initial admin secret after first login (`argocd-initial-admin-secret`).
