#### 1.操作系统环境配置

* 修改主机名添加hosts解析

  ```bash
  hostnamectl set-hostname master01  ##其他节点相应修改
  cat << EOF >> /etc/hosts  ##复制hosts文件到其他每个节点
  192.168.135.231  master01
  192.168.135.240  master02
  192.168.135.199  master03
  192.168.135.206  node01
  192.168.135.226  node02
  192.168.135.211  node03
  192.168.135.245  node04
  192.168.135.234  node05
  192.168.135.228  node06
  192.168.135.238  node07
  EOF
  ```

* 格式化数据盘并挂载

  ```bash
  fdisk /dev/vdb     ##创建分区
  mkfs.xfs /dev/vdb1    ##格式化分区为xfs
  mkdir /var/lib/docker  ##预先创建docker目录
  mount /dev/vdb1 /var/lib/docker  ## 用UUID挂载写入fstab
  ```

* 上传操作系统镜像并挂载制作本地yum源

  ```bash
  mkdir /mnt/iso   ##创建镜像挂载目录
  mount -o loop CentOS-7-x86_64-DVD-2009.iso /mnt/iso/  ##挂载镜像作为源
  cat << EOF > /etc/yum.repos.d/local.repo
  [base-local]
  name=CentOS7.9-local    ## 操作系统镜像源
  baseurl=file:///mnt/iso
  enabled=1 
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
  [base-2-local]
  name=base
  baseurl=file:///packages/base-packages  ##本次安装所需的base基础rpm包
  enable=1
  gpgcheck=0
  [extra-local]
  name=extra
  baseurl=file:///packages/extra-packages ## extra包
  enable=1
  gpgcheck=0
  [docker-local]
  name=Docker-ce
  baseurl=file:///packages/docker-packages ##docker-ce包
  enable=1 
  gpgcheck=0
  [kubernetes-local]
  name=kubernetes
  baseurl=file:///packages/kubernetes-packages ##kubernetes组件包
  enable=1
  gpgcheck=0
  EOF
  ```

* 操作系统参数优化设置

  ```bash
  cat << EOF > /etc/sysctl.d/k8s.conf
  net.bridge.bridge-nf-call-iptables = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  net.bridge.bridge-nf-call-arptables = 1
  net.ipv4.ip_forward = 1
  net.ipv4.tcp_tw_recycle = 0  ##用来快速回收TIME_WAIT连接，不过如果在NAT环境下会引发问题 tcp_tw_recycle 和 Kubernetes的NAT冲突，必须关闭，否则会导致服务不通或丢包，在4.12之后的内核已移除tcp_tw_recycle内核参数: 
  vm.swappiness = 0    #最大限度使用物理内存，然后才是 swap空间
  vm.overcommit_memory = 1
  vm.panic_on_oom = 0
  fs.inotify.max_user_watches = 89100
  fs.file-max = 52706963
  fs.nr_open = 52706963
  
  
  net.ipv6.conf.all.disable_ipv6 = 1 #默认0
  net.ipv6.conf.default.disable_ipv6 = 1 #默认0 关闭不使用的 IPV6 协议栈，防止触发 docker BUG。
  
  net.netfilter.nf_conntrack_max = 2310720
  net.ipv4.tcp_max_tw_buckets = 5000
  net.ipv4.tcp_syncookies = 1
  net.ipv4.tcp_max_syn_backlog = 1024
  net.ipv4.tcp_synack_retries = 2
  fs.inotify.max_user_watches = 89100
  EOF
  ```

* 验证并重启网络

  ```bash
  sysctl --system
  sysctl -p /etc/sysctl.d/k8s.conf
  systemctl restart network
  ```

* 关闭 Swap，自 1.8 开始，k8s 要求关闭系统 Swap，如果不关闭，kubelet 无法启动

  ```bash
  swapoff -a   ##关闭swap
  sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  ##注释开机swap挂载
  #sed -i '/swap/d' /etc/fstab
  ```

* 关闭防火墙和 SELinux

  ```bash
  systemctl disable firewalld && systemctl stop firewalld  ##关闭防火墙并禁止开机启动
  setenforce 0 && sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config ##关闭selinux并写入配置文件永久关闭
  ```

* 配置时间同步

  ```bash
  yum install chrony -y  ##安装chrony
  systemctl start chronyd  ##启动chronyd服务
  systemctl enable chronyd ##设置开机启动chronyd
  chronyc sources  ##同步时钟源，在本次安装过程中由于无法同步公网时钟源，而将master01作为整个集群的时钟源，其他节点同步master01
  ```

* 禁用postfix

  ```bash
  systemctl stop postfix && systemctl disable postfix ##关闭并禁止开机启动
  ```

* kube-proxy 开启 ipvs 的前置条件

  ```bash
  cat > /etc/sysconfig/modules/ipvs.modules <<EOF
  #!/bin/bash
  modprobe -- ip_vs
  modprobe -- ip_vs_rr
  modprobe -- ip_vs_wrr
  modprobe -- ip_vs_sh
  modprobe -- nf_conntrack_ipv4
  modprobe -- br_netfilter ##高版本内核已经编译进内核功能而不是模块CONFIG_BRIDGE_NETFILTER=y)cat /boot/config-$(uname -r) |grep -C5 BRIDGE
  EOF
  chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
  ```

  

