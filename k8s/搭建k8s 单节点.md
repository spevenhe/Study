## Prerequisites
```
service firewalld stop
chkconfig firewalld off
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
```

```
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

## install docker
https://docs.docker.com/engine/install/centos.

## install kubeadm
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm

```
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```
## proxy
```
mkdir -pv /etc/systemd/system/containerd.service.d/
 
cat << EOF > /etc/systemd/system/containerd.service.d/http-proxy.conf
[Service]
Environment="HTTP_PROXY=http://child-prc.intel.com:913"
Environment="HTTPS_PROXY=http://child-prc.intel.com:913"
Environment="NO_PROXY=127.0.0.1,10.67.127.0/23,10.244.0.0/16,10.96.0.0/16,10.67.112.0/23,10.67.126.0/23,192.168.8.0/24,${HOST_IP}"
EOF


export http_proxy=http://child-prc.intel.com:913
export https_proxy=http://child-prc.intel.com:913
```

## init cluster
```
kubeadm init --kubernetes-version=v1.28.5 \
--pod-network-cidr=10.244.0.0/16 \
--apiserver-advertise-address={host_ip} \
--cri-socket=unix:///var/run/containerd/containerd.sock \
--token-ttl 0 \
--ignore-preflight-errors=SystemVerification | tee kubeadm-init.log
```

## untaint master node
```
kubectl taint node master node-role.kubernetes.io/control-plane:NoSchedule-
```
