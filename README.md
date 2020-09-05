
## Purpose

This repo can be used to quickly setup a multi node Kubernetes cluster for developing, testing, training and demo purposes locally, by using Vagrant and Ansible. 
The environment will also create sample NFS type Persistent Volumes, which have provisioned from the Vagrant host NFS server.

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

## Deploy example 1 - `hashicorp/consul`
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
sh get_helm.sh

helm repo add hashicorp https://helm.releases.hashicorp.com

[vagrant@k8s-master ~]$ helm search repo hashicorp/consul
NAME            	CHART VERSION	APP VERSION	DESCRIPTION                    
hashicorp/consul	0.24.1       	1.8.2      	Official HashiCorp Consul Chart
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ kubectl get services -n kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   20m
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ helm install -f helm-consul-values.yaml hashicorp hashicorp/consul
NAME: hashicorp
LAST DEPLOYED: Fri Sep  4 05:58:30 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Consul!

Now that you have deployed Consul, you should look over the docs on using 
Consul with Kubernetes available here: 

https://www.consul.io/docs/platform/k8s/index.html


Your release is named hashicorp.

To learn more about the release if you are using Helm 2, run:

  $ helm status hashicorp
  $ helm get hashicorp

To learn more about the release if you are using Helm 3, run:

  $ helm status hashicorp
  $ helm get all hashicorp
[vagrant@k8s-master ~]$ 
```

* Come cross 1st error `no persistent volumes available for this claim and no storage class is set`
```
[vagrant@k8s-master ~]$ kubectl get events
LAST SEEN   TYPE      REASON                    OBJECT                                                                       MESSAGE
14s         Normal    FailedBinding             persistentvolumeclaim/data-default-hashicorp-consul-server-0                 no persistent volumes available for this claim and no storage class is set
41s         Normal    SuccessfulCreate          replicaset/hashicorp-consul-connect-injector-webhook-deployment-744454c968   Created pod: hashicorp-consul-connect-injector-webhook-deployment-74445cn8jl
41s         Normal    Scheduled                 pod/hashicorp-consul-connect-injector-webhook-deployment-74445cn8jl          Successfully assigned default/hashicorp-consul-connect-injector-webhook-deployment-74445cn8jl to node-2
37s         Normal    Pulling                   pod/hashicorp-consul-connect-injector-webhook-deployment-74445cn8jl          Pulling image "hashicorp/consul-k8s:0.18.1"
41s         Normal    ScalingReplicaSet         deployment/hashicorp-consul-connect-injector-webhook-deployment              Scaled up replica set hashicorp-consul-connect-injector-webhook-deployment-744454c968 to 1
41s         Normal    Scheduled                 pod/hashicorp-consul-f4d45                                                   Successfully assigned default/hashicorp-consul-f4d45 to node-1
40s         Warning   FailedMount               pod/hashicorp-consul-f4d45                                                   MountVolume.SetUp failed for volume "config" : failed to sync configmap cache: timed out waiting for the condition
36s         Normal    Pulling                   pod/hashicorp-consul-f4d45                                                   Pulling image "consul:1.8.2"
40s         Normal    Scheduled                 pod/hashicorp-consul-jvjf4                                                   Successfully assigned default/hashicorp-consul-jvjf4 to node-2
36s         Normal    Pulling                   pod/hashicorp-consul-jvjf4                                                   Pulling image "consul:1.8.2"
39s         Warning   FailedScheduling          pod/hashicorp-consul-server-0                                                0/3 nodes are available: 3 pod has unbound immediate PersistentVolumeClaims.
42s         Normal    NoPods                    poddisruptionbudget/hashicorp-consul-server                                  No matching pods found
41s         Normal    SuccessfulCreate          statefulset/hashicorp-consul-server                                          create Claim data-default-hashicorp-consul-server-0 Pod hashicorp-consul-server-0 in StatefulSet hashicorp-consul-server success
39s         Normal    SuccessfulCreate          statefulset/hashicorp-consul-server                                          create Pod hashicorp-consul-server-0 in StatefulSet hashicorp-consul-server successful
41s         Normal    SuccessfulCreate          daemonset/hashicorp-consul                                                   Created pod: hashicorp-consul-f4d45
40s         Normal    SuccessfulCreate          daemonset/hashicorp-consul                                                   Created pod: hashicorp-consul-jvjf4
...
[vagrant@k8s-master ~]$ 
```

* fix the issue


  1. find storage requirement from [`hashicorp/consul` server persistent volume request]((https://github.com/hashicorp/consul-helm/blob/master/values.yaml))

```
  # storage and storageClass are the settings for configuring stateful
  # storage for the server pods. storage should be set to the disk size of
  # the attached volume. storageClass is the class of storage which defaults
  # to null (the Kube cluster will pick the default).
  storage: 10Gi
  storageClass: null
```

  2. Provide qualified persistent volume in k8s cluser.

```
[vagrant@k8s-master ~]$ cat /vagrant/k8s-setup/nfs-pv.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001-nfs
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /srv/nfs
    server: 192.168.122.1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0002-nfs
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: null
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /srv/nfs
    server: 192.168.122.1