#### 2. 安装docker

* 上传制作好的docker本地源及依赖包（如上local.repo）

* 安装docker

  ```bash
  yum install  docker-ce-19.03.15*
  ```

* docker基础配置

  ```bash
  mkdir /etc/docker ##创建docker配置目录
  cat >> /etc/docker/daemon.json << EOF  ##写入相应的配置,tc-minbao.docker:1985为私有harbor地址
  {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "insecure-registries":["tc-minbao.docker:1985"],
    "log-driver":"json-file",
    "log-opts": {"max-size":"1024m", "max-file":"3"}
  }
  EOF
  ```

* 启动docker

  ```bash
  systemctl start docker  && systemctl enable docker ##启动docker并设置开机启动
  ```

#### 3. 安装kubernetes

* 上传制作好的kubernetes组件本地源和依赖包(如上local.repo)

* 上传制作好的编译修改后的kubeadm二进制包

* 上传kubernetes初始化所需要的image镜像

* 安装kubernetes组件

  ```bash
  yum install  kubelet-1.20* kubeadm-1.20* kubectl-1.20* ipvsadm
  ```

* 启动kubelet

  ```bash
  systemctl enable kubelet  && systemctl start kubelet
  ```

  

#### 4. 初始化kubernetes

* 通过kubeadm初始化集群

  ```bash
  kubeadm init  --kubernetes-version=v1.20.15 \
  --control-plane-endpoint "192.168.135.231:6443" \
  --apiserver-advertise-address "192.168.135.231" \
  --apiserver-bind-port 6443 \
  --upload-certs \
  --image-repository "registry.aliyuncs.com/google_containers" \
  --service-cidr "10.96.0.0/12" \
  --pod-network-cidr "10.244.0.0/16" | tee kube-init.log
  ```

* 上传网络插件CNI镜像及yaml并部署

  ```bash
  kubectl apply -f kube-flannel.yml
  ```

* kube-proxy开起ipvs

  *修改ConfigMap的kube-system/kube-proxy中的config.conf，mode: “ipvs”*

  ```bash
  [root@node-1 k8s]# kubectl edit cm kube-proxy -n kube-system
  ...
      ipvs:
        excludeCIDRs: null
        minSyncPeriod: 0s
        scheduler: ""
        strictARP: false
        syncPeriod: 30s
      kind: KubeProxyConfiguration
      metricsBindAddress: 127.0.0.1:10249
      mode: "ipvs"
      nodePortAddresses: null
      oomScoreAdj: -999
      portRange: ""
      resourceContainer: /kube-proxy
  ...
  
  configmap/kube-proxy edited
  ```

* 重启kube-proxy使ipvs生效

  ```bash
  kubectl get pod -n kube-system | grep kube-proxy |  awk '{"system(kubectl delete pod "$1" -n kube-system")}'
  ```

* 修改master接受调度(可选)

  ```bash
  kubectl taint node --all  node-role.kubernetes.io/master-
  kubectl describe node master01 | grep Taints
  
  Taints:     <none>
  ```

* 规划集群节点调度，taint及label

#### 5.安装apisix

* 上传apisix的helm chart包和镜像

  ```bash
  helm install apisix ./apisix -n minbao
  ```

#### 6.安装nacos

* 上传nacos编排文件和镜像





### 其他



##### 创建密钥

```
kubectl create secret docker-registry registry-secret --docker-server=harbor.xxxx.com  --docker-username=admin --docker-password=xxxx -n {namespace}
```

##### 升级现有集群

```
先升级kubeadm  kubelet kubectl到指定版本,例如从1.14版本升级到1.15版本
yum install  kubeadm-1.15.0  kubelet-1.15.0 kubectl-1.15.0 -y

在每个master节点上升级版本
kubeadm upgrade apply v1.15.0

之后重启kubelet
systemctl daemon-reload && systemctl restart kubelet
```

##### 较低版本etcd访问及查看状态问题

```
   kubectl exec -it etcd-node1 -n kube-system -- etcdctl member list 返回：
    client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:4001: connect: connection refused
    ; error #1: net/http: HTTP/1.x transport connection broken: malformed HTTP response "\x15\x03\x01\x00\x02\x02"

  etcdctl --endpoints=https://192.168.1.201:2379 member list 返回“
     client: etcd cluster is unavailable or misconfigured; error #0: x509: failed to load system roots and no roots provided

   正常需要https和ca,证书,证书key
   kubectl exec -it etcd-node1 -n kube-system -- etcdctl --endpoints=https://127.0.0.1:2379 --ca-file=/etc/kubernetes/pki/etcd/ca.crt --cert-file=/etc/kubernetes/pki/etcd/server.crt --key-file=/etc/kubernetes/pki/etcd/server.key member list
   
   
kubectl exec -it etcd-node1 -n kube-system -- etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key get / --prefix --keys-only > a.key


```

#### 集群突然出现notready问题排查

* 查看kubelet日志

  ```
  journalctl -f -u kubelet
  ```

* 尝试重启网卡

  ```
  systemctl restart network
  ```

  