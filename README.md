# Kubernetes Cluster on a stick

## Setup VMs

Create the VMs

```sh
vagrant up
```

Export the ssh config

```sh
vagrant ssh-config master01.grenier.local >> ./ssh_config
vagrant ssh-config worker01.grenier.local >> ./ssh_config
vagrant ssh-config worker02.grenier.local >> ./ssh_config
vagrant ssh-config worker03.grenier.local >> ./ssh_config
vagrant ssh-config nfs01.grenier.local >> ./ssh_config
```

## Configure

```sh
ansible-playbook ./microk8s.yaml -i ./hosts.yaml
```