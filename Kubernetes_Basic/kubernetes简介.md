---
#### 1.1应用部署方式演变
- 在部署应用程序的方式上，主要经历了三个时代：
 - **传统部署**：互联网早期，会直接将应用程序部署在物理机上
   - 优点：简单，不需要其它技术的参与
   - 缺点：不能为应用程序定义资源使用边界，很难合理地分配计算资源，而且程序之间容易产生影响

 - **虚拟化部署**：可以在一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境
   - 优点：程序环境不会相互产生影响，提供了一定程度的安全性
   - 缺点：增加了操作系统，浪费了部分资源

 - **容器化部署**：物理机作为容器的载体，只负责容器的调度
   - 优点：可以保证每个容器拥有自己的文件系统、CPU、内存、进程空间等运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署

![pic1](../pics/image-20200505183738289.png)

+ 容器化部署方式给带来很多的便利，但是也会出现一些问题，比如说：
  - 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
  - 当并发访问量变大的时候，怎么样做到横向扩展容器数量
+ 这些容器管理的问题统称为**容器编排**问题，为了解决这些容器编排问题，就产生了一些容器编排的软件：
  - **Swarm**：Docker自己的容器编排工具
  - **Mesos**：Apache的一个资源统一管控的工具，需要和Marathon结合使用
  - **Kubernetes**：Google开源的的容器编排工具

![pic2](../pics/image-20200524150339551.png)

#### 1.2 kubernetes简介

![pic3](../pics/image-20200406232838722.png)
+ kubernetes，是一个全新的基于容器技术的分布式架构领先方案，是谷歌严格保密十几年的秘密武器----Borg系统的一个开源版本，于2014年9月发布第一个版本，2015年7月发布第一个正式版本。
+ kubernetes的本质是**一组服务器集群**，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化，主要提供了如下的主要功能：
  - **自我修复**：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
  - **弹性伸缩**：可以根据需要，自动对集群中正在运行的容器数量进行调整
  - **服务发现**：服务可以通过自动发现的形式找到它所依赖的服务
  - **负载均衡**：如果一个服务起动了多个容器，能够自动实现请求的负载均衡
  - **版本回退**：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
  - **存储编排**：可以根据容器自身的需求自动创建存储卷

####1.3 kubernetes组件

+ 一个kubernetes集群主要是由**控制节点(master)**、**工作节点(node)**构成，每个节点上都会安装不同的组件。
  - **master：集群的控制平面，负责集群的决策 ( 管理 )**
    - **ApiServer** : 资源操作的唯一入口，接收用户输入的命令，提供认证、授权、API注册和发现等机制
    - **Scheduler** : 负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上
    - **ControllerManager** : 负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等
    - **Etcd** ：负责存储集群中各种资源对象的信息
  - **node：集群的数据平面，负责为容器提供运行环境 ( 干活 )**
    - **Kubelet** : 负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器
    - **KubeProxy** : 负责提供集群内部的服务发现和负载均衡
    - **Docker** : 负责节点上容器的各种操作

![pic3](../pics/image-20200406184656917.png)
+ 下面，以部署一个nginx服务来说明kubernetes系统各个组件调用关系：
 - 1.首先要明确，一旦kubernetes环境启动之后，master和node都会将自身的信息存储到etcd数据库中
 - 2.一个nginx服务的安装请求会首先被发送到master节点的apiServer组件
 - 3.apiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上在此时，它会从etcd中读取各个node节点的信息，然后按照一定的算法进行选择，并将结果告知apiServer
 - 4.apiServer调用controller-manager去调度Node节点安装nginx服务
 - 5.kubelet接收到指令后，会通知docker，然后由docker来启动一个nginx的pod;pod是kubernetes的最小操作单元，容器必须跑在pod中至此，

 - 6.一个nginx服务就运行了，如果需要访问nginx，就需要通过kube-proxy来对pod产生访问的代理这样，外界用户就可以访问集群中的nginx服务

#### 1.4 kubernetes概念
+ **Master**：集群控制节点，每个集群需要至少一个master节点负责集群的管控
+ **Node**：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行
+ **Pod**：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器
+ **Controller**：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等
+ **Service**：pod对外服务的统一入口，下面可以维护者同一类的多个pod
+ **Label**：标签，用于对pod进行分类，同一类pod会拥有相同的标签
+ **NameSpace**：命名空间，用来隔离pod的运行环境

