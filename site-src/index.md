<!-- TRANSLATED by md-translate -->
# 简介

gateway API 是 Kubernetes 的一个官方项目，专注于 Kubernetes 中的 L4 和 L7 路由。 该项目代表了新一代的 Kubernetes ingress、负载平衡和服务网格 API。 从一开始，它就被设计成通用的、有表现力的和面向角色的。

整体资源模型的重点是 3 个独立的[角色]（/概念/角色和角色）以及他们要管理的相应资源：

<!-- Source: https://docs.google.com/presentation/d/11HEYCgFi-aya7FS91JvAfllHiIlvfgcp7qpi_Azjk4E/edit#slide=id.g292839eca6d_1_0 -->

<img src="/images/resource-model.png" alt="Gateway API Resource Model" class="center" />

该 API 中的大部分配置都包含在路由层中。 这些特定于协议的资源（[HTTPRoute](/api-types/httproute)、[GRPCRoute](/api-types/grpcroute)等）可实现 ingress 和 Mesh 的高级路由功能。

gateway API 徽标有助于说明此 API 的双重用途，使南北（ingress）和东西（Mesh）流量的路由选择能够共享相同的配置。

<img src="/images/logo/logo-text-horizontal.png" alt="Gateway API Logo" class="center" />

## 用于 ingress 的 gateway 应用程序接口<a name="for-ingress"></a>

成功 "v0.5.0+标准"。

```
Gateway, GatewayClass, and HTTPRoute have been part of the Standard Channel
of Gateway API since v0.5.0 and are considered stable APIs. For more
information refer to our [versioning guide](/concepts/versioning).
```

