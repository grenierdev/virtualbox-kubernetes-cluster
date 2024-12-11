# Kubernetes Cluster on a stick

## Setup

Create the VMs

```sh
vagrant up
sh ./export-ssh-config
ansible-playbook ./microk8s.yaml -i ./hosts.yaml
```