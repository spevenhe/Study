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
