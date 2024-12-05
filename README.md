- Ubuntu Server 24 Minimized
- User `ubuntu:ubuntu`

## References
- https://medium.com/@mojabi.rafi/create-a-kubernetes-cluster-using-virtualbox-and-without-vagrant-90a14d791617
- https://kevinhoffman.medium.com/building-a-kubernetes-cluster-in-virtualbox-with-ubuntu-22cd338846dd
- https://www.profiq.com/kubernetes-cluster-setup-using-virtual-machines/
- https://errindam.medium.com/taming-kubernetes-for-fun-and-profit-60a1d7b353de

## See also
- https://rook.io/docs/rook/latest-release/Getting-Started/Prerequisites/prerequisites/#ceph-prerequisites

## Base image
```
# Install required packages
sudo apt-get update
sudo apt-get install -y vim
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl docker.io

# Configure system
sudo systemctl disable firewalld && systemctl stop firewalld	
sudo swapoff -a
sudo sed -i '/swap/ s/^\(.*\)$/#\1/g' /etc/fstab

# Hold the current version of kubelet, kubeadm, kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Configure cgroup
cat <<EOF | sudo tee /etc/docker/daemon.json
{ "exec-opts": ["native.cgroupdriver=systemd"], "log-driver": "json-file", "log-opts": { "max-size": "100m" }, "storage-driver": "overlay2" }
EOF

# Configure kubernetes
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
cat <<EOF | sudo tee /etc/default/kubelet
KUBELET_EXTRA_ARGS="--cgroup-driver=cgroupfs"
EOF
cat <<EOF | sudo tee -a /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"
EOF
sudo systemctl stop apparmor && sudo systemctl disable apparmor
sudo systemctl daemon-reload && sudo systemctl restart kubelet

# Enable and start kubernetes and docker
sudo systemctl enable --now kubelet
sudo systemctl enable --now docker
sudo systemctl restart kubelet
sudo systemctl restart docker
```

## Master
```
# Configure hostname
sudo hostnamectl set-hostname kubemaster01
sudo vi /etc/hosts

# Change static IP
sudo vi /etc/netplan/50-cloud-init.yaml

# Initialize cluster
sudo kubeadm init --apiserver-advertise-address=192.168.56.11 --control-plane-endpoint=192.168.56.11 --pod-network-cidr=10.244.0.0/16

# Configure local kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Add network
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Patch inter-node interface
kubectl patch daemonsets kube-flannel-ds --patch-file flannel-patch.yml --namespace kube-flannel

# ???
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

# Loadbalancer
kubectl get configmap kube-proxy -n kube-system -o yaml | sed -e "s/strictARP: false/strictARP: true/" | kubectl apply -f - -n kube-system
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
kubectl apply -f metallb-config.yaml
```

```
You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 192.168.56.11:6443 --token tyvx3b.2ty0qmtzjehbq7bv \
	--discovery-token-ca-cert-hash sha256:353e2063c5a4702ab3cb39990cf3dd61862e41349a882ce6a4c8e3ffb1fda0fd \
	--control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.11:6443 --token tyvx3b.2ty0qmtzjehbq7bv \
	--discovery-token-ca-cert-hash sha256:353e2063c5a4702ab3cb39990cf3dd61862e41349a882ce6a4c8e3ffb1fda0fd
```

## Worker
```
# Configure hostname
sudo hostnamectl set-hostname kubeworker01
sudo vi /etc/hosts

# Change static IP
sudo vi /etc/netplan/50-cloud-init.yaml

sudo kubeadm join 192.168.56.11:6443 --token tyvx3b.2ty0qmtzjehbq7bv \
	--discovery-token-ca-cert-hash sha256:353e2063c5a4702ab3cb39990cf3dd61862e41349a882ce6a4c8e3ffb1fda0fd
```

## Routing to loadbalancer

Compare the routing of the kubemaster01 to 192.168.56.0/24 to host and change accordingly
```
# sudo ip route replace 192.168.56.0/24 via 192.168.56.11 dev vboxnet0
```