#### 1.5 资源管理介绍
+ 在kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes。
  - kubernetes的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在kubernetes集群中运行一个个的容器，并将指定的程序跑在容器中。
  - kubernetes的最小管理单元是pod而不是容器，所以只能将容器放在`Pod`中，而kubernetes一般也不会直接管理Pod，而是通过`Pod控制器`来管理Pod的。
  - Pod可以提供服务之后，就要考虑如何访问Pod中服务，kubernetes提供了`Service`资源实现这个功能。
  - 当然，如果Pod中程序的数据需要持久化，kubernetes还提供了各种`存储`系统。

![pic4](../pics/image-20200406225334627.png)
+ 学习kubernetes的核心，就是学习如何对集群上的`Pod、Pod控制器、Service、存储`等各种资源进行操作

#### 1.6 资源管理方式
- 命令式对象管理：直接使用命令去操作kubernetes资源

```
kubectl run nginx-pod --image=nginx:1.17.1 --port=80
```

- 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源

```
kubectl create/patch -f nginx-pod.yaml
```

- 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源

```
kubectl apply -f nginx-pod.yaml
```

| 类型           | 操作对象 | 适用环境 | 优点           | 缺点                             |
| :------------- | :------- | :------- | :------------- | :------------------------------- |
| 命令式对象管理 | 对象     | 测试     | 简单           | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 | 文件     | 开发     | 可以审计、跟踪 | 项目大时，配置文件多，操作麻烦   |
| 声明式对象配置 | 目录     | 开发     | 支持目录操作   | 意外情况下难以调试               |

##### 1.6.1 命令式对象管理
- **kubectl命令**
  - kubectl是kubernetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署。kubectl命令的语法如下：

```
kubectl [command] [type] [name] [flags]
```
  - **comand**：指定要对资源执行的操作，例如create、get、delete
  - **type**：指定资源类型，比如deployment、pod、service
  - **name**：指定资源的名称，名称大小写敏感
  - **flags**：指定额外的可选参数

```
# 查看所有pod
kubectl get pod

# 查看某个pod
kubectl get pod pod_name

# 查看某个pod,以yaml格式展示结果
kubectl get pod pod_name -o yaml
```

- **资源类型**
  - kubernetes中所有的内容都抽象为资源，可以通过下面的命令进行查看:

```
kubectl api-resources
```
  - 经常使用的资源有下面这些：

| 资源分类      | 资源名称                 | 缩写    | 资源作用        |
| :------------ | :----------------------- | :------ | :-------------- |
| 集群级别资源  | nodes                    | no      | 集群组成部分    |
| namespaces    | ns                       | 隔离Pod |                 |
| pod资源       | pods                     | po      | 装载容器        |
| pod资源控制器 | replicationcontrollers   | rc      | 控制pod资源     |
|               | replicasets              | rs      | 控制pod资源     |
|               | deployments              | deploy  | 控制pod资源     |
|               | daemonsets               | ds      | 控制pod资源     |
|               | jobs                     |         | 控制pod资源     |
|               | cronjobs                 | cj      | 控制pod资源     |
|               | horizontalpodautoscalers | hpa     | 控制pod资源     |
|               | statefulsets             | sts     | 控制pod资源     |
| 服务发现资源  | services                 | svc     | 统一pod对外接口 |
|               | ingress                  | ing     | 统一pod对外接口 |
| 存储资源      | volumeattachments        |         | 存储            |
|               | persistentvolumes        | pv      | 存储            |
|               | persistentvolumeclaims   | pvc     | 存储            |
| 配置资源      | configmaps               | cm      | 配置            |
|               | secrets                  |         | 配置            |

- **操作**
  - kubernetes允许对资源进行多种操作，可以通过--help查看详细的操作命令
  
  ```
  kubectl --help
  ```
  - 经常使用的操作有下面这些：

