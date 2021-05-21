# Install Kubernetes Cluster using kubeadm on a simple Master and Worker node
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__.

This documentation guides you in setting up a cluster with one master node and one worker node.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|kmaster.example.com|172.16.16.100|CentOS 7|2G|2|
|Worker|kworker.example.com|172.16.16.101|CentOS 7|1G|1|

## On both Kmaster and Kworker
Perform all the commands as root user unless otherwise specified
##### Disable Firewall
```
systemctl disable firewalld; systemctl stop firewalld
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Update sysctl settings for Kubernetes networking
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker
systemctl enable --now docker
```
### Kubernetes Setup
##### Add yum repository
```
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
##### Install Kubernetes components
```
yum install -y kubeadm kubelet kubectl
```
##### Enable and Start kubelet service
```
systemctl enable --now kubelet
```
## On kmaster
##### Initialize Kubernetes Cluster
```
kubeadm init --apiserver-advertise-address=172.16.16.100 --pod-network-cidr=192.168.0.0/16
```
##### Deploy Calico network
```
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```
##### Cluster join command
```
kubeadm token create --print-join-command
```
##### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs
```
# Install Calicoctl on Cluster on a Single host
1. Log into the host, open a terminal prompt, and navigate to the location where you want to install the binary.
Tip: Consider navigating to a location that’s in your PATH. For example, ``` /usr/local/bin/. ```

2. Use the following command to download the calicoctl binary.
```
curl -o calicoctl -O -L  "https://github.com/projectcalico/calicoctl/releases/download/v3.19.0/calicoctl" 
```
3. Set the file to be executable.
```
chmod +x calicoctl
```
check if calicoctl is installed correctly
```
calicoctl version
```
Note: If the location of calicoctl is not already in your PATH, move the file to one that is or add its location to your PATH. This will allow you to invoke it without having to prepend its location.


# To install BGP route-to-router reflector
By default calico network work in a node-to-node mesh as follows
![image](https://user-images.githubusercontent.com/14257200/119192272-d85aa780-ba4d-11eb-87b6-54393d6fe58f.png)

When requirement grows (100's of nodes), node-to-node mesh network may not be sustainable or scalable. To avoid this, route-reflectors are dedicated nodes to make the routes more efficient and scalable option.
![image](https://user-images.githubusercontent.com/14257200/119192247-ced13f80-ba4d-11eb-95ca-90056d681835.png)

Create the following yaml files 

## rr-peers.yaml
```
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: rr-mesh
spec:
  nodeSelector: has(route-reflector)
  peerSelector: has(route-reflector)
```
```
calicoctl apply -f rr-peers.yaml
```
## bgp-peer.yaml

```
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: peer-to-rrs
spec:
  nodeSelector: !has(route-reflector)
  peerSelector: has(route-reflector)
```
```
calicoctl apply -f bgp-peer.yaml
```

Try to get 

calicoctl get bgpconfig default  {if you get the below error, create one}
resource does not exist: BGPConfiguration(default) with error: bgpconfigurations.crd.projectcalico.org "default" not found

## bgp-config.yaml

```
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  logSeverityScreen: Info
  nodeToNodeMeshEnabled: false
  asNumber: 63400
```
```
calicoctl apply -f bgp-config.yaml
```
