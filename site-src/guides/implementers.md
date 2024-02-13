<!-- TRANSLATED by md-translate -->
# 实施者指南

关于构建 gateway API 实现，你想知道但又不敢问的一切。

本文档收集了编写 gateway API 实现的技巧和窍门，这些技巧和窍门在相应类型的 godoc 字段中没有直接的位置。

我们还打算在这里写下一些指导原则，帮助应用程序接口的实施者避免犯常见错误。

如果你打算作为最终用户被引用这个 API，而不是构建使用它的东西，那么它可能并不十分重要。

这是一份活的文件，如果您发现有遗漏，欢迎提出修改意见！

## 关于 gateway 应用程序接口的重要事项

但愿其中大多数并不令人吃惊，但它们有时会产生一些不明显的影响，我们将在此尝试加以阐述。

###gateway应用程序接口是`kubernetes.io`应用程序接口

gateway API 使用 `gateway.networking.k8s.io` API group。 这意味着，与核心二进制文件中交付的 API 一样，每次发布时，API 都已通过上游 Kubernetes 审核人员的审核。

#### gateway 应用程序接口被引用到 CRD 中

gateway 应用程序接口以一组 CRD 的形式提供，使用我们的[版本控制政策](/concepts/versioning)进行版本控制。

版本管理策略中最重要的部分是，看似相同的对象（即具有相同的 "组"、"版本 "和 "种类"）可能具有略微不同的模式。 我们以 "兼容 "的方式进行更改，因此一般情况下应该 "能正常工作"，但为了使 "能正常工作 "更加可靠，实现者需要采取一些措施；下文将详细介绍这些措施。

基于 CRD 的交付还意味着，如果一个实施方案在 CRD 尚未安装的情况下尝试使用（即获取、列表、观察等）gateway API 对象，那么你的 Kubernetes 客户端代码很可能会返回严重错误。 下面还将详细介绍处理这种情况的技巧。

Gateway API 对象的 CRD 定义都包含两个特定的 Annotations：

* `gateway.networking.k8s.io/bundle-version:<semver-release-version>`
* `gateway.networking.k8s.io/channel：<channel-name>`

捆绑版本 "和 "通道"（"发布通道 "的简称）的概念在我们的 [versioning](/concepts/versioning) 文档中有解释。

如果集群中安装了模式版本，实施可利用这些版本来确定集群中安装了哪些模式版本。

###对标准通道 CRD 的更改向后兼容

标准通道 CRD 合同的部分内容是，API 版本内的更改必须_兼容_。 请注意，属于实验通道的 CRD 不提供任何向后兼容性保证。

虽然 [gateway API 版本化策略](/concepts/versioning) 在很大程度上与上游的 Kubernetes API 保持一致，但它确实允许 "对验证进行更正"。 例如，如果 API 规范指出某个值无效，但相应的验证并没有涵盖这一点，那么未来的发布版本就有可能添加验证，以防止该无效输入。

这种契约还意味着，在使用比编写时版本更高的 API 时，实现不会失败，因为 Kubernetes 存储的较新模式肯定能够序列化到实现在代码中引用的较旧版本中。

同样，如果一个实现是用较高版本编写的，那么它所理解的较新 Values 将永远不会被使用，因为较旧版本中没有这些值。

## 实施规则和准则

### CRD 管理

有关如何管理 gateway API CRD 的信息，包括何时可以将 CRD 安装与实施捆绑在一起，请参阅我们的[CRD 管理指南](/guides/crd-management)。

### 一致性和版本兼容性

符合要求的 gateway API 实现是指通过一致性测试的实现，这些测试包含在每个 gateway API 捆绑版本的发布中。

在开发过程中，测试可能会被跳过，但要符合要求的版本必须没有被跳过的测试。

根据扩展状态合同，可禁用扩展功能。

gateway 应用程序接口的一致性与版本有关。 如果不做修改，N 版本的一致性通过的实现可能无法通过 N+1 版本的一致性。

实施者应向 gateway API Github 软件仓库提交一致性测试套件报告，其中应包含测试的详细信息。

一致性套件输出包括支持的 gateway API 版本。

#### 版本兼容性

一旦发布 v1.0，对于支持 gateway 和 GatewayClass 的实现，它们必须设置一个新的 Condition，即 "SupportedVersion"，其中 "status: true "表示支持已安装的 CRD 版本，"status: false "表示不支持。

#### 标准状态字段和条件

gateway API 拥有很多资源，但在设计时，我们努力使用 Condition 类型和 `status.conditions` 字段，使不同对象的状态体验尽可能保持一致。

大多数资源都有一个`status.conditions`字段，但有些资源也有一个_包含_`conditions`字段的 namespace 字段。

对于后者，gateway 的 "status.listeners "和 Route 的 "status.parents "字段就是例子，其中片段中的每个项目都标识了与某些配置子集相关的 "条件"。

