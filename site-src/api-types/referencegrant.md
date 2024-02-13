<!-- TRANSLATED by md-translate -->
# 参考资料

成功 "V0.6.0+ 版本中的标准通道"。

```
The `ReferenceGrant` resource is Beta and part of the Standard Channel in `v0.6.0+`.
```

本资源原名为 "ReferencePolicy"，后更名为 "ReferenceGrant"，以避免与政策附件混淆。

引用授权（ReferenceGrant）可用于在 gateway API 中实现跨名称空间引用，特别是路由（Routes）可将流量转发到其他名称空间的后端（backend），或 gateway 可引用其他名称空间的 Secret。

参考补助金](/镜像/参考补助金-simple.svg)

<!-- Source: https://docs.google.com/presentation/d/11HEYCgFi-aya7FS91JvAfllHiIlvfgcp7qpi_Azjk4E/edit#slide=id.g13c18e3a7ab_0_171 -->

在过去，我们已经看到，跨名称空间边界转发流量是一种理想的功能，但如果没有像 ReferenceGrant 这样的保障措施，[漏洞](https://github.com/kubernetes/kubernetes/issues/103675) 就会出现。

如果一个对象在其名称空间之外被引用，该对象的 Owners 必须创建一个 ReferenceGrant 资源，以明确允许这种引用。 没有 ReferenceGrant，跨名称空间的引用是无效的。

## 结构

从根本上说，ReferenceGrant 由两个列表组成，一个是可能来自引用的资源列表，另一个是可能被引用的资源列表。

来自 "列表允许您指定可引用 "至 "列表中描述的项目的资源组、种类和名称空间。

到 "列表允许您指定可被 "从 "列表中描述的项目引用的资源组和资源类型。 在 "到 "列表中不需要名称空间，因为引用授权只能被用来允许引用与引用授权在同一名称空间中的资源。

## 示例

下面的示例展示了名称空间 `foo` 中的 HTTPRoute 如何引用名称空间 `bar` 中的服务。 在这个示例中，名称空间 `bar` 中的 ReferenceGrant 明确允许从名称空间 `foo` 中的 HTTPRoute 引用服务。

```yaml
kind: HTTPRoute
metadata:
  name: foo
  namespace: foo
spec:
  rules:
  - matches:
    - path: /bar
    backendRefs:
      - name: bar
        namespace: bar
---
kind: ReferenceGrant
metadata:
  name: bar
  namespace: bar
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: HTTPRoute
    namespace: foo
  to:
  - group: ""
    kind: Service
```

## 应用程序接口设计决策

虽然应用程序接口在本质上很简单，但也有一些值得注意的决定：

1. 每个 ReferenceGrant 只支持一个 "发件人 "和 "收件人 "部分。额外的信任关系必须使用额外的 ReferenceGrant 资源来建模。
2. 资源名称被有意排除在 ReferenceGrant 的 "发件人 "部分之外，因为它们很少提供有意义的保护。能够在 namespace 中写入某类资源的用户可以随时重命名资源或更改资源结构，以便与给定的授权相匹配。
3. 每个 "From "结构只允许使用一个命名空间。虽然选择器的功能会更强大，但它会助长不必要的不安全配置。
4. 这些资源的效果纯粹是相加的，它们相互叠加。这样，它们就不可能相互冲突。

请参阅 [API 规范](/reference/spec#gateway.networking.k8s.io/v1alpha2.ReferenceGrant)，了解如何解释特定 ReferenceGrant 字段的更多详情。

## 实施指南

该应用程序接口依赖于运行时验证。 实现必须注意这些资源的更改，并在每次更改或删除后重新计算跨 namespace 引用的有效性。

在通报跨名称空间引用的状态时，除非存在允许进行引用的 ReferenceGrant，否则实现不 得公开另一名称空间中是否存在资源的信息。 这意味着，如果在没有 ReferenceGrant 的情况下对不存在的资源进行了跨名称空间引用，则任何状态条件或警告信息都必须重点说明不存在允许进行引用的 ReferenceGrant 这一事实。 不应提供有关被引用资源是否存在的提示。

## 例外情况

跨 namespace Route -&gt; Gateway 绑定采用的模式略有不同，握手机制内置于 Gateway 资源中。 有关该方法的更多信息，请参阅相关的[安全模型文档](/concepts/security-model)。 虽然在概念上与 ReferenceGrant 相似，但该配置直接内置于 Gateway 监听器中，并允许对每个监听器进行细粒度配置，而这在 ReferenceGrant 中是不可能实现的。

在某些情况下，忽略 ReferenceGrant 而采用其他安全机制也是可以接受的。 只有在其他机制（如 NetworkPolicy）能有效限制实现跨 namespace 引用的情况下，才可以这样做。

选择作为例外的实现必须明确记录其实现不遵守 ReferenceGrant，并详细说明有哪些替代保障措施。 请注意，这不大可能适用于 API 的入口实现，也不会适用于所有网格实现。

有关跨名称空间引用所涉及风险的示例，请参阅 [CVE-2021-25740](https://github.com/kubernetes/kubernetes/issues/103675)。该 API 的实现需要非常小心，以避免混淆的副手攻击。 ReferenceGrant 为此提供了一种保障措施。只有绝对确定已采取其他同样有效的保障措施的实现才必须例外。

## 一致性级别

对于源自以下对象的跨名称空间引用，ReferenceGrant 支持是一项 "核心 "一致性级别要求：

* gateway
* GRPCRoute
* HTTP 路由
* TLS 路由
* TCPRoute
* UDPRoute

也就是说，除了上文 "例外情况 "部分所述的情况外，所有实现都必须在 gateway 和任何核心 xRoute 类型的跨名称空间引用中使用该流程。

其他 "特定实现 "对象和引用也必须使用此流程进行跨名称空间引用，上文 "例外 "部分所述情况除外。

## 未来 API 组的潜在变化

在 gateway API 和 SIG 网络用例之外，ReferenceGrant 也开始受到关注。 这一资源有可能转移到一个更中立的位置。 ReferenceGrant API 的用户可能需要在未来某个时间过渡到不同的 API 组（而非 "gateway.networking.k8s.io"）。