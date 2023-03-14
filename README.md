# Deploy K3s
> Install k3s on a Raspberry Pi 4 Cluster

## Install K3s

### Master
Setup master as root
```bash
export K3S_KUBECONFIG_MODE="644"
curl -sfL https://get.k3s.io | sh -
```

Get Token
```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```
### Workers
Setup workers as root
```bash
export K3S_URL="https://MASTER_LOCAL_IP:6443"
export K3S_KUBECONFIG_MODE="644"
export K3S_TOKEN="TOKEN_FROM_ABOVE_COMMAND"
curl -sfL https://get.k3s.io | sh -
```

## Cert Manager
[Installation Docs](https://cert-manager.io/docs/installation/helm/)

```bash
# add helm repository
helm repo add jetstack https://charts.jetstack.io
# update local helm chart
helm repo update
# install cert manager with CustomResourceDefinitions
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.11.0 \
  --set installCRDs=true
```

## Certificates
> Lets Encrypt

[Troubleshooting](https://cert-manager.io/docs/troubleshooting/acme/)
```bash
# create namespace for cert
kubectl create namespace NAME_OF_NAMESPACE
# create cloudflare secret
kubectl apply -f secret-cloudflare.yml
# create cluster issueer
kubectl apply -f clusterisser-acme.yml
# create certificate
# make name and secretName ex domain-dev
kubectl apply -f certificate.yml
```

## Middleware
```bash
# http to https redirect
kubectl apply -f middleware-https-redirect.yml
```

## Ingress
Annotations
```
key: traefik.ingress.kubernetes.io/router.middlewares
value: default-redirect@kubernetescrd
```

## Traefik
Expose Dashboard
```bash
kubectl port-forward -n kube-system $(kubectl -n kube-system get pods --selector "app.kubernetes.io/name=traefik" --output=name) 9000:9000
```