[vagrant@k8s-master ~]$ 

kubectl apply -f /vagrant/k8s-setup/nfs-pv.yaml
How To Secure Consul with TLS Encryption on Ubuntu 14.04
[vagrant@k8s-master ~]$ kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                            STORAGECLASS   REASON   AGE
pv0001-nfs   5Gi        RWX            Recycle          Available                                                    slow                    151m
pv0002-nfs   50Gi       RWO            Recycle          Bound       default/data-default-hashicorp-consul-server-0                           151m
[vagrant@k8s-master ~]$ 
```

* Come cross 2nd error `TLS handshake error from 10.0.2.1:39114: No certificate available.`
```
[vagrant@k8s-master ~]$ kubectl get all
NAME                                                                  READY   STATUS             RESTARTS   AGE
pod/hashicorp-consul-connect-injector-webhook-deployment-744452qgqk   0/1     CrashLoopBackOff   3          50s
pod/hashicorp-consul-qjr52                                            1/1     Running            0          49s
pod/hashicorp-consul-server-0                                         1/1     Running            0          50s
pod/hashicorp-consul-vffbw                                            1/1     Running            0          51s

NAME                                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                   AGE
service/hashicorp-consul-connect-injector-svc   ClusterIP   10.102.17.65     <none>        443/TCP                                                                   51s
service/hashicorp-consul-dns                    ClusterIP   10.109.147.48    <none>        53/TCP,53/UDP                                                             51s
service/hashicorp-consul-server                 ClusterIP   None             <none>        8500/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   51s
service/hashicorp-consul-ui                     NodePort    10.106.106.126   <none>        80:30680/TCP                                                              51s
service/kubernetes                              ClusterIP   10.96.0.1        <none>        443/TCP                                                                   146m

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/hashicorp-consul   2         2         2       2            2           <none>          51s

NAME                                                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hashicorp-consul-connect-injector-webhook-deployment   0/1     1            0           51s

NAME                                                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/hashicorp-consul-connect-injector-webhook-deployment-744454c968   1         1         0       51s

NAME                                       READY   AGE
statefulset.apps/hashicorp-consul-server   1/1     51s
[vagrant@k8s-master ~]$ 

[vagrant@k8s-master ~]$ kubectl logs hashicorp-consul-connect-injector-webhook-deployment-744452qgqk
Listening on ":8080"...
Updated certificate bundle received. Updating certs...
2020/09/04 09:10:13 http: TLS handshake error from 10.0.2.1:39114: No certificate available.
2020/09/04 09:10:13 http: TLS handshake error from 10.0.2.1:39116: No certificate available.
2020/09/04 09:10:15 http: TLS handshake error from 10.0.2.1:39120: No certificate available.
2020/09/04 09:10:15 http: TLS handshake error from 10.0.2.1:39124: No certificate available.
[vagrant@k8s-master ~]$ 
```

* fix the issue
[Consul verified TLS from Pods in Kubernetes cluster](https://discuss.hashicorp.com/t/consul-verified-tls-from-pods-in-kubernetes-cluster/9208)

```
[vagrant@k8s-master ~]$ kubectl exec -it hashicorp-consul-server-0 -- /bin/sh
/ # consul members
Node                       Address        Status  Type    Build  Protocol  DC    Segment
hashicorp-consul-server-0  10.0.1.3:8301  alive   server  1.8.2  2         demo  <all>
node-1                     10.0.1.2:8301  alive   client  1.8.2  2         demo  <default>
node-2                     10.0.2.3:8301  alive   client  1.8.2  2         demo  <default>
/ # 

/ # consul tls ca create
==> Saved consul-agent-ca.pem
==> Saved consul-agent-ca-key.pem
/ # 

/ # consul tls cert create -server
==> WARNING: Server Certificates grants authority to become a
    server and access all state in the cluster including root keys
    and all ACL tokens. Do not distribute them to production hosts
    that are not server nodes. Store them as securely as CA keys.
==> Using consul-agent-ca.pem and consul-agent-ca-key.pem
==> Saved dc1-server-consul-0.pem
==> Saved dc1-server-consul-0-key.pem
/ # 

/ # ls
bin                          home                         run
consul                       lib                          sbin
consul-agent-ca-key.pem      lib64                        srv
consul-agent-ca.pem          media                        sys
dc1-server-consul-0-key.pem  mnt                          tmp
dc1-server-consul-0.pem      opt                          usr
dev                          proc                         var
etc                          root
/ # 

