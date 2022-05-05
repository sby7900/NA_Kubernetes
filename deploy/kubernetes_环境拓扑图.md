---
#### 2. kubernetes集群环境搭建:
+ ##### 2.1 前置知识点
  - 目前生产部署Kubernetes 集群主要有两种方式：
  - **kubeadm**:
    - Kubeadm 是一个K8s 部署工具，提供kubeadm init 和kubeadm join，用于快速部署Kubernetes 集群。
  - **二进制包**:
    - 从github 下载发行版的二进制包，手动部署每个组件，组成Kubernetes 集群。Kubeadm 降低部署门槛，但屏蔽了很多细节，遇到问题很难排查。如果想更容易可控，推荐使用二进制包部署Kubernetes 集群，虽然手动部署麻烦点，期间可以学习很多工作原理，也利于后期维护。

![pics1](../pics/image-20200404094800622.png)

+ ##### 2.2 kubeadm 部署方式介绍
  - kubeadm 是官方社区推出的一个用于快速部署kubernetes 集群的工具，这个工具能通过两条指令完成一个kubernetes 集群的部署：
  - 创建一个Master 节点 `kubeadm init`
  - 将Node 节点加入到当前集群中 `kubeadm join <Master 节点的IP 和端口>`

+ ##### 2.3 安装要求
  - 在开始之前，部署Kubernetes 集群机器需要满足以下几个条件：
   - 一台或多台机器，操作系统CentOS7.x-86_x64
   - 硬件配置：2GB 或更多RAM，2 个CPU 或更多CPU，硬盘30GB 或更多
   - 集群中所有机器之间网络互通
   - 可以访问外网，需要拉取镜像
   - 禁止swap 分区

+ ##### 2.4 最终目标
 - 在所有节点上安装Docker 和kubeadm
 - 部署Kubernetes Master
 - 部署容器网络插件
 - 部署Kubernetes Node，将节点加入Kubernetes 集群中
 - 部署Dashboard Web 页面，可视化查看Kubernetes 资源
 
+ ##### 2.5 准备环境
![pics2](../pics/image-20210609000002940.png)

 - 配置环境介绍:
    - Master节点系统: RockyLinux8.5
    - Node节点系统: RockyLinux8.5
    - Containerd版本:
    - Kubernetes版本: 1.23.5
    - Kubernetes容器版本:
        - kube-apiserver 1.23.5
        - kube-controller-manager 1.23.5
        - kube-proxy 1.23.5
        - kube-scheduler 1.23.5
        - pause 3.5和3.6
        - etcd 3.5.1-0
        - coredns v1.8.6
        - 网络插件:
            + flannel v0.17.0
            + flannel-cni-plugin v1.0.1

| 角色         | IP地址          | 组件                              |
| :-------     | :----------     | :-------------------------------- |
| kube-master  | 192.168.107.152 | containerd，kubectl，kubeadm，kubelet |
| kube-node1   | 192.168.107.153 | containerd，kubectl，kubeadm，kubelet |
| kube-node2   | 192.168.107.154 | containerd，kubectl，kubeadm，kubelet |

 