使用网关 API 管理 ingress 流量时，[Gateway](/api-types/gateway) 资源定义了一个接入点，流量可在该接入点跨多个上下文进行路由--例如，从集群外部到集群内部（[north/south traffic](/concepts/glossary#northsouth-traffic)）。

每个网关都与一个[GatewayClass]（/api-types/gatewayclass）相关联，它描述了将为网关处理流量的[网关控制器]（/concepts/glossary#gateway-controller）的实际类型；然后，单个路由资源（如[HTTPRoute]（/api-types/httproute））[与网关资源相关联]（/concepts/api-overview#attaching-routes-to-gateways）。 将这些不同的关注点分离成不同的资源，是网关 API 面向角色特性的关键部分，同时也允许在同一集群中使用多种网关控制器（由 GatewayClass 资源表示），每种控制器都有多个实例（由 Gateway 资源表示）。

### 服务网格的 gateway API（[GAMMA 计划](/mesh/gamma)<a name="for-service-mesh"></a>

示例 "V0.8.0+ 版本中的实验"。

```
The [GAMMA initiative](/mesh/gamma) work for supporting service mesh use cases
is _experimental_ in `v0.8.0`+. It is possible that it will change; we do
not recommend it in production at this point.
```

在使用 gateway API 管理[服务网格](/concepts/glossary#service-mesh)时，情况有些不同。 由于集群中通常只有一个网格处于活动状态，因此不会使用[Gateway](/api-types/gateway) 和[GatewayClass](/api-types/gatewayclass) 资源；取而代之的是[直接与服务资源关联](/concepts/gamma#gateway-api-for-mesh)的单个路由资源（如[HTTPRoute](/api-types/httproute)），允许网格管理任何指向该服务的流量，同时保留 Gateway API 面向角色的特性。

迄今为止，[GAMMA](/mesh/gamma) 只需对 gateway API 进行相当小的改动，就能支持网格功能。 不过，对 GAMMA 而言，一个迅速变得至关重要的特殊领域是不同[服务资源面](/concepts/service-facets) 的定义。

Providers 对服务网格的支持是**试验性的**。 我们鼓励使用它并提供反馈，但您***必须做好准备，迎接 GAMMA 应用程序接口的变化。

## 开始

无论你是对使用 gateway API 感兴趣的用户，还是对符合 API 感兴趣的实施者，以下资源都能帮助你了解必要的背景知识：

* [应用程序接口概述](/concepts/api-overview)
* [用户指南](/guides)
* [实施](/implementations)
* [应用程序接口参考规范](/reference/spec)
* [社区链接](/contributing/community)和[开发指南](/contributing/devguide)

## gateway 应用程序接口概念

以下设计目标是 gateway 应用程序接口概念的驱动力，它们展示了 gateway 如何在 ingress 等现有标准的基础上进行改进。

* ** 面向角色** - gateway 由应用程序接口资源组成，这些资源可模拟

被引用和配置 Kubernetes 服务网络的组织角色。

* **便携式** - 这不是一种改进，而是一种

正如 ingress 是一种具有[众多实现](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) 的通用规范一样，gateway API 也被设计成一种可移植的规范，由许多实现来支持。

* **Expressive** - gateway 应用程序接口资源支持各种核心功能

比如基于报头的匹配、流量权重以及其他只有在 ingress 中通过自定义 Annotations 才能实现的功能。

* **可扩展** - gateway 应用程序接口允许在以下位置链接自定义资源

这样就可以在应用程序接口结构的适当位置进行细化定制。

其他一些显著功能包括

* ** 网关类** - 网关类正式确定了负载平衡的类型

这些类能让用户轻松、清晰地了解 Kubernetes 资源模型提供了哪些功能。

* ** 共享 gateway 和跨 namespace 支持** - 它们允许共享

这样，团队（甚至跨 namespace）就可以安全地共享基础设施，而无需直接协调。

* ** 类型化路由和类型化后端** - 网关应用程序接口支持类型化路由

这样，API 就能灵活地支持各种协议（如 HTTP 和 gRPC）和各种后端目标（如 Kubernetes 服务、存储桶或函数）。

* 利用[GAMMA 计划]（/concepts/gamma/）试验性地**服务网格支持** -

gateway API 支持将路由资源与服务资源关联起来，以配置服务网格和 ingress 控制器。

## 面向角色的应用程序接口为何重要？

无论是道路、电力、数据中心还是 Kubernetes 集群，基础设施都是为共享而建的。 然而，共享基础设施提出了一个共同的挑战--如何为基础设施的用户提供灵活性，同时保持基础设施所有者的控制权？

gateway API 通过面向角色的 Kubernetes 服务网络设计实现了这一目标，在分布式灵活性和集中式控制之间取得了平衡。 它允许共享网络基础设施（硬件负载平衡器、云网络、集群托管代理等）被许多不同的非协调团队引用使用，所有这些都受集群操作员设置的策略和约束的限制。

gateway 应用程序接口设计所引用的角色是由三个 "角色 "定义的：

### 个性

* **伊恩**（他/他）是基础设施的提供者。他的职责是
一套基础设施，允许多个孤立的集群
为多个租户提供服务。他不受制于任何一个租户；相反、
他关心的是所有租户的集体利益。
* **Chihiro**（他们/他们）是集群操作员。他们的职责是管理
集群，确保其满足多个用户的需求。
同样，Chihiro 不对集群的任何一个用户负责；他们需要
确保集群为所有用户提供所需的服务。
* **Ana**（她/她）是一名应用程序开发人员。在网关 API 角色中，Ana
在 gateway 应用程序接口角色中处于一个独特的位置：她关注的重点是她的应用程序所要服务的业务需求。
而不是 Kubernetes 或 gateway API。事实上、
Ana 很可能把 gateway API 和 Kubernetes 视为纯粹的摩擦因素
阻碍她完成工作的因素。

(角色和人物](/concepts/roles-and-personas)中对这三者有更详细的论述）。

应该清楚的是，虽然安娜、千寻和伊恩并不一定对所有事情都有一致的看法，但他们需要通力合作，以保证事情的顺利进行。 这就是 gateway API 的核心挑战，一言以蔽之。

#### 使用案例

示例用例](/concepts/use-cases) 展示了这种面向角色模型的工作原理。 它的灵活性使应用程序接口能够适应千差万别的组织模式和实现方式，同时保持可移植的标准应用程序接口。

所介绍的用例都是特意从上述角色的角度出发的。 gateway API 最终是要供人类使用的，这意味着它必须符合安娜、千寻和伊恩各自的用途。

## Gateway API 和 API Gateway 有什么区别？

API Gateway 是一个笼统的概念，它描述了任何暴露后端服务功能的东西，同时为流量路由和操作提供额外的功能，如负载平衡、请求和响应转换，有时还提供更高级的功能，如身份验证和授权、速率限制和断路。

Gateway API 是一个接口或一组资源，用于为 Kubernetes 中的服务网络建模。 其中一个主要资源是 "Gateway"，它声明了要实例化的 Gateway 类型（或类）及其配置。 作为一个 Gateway Provider，您可以实现 Gateway API，以富有表现力、可扩展和面向角色的方式为 Kubernetes 服务网络建模。

大多数 gateway API 实现在某种程度上都是 API Gateway，但并非所有 API Gateway 都是 Gateway API 实现。

## 谁在开发 gateway API？

Gateway API 是一个[SIG-Network](https://github.com/kubernetes/community/tree/master/sig-network)项目，旨在改进 Kubernetes 中的服务网络并使其标准化。请查看[实施参考](implementations.md)，了解支持 Gateway 的最新项目和产品。如果您有兴趣参与或使用 Gateway API 构建实施，请随时[get involved!](/contributing/community)。