# Kubernetes Cluster on a stick

## Setup

Create the VMs

```sh
vagrant up
sh ./export_ssh_config
ansible-playbook ./microk8s_install.yaml -i ./hosts.yaml
ansible-playbook ./microk8s_join.yaml -i ./hosts.yaml

ansible -i ./hosts.yaml 'masters[0]' --become -a 'microk8s config' | tail -n +2 > ~/.kube/config
# edit ~/.kube/config to point to reachable IP
```

## Cloudflare Ingress

```sh
helm repo add strrl.dev https://helm.strrl.dev
helm repo update
helm upgrade --install --wait \
  -n cloudflare-tunnel-ingress-controller --create-namespace \
  cloudflare-tunnel-ingress-controller \
  strrl.dev/cloudflare-tunnel-ingress-controller \
  --set=cloudflare.apiToken="<cloudflare-api-token>",cloudflare.accountId="<cloudflare-account-id>",cloudflare.tunnelName="<your-favorite-tunnel-name>"
```

## Dashboard

```
microk8s enable dashboard
# Extract the token
microk8s kubectl describe secret -n kube-system microk8s-dashboard-token
# Expose dashboard to localhost
kubectl port-forward -n kube-system service/kubernetes-dashboard 10443:443
# Expose dashboard to the world (DONT)
kubectl -n kube-system \
  create ingress kubernetes-dashboard-via-cf-tunnel \
  --rule="<you're favorite domain>/*=kubernetes-dashboard:443" \
  --annotation cloudflare-tunnel-ingress-controller.strrl.dev/backend-protocol=https \
  --class cloudflare-tunnel
```

## Whoami
```
kubectl apply -f whoami.yaml

kubectl create ingress whoami-via-cf-tunnel --rule="<you're favorite domain>/*=whoami:80" --class cloudflare-tunnel
```


# Kubernetes on Windows Server 2025 Standard

https://microk8s.io/docs/add-a-windows-worker-node-to-microk8s

```
notepad 'C:\Program Files\containerd\config.toml'
# sandbox_image = "rancher/pause:3.6-windows-ltsc2022-amd64"

```