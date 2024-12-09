# Kubernetes Cluster on a stick

## Setup VMs

Create the VMs

```sh
vagrant up
```

Export the ssh config

```sh
vagrant ssh-config master01.k8s.local >> ./inventory/ssh_config
vagrant ssh-config worker01.k8s.local >> ./inventory/ssh_config
vagrant ssh-config worker02.k8s.local >> ./inventory/ssh_config
vagrant ssh-config worker03.k8s.local >> ./inventory/ssh_config
vagrant ssh-config nfs01.k8s.local >> ./inventory/ssh_config
```

## Configure

```sh
ansible-playbook ./playbooks/setup.yaml -i ./inventory/hosts.yaml
```