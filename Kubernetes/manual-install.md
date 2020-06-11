# Install Kubernetes Manual

## Create user kube and Install some package
```
adduser kube; \
echo -e "P@ssw0rd" | passwd kube --stdin; \
echo "kube  ALL=(ALL)  NOPASSWD: ALL" > /etc/sudoers.d/kube; \
su - kube; \
sudo dnf install -y bash-completion tmux epel-release; \
sudo dnf install -y htop iftop

```

## Configure before begin install
- Set `/etc/host`
- check `br_netfilter` with `lsmod | grep br_netfilter`
- if not enabled yet `sudo modprob br_netfilter`
- then add some variable to `sysctl`
```
sudo modprob br_netfilter; \
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
- allow port before install
**Control-plane node(s)**
Protocol	Direction	Port Range	Purpose	Used By
TCP	Inbound	6443*	Kubernetes API server	All
TCP	Inbound	2379-2380	etcd server client API	kube-apiserver, etcd
TCP	Inbound	10250	Kubelet API	Self, Control plane
TCP	Inbound	10251	kube-scheduler	Self
TCP	Inbound	10252	kube-controller-manager	Self
```
sudo firewall-cmd --add-port=6443/tcp --permanent --zone=public; \
sudo firewall-cmd --add-port=2379-2380/tcp --permanent --zone=public; \
sudo firewall-cmd --add-port=10250/tcp --permanent --zone=public; \
sudo firewall-cmd --add-port=10251/tcp --permanent --zone=public; \
sudo firewall-cmd --add-port=10252/tcp --permanent --zone=public; \
sudo firewall-cmd --reload
```
**Worker node(s)**
Protocol	Direction	Port Range	Purpose	Used By
TCP	Inbound	10250	Kubelet API	Self, Control plane
TCP	Inbound	30000-32767	NodePort Services†	All
```
sudo firewall-cmd --add-port=10250/tcp --permanent --zone=public; \
sudo firewall-cmd --add-port=30000-32767/tcp --permanent --zone=public; \
sudo firewall-cmd --reload
```

- install docker or cri-o
```
sudo yum install -y yum-utils; \
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo; \
sudo yum-config-manager --enable docker-ce-nightly; \
sudo yum install docker-ce docker-ce-cli containerd.io -y --nobest; \
sudo systemctl enable docker.service --now
```
- disable SELinux
```
sudo setenforce 0; \
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

```
- install kubeadmin, kubelet, kubectl
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes; \
sudo systemctl enable --now kubelet

```
- start docker and kubelet
- run kubeadm init `kubeadm init --control-plane-endpoint=<loadbalancer> --pod-network-cidr=10.244.0.0/16` **Using flannel CNI**
- after finish save the result
- apply CNI `kubectl apply -f <from cni plugin>`  for example `flannel kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`
- using flannel here https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md
- 