| 命令分类   | 命令         | 翻译                        | 命令作用                     |
| :--------- | :----------- | :-------------------------- | :--------------------------- |
| 基本命令   | create       | 创建                        | 创建一个资源                 |
|            | edit         | 编辑                        | 编辑一个资源                 |
|            | get          | 获取                        | 获取一个资源                 |
|            | patch        | 更新                        | 更新一个资源                 |
|            | delete       | 删除                        | 删除一个资源                 |
|            | explain      | 解释                        | 展示资源文档                 |
| 运行和调试 | run          | 运行                        | 在集群中运行一个指定的镜像   |
|            | expose       | 暴露                        | 暴露资源为Service            |
|            | describe     | 描述                        | 显示资源内部信息             |
|            | logs         | 日志输出容器在 pod 中的日志 | 输出容器在 pod 中的日志      |
|            | attach       | 缠绕进入运行中的容器        | 进入运行中的容器             |
|            | exec         | 执行容器中的一个命令        | 执行容器中的一个命令         |
|            | cp           | 复制                        | 在Pod内外复制文件            |
|            | rollout      | 首次展示                    | 管理资源的发布               |
|            | scale        | 规模                        | 扩(缩)容Pod的数量            |
|            | autoscale    | 自动调整                    | 自动调整Pod的数量            |
| 高级命令   | apply        | rc                          | 通过文件对资源进行配置       |
|            | label        | 标签                        | 更新资源上的标签             |
| 其他命令   | cluster-info | 集群信息                    | 显示集群信息                 |
|            | version      | 版本                        | 显示当前Server和Client的版本 |

  - 下面以一个namespace / pod的创建和删除简单演示下命令的使用：
  
  ```
  # 创建一个namespace
  [root@master ~]# kubectl create namespace dev
  namespace/dev created

  # 获取namespace
  [root@master ~]# kubectl get ns
  NAME              STATUS   AGE
  default           Active   21h
  dev               Active   21s
  kube-node-lease   Active   21h
  kube-public       Active   21h
  kube-system       Active   21h

  # 在此namespace下创建并运行一个nginx的Pod
  [root@master ~]# kubectl run pod --image=nginx:latest -n dev
  kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use      kubectl run --generator=run-pod/v1 or kubectl create instead.
  deployment.apps/pod created

  # 查看新创建的pod
  [root@master ~]# kubectl get pod -n dev
  NAME  READY   STATUS    RESTARTS   AGE
  pod   1/1     Running   0          21s

  # 删除指定的pod
  [root@master ~]# kubectl delete pod pod-864f9875b9-pcw7x
  pod "pod" deleted

  # 删除指定的namespace
  [root@master ~]# kubectl delete ns dev
  namespace "dev" deleted
  ```

##### 1.6.2 命令式对象配置
- 命令式对象配置就是使用命令配合配置文件一起来操作kubernetes资源。
  - 1.创建一个nginxpod.yaml，内容如下：

``` 
apiVersion: v1
kind: Namespace
metadata:
  name: dev

\*---\*

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx:latest
```
  
  - 2.执行create命令，创建资源：

```
[root@master ~]# kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created
#此时发现创建了两个资源对象，分别是namespace和pod
```
  - 3.执行get命令，查看资源：

```
[root@master ~]#  kubectl get -f nginxpod.yaml
NAME            STATUS   AGE
namespace/dev   Active   18s

NAME            READY   STATUS    RESTARTS   AGE
pod/nginxpod    1/1     Running   0          17s
#这样就显示了两个资源对象的信息
```

  - 4.执行delete命令，删除资源：
```
[root@master ~]# kubectl delete -f nginxpod.yaml
namespace "dev" deleted
pod "nginxpod" deleted
#此时发现两个资源对象被删除了
```

> **总结: **
>> 命令式对象配置的方式操作资源，可以简单的认为：命令  +  yaml配置文件（里面是命令需要的各种参数）

##### 1.6.3 声明式对象配置
- 声明式对象配置跟命令式对象配置很相似，但是它只有一个命令apply。

```
# 首先执行一次kubectl apply -f yaml文件，发现创建了资源
[root@master ~]#  kubectl apply -f nginxpod.yaml
namespace/dev created
pod/nginxpod created

# 再次执行一次kubectl apply -f yaml文件，发现说资源没有变动
[root@master ~]#  kubectl apply -f nginxpod.yaml
namespace/dev unchanged
pod/nginxpod unchanged
```

> **总结**:
>>     其实声明式对象配置就是使用apply描述一个资源最终的状态（在yaml中定义状态）
    使用apply操作资源：
        如果资源不存在，就创建，相当于 kubectl create
        如果资源已存在，就更新，相当于 kubectl patch


> **扩展**
>> kubectl的运行是需要进行配置的，它的配置文件是$HOME/.kube，如果想要在node节点运行此命令，需要将master上的.kube文件复制到node节点上，即在master节点上执行下面操作：

```
scp  -r  HOME/.kube   node1: HOME/
```

> 使用推荐: 三种方式应该怎么用?
>> 创建/更新资源 使用声明式对象配置 kubectl apply -f XXX.yaml

>> 删除资源 使用命令式对象配置 kubectl delete -f XXX.yaml

>> 查询资源 使用命令式对象管理 kubectl get(describe) 资源名称

