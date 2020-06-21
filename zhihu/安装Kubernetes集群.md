



# 在fedora32/centos8上安装Kubernetes集群

> [Kubernetes](http://kubernetes.io)是Google基于Borg开源的容器编排调度引擎，作为[CNCF](http://cncf.io)（Cloud Native Computing Foundation）最重要的组件之一，它的目标不仅仅是一个编排系统，而是提供一个规范，可以让你来描述集群的架构，定义服务的最终状态，kubernetes可以帮你将系统自动得达到和维持在这个状态。
>
> 更直白的说，Kubernetes用户可以通过编写一个yaml或者json格式的配置文件，也可以通过工具/代码生成或直接请求kubernetes API创建应用，该配置文件中包含了用户想要应用程序保持的状态，不论整个kubernetes集群中的个别主机发生什么问题，都不会影响应用程序的状态，你还可以通过改变该配置文件或请求kubernetes API来改变应用程序的状态。



**Repository:** [Author: GitHub](https://github.com/tangtaoshadow/)  

**Introduction:**  

**Author:**[杭州电子科技大学](http://www.hdu.edu.cn/)  [唐涛](https://www.promiselee.cn/tao) 

**CreateTime:**`2020-6-21 09:11:19`

**UpdateTime:**`2020-6-21 17:39:16`

**Copyright:**  唐涛 [HOME | 首页](https://www.promiselee.cn/tao) **©**  ***2020***  [<img alt="tangtao" style="width:35px;display:inline;" src="https://www.promiselee.cn/share_static/files/github/tao-logo.svg"/>](https://www.promiselee.cn/tao)  [<img style="width:25px;display:inline;margin-bottom:5px;" alt="github" src="https://www.promiselee.cn/share_static/files/github/github-logo.svg"/>](https://github.com/tangtaoshadow)

**Email:**  <tangtao2099@outlook.com>

**Link:**  [知乎](https://www.zhihu.com/people/tang-tao-24-36/activities)   [GitHub](https://github.com/tangtaoshadow) [语雀](https://www.yuque.com/tangtao-frbji)



---

[toc]

运行环境

```shell
[root@localhost ~]# cat /etc/redhat-release 
Fedora release 32 (Thirty Two)
[root@localhost ~]# cat /proc/version
Linux version 5.6.19-300.fc32.x86_64 (mockbuild@bkernel01.iad2.fedoraproject.org) (gcc version 10.1.1 20200507 (Red Hat 10.1.1-1) (GCC)) #1 SMP Wed Jun 17 16:10:48 UTC 2020
```



kubernetes版本 v1.18.4

```shell
[root@localhost ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.4", GitCommit:"c96aede7b5205121079932896c4ad89bb93260af", GitTreeState:"clean", BuildDate:"2020-06-17T11:41:22Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.4", GitCommit:"c96aede7b5205121079932896c4ad89bb93260af", GitTreeState:"clean", BuildDate:"2020-06-17T11:33:59Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```



---



### 开始搭建集群环境

在安装新组件之前，先升级系统

```shell
dnf -y upgrade
```



禁用SELinux

```shell
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```



启用透明伪装并促进虚拟可扩展LAN（`VxLAN`）通信，以在整个集群中的Kubernetes Pod之间进行通信。

```sh
modprobe br_netfilter
```



防火墙上启用IP伪装

```shell
firewall-cmd --add-masquerade --permanent
firewall-cmd --reload
```



设置桥接数据包以遍历iptables规则

```shell
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```



加载新规则

```shell
sysctl --system
```



禁用所有内存交换以提高性能

> 也可以通过kubernetes手动开启忽略交换分区，但这样可能会降低集群性能💥。

```shell
swapoff -a
vim /etc/fstab
---------------注释掉 swap 重启系统----------------

=========================示例=======================
[root@localhost tao]# swapoff -a
[root@localhost tao]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Mon Jun 15 02:13:37 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/fedora_localhost--live-root /                       ext4    defaults        1 1
UUID=fbdf9a0f-5f7a-4542-9988-b579662fd622 /boot                   ext4    defaults        1 2
/dev/mapper/fedora_localhost--live-swap none                    swap    defaults        0 0
-----------重启之后----------
[root@localhost tao]# free -h
              total        used        free      shared  buff/cache   available
Mem:          7.7Gi       670Mi       6.4Gi       7.0Mi       709Mi       6.8Gi
Swap:            0B          0B          0B
```



### 安装docker

> 参考前文，或参考[官网](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/)



### 在主节点和工作节点上安装Kubernetes

将Kubernetes存储库添加到您的包管理器中。

> 链接：[谷歌存储库](https://packages.cloud.google.com/yum/repos)

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```



更新存储库信息

```shell
dnf upgrade -y
```



安装Kubernetes的所有必需组件

```shell
dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```



启动Kubernetes服务，并使其在启动时运行

> 先要初始化 `kubeadm init` 才能查看`kubelet`的状态🔻。

```shell
systemctl enable kubelet
systemctl start kubelet
-----------------------------
[root@localhost system]# systemctl enable kubelet
Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /usr/lib/systemd/system/kubelet.service.
[root@localhost system]# systemctl start kubelet
```

> 注意：
>
> 通过上面的输出可以发现 `Created symlink /etc/systemd/system/multi-user.target.wants/kubelet.service → /usr/lib/systemd/system/kubelet.service`，因此kubernetes的service在 `/usr/lib/systemd/system/kubelet.service`



kubelet启动报错

```shell
21 02:51:57 localhost.localdomain kubelet[24920]: F0621 02:51:57.778480   24920 kubelet.go:1383] Failed to start ContainerManager system validation failed - Following Cgroup subsystem not mounted: [cpu c

----解决----
dnf install -y grubby
grubby --update-kernel=ALL --args="systemd.unified_cgroup_hierarchy=0"
reboot
```



kubelet `unknown container "/system.slice/docker.service"`

```shell
----------- 错误日志 关闭kubelet 重启系统就好了 -------------
6月 21 04:02:07 localhost.localdomain kubelet[782]: E0621 04:02:07.593760     782 summary_sys_containers.go:47] Failed to get system container stats for "/system.slice/docker.service": failed to get cgroup stats for "/system.slice/docker.service": failed to get container info for "/system.slice/docker.service": unknown container "/system.slice/docker.service"
```





### 在master上配置Kubernetes

配置kubeadm

```shell
kubeadm config images pull
```



开放Kubernetes的必要端口

> 虽然生产环境集群部署在内网，但还是不建议关闭防火墙💢。

```shell
firewall-cmd --zone=public --permanent --add-port={6443,2379,2380,10250,10251,10252}/tcp
```



允许docker从另一个节点访问，注意替换成你的**`worker-IP`**❗

```shell
firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=worker-IP-address/32 accept'

------------------------------------------
[root@localhost tao]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a5:06:fe brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    inet 192.168.253.130/24 brd 192.168.253.255 scope global dynamic noprefixroute ens33
       valid_lft 1341sec preferred_lft 1341sec
    inet6 fe80::b7ad:a479:c3f4:909/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:b3:47:db brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
    link/ether 52:54:00:b3:47:db brd ff:ff:ff:ff:ff:ff
5: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:fb:fd:1f:c8 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
[root@localhost tao]# firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=192.168.253.0/24 accept'
success
```



允许从Docker容器访问主机的本地主机

```shell
firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=172.17.0.0/16 accept'
```



刷新防火墙配置

```shell
firewall-cmd --reload
```



为Kubernetes安装CNI（容器网络接口）插件

> 参考：https://docs.projectcalico.org/getting-started/kubernetes/quickstart#overview



#### 集群初始化

> 如果您的网络中已经使用了`192.168.0.0/16`，则必须选择其他Pod网络CIDR，以替换上述命令中的`192.168.0.0/16`。

```shell
kubeadm init --pod-network-cidr 192.169.0.0/16

-----初始化失败记得 -----
kubeadm reset

----- 查看错误日志 ---------
journalctl -xe
```

启动成功输出

```shell
[root@localhost tao]# kubeadm init --pod-network-cidr 192.169.0.0/16
W0621 03:03:45.439927    3684 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.4
[preflight] Running pre-flight checks
        [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [localhost.localdomain kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.253.130]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [192.168.253.130 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost.localdomain localhost] and IPs [192.168.253.130 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0621 03:03:50.888041    3684 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0621 03:03:50.891060    3684 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 18.009144 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node localhost.localdomain as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 20sbm6.o0csnemiz75cs7nx
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.253.130:6443 --token 20sbm6.o0csnemiz75cs7nx \
    --discovery-token-ca-cert-hash sha256:27c27955f6c2e2251ab0864fb8194e5fe30d050c5ae1a4b8316671234c9f81f9 
```



```
上面输出的内容
kubeadm join 192.168.253.130:6443 --token 20sbm6.o0csnemiz75cs7nx \
    --discovery-token-ca-cert-hash sha256:27c27955f6c2e2251ab0864fb8194e5fe30d050c5ae1a4b8316671234c9f81f9 

用于新节点加入集群
```



##### 解决初始化问题

> 参考链接：[官网](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/)

###### Cgroup 驱动程序

```shell
[root@localhost prometheus]# kubeadm init --pod-network-cidr 192.168.0.0/16
W0621 01:47:20.228892   31202 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.4
[preflight] Running pre-flight checks
        [WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
```

解决

```shell
## Create /etc/docker directory.
mkdir /etc/docker

# Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

systemctl daemon-reload
systemctl restart docker
```



```shell
[root@localhost tao]# docker info | grep -i cgroup
 Cgroup Driver: systemd
```



> 🛑注意：请确保kublet正常启动

查看systemctk启动日志

```shell
journalctl -f 
journalctl -xe
```



##### 在控制平面节点上配置 kubelet 使用的 cgroup 驱动程序

```shell
vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=

---修改后 ---------------
[root@localhost tao]# cat /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--cgroup-driver=systemd

systemctl daemon-reload
systemctl restart kubelet
```



#### 制作以下目录和配置文件

```shell
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```



#### 开始安装Calico

安装Tigera Calico

```shell
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
```



通过创建必要的自定义资源来安装Calico，有关此清单中可用配置选项的更多信息，请参阅[安装参考](https://docs.projectcalico.org/reference/installation/api)。

```shell
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml
```



确认所有Pod正在运行

```shell
kubectl get pods --all-namespaces  

[root@localhost tao]# kubectl get pods --all-namespaces  
NAMESPACE         NAME                                            READY   STATUS    RESTARTS   AGE
kube-system       calico-kube-controllers-58b656d69f-pd65v        1/1     Running   0          8m58s
kube-system       calico-node-vvbl9                               1/1     Running   0          8m58s
kube-system       coredns-66bff467f8-j947g                        1/1     Running   0          24m
kube-system       coredns-66bff467f8-mr7lb                        1/1     Running   0          24m
kube-system       etcd-localhost.localdomain                      1/1     Running   0          24m
kube-system       kube-apiserver-localhost.localdomain            1/1     Running   0          24m
kube-system       kube-controller-manager-localhost.localdomain   1/1     Running   0          24m
kube-system       kube-proxy-f28jk                                1/1     Running   0          24m
kube-system       kube-scheduler-localhost.localdomain            1/1     Running   0          24m
tigera-operator   tigera-operator-c9cf5b94d-wnt9t                 1/1     Running   0          8m32s
```



使pod可以在Master上运行，仅用于学习环境，不建议生产环境

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-

-------------输出---------------
[root@localhost tao]# kubectl taint nodes --all node-role.kubernetes.io/master-
node/localhost.localdomain untainted
```



确认集群中现在有一个节点

```shell
kubectl get nodes -o wide
===============================
[root@localhost tao]# kubectl get nodes -o wide
NAME                    STATUS   ROLES    AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                          KERNEL-VERSION           CONTAINER-RUNTIME
localhost.localdomain   Ready    master   31m   v1.18.3   192.168.253.130   <none>        Fedora 32 (Workstation Edition)   5.6.19-300.fc32.x86_64   docker://19.3.11
```



至此，带有Calico的单主机Kubernetes集群配置完成👌。



### 在Worker节点上配置Kubernetes

> 通常每个Kubernetes安装都需要有一个或多个运行容器化应用程序的工作程序节点。在这里仅配置一个工作服务器，可以重复这些步骤将更多节点加入集群中。



开放Kubernetes端口

```shell
firewall-cmd --zone=public --permanent --add-port={10250,30000-32767}/tcp
firewall-cmd --reload
```



如果曾经配置过，先重置❗，强制重置

```shell
kubeadm reset --force
```



#### 加入集群

查看master的token，或重新创建token

```shell
[root@localhost tao]# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
20sbm6.o0csnemiz75cs7nx   23h         2020-06-22T03:04:10-04:00   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token

************** 如果过期了，使用 kubeadm token create 重新创建 *********
[root@localhost tao]# kubeadm token create
W0621 03:45:29.067220    6207 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
ay51j4.wxzqv81jz6c4htxi
[root@localhost tao]# kubeadm token list
TOKEN                     TTL         EXPIRES                     USAGES                   DESCRIPTION                                                EXTRA GROUPS
20sbm6.o0csnemiz75cs7nx   23h         2020-06-22T03:04:10-04:00   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token
ay51j4.wxzqv81jz6c4htxi   23h         2020-06-22T03:45:29-04:00   authentication,signing   <none>                                                     system:bootstrappers:kubeadm:default-node-token

```



获取 CA 证书 sha256 编码 hash 值

> 因为集群都是采用ssl

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
[root@localhost tao]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
27c27955f6c2e2251ab0864fb8194e5fe30d050c5ae1a4b8316671234c9f81f9
```



新节点申请加入集群

> 初始化的时候有输出✅，如果过期了，使用上面的命令重新生成即可。

```shell
kubeadm join 192.168.253.130:6443 --token 20sbm6.o0csnemiz75cs7nx \
    --discovery-token-ca-cert-hash sha256:27c27955f6c2e2251ab0864fb8194e5fe30d050c5ae1a4b8316671234c9f81f9 
    
    
==================输出 ======================
[root@tangtao tao]# kubeadm join 192.168.253.130:6443 --token 20sbm6.o0csnemiz75cs7nx \
>     --discovery-token-ca-cert-hash sha256:27c27955f6c2e2251ab0864fb8194e5fe30d050c5ae1a4b8316671234c9f81f9 
W0621 05:10:24.959560    2291 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING Hostname]: hostname "tangtao.k8s-node1" could not be reached
        [WARNING Hostname]: hostname "tangtao.k8s-node1": lookup tangtao.k8s-node1 on 192.168.253.2:53: no such host
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```



master节点查看

```shell
[root@localhost tao]# kubectl get nodes
NAME                    STATUS   ROLES    AGE    VERSION
localhost.localdomain   Ready    master   127m   v1.18.3
tangtao.k8s-node1       Ready    <none>   82s    v1.18.3
```





---

更新时间：`2020-6-21 18:54:57`

原文链接：[GitHub](https://github.com/tangtaoshadow/public_knowledge-tao/blob/master/zhihu/%E5%AE%89%E8%A3%85Kubernetes%E9%9B%86%E7%BE%A4.md)  [Web](https://www.promiselee.cn/share_static/files/public_knowledge/zhihu/%E5%AE%89%E8%A3%85Kubernetes%E9%9B%86%E7%BE%A4.html)