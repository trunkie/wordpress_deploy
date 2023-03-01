# Install Kubernetes Cluster using kubeadm by Ansible server
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__.

This documentation guides you in setting up a cluster with one master node and one worker node.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Ansible|ansible-srv|192.168.2.16|CentOS 7|2G|2|
|Master1|k8s-master1|192.168.2.88|CentOS 7|2G|2|
|Worker1|k8s-worker1|192.168.2.215|CentOS 7|2G|2|
|Worker2|k8s-worker2|192.168.2.93|CentOS 7|2G|2|

## Ansible Controller Configuration
1. Check /etc/hosts and hostname



## Install on both k8s-master* and k8s-worker*
Perform all the commands as root user unless otherwise specified
##### Disable Firewall
```sh
systemctl disable firewalld; systemctl stop firewalld
```
##### Disable swap
```sh
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Disable SELinux
```sh
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Update sysctl settings for Kubernetes networking
```sh
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
##### Install docker engine
```sh
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-19.03.12 
systemctl enable --now docker
```
### Kubernetes Setup
##### Add yum repository
```sh
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
```sh
yum install -y kubeadm-1.18.5-0 kubelet-1.18.5-0 kubectl-1.18.5-0
```
##### Enable and Start kubelet service
```sh
systemctl enable --now kubelet
```
## Only install on k8s-master*
##### Initialize Kubernetes Cluster

```sh
kubeadm init --pod-network-cidr=172.16.0.0/12
```
After kubeadm init have created a serial key like this
```sh
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.2.42:6443 --token muym0f.gwow6jwzvg36n9io \
    --discovery-token-ca-cert-hash sha256:6ba1a2fe2ebdf54f79e930b5c516c0ab869a828f822198e040ef5531fceeb5e9
[root@k8s-master1 ~]#
```
Copy this key and paste on k8s-worker*
##### Deploy Calico network
```sh
kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
```
##### Cluster join command
```sh
kubeadm token create --print-join-command
```
##### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```
If you have an error
```sh 
[root@k8s-master ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[root@k8s-master ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
[root@k8s-master ~]# kubectl get nodes
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
run this command in k8s-master
```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Install only k8s-worker*
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```sh
kubectl get nodes
```
and result
```sh
[root@k8s-master1 ~]# kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
k8s-master1   Ready    master   3m22s   v1.18.5
k8s-worker1   Ready    <none>   2m25s   v1.18.5
k8s-worker2   Ready    <none>   2m19s   v1.18.5

```
##### Get component status
```
kubectl get cs
```



Have Fun!!