/ # cat consul-agent-ca.pem
-----BEGIN CERTIFICATE-----
MIIC7zCCApSgAwIBAgIRANmwchV1y+JQEcdEJUSlVkQwCgYIKoZIzj0EAwIwgbkx
CzAJBgNVBAYTAlVTMQswCQYDVQQIEwJDQTEWMBQGA1UEBxMNU2FuIEZyYW5jaXNj
bzEaMBgGA1UECRMRMTAxIFNlY29uZCBTdHJlZXQxDjAMBgNVBBETBTk0MTA1MRcw
FQYDVQQKEw5IYXNoaUNvcnAgSW5jLjFAMD4GA1UEAxM3Q29uc3VsIEFnZW50IENB
IDI4OTM1ODYzMzIyNzM3MTMyOTk5NTA1MTc4MzYzMTU2MDU5NTAxMjAeFw0yMDA5
MDUwNjM5NTlaFw0yNTA5MDQwNjM5NTlaMIG5MQswCQYDVQQGEwJVUzELMAkGA1UE
CBMCQ0ExFjAUBgNVBAcTDVNhbiBGcmFuY2lzY28xGjAYBgNVBAkTETEwMSBTZWNv
bmQgU3RyZWV0MQ4wDAYDVQQREwU5NDEwNTEXMBUGA1UEChMOSGFzaGlDb3JwIElu
Yy4xQDA+BgNVBAMTN0NvbnN1bCBBZ2VudCBDQSAyODkzNTg2MzMyMjczNzEzMjk5
OTUwNTE3ODM2MzE1NjA1OTUwMTIwWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAARW
5yKIE2c50g3uNNJk2uhRCgvxtXeJAuKFLcftSbY1K1NODZ+9sQ0PXEkv1w0d6dU7
rbPIhjwVqM+MCxkxAZ8Qo3sweTAOBgNVHQ8BAf8EBAMCAYYwDwYDVR0TAQH/BAUw
AwEB/zApBgNVHQ4EIgQgLa1HQjHFyrilTQcHIUR3VDmif/qyEKjDzV1tOI+ZtNow
KwYDVR0jBCQwIoAgLa1HQjHFyrilTQcHIUR3VDmif/qyEKjDzV1tOI+ZtNowCgYI
KoZIzj0EAwIDSQAwRgIhAKR7abq2cymdTUwj0vWG1pziMRWSKJEjedo6sYvELn6g
AiEAvx6bSgEGhLg58ik6Fu1f8dTnfXfHPaLMqbGoGJcl5QA=
-----END CERTIFICATE-----
/ # cat consul-agent-ca-key.pem
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIE41/I8gi8CKFxMXygCU4opgjwe/UH672FsWOXuuHphWoAoGCCqGSM49
AwEHoUQDQgAEVuciiBNnOdIN7jTSZNroUQoL8bV3iQLihS3H7Um2NStTTg2fvbEN
D1xJL9cNHenVO62zyIY8FajPjAsZMQGfEA==
-----END EC PRIVATE KEY-----
/ # 

/ # cat dc1-server-consul-0.pem
-----BEGIN CERTIFICATE-----
MIICnTCCAkOgAwIBAgIRALwr7daJznyQPeiDv2WOis4wCgYIKoZIzj0EAwIwgbkx
CzAJBgNVBAYTAlVTMQswCQYDVQQIEwJDQTEWMBQGA1UEBxMNU2FuIEZyYW5jaXNj
bzEaMBgGA1UECRMRMTAxIFNlY29uZCBTdHJlZXQxDjAMBgNVBBETBTk0MTA1MRcw
FQYDVQQKEw5IYXNoaUNvcnAgSW5jLjFAMD4GA1UEAxM3Q29uc3VsIEFnZW50IENB
IDI4OTM1ODYzMzIyNzM3MTMyOTk5NTA1MTc4MzYzMTU2MDU5NTAxMjAeFw0yMDA5
MDUwNjQwMjlaFw0yMTA5MDUwNjQwMjlaMBwxGjAYBgNVBAMTEXNlcnZlci5kYzEu
Y29uc3VsMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEtwFCyQhl1ocO3McZcV4z
52/SLimh0wlpy1YUtiLGva/BEzcfEa48/ITsDaU4YazZ7BiVTHwVrLa5/UiEF2Gi
wKOBxzCBxDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYwFAYIKwYBBQUHAwEGCCsG
AQUFBwMCMAwGA1UdEwEB/wQCMAAwKQYDVR0OBCIEINjQx1T1kFTATjbupss+mqfI
qqdvTRvhcixocizQxJUSMCsGA1UdIwQkMCKAIC2tR0Ixxcq4pU0HByFEd1Q5on/6
shCow81dbTiPmbTaMC0GA1UdEQQmMCSCEXNlcnZlci5kYzEuY29uc3Vsgglsb2Nh
bGhvc3SHBH8AAAEwCgYIKoZIzj0EAwIDSAAwRQIgFW3OibG80tOgWALgFhCaXmhN
To5oEv5rMj888+5ou2ECIQD8NF/5VNdmRpE+ymKAqaUnQhuIurtyYH3KPbqNYkS5
bQ==
-----END CERTIFICATE-----
/ # cat dc1-server-consul-0-key.pem
-----BEGIN EC PRIVATE KEY-----
MHcCAQEEIHr+YYYqtQhkqBfoYy2xEEL9ED2yqn89xG6zmj7MCYpioAoGCCqGSM49
AwEHoUQDQgAEtwFCyQhl1ocO3McZcV4z52/SLimh0wlpy1YUtiLGva/BEzcfEa48
/ITsDaU4YazZ7BiVTHwVrLa5/UiEF2GiwA==
-----END EC PRIVATE KEY-----
/ # 
```