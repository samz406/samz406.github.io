---
title: k8s常用术语
date: 2021-07-09 19:10:46
tags: [技术]
categories: K8S
---



Kubernetes 常用术语介绍

<!-- more -->

Kubernetes包含Node、Pod、Replication Controller、Service等资源对象，这些资源对象可通过Kubernetes提供的kubectl工具管理。下面简单介绍下常用术语。



### Container

Container（容器）是一种便携式、轻量级的操作系统级虚拟化技术。它使用 namespace 和Cgroups隔离不同的软件运行环境，并通过镜像自包含软件的运行环境，从而使得容器可以很方便的在任何地方运行。由于容器体积小且启动快，因此可以在每个容器镜像中打包一个应用程序。这种一对一的应用镜像关系拥有很多好处。使用容器，不需要与外部的基础架构环境绑定, 因为每一个应用程序都不需要外部依赖，更不需要与外部的基础架构环境依赖。完美解决了从开发到生产环境的一致性问题。

容器有如下优点：

- 敏捷的应用程序创建和部署: 与虚拟机镜像相比，容器镜像更易用、更高效。

- 持续开发、集成和部署: 提供可靠与频繁的容器镜像构建、部署和快速简便的回滚（镜像是不可变的）。

- 开发与运维的关注分离: 在构建/发布时即创建容器镜像，从而将应用与基础架构分离。

- 开发、测试与生产环境的一致性: 在笔记本电脑上运行和云中一样。

- 可观测：不仅显示操作系统的信息和度量，还显示应用自身的信息和度量。

- 云和操作系统的分发可移植性: 可运行在 Ubuntu, RHEL, CoreOS, 物理机, GKE 以及其他任何地方。

- 以应用为中心的管理: 从传统的硬件上部署操作系统提升到操作系统中部署应用程序。

- 松耦合、分布式、弹性伸缩、微服务: 应用程序被分成更小，更独立的模块，并可以动态管理和部署 - 而不是运行在专用设备上的大型单体程序。

- 实现资源隔离和高效利用。

### Pod

K8s 使用Pod来管理容器，每个 Pod 可以包含一个或多个容器。它们共享 IPC 和 Network namespace，是 Kubernetes 调度的基本单位。Pod 内的多个容器共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。

