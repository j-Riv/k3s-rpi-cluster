# Deploy K3s
> Install [K3s](https://www.rancher.com/products/k3s) on a Raspberry Pi 4 Cluster running [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/raspberry-pi). The cluster will be able to be imported into [Rancher](https://www.rancher.com/why-rancher). Putting this here in hopes it helps someone or myself later on :D.

What is K3s?

K3s is a lightweight Kubernetes distribution created by Rancher Labs, and it is fully certified by the Cloud Native Computing Foundation (CNCF). K3s is highly available and production-ready. It has a very small binary size and very low resource requirements.

## Raspberry Pi Setup

Install Ubuntu Server 22.04 LTS on each Raspberry Pi.

Then do the following for each pi.

Edit the host name ex: `k3s-master` for the master and `k3s-worker-01`, `k3s-worker-02` etc for the workers.
```bash
sudo nano /etc/hostname
```

Configure boot options.

Add `cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1` to the end of the first line.

```bash
sudo nano /boot/firmware/cmdline.txt
```

Install updates and reboot

```bash
sudo apt update && sudo apt dist-upgrade
sudo reboot
```

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

```bash
# create namespace for cert
kubectl create namespace NAME_OF_NAMESPACE
# create cloudflare secret
# use global api key or create a token and grant all zones
kubectl apply -f secret-cloudflare.yml
# create cluster issuer
# choose server: staging or production, replace email set reference to api key or token from above
kubectl apply -f clusterisser-acme.yml
# create certificate
# make name and secretName the same so its not confusing later, add namespace
kubectl apply -f certificate.yml
```

[Troubleshooting Certificates](https://cert-manager.io/docs/troubleshooting/acme/)

```bash
# get certificates
kubectl get certificates;
# view certificates
kubectl describe certificate name-of-certificate
# get certificate requests
kubectl get certificaterequest
# view certificate request
kubectl describe certificaterequest name-of-certificate-request
# get orders
kubectl get orders
# view order
kubectl describe order name-of-order
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