# argocd-progressive-sync-example

First, create a kind cluster

```yaml
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# wait until all pods are running
kubectl get pods -n argocd                                                                               

NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          10s
argocd-applicationset-controller-66b94549f6-46nn2   1/1     Running   0          10s
argocd-dex-server-6cd4c7498f-vgwh9                  1/1     Running   0          10s
argocd-notifications-controller-65cddcf9d6-7z8c6    1/1     Running   0          10s
argocd-redis-74d77964b-ddkjh                        1/1     Running   0          10s
argocd-repo-server-96b577c5-dwdhk                   1/1     Running   0          10s
argocd-server-7c7b5568cc-wlf2b                      1/1     Running   0          10s
```

to enable Progressive Sync, add following config to `argocd-cmd-params-cm` ConfigMap

```yaml
  applicationsetcontroller.enable.progressive.syncs: "true"
```

then restart ArgoCD ApplicationSet Controller pods

```bash
kubectl rollout restart deploy argocd-applicationset-controller
```

apply the `appset.yaml`

```bash
kubectl apply -f appset.yaml -n argocd
```

check the apps on ui
```
# get password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

# user: admin

kubectl port-forward svc/argocd-server 8080:80
```