For the Gateway case, it's to allow Conditions per _Listener_, and in the Route
case, it's to allow Conditions per _implementation_ (since Route objects can
be used in multiple Gateways, and those Gateways can be reconciled by different
implementations).

在所有这些情况中，有一些相对常见的 Condition 类型具有相似的含义：

* 接受"--资源或资源的一部分包含可接受的配置，将

这并不意味着整个配置都是有效的，只是说有足够的有效配置来产生某种效果。

* 已编程"（Programmed）--这是在 "已接受"（Accepted）之后的一个操作阶段、

when the resource or part thereof has been Accepted and programmed into the
underlying dataplane. Users should expect the configuration to be ready for
traffic to flow _at some point in the near future_. This Condition does _not_
say that the dataplane is ready _when it's set_, just that everything is valid
and it _will become ready soon_. "Soon" may have different meanings depending
on the implementation.

* ResolvedRefs`（已解决的引用）--该条件表明资源中的所有引用

如果该条件被设置为 `status: false`，则表示资源或其部分中至少有一个_引用因某种原因无效，`message`字段应指明哪个引用无效。

实施者应查看每种类型的 godoc，以了解这些条件在每种资源或其部分上的具体细节。

此外，上游的 `Conditions` 结构包含一个可选的 `observedGeneration` 字段 - 实现必须使用该字段，并在生成状态时将其设置为对象的 `metadata.generation` 字段。 这样，API 的用户就可以确定状态是否与对象的当前版本相关。

### 资源详情

对于每个当前可用的一致性配置文件，都有一组资源需要实现调和。

下文将介绍 gateway API 的每个对象，并指出预期的行为。

#### GatewayClass

GatewayClass 有一个主要的 `spec` 字段--`controllerName`。每个实现都要声称一个域前缀字符串值（如 `example.com/example-ingress`）作为其`controllerName`。

实现必须监控_all_ GatewayClasses（所有 GatewayClasses），并对具有匹配 "controllerName "的 GatewayClasses 进行对账。 实现必须从具有匹配 "controllerName "的 GatewayClasses 中选择至少一个兼容的 GatewayClass，并通过将每个 GatewayClass 中的 "Accepted "条件设置为 "status: true "来表示接受处理该 GatewayClass。 任何具有匹配 "controllerName "但_not_ Accepted 的 GatewayClasses 必须将 "Accepted "条件设置为 "status: false"。

如果只能调和一个网关类，实施者可以从其他可接受的网关类中只选择一个网关类；如果能调和多个网关类，实施者也可以选择任意多个网关类。

如果 GatewayClass 中的某些内容导致其不兼容（在撰写本文时，唯一可能的原因是有一个指向实现不支持的 `paramsRef` 对象的指针），则实现应将不兼容的 GatewayClass 标记为不 "接受"。

#### Gateway

gateway 对象必须在`spec.gatewayClassName`字段中引用一个已存在并被某个实现`接受'的 gatewayClass，以便该实现对它们进行调节。

如果 gateway 对象退出了调节范围（例如，由于其引用的 gatewayClass 已被删除），那么作为删除过程的一部分，其状态可能会被实现删除，但这并不是必须的。

#### 一般线路信息

所有路由对象都共享某些属性：

* 它们必须连接到范围内的父节点，执行才会考虑

它们是可以调和的。

* 实施必须更新每个范围内路由的状态，并使用

有关详细信息，请参阅具体的路由类型，但通常包括 `Accepted`、`Programmed` 和 `ResolvedRefs` 条件。

* 超出范围的路由不应更新状态，因为它有可能

observedGeneration "字段将表明任何剩余状态都已过时。

#### HTTPRoute

HTTPRoutes 会路由未加密且可供检查的 HTTP 流量，包括在 gateway 终止的 HTTPS 流量（因为这些流量随后会被解密），并允许 HTTPRoute 在路由指令中引用 HTTP 属性，如路径、方法或标头。

#### TLSRoute

TLSRoutes 使用 SNI 标头将加密的 TLS 流量路由到相关后端，而不对流量流进行解密。

#### TCPRoute

TCPRoutes 会将到达监听器的 TCP 流路由到给定的后端之一。

#### UDPRoute

UDPRoutes 会将到达监听器的 UDP 数据包路由到给定的后端之一。

#### ReferenceGrant

ReferenceGrant 是一种特殊资源，被一个命名空间中的资源 Owners 用来_选择性地_允许其他命名空间中的 Gateway API 对象引用。

ReferenceGrant 在与授予引用访问权的事物相同的 namespace 中创建，允许从其他命名空间、其他 Kinds 或两者中访问。

支持跨名称空间引用的实现必须监控 ReferenceGrant，并对指向被范围内 gateway API 对象引用的对象的任何 ReferenceGrant 进行调节。