<!-- TRANSLATED by md-translate -->
# CRD 管理

gateway API 采用 CRDs 构建。 这带来了许多显著的好处，尤其是每次发布的 gateway API 都支持 Kubernetes 最新的 5 个次要版本。 这意味着你可能不需要升级 Kubernetes 集群就能获得最新版本的 API。

本指南旨在回答与 gateway API CRD 管理相关的一些最常见问题。

## 谁来管理 CRD？

归根结底，CRD 是一种具有高度权限的集群范围资源。 这意味着集群管理员或集群 Provider 应负责管理集群中的 CRD。

实际上，这意味着以下任何一种方法都是合理的：

* 集群管理员安装 CRD
* 集群供应工具或 Provider 安装和管理 CRD

有些实施方案可能还希望捆绑 CRD 以简化安装。 只要它们从未捆绑 CRD，这种做法是可以接受的：

1. 覆盖未识别或版本较新的 gateway API CRD。
2. 覆盖具有不同发布渠道的 gateway API CRD。
3. 删除 gateway API CRD。

[Issue #2678](https://github.com/kubernetes-sigs/gateway-api/issues/2678)探讨了实现这一目标的一种可能方法。

## 升级到新版本

gateway API 以两种[发布通道]（/concepts/versioning/#release-channels-eg-experimental-standard）发布 CRD。 坚持使用标准通道 CRD 将确保 CRD 升级更简单、更安全。

### 总体指导方针

1. 避免向后移动。新版本的 CRD 可能会添加新字段和功能。回滚到这些 CRD 的旧版本可能会导致配置丢失。
2. 升级前请阅读发布说明。在某些情况下，它们可能包含一些升级前需要遵循的指导原则。
3. 了解[Gateway API 版本政策](/concepts/versioning)，以便知道哪些内容可以更改。
4. 尽管同时跨多个 gateway API 次版本升级通常是安全的，但最安全且经过最广泛测试的途径还是一次升级一个次版本。

### 验证 webhook

Gateway API 早期版本中包含一个验证 webhook。 从 v1.0 版开始，该 webhook 已被正式弃用，转而使用直接包含在 CRD 中的 CEL 验证。 在 Gateway API v1.1 版中，该 webhook 将被完全删除。 这意味着在升级到较新的 Gateway API 版本时，验证 webhook 不再是一个考虑因素。

###应用程序接口版本删除

注意 这是一个高级用例，目前只适用于自 v0.5.0 版起就在同一集群中使用 gateway API 的用户。

在拥有更新或更稳定 API 版本的 CRD 中，gateway API 发布有可能会移除类似于 v1alpha2 这样的 alpha API 版本。 在标准通道内，一个 API 版本的移除会分散到至少四个次要发布中：

1. 较新的 API 版本被配置为存储版本。
2. 版本已过时（将在发布说明中注明，并在使用已过时的 API 版本时通过过时警告说明）。
3. 版本不再提供服务，但仍包含在 CRD 中，以便在 API 版本之间进行自动转换。
4. 版本不再包含在 CRD 中。

如果你正在使用的 CRD 经历了这个过程（包括存储版本迁移），那么你的一些资源有可能被卡在旧的（已废弃的）存储版本上。 当 CRD 存储版本更新时，只有当使用该 CRD 的单个资源再次保存时才会生效。

例如，如果您使用 Gateway API v0.5.0 CRD 创建了一个 "foo "GatewayClass，那么该 GatewayClass 的存储版本将是 v1alpha2。 如果该 "foo "GatewayClass 从未被修改或更新过，到时您将无法升级到 Gateway API v1.0.0 CRD，因为我们的某个资源仍在使用 v1alpha2 作为存储版本，而该版本已不再包含在 CRD 中（上述步骤 4）。

为了升级，您需要采取一些措施来更新使用旧版本存储的 GatewayClass。 例如，向每个 GatewayClass 发送一个空的 kubectl 补丁就能达到这个效果。幸运的是，有一个工具可以自动帮我们完成这项工作--[kube-storage-version-migrator](https://github.com/kubernetes-sigs/kube-storage-version-migrator) 会自动更新资源，确保它们使用的是最新的存储版本。

### 实验频道

顾名思义，"实验通道 "并不像 "标准通道 "那样提供稳定性保证。 当涉及到次要发布时，"实验通道 "CRD 有可能出现以下任何一种情况：

* 对现有 API 字段或资源进行破坏性更改
* 删除 API 字段或资源，但不提前弃用

如果出现这种情况，我们会在发布说明中明确告知。