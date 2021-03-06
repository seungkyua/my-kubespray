# my-kubespray
You can easily add your custom logic without modifying kubespray.

The my-kubespray has serveral parameters to override and can deploy ceph rbd provisioner into namespace.

## Prerequisites

- All servers must have the same user, and the user must set root permissions to NOPASSWD in sudo.
- In the deployment node (or master 1 node), you need to enable ssh access to all other servers (including localhost) without a password to the corresponding account.
- Register all cluster servers in the /etc/hosts file.
- Time synchronization between servers with ntp.
- Enable network restart with `systemctl restart network.service` .
- Disable firewall such as selinux or ufw.
- swapoff.
- Install python 2.7.x in ansible runner node.  

## Quick Start

To deploy the cluster you can use :

```
# Download kubespray and my-kubespray
$ git clone https://github.com/kubernetes-sigs/kubespray.git
$ git clone https://github.com/seungkyua/my-kubespray.git

# Change directory into my-kubespray
$ cd my-kubespray

# Install dependencies from ``requirements.txt``
$ sudo pip install -r ../kubespray/requirements.txt

# Copy ``inventory/k2-seungkyua`` as ``inventory/mycluster``
$ cp -rfp inventory/k2-seungkyua inventory/mycluster

# Update Ansible inventory file
$ vi inventory/mycluster/hosts.ini
[all]
k2-master01 ip=192.168.30.151
k2-master02 ip=192.168.30.152
k2-master03 ip=192.168.30.153
k2-ctrl01 ip=192.168.30.154
k2-ctrl02 ip=192.168.30.155
k2-ctrl03 ip=192.168.30.156
k2-cn01 ip=192.168.30.157

[etcd]
k2-master01
k2-master02
k2-master03

[kube-master]
k2-master01
k2-master02
k2-master03

[kube-node]
k2-ctrl01
k2-ctrl02
k2-ctrl03
k2-cn01

[k8s-cluster:children]
kube-node
kube-master

# Review and change parameters under ``inventory/mycluster/group_vars/k8s-cluster.yml``
$ cat inventory/mycluster/group_vars/k8s-cluster.yml
populate_inventory_to_hosts_file: false
override_system_hostname: false
helm_enabled: true
etcd_memory_limit: 8192M
kubeconfig_localhost: true
kubectl_localhost: true
ipip_mode: Never
calico_ip_auto_method: "can-reach=8.8.8.8"
kubeadm_enabled: true
docker_insecure_registries:
  - seungkyua:5000
dashboard_enabled: true
local_volume_provisioner_enabled: true
ingress_nginx_enabled: true
ingress_nginx_host_network: true
ingress_nginx_nodeselector:
  node-role.kubernetes.io/ingress: true
ceph_version: mimic
storageclass_name: rbd
monitors: 192.168.30.23:6789,192.168.30.24:6789,192.168.30.25:6789
admin_token: QBTEBmRVVlh5UmZoTlJBQTMyZTh6Qk5uajV1VElrMDJEbWFwWmc9WA==
user_secret_namespace: default
pool_name: kubes
user_id: kube
user_token: QEFESG9CcFlpZ0o3TVJBQTV2eStjbDM5RXNLcFkzQyt0WEVHckE9WA==

# Run Ansible Playbook to deploy kubernetes cluster
$ ansible-playbook -b -f 30 -i inventory/mycluster/hosts.ini ../kubespray/cluster.yml

# Run Ansible Playbook to deploy ceph rbd provisioner
$ ansible-playbook -b -f 30 -i inventory/mycluster/hosts.ini storage.yml
```

## Miscellaneous

After installation, you can find the artifacts which are `kubectl` and `admin.conf` in inventory/mycluster/artifacts directorin on deploy node(ansible runner).

```
$ ls -al inventory/mycluster/artifacts
total 173620
drwxr-x--- 2 seungkyua seungkyua      4096 Feb 22 05:25 .
drwxrwxr-x 4 seungkyua seungkyua      4096 Feb 22 02:38 ..
-rw-r----- 1 seungkyua seungkyua      5449 Feb 22 05:25 admin.conf
-rwxr-xr-x 1 seungkyua seungkyua 177766296 Feb 22 05:25 kubectl
-rwxr-xr-x 1 seungkyua seungkyua        65 Feb 22 05:25 kubectl.sh
```

If desired, copy `admin.conf` to `~/.kube/config` and `kubectl` to `/usr/local/bin/kubectl`

```
$ mkdir -p ~/.kube
$ cp inventory/mycluster/artifacts/admin.conf ~/.kube/config
$ sudo cp inventory/mycluster/artifacts/kubectl /usr/local/bin/kubectl
```

