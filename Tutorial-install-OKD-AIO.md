# Tutorial Install OKD AIO

#1. matikan firewalld
```
systemctl stop firewalld
systemctl disable firewalld
```

#2. install package openshift origin 3.11
```
yum -y install centos-release-openshift-origin311 epel-release docker git pyOpenSSL 
```

#3. aktifkan service docker
```
systemctl start docker
systemctl enable docker
systemctl status docker
```

#4. konfigurasi docker storage
### Siapkan 2 disk, sda --> 50 GB (untuk os) dan sdb --> 50 GB (untuk docker storage)

```
fdisk -l
vgcreate docker-vg /dev/sdb
vgdisplay
sudo cat <<EOF > /etc/sysconfig/docker-storage-setup
VG=docker-vg
EOF

mv /etc/sysconfig/docker-storage /etc/sysconfig/docker-storage-backup.back
docker-storage-setup
```

#5. Edit hosts
```
vi /etc/hosts
192.168.1.50 okd.i3datacenter.com registry.i3datacenter.com
```

#6. Push ssh public key
```
ssh-keygen
ssh-copy-id root@okd.i3datacenter.com
```

#7. install openshift-ansible 
```
sudo yum install openshift-ansible -y
```
note: untuk ansible pastikan version 2.6

#8. Edit Inventory
```
[OSEv3:children]
masters
nodes
etcd

# Set variables common for all OSEv3 hosts
[OSEv3:vars]
# SSH user, this user should allow ssh based auth without requiring a password
ansible_ssh_user=root

# If ansible_ssh_user is not root, ansible_become must be set to true
#ansible_become=true

#type_deployment
openshift_deployment_type=origin

# metrics
openshift_metrics_install_metrics=true

# prometheous operator
openshift_cluster_monitoring_operator_install=true

openshift_release=3.11

# # define default sub-domain for Master node
openshift_master_cluster_hostname=console.i3datacenter.com
openshift_master_cluster_public_hostname=console.i3datacenter.com
openshift_master_default_subdomain=apps.i3datacenter.com
openshift_master_api_port=8443
openshift_master_console_port=8443

# uncomment the following to enable htpasswd authentication; defaults to DenyAllPasswordIdentityProvider
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
openshift_master_htpasswd_users={'admin' : '$apr1$cHq4R9bg$WznMbawFn6tMFmCwI76Nb0', 'developer' : '$apr1$qPoxMv8g$7UGodl3cZP2rdfu5dqbdl1'}
openshift_disable_check=memory_availability,disk_availability,docker_image_availability,docker_storage

# host group for masters
[masters]
okd.i3datacenter.com openshift_schedulable=true containerized=false

# host group for etcd
[etcd]
okd.i3datacenter.com

# host group for nodes, includes region info
[nodes]
okd.i3datacenter.com openshift_node_group_name='node-config-all-in-one'
```

#9. Jalankan prereq playbook
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/prerequisites.yml 

#10. jalankan deploy cluster playbook
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/deploy_cluster.yml 


#11. membuat user - di master
sudo htpasswd /etc/origin/master/htpasswd alan 

#12. memberikan hak akses admin ke user
oc adm policy add-cluster-role-to-user cluster-admin alan

Ref: 
https://docs.openshift.com/container-platform/3.3/admin_solutions/user_role_mgmt.html
https://docs.openshift.com/container-platform/3.3/architecture/additional_concepts/authorization.html#cluster-policy-and-local-policy