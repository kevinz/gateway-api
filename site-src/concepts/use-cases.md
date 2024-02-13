<!-- TRANSLATED by md-translate -->
# 被引用的案例

Gateway API 涵盖了_非常_广泛的使用案例（这既是优点也是缺点！）。 本页强调_并不是要详尽无遗地列出这些使用案例：相反，它旨在提供一些示例，以帮助演示如何使用 API。

在任何情况下，牢记 gateway API 中使用的[角色和人物]（/concepts/roles-and-personas）都是非常重要的。 这里介绍的用例特意以[Ana]（/concepts/roles-and-personas#ana）、[Chihiro]（/concepts/roles-and-personas#chihiro）和[Ian]（/concepts/roles-and-personas#ian）来描述：API 必须对他们可用（同样重要的是要记住，即使这些角色可能由同一个人担任，特别是在较小的组织中，他们都有不同的关注点，我们需要分别考虑）。

## 被引用的案例

* [基本南北用例](#basic-northsouth-use-case)
* 单个
网关后的多个应用程序](#multiple-applications-behind-a-single-gateway)
* [基本东西向用例]（#basic-eastwest-use-case） -- **实验性**
* [网关和网格用例](#gateway-and-mesh-use-case) -- **实验性**

## Basic [north/south](/concepts/glossary#northsouth-traffic) use case

成功 "v0.8.0+ 版本中的标准通道"。

```
The [north/south] use case is fully supported by the Standard Channel
in `v0.8.0+`.
```

Ana 创建了一个微服务应用程序，她希望在 Kubernetes 中运行该应用程序。 她的应用程序将被集群外的客户引用，虽然 Ana 已经创建了应用程序，但设置集群并不是她的专长。

1. 安娜去找千寻，让他们成立一个集群。安娜告诉千寻
她的客户希望她的应用程序接口可以使用根植于
https://ana.application.com/。
2.Chihiro 去找 Ian，请求建立一个集群。
3.Ian 提供了一个运行网关控制器的集群，该控制器具有 [GatewayClass](/api-types/gatewayclass)
资源，名为 "basic-gateway-class"。网关控制器管理与路由来自外部流量的
基础设施。
集群内部流量的相关基础设施。
4.Ian 给 Chihiro 提供了新集群的凭据，并告诉 Chihiro
他们可以使用 gatewayClass `basic-gateway-class` 进行设置。
5.Chihiro 将名为 `ana-gateway` 的 [Gateway](/api-types/gateway) 应用到集群，告诉它
为 TLS 流量侦听 443 端口，并向其提供 TLS 证书
主题 CN 为 `ana.application.com`。它们将该网关与 `basic-gateway-class` GatewayClass 关联。
6.伊恩在第 3 步中配置的网关控制器为 "ana.application.com "分配了一个负载平衡器和一个 IP 地址。
网关控制器分配一个负载平衡器和一个 IP 地址，并提供数据平面
组件，这些组件可以路由到达负载平衡器端口
443 的数据平面组件，并开始监视与`ana-gateway`相关联的路由资源。
ana-gateway "相关的路由资源。
7.Chihiro 获取 "ana-gateway "的 IP 地址，并在集群外为 "ana.application "创建 DNS 记录。
与集群外的 `ana.application.com` 匹配。
8.千寻告诉安娜，她可以使用名为 gateway
`ana-gateway` 的网关。
9.安娜编写并应用 [HTTPRoute](/api-types/httproute) 资源，以配置允许哪些 URL 路径和哪些微服务。
以及哪些微服务应处理它们。她将这些
这些 HTTPRoutes 被引用到 gateway `ana-gateway` 的网关 API [路由
Attachment Process]（/concepts/api-overview#attaching-routes-to-gateway）将这些 HTTPRoutes 与 gateway 关联起来。
10.此时，当请求到达负载平衡器时，它们会被路由
到 Ana 的应用程序。

这使得 Chihiro 可以在 gateway 执行集中式策略[如 TLS](/guides/tls#downstream-tls)，同时允许 Ana 和她的同事控制应用程序的[路由逻辑](/guides/http-routing)和推出计划（如[流量分割推出](/guides/traffic-splitting)）。

## 单个 gateway 后面有多个应用程序

成功 "v0.8.0+ 版本中的标准通道"。

```
The [north/south] use case is fully supported by the Standard Channel
in `v0.8.0+`.
```

这与【基本南北用例】(#basic-northsouth-use-case) 非常相似，但有多个应用程序团队：Ana 和她的团队在 `store` 名称空间中管理一个店面应用程序，而 Allison 和她的团队在 `site` 名称空间中管理一个网站。

* Ian 和 Chihiro 合作提供集群、`GatewayClass`和
gateway"。
* Ana 和 Allison 独立部署工作负载和 HTTPRoutes，并将其绑定到同一`Gateway`资源。
gateway` 资源的工作负载和 HTTPRoutes。

同样，这种关注点的分离允许 Chihiro 在 gateway 上执行集中策略[如 TLS](/guides/tls#downstream-tls)。 与此同时，Ana 和 Allison 在各自的 namespace 中运行自己的应用程序](/guides/multiple-ns)，但将他们的 Routes 附加到同一个共享 gateway 上，这样他们就可以独立控制自己的[路由逻辑](/guides/http-routing)、[流量分割展开](/guides/traffic-splitting)等，而不用担心 Chihiro 和 Ian 正在处理的事情。

![gateway API 角色](/images/gateway-roles.png)

## Basic [east/west](/concepts/glossary#eastwest-traffic) use case

危险 "v0.8.0 中的实验"。

```
The [GAMMA initiative][gamma] work for supporting service mesh use cases
is _experimental_ in `v0.8.0`. It is possible that it will change; we do
not recommend it in production at this point.
```

在这种情况下，Ana 建立了一个工作负载，该负载已经在一个带有符合 [GAMMA](/concepts/gamma/) 标准的 [service mesh](/concepts/glossary#service-mesh) 的集群中运行。 她希望使用 mesh 来保护她的工作负载，具体做法是拒绝对她的工作负载调用不正确的 URL 路径，并在任何人对她的工作负载提出请求时强制执行超时。

* 千寻和伊恩已经提供了一个具有运行服务网格的集群。
安娜不需要向他们提出任何请求。
* Ana 编写了一个 HTTPRoute，该 HTTPRoute 定义了可接受的路由和超时，其
父路 由 "是她的工作负载的服务。
* Ana 在与其工作负载相同的 namespace 中应用 HTTPRoute。
* 网格会自动开始执行由
描述的路由策略。

在这种情况下，角色间的关注点分离使 Ana 能够利用服务网格，通过自定义路由逻辑，在向 Chihiro 或 Ian 发出请求时不会出现任何瓶颈。

## ＃＃网关和网格被引用的情况

危险 "v0.8.0 中的实验"。

```
The [GAMMA initiative][gamma] work for supporting service mesh use cases
is _experimental_ in `v0.8.0`. It is possible that it will change; we do
not recommend it in production at this point.
```

这实际上是 [单个 gateway 后面的多个应用程序](#multiple-applications-behind-a-single-gateway) 和 [basic east/west](#basic-eastwest-use-case) 用例的组合：

* Chihiro 和 Ian 将配置一个集群、一个 [GatewayClass](/api-types/gatewayclass) 和一个 [Gateway](/api-types/gateway)。
* Ana 和 Allison 将在相应的
namespace 中部署应用程序。
* 然后，Ana 和 Allison 将酌情应用 HTTPRoute 资源。

不过，由于涉及到网格，这种情况下有两个非常重要的变化：

1. 如果 Chihiro 部署的 [gateway controller]（/concepts/glossary#gateway-controller）默认为 [Service
路由](/concepts/glossary#service-routing)，他们可能需要重新配置为[端点路由](/concepts/glossary#endpoint-routing)。
(这是[GAMMA](/concepts/gamma/)正在进行的一项工作，但预计
端点路由将被推荐）。
2.Ana 和/或 Allison 需要将 HTTPRoutes 与各自的
工作负载的服务绑定 HTTPRoutes，以配置网状路由逻辑。这些可以是
不同的 HTTPRoutes，或者他们可以应用单一的
HTTPRoutes 同时绑定到 gateway 和服务。

一如既往，以这种方式分离关注点的最终目的是允许 Chihiro 在 gateway 执行集中式策略[如 TLS](/guides/tls#downstream-tls) ，同时允许 Ana 和 Allison 保留对[路由逻辑](/guides/http-routing)、[流量分割展开](/guides/traffic-splitting)等的独立控制，既可用于[北/南](/concepts/glossary#northsouth-traffic)，也可用于[东/西](/concepts/glossary#eastwest-traffic)路由。