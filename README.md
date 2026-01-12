# Pleiades cluster configuration

## Bootstrap

The Hetzner cloud controller needs to be added first otherwise the node will be tainted:

```bash
kubectl -n kube-system create secret generic hcloud --from-literal=token="${PLEIADES_HCLOUD_TOKEN:?}"
helm repo add hcloud https://charts.hetzner.cloud
helm repo update hcloud
helm install hccm hcloud/hcloud-cloud-controller-manager -n kube-system
```

```bash
kubectl apply -k ops/kubelet-serving-cert-approver
kubectl apply -k ops/argocd
```

```bash
initial_admin_password="$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)"
argocd login --port-forward --port-forward-namespace argocd --username=admin --password="$initial_admin_password"
argocd admin dashboard -n argocd
```

```bash
argocd app create apps \
    --port-forward --port-forward-namespace argocd \
    --dest-namespace argocd \
    --dest-server https://kubernetes.default.svc \
    --repo https://github.com/stargrid-systems/pleiades-gitops.git \
    --path apps
argocd app sync apps --port-forward --port-forward-namespace argocd
```

## License

This repository is licensed under GNU Affero General Public License v3.0 or later (AGPL-3.0-or-later).
See the [LICENSE](LICENSE) file for details.
