
## Purpose

This repo can be used to quickly setup a multi node Kubernetes cluster for developing, testing, training and demo purposes locally, by using Vagrant and Ansible. 
The environment will also create a NFS type Persistent Volume, which comes from the Vagrant host.

## Prerequites

  - A suitable host here should be a CentOS 7 machine, physical or virtual.
  - Vagrant and vagrant-libvirt have been installed on this host and `libvirt` will be used as Vagrant VM provider.
  - Ansible has been installed on this host and Vagrant VM will be provisioned by using Ansible playbooks.
  - NFS server has been configured and tested.

```
[...@koala ~]$ uname -a
Linux koala.fen9.li 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
[...@koala ~]$ 

[...@koala ~]$ vagrant version
Installed Version: 2.2.10
Latest Version: 2.2.10
 
You're running an up-to-date version of Vagrant!
[...@koala ~]$ 

[...@koala ~]$ ansible --version
ansible 2.9.10
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/home/fli/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Aug  7 2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
[...@koala ~]$ 

[...@koala ~]$ showmount -e localhost
Export list for localhost:
/srv/nfs 192.168.122.*
[...@koala ~]$ 
```

## Usage

1. clone this repo.

2. change directory to `vagrant-ansible-k8s`, the file structure should be looked similar as below:

```
[... vagrant-ansible-k8s]$ tree
.
├── k8s-setup
│   ├── master-playbook.yml
│   └── node-playbook.yml
├── LICENSE
├── REAEDME.md
└── Vagrantfile

1 directory, 5 files
[... vagrant-ansible-k8s]$ 
```

3. run `vagrant up` to spin up the kubernetes cluster, the running result should be very similar to below:

```
[... vagrant-ansible-k8s]$ vagrant up
Bringing machine 'k8s-master' up with 'libvirt' provider...
Bringing machine 'node-1' up with 'libvirt' provider...
Bringing machine 'node-2' up with 'libvirt' provider...
...
PLAY RECAP *********************************************************************
k8s-master                 : ok=22   changed=17   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   

PLAY RECAP *********************************************************************
node-2                     : ok=16   changed=13   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

PLAY RECAP *********************************************************************
node-1                     : ok=16   changed=13   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

[... vagrant-ansible-k8s]$ 
```

4. once `vagrant up` command finishes successfully, ssh to kubernetes cluser master node and check the cluser running status

```
[... vagrant-ansible-k8s]$ vagrant ssh k8s-master
Last login: Thu Sep  3 08:24:50 2020 from 192.168.121.1
[vagrant@k8s-master ~]$

[vagrant@k8s-master ~]$ kubectl cluster-info
Kubernetes master is running at https://192.168.122.88:6443
KubeDNS is running at https://192.168.122.88:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
[vagrant@k8s-master ~]$

[vagrant@k8s-master ~]$ kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   60m   v1.19.0
node-1       Ready    <none>   53m   v1.19.0
node-2       Ready    <none>   53m   v1.19.0
[vagrant@k8s-master ~]$

[vagrant@k8s-master ~]$ kubectl get services --all-namespaces
NAMESPACE     NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  60m
kube-system   kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   60m
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-f9fd979d6-4bkhq              1/1     Running   0          60m
kube-system   coredns-f9fd979d6-58zmw              1/1     Running   0          60m
kube-system   etcd-k8s-master                      1/1     Running   0          60m
kube-system   kube-apiserver-k8s-master            1/1     Running   0          60m
kube-system   kube-controller-manager-k8s-master   1/1     Running   0          60m
kube-system   kube-flannel-ds-amd64-9d47c          1/1     Running   0          53m
kube-system   kube-flannel-ds-amd64-9kvdk          1/1     Running   0          53m
kube-system   kube-flannel-ds-amd64-hnrzv          1/1     Running   0          60m
kube-system   kube-proxy-qb942                     1/1     Running   0          53m
kube-system   kube-proxy-vhkg5                     1/1     Running   0          53m
kube-system   kube-proxy-znddm                     1/1     Running   0          60m
kube-system   kube-scheduler-k8s-master            1/1     Running   0          60m
[vagrant@k8s-master ~]$

[vagrant@k8s-master ~]$ kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv0001-nfs   5Gi        RWX            Recycle          Available           slow                    8m23s
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ df -hT
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  912M     0  912M   0% /dev
tmpfs          tmpfs     919M     0  919M   0% /dev/shm
tmpfs          tmpfs     919M   11M  909M   2% /run
tmpfs          tmpfs     919M     0  919M   0% /sys/fs/cgroup
/dev/vda1      xfs        40G  4.6G   36G  12% /
tmpfs          tmpfs     184M     0  184M   0% /run/user/0
tmpfs          tmpfs     184M     0  184M   0% /run/user/1000
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ exit
logout
Connection to 192.168.121.210 closed.
[... vagrant-ansible-k8s]$ 
```

## Change the default configuration

In the `Vagrantfile`

1. The number of worker nodes is defined in `N = 2`.

2. The VM memory and cpus are defined in 
```
v.memory = 2048
v.cpus = 2
```

## Clean up
```
vagrant halt
vagrant destroy
```

## Reference
1. [Kubernetes Setup Using Ansible and Vagrant](https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/)
2. [Vagrant](https://www.vagrantup.com/)
3. [Ansible](https://www.ansible.com/)

