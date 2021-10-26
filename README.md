# Raspberry Pi Kubernetes Cluster

## Installation

### Hardware

### CRI-O

```
export OS=xUbuntu_20.04
export VERSION=1.22
```

#### Add Repositories
```
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" | sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list

curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | sudo apt-key add -
curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | sudo apt-key add -
```

#### Install

```
sudo apt-get update
sudo apt-get install cri-o cri-o-runc

sudo apt-mark hold cri-o cri-o-runc
```

### Kubernetes (kubeadm, kubelet, kubectl)

#### Add Repository

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

#### Install

```
sudo apt-get install kubeadm kubelet kubectl
sudo apt-mark hold kubeadm kubelet kubectl
```

### Networking

#### Ensure you load modules

```
sudo modprobe overlay
sudo modprobe br_netfilter
```

#### Set up required sysctl params

```
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

#### Reload sysctl

```
sudo sysctl --system
```

### Useful Commands
sudo kubeadm init phase upload-certs --upload-certs

sudo kubeadm token create --print-join-command --certificate-key

### Cilium

helm repo add cilium https://helm.cilium.io/

helm repo update

helm install cilium cilium/cilium --version 1.10.4 \
--namespace cilium-system \
--set ipam.mode=cluster-pool \
--set ipam.operator.clusterPoolIPv4PodCIDR=100.64.0.0/16 \
--set ipam.operator.clusterPoolIPv4MaskSize=24 \
--set nodeinit.enabled=true \
--set tunnel=vxlan \
--set kubeProxyReplacement=strict \
--set k8sServiceHost=k8s.bruckers.org \
--set k8sServicePort=6443
