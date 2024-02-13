<!-- TRANSLATED by md-translate -->
# 应用程序接口设计指南

本应用程序接口通篇被引用了一些通用设计准则。

注意 在整个 gateway API 文档和规范中，"MUST"、"MAY "和 "SHOULD "等关键字被广泛引用。 这些关键字应按照 RFC 2119 中的描述来解释。

## 单一资源一致性

Kubernetes API 仅在单个资源级别上保证一致性。 相对于单个资源，复杂的资源图会产生一些后果：

* 对跨越多个资源的属性进行错误检查将是异步的，并最终保持一致。简单的语法检查可以在单个资源级别进行，但跨资源的依赖关系需要由控制器处理。
* 控制器需要处理资源之间的断开链接和/或不匹配的配置。

## 冲突

独立行动者（如集群运行人员和应用程序开发人员之间）之间的责任分离和授权可能会导致配置冲突。 例如，两个应用程序团队可能会无意中为相同的 HTTP 路径提交配置。

在大多数情况下，在为可能存在冲突的字段提供文件的同时，也提供了解决冲突的指 导。 如果冲突没有规定的解决方案，则应适用以下指导原则：

* 最好不要破坏正在运行的设备。
* 尽可能减少流量。
* 发生冲突时，提供一致的体验。
* 在发现冲突时，明确选择哪条路径。在可能的情况下，应通过在相关资源上设置适当的状态条件来传达这一点。
* 更具体的匹配应优先于不太具体的匹配。
* 创建时间戳最早的资源胜出。
* 如果其他条件相同（包括创建时间戳），则应优先考虑按字母顺序（namespace/name）排在前面的资源。例如，foo/bar 优先于 foo/baz。

## 优雅地处理未来的应用程序接口版本

在实施此 API 时，一个重要的考虑因素是它将来可能会如何变化。 与之前的 ingress API 类似，此 API 的设计目的是由同一集群中的各种不同产品来实施。 这意味着您的实施方案所开发的 API 版本可能与它被引用的 API 版本不同。

至少必须满足以下要求，以确保未来版本的 API 不会破坏您的实现：

* 处理验证松动的字段而不会崩溃
* 处理从必填项过渡到可选项的字段而不会崩溃

### 支持的应用程序接口版本

可通过查看每个 CRD 上的 `gateway.networking.k8s.io/bundle-version` 注解来确定集群中安装的 Gateway API CRD 的版本。 每个实现必须将其与它所识别和支持的版本列表进行比较。 具有 GatewayClass 的实现必须在 GatewayClass 上发布 `SupportedVersion` 条件，以表明集群中安装的 CRD 是否受支持。

## CRD 和 Webhook 验证的局限性

备注 "webhook 验证已过时"？

```
Webhook validation in Gateway API has been deprecated and will be fully
removed in v1.1.0. With that said, all of this guidance will still apply for
implementations as long as they support v1.0.0 or older releases of the API.
```

CRD 和 webhook 验证不是最终的验证，即 webhook 是 "不错的用户体验"，但不是模式执行。 这种验证的目的是在用户提供无效配置时立即向用户提供反馈。 编写代码时要有防御性，假设至少有一些无效输入（gateway API 资源）会到达你的控制器。 Webhook 和 CRD 验证都不是完全可靠的，因为它：

* 可能无法正确部署。
* 可能会在未来的 API 发布中被放宽。(在较新版本的 API 中，字段可能包含限制性验证较松的值）。

注意：这些限制并非 gateway API 所独有，它们更广泛地适用于任何 Kubernetes CRD 和 webhook。

实现者应确保，即使在 API 中遇到意外值，他们的实现仍应尽可能安全，并从容应对这种输入。 最常见的应对方法是将配置视为畸形而拒绝，并通过状态块中的条件向用户发出信号。 为避免重复工作，gateway API 维护者正在考虑添加一个共享验证包，供实现者用于此目的。这已被 [#926](https://github.com/kubernetes-sigs/gateway-api/issues/926) 所引用。

### 期望

我们预计，在此 API 的早期阶段，不同 Provider 之间的一致性会有不同程度的差异。 用户可以通过一致性测试的结果来了解哪些方面的行为可能与规范存在差异。

### 具体实施

在 API 的某些方面，我们允许用户指定功能的用法，但具体行为可能取决于相应的实现。 例如，正则表达式匹配存在于所有实现中，但由于所使用的相应库（如 PCRE、ECMA、Re2）之间存在细微差别，因此不可能指定具体的行为。 对于我们的用户来说，尽可能指定功能的用法仍然很有用，但我们承认 API 某些子集的行为可能仍然存在差异（这也没关系）。

这些情况将作为应用程序接口中 "特定于实现 "的部分进行定义。

## 种类与资源

与其他 Kubernetes API 类似，gateway API 在整个 API 的对象引用中使用了 "Kind"（种类）而不是 "Resource"（资源）。 大多数 Kubernetes 用户应该都熟悉这种模式。

根据[Kubernetes API 公约](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)，这意味着该 API 的所有实现都应该在种类和资源之间有一个预定义的映射。依赖动态资源映射是不安全的。

## 应用程序接口约定

gateway API 遵循 Kubernetes API [约定](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md)。这些约定旨在简化客户端开发，并确保配置机制可以在各种不同的用例中一致实施。 除了 Kubernetes API 约定之外，Gateway API 还有以下约定：

### 列表名称

本项目使用的另一个惯例是 CRD 中列表的复数字段名称。 我们使用以下规则：

* 如果字段名称是名词，请引用复数值。
* 如果字段名称是动词，则被引用为单数。
