# Lecture 9 (CentOS 7)

https://edu.slurm.io/courses/slurm-base-school/unit/136

https://github.com/southbridgeio/slurm-school-k8s/tree/master/6.Kubespray

https://github.com/southbridgeio/kubespray

## VMs
3 master nodes
- k8s-master-1.local
  192.168.1.151
- k8s-master-2.local
  192.168.1.152
- k8s-master-3.local
  192.168.1.153

1 ingress node
- k8s-ingress-1.local
  192.168.1.155

2 worker nodes
- k8s-node-1.local
  192.168.1.161
- k8s-node-2.local
  192.168.1.162

### network settings
https://lintut.com/how-to-setup-network-after-rhelcentos-7-minimal-installation/
```
$ nmcli d
$ nmtui
```
### disable firewall for master nodes
```
systemctl status firewalld.service
systemctl stop firewalld.service
systemctl disable firewalld.service
```
Installation might fail on etcd health check
https://github.com/etcd-io/etcd/issues/8377


### openssh
```
yum install -y openssh-server
```


## Master node 1
### ssh
```
ssh-keygen -t rsa
ssh-copy-id root@192.168.1.151
ssh-copy-id root@192.168.1.152
ssh-copy-id root@192.168.1.153
ssh-copy-id root@192.168.1.155
ssh-copy-id root@192.168.1.161
ssh-copy-id root@192.168.1.162
```

### install pip
https://www.liquidweb.com/kb/how-to-install-pip-on-centos-7/
```
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py
```

### get kubespray ready
```
mkdir srv
cd srv
git clone https://github.com/southbridgeio/kubespray
cd kubespray

pip install -r requirements.txt
```
#### kubespray settings
```
cp -r inventory/s000/ inventory/zxspec/
```
inventory/zxspec/inventory.ini
```
k8s-master-1.local ansible_host=192.168.1.151 ip=192.168.1.151
k8s-master-2.local ansible_host=192.168.1.152 ip=192.168.1.152
k8s-master-3.local ansible_host=192.168.1.153 ip=192.168.1.153
k8s-ingress-1.local ansible_host=192.168.1.155 ip=192.168.1.155
k8s-node-1.local ansible_host=192.168.1.161 ip=192.168.1.161
k8s-node-2.local ansible_host=192.168.1.162 ip=192.168.1.162

[kube-master]
k8s-master-1.local
k8s-master-2.local
k8s-master-3.local

[etcd]
k8s-master-1.local
k8s-master-2.local
k8s-master-3.local

[kube-node]
k8s-node-1.local
k8s-node-2.local
k8s-ingress-1.local

[kube-ingress]
k8s-ingress-1.local

[k8s-cluster:children]
kube-node
kube-master

```

inventory/zxspec/group_vars/k8s-cluster/k8s-cluster.yml
```
. . .
## Change this to use another Kubernetes version, e.g. a current beta release
kube_version: v1.16.9
. . .
# DNS configuration.
# Kubernetes cluster name, also will be used as DNS domain
cluster_name: zxspec.local
```

inventory/zxspec/group_vars/k8s-cluster/k8s-net-flannel.yml
```
## Select interface that should be used for flannel operations by regexp on Name or IP
## This is actually an inventory cluster-level item
## example: select interface with ip from net 10.0.0.0/23
## single quote and escape backslashes
flannel_interface_regexp: '192\\.168\\.1\\.\\d{1,3}'
```

_deploy_cluster.sh 
```shell
#!/bin/sh
. . .
ansible-playbook -u "$1" -i inventory/zxspec/inventory.ini cluster.yml -b --diff        
```
###  start installation
update _deploy_cluster.sh 
```
sh _deploy_cluster.sh root
```
