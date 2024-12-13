# Kubernetes Cluster on a stick

## Setup

Create the VMs

```sh
vagrant up
sh ./export_ssh_config
ansible-playbook ./microk8s_install.yaml -i ./hosts.yaml
ansible-playbook ./microk8s_join.yaml -i ./hosts.yaml

ansible -i ./hosts.yaml masters --become -a 'microk8s kubectl get node'
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