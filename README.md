# Service Mesh 深入调研
_Made By XinyaoTian_

> “A service mesh is a dedicated infrastructure layer for handling service-to-service communication. “ —— William Morgan

Service Mesh 是一个专注于处理服务间通信的基础设施层。

云原生应用有着复杂的服务拓扑，而 Service Mesh 保证请求可以在这些拓扑中可靠地穿梭。在实际应用当中，Service Mesh 通常是由一系列轻量级的网络代理组成的，它们与应用程序部署在一起，但应用程序不需要知道它们的存在。


## 技术起源

Service Mesh 的概念最早由前 twitter 的基础设施工程师 William Morgan 于 2017 年 4 月 25 日提出。虽然在此之前，微服务领域也有类似的概念被提出或用于开发项目，但在业界始终没有一个统一的名称。 William Morgan 在自己的博文 "What’s a service mesh? And why do I need one?" 中正式给 Service Mesh 做出了权威的定义。至此，"Service Mesh" 这个名词正式出现在各大公司以及技术社区的视野中。


## 出现背景

随着云计算的普及，越来越多的开发者和企业开始使用“微服务”的开发模式。这种开发模式拥有诸如耦合度低、跨语言开发、更小粒度扩容等许多优势，但同样也面临着许多挑战。
微服务从出现至今总共经历了三个阶段：微服务初期、Sidecar 时期和 Service Mesh 时期。

- 微服务初期

    在微服务初期( 2015年前 )开发微服务应用的过程中，我们需要重复性地处理一系列基础工作，比如：服务注册、服务发现、得到服务实例后的负载均衡、熔断机制等。这些工作在 Service Mesh 出现之前统统都要开发人员在项目中用代码解决并实现，导致应用程序中加入了大量的非功能性代码。即使使用类似 Netflix OSS 的库和 Spring Cloud 的框架，开发人员依然面临着需掌握内容多、技术门槛高等诸多困难。

- Sidecar 的出现

    "Sidecar" 这个词，本人使用 Google 搜索引擎同时检索  sidecar 和 microservices 这两个关键字并按时间顺序排序，最早出现的检索结果是在 2014 年 5 月 14 日的这篇演讲中。根据本人查阅的资料显示，"Sidecar" 这个名词最早由 Netflix 提出并被用于 Eureka 项目。由于这个项目的广泛应用，故在此之后，凡是微服务中“用于端对端通信的、被单独分离出来的“组件，就都被称为 "Sidecar" 了。

    仔细分析上述一系列的重复性工作，我们可以发现，这些工作几乎全部集中在处理各个服务间的通信问题。那么，为何我们不把这些工作从业务逻辑中抽离出来，使其专注于服务间通信，并形成单独的组件呢？
Sidecar 模式，即在微服务中将关于服务通讯的功能抽离出来，并作为一个单独的组件运行在微服务中。这种在微服务中独立负责端对端通信的组件，我们称之为 Sidecar 。这种在微服务中将业务逻辑与服务通信解藕，并分离为两个独立运行组件的做法，正是 Service Mesh 概念的雏形。
但在这个阶段，每个微服务中的 Sidecar 还无法通用，即这个微服务的 Sidecar 没有办法拆出来给另一个微服务使用。

- Service Mesh 的提出

    Service Mesh 在 Sidecar 模式的基础上更进一步。Service Mesh 的定义——一个专注于处理服务间通信的基础设施层——站在开发者的角度来讲，就是在每一个微服务中将用于通信的部分从业务中彻底解藕，应用程序甚至不需要知道它们的存在。在 Service Mesh 中，每个微服务至少含有两个组件：一个用于处理业务功能的“应用程序”和一个专职处理服务间通信的“ Sidecar ”( 类似代理 )。
    
    Service Mesh 的愿景是希望开发者再也不需要将精力花费在服务通信上。服务通信由每个微服务的 Sidecar 负责，而 Sidecar 由专门的项目来接管。目前，许多被熟知的项目都可以被我们当作 Sidecar 来运用，比如 Envoy 、 HAProxy 和 Nginx 。

    使用了 Service Mesh 之后，开发团队和运维团队就可以更加明确的划清自己的职责范围——开发团队专注于业务的开发，而运维团队只需关注微服务中的 Sidecar 就可以明确地了解到每个微服务的健康情况和各种指标。

## Service Mesh 的设计理念和作用

随着云原生应用的崛起，Service Mesh 逐渐成为一个独立的基础设施层。在云原生模型里，一个应用可以由数百个服务组成，每个服务可能有数千个实例，而每个实例可能会持续地发生变化。服务间通信不仅异常复杂，而且也是运行时行为的基础。管理好服务间通信对于保证端到端的性能和可靠性来说是非常重要的。

Service Mesh 实际上就是处于 TCP/IP 之上的一个抽象层，它假设底层的 L3/L4 网络能够点对点地传输字节（当然，它也假设网络环境是不可靠的，所以 Service Mesh 也必须具备处理网络故障的能力）。

从某种程度上说，Service Mesh 有点类似 TCP/IP 。TCP 对网络端点间传输字节的机制进行了抽象，而Service Mesh则是对服务节点间请求的路由机制进行了抽象。Service Mesh 不关心消息体是什么，也不关心它们是如何编码的。应用程序的目标是“将某些东西从A传送到B”，而 Service Mesh 所要做的就是实现这个目标，并处理传送过程中可能出现的任何故障。

与TCP不同的是，Service Mesh有着更高的目标：为应用运行时提供统一的、应用层面的可见性和可控性。通过每个微服务中的 Sidecar ，Service Mesh 得以将服务间通信从底层的基础设施中分离出来，让它成为整个生态系统的一等公民——它不再是单纯的基础设施，更可以被监控、托管和控制。

## Service Mesh 的未来

尽管 Service Mesh 在云原生系统方面的应用已经有了快速的增长，但仍然存在巨大的提升空间。服务发现和访问策略在云原生环境中仍显初级，而 Service Mesh 毫无疑问将成为这方面不可或缺的基础。就像 TCP/IP 作为互联网的基础一样，Service Mesh 将在微服务的底层基础设施这条路上更进一步。

## Service Mesh 的发展现状
目前，由 Google , IBM 和 Lyft 公司共同研发的 Istio 非常火爆。Istio 是 Service Mesh 的一个实现，可以看作是一个“微服务管理框架”，一般配合 Kubernetes 使用。Istio 使用 Envoy 作为 Sidecar，并精细地实现了 Service Mesh 对于上述微服务之间传输的诸多设想，并且添加了许多强大的额外功能。


## 参考资料及文献：

1. "What’s a service mesh? And why do I need one?",  William Morgan , 薛命灯 译
2. "Service Mesh：下一代微服务", 敖小剑
3. "What is Istio?", Istio documents
4. Service Mesh 中文网