在 K8s中，所有对象都使用 manifest（yaml 或 json）来定义，比如一个简单的 nginx 服务可以定义为 nginx.yaml，它包含一个镜像为 nginx 的容器



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```



### Node

Node可以是一台物理主机，也可以是一台虚拟机。Node是Kubernetes集群中的工作负载节点。Node可以在运行期间动态增加到Kubernetes集群中，前提是在这个节点上已经正确安装、配置和启动了上述关键进程，在默认情况下kubelet会向Master注册自己，这也是Kubernetes推荐的Node管理方式。一旦Node被纳入集群管理范围，kubelet进程就会定时向Master汇报自身的情报，例如操作系统、Docker版本、机器的CPU和内存情况，以及当前有哪些Pod在运行等，这样Master就可以获知每个Node的资源使用情况，并实现高效均衡的资源调度策略。而某个Node在超过指定时间不上报信息时，会被Master判定为“失联”，Node的状态被标记为不可用（Not Ready）。

每个Node上都运行如下关键进程：

-  **kubelet**：负责Pod对应的容器的创建、启停等任务，同时与Master密切协作，实现集群管理的基本功能。
-  **kube-proxy**：实现Kubernetes Service的通信与负载均衡机制的重要组件。
- Docker Engine（docker）：Docker引擎，负责本机的容器创建和管理工作。



### Master

Kubernetes里的Master指的是集群控制节点，在每个Kubernetes集群里都需要有一个Master来负责整个集群的管理和控制，基本上Kubernetes的所有控制命令都发给它，它负责具体的执行过程，

Master上运行如下关键进程:

- Kubernetes API Server（**kube-apiserver**）：提供了HTTP Rest接口的关键服务进程，是Kubernetes里所有资源的增、删、改、查等操作的唯一入口，也是集群控制的入口进程。
-  Kubernetes Controller Manager（**kube-controller-manage**r）：Kubernetes里所有资源对象的自动化控制中心，可以将其理解为资源对象的“大总管”。
- Kubernetes Scheduler（**kube-scheduler**）：负责资源调度（Pod调度）的进程，相当于公交公司的“调度室”。



### Namespace

​	Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, replication controllers 和 deployments 等都是属于某一个 namespace 的（默认是 **default**），而 node, persistentVolumes 等则不属于任何 namespace

​	

### Label

Label 是识别 Kubernetes 对象的标签，以 key/value 的方式附加到对象上（key 最长不能超过 63 字节，value 可以为空，也可以是不超过 253 字节的字符串），Label 不提供唯一性，并且实际上经常是很多对象（如 Pods）都使用相同的 label 来标志具体的应用。Label可以被附加到各种资源对象上，例如Node、Pod、Service、RC等，一个资源对象可以定义任意数量的Label，同一个Label也可以被添加到任意数量的资源对象上。Label通常在资源对象定义时确定，也可以在对象创建后动态添加或者删除。

Label Selector在Kubernetes中的重要使用场景如下

- kube-controller进程通过在资源对象RC上定义的Label Selector来筛选要监控的Pod副本数量，使Pod副本数量始终符合预期设定的全自动控制流程。◎
- kube-proxy进程通过Service的Label Selector来选择对应的Pod，自动建立每个Service到对应Pod的请求转发路由表，从而实现Service的智能负载均衡机制。
-  通过对Node定义特定的Label，并在Pod定义文件中使用NodeSelector这种标签调度策略，kube-scheduler进程可以实现Pod定向调度的特性。



### Service



Service 是应用服务的抽象，通过 labels 为应用提供负载均衡和服务发现。匹配 labels 的 Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些 endpoints 上。每个 Service 都会自动分配一个 cluster IP（仅在集群内部可访问的虚拟地址）和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后端容器的运行。



### Deployment

  Deployment是Kubernetes在1.2版本中引入的新概念，Deployment在内部使用了Replica Set来实现目的，无论从Deployment的作用与目的、YAML定义，还是从它的具体命令行操作来看，都可以把它看作RC的一次升级，两者的相似度超过90%。Deployment相对于RC的一个最大升级是以随时知道当前Pod“部署”的进度。实际上由于一个Pod的创建、调度、绑定节点及在目标Node上启动对应的容器这一完整过程需要一定的时间，所以我们期待系统启动N个Pod副本的目标状态，实际上是一个连续变化的“部署过程”导致的最终状态。



Deployment应用场景有如下场景：

- 创建一个Deployment对象来生成对应的Replica Set并完成Pod副本的创建。
- 检查Deployment的状态来看部署动作是否完成（Pod副本数量是否达到预期的值）。
-  更新Deployment以创建新的Pod（比如镜像升级）。
-  如果当前Deployment不稳定，则回滚到一个早先的Deployment版本。
- 暂停Deployment以便于一次性修改多个PodTemplateSpec的配置项，之后再恢复Deployment，进行新的发布。
-  扩展Deployment以应对高负载。
- 查看Deployment的状态，以此作为发布是否成功的指标。
- 清理不再需要的旧版本ReplicaSets。



### Replication Controller



它其实定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，所以RC的定义包括如下几个部分。

- Pod期待的副本数量。
-  用于筛选目标Pod的Label Selector。
-  当Pod的副本数量小于预期数量时，用于创建新Pod的Pod模板（template）



定义一个RC并将其提交到Kubernetes集群中后，Master上的Controller Manager组件就得到通知，定期check当前存活的目标Pod，并确保目标Pod实例的数量刚好等于此RC的期望值，如果有过多的Pod副本在运行，系统就会停掉一些Pod，否则系统会再自动创建一些Pod。可以说，通过RC，Kubernetes实现了用户应用集群的高可用性。



> 主要是管理Pod



学习资料

- https://kubernetes.feisky.xyz/

- 《Kubernetes权威指南》







