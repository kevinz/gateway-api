<!-- TRANSLATED by md-translate -->
# 常见问题 (FAQ)

#### 如何参与 gateway API？

社区](/contributing/community)页面记录了如何参与项目。

#### gateway API 是否会取代 ingress API？

没有。Ingress API 自 Kubernetes 1.19 起就是 GA 的。我们没有计划废弃该 API，而且我们预计大多数 Ingress 控制器都将无限期地支持它。

#### ingress 和 gateway API 有哪些区别？

Ingress 主要针对以简单的声明式语法公开 HTTP 应用程序。 Gateway API 公开了更通用的代理 API，可用于更多协议，而不仅仅是 HTTP，并为更多基础架构组件建模，为集群操作员提供更好的部署和管理选项。

更多信息，请参阅 [从 ingress 迁移](/guides/migrating-from-ingress/) 指南。

#### 会有默认的控制器实现吗？

不，已经有很多优秀的 [实现](/implementations)可供选择。 本项目的范围是定义应用程序接口、一致性测试和整体文档。

#### 如何通过 gateway API 公开自定义功能？

有几种机制可用来扩展应用程序接口，使其具有特定的实现功能：

* 通过 [政策附件](/reference/policy-attachment/) 模型，您可以

策略或配置对象可通过名称或显式对象引用来匹配 gateway API 对象。

* 对 gateway API 资源中的字符串字段使用特定于实现的值。
* 万不得已时，可在 gateway API 资源中使用特定于实现的 Annotations
对象上使用特定于实现的注解。
* 使用 API 定义的扩展点。有些 gateway API 对象有明确的

[扩展点](/concepts/api-overview#extension-points)，供实现时被引用。

#### 在哪里可以找到 gateway API 发布？

gateway API 发布是[Github 仓库](https://github.com/kubernetes-sigs/gateway-api) 的标签。[Github 发布](https://github.com/kubernetes-sigs/gateway-api/releases) 页面显示了所有发布。

#### 应该如何考虑 alpha API 版本？

与上游 Kubernetes 类似，alpha API 版本表示资源仍处于试验性质，可能会在 gateway API 的未来发布版本中被移除或以破坏性方式更改。

更多信息，请参阅 [Versioning](/concepts/versioning/) 文档。

#### 支持哪些 Kubernetes 版本？

请参阅我们的 [支持版本] 政策（/concepts/versioning/#supported-version）。

#### 是否支持 SSL 直通？

TLSRoutes](/concepts/api-overview#tlsroute) 支持 SSL Passthrough（网关将使用 [Transport Layer Security (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security) 加密的流量_intact_路由到后端服务，而不是终止它）。有关 Passthrough 和其他 TLS 配置的更多详情，请参阅 [TLS Guide](/guides/tls) 。

#### Gateway API 和 API Gateway 有什么区别？

API Gateway 是一个笼统的概念，它描述了任何暴露后端服务功能的东西，同时为流量路由和操作提供额外的功能，如负载平衡、请求和响应转换，有时还提供更高级的功能，如身份验证和授权、速率限制和断路。

Gateway API 是一个接口或一组资源，用于为 Kubernetes 中的服务网络建模。 其中一个主要资源是 "Gateway"，它声明了要实例化的 Gateway 类型（或类）及其配置。 作为一个 Gateway Provider，您可以实现 Gateway API，以富有表现力、可扩展和面向角色的方式为 Kubernetes 服务网络建模。

大多数 gateway API 实现在某种程度上都是 API Gateway，但并非所有 API Gateway 都是 Gateway API 实现。

#### gateway API 是 API 管理的标准吗？

API 管理的概念要比 gateway API 的目标或 API Gateway 所要提供的内容广泛得多。 API Gateway 可以是 API 管理解决方案的重要组成部分。 Gateway API 可以被视为 API 管理标准化的一种方式。