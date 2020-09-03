
## Purpose

This repo can be used to quickly setup a multi node Kubernetes cluster for developing, testing, training and demo purposes locally, by using Vagrant and Ansible. 

## Prerequites

  - A suitable host here should be a CentOS 7 machine, physical or virtual.
  - Vagrant and vagrant-libvirt have been installed on this host and `libvirt` will be used as Vagrant VM provider.
  - Ansible has been installed on this host and Vagrant VM will be provisioned by using Ansible playbooks.

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

