<!-- TRANSLATED by md-translate -->
# 安全模式

## 简介

gateway 应用程序接口的设计可实现对典型组织中每个角色的细粒度授权。

## 资源

gateway API 有 3 个主要的 API 资源：

* **GatewayClass** 定义了一组具有共同配置和行为的网关。
* **网关**请求将流量转换到集群内服务的一个点。
* **路由**描述了通过 gateway 来的流量如何映射到服务。

## 角色和人物

gateway API 中有 3 种主要角色，详见 [roles and personas](/concepts/roles-and-personas)：

* **伊恩**（他/他）：基础设施提供商
* **Chihiro**（他们/他们）：集群操作员
* **安娜**（她/他）：应用程序开发人员应用程序开发人员

### RBAC

RBAC（基于角色的访问控制）是用于 Kubernetes 授权的标准。 它允许用户配置谁可以在特定范围内对资源执行操作。 RBAC 可用于启用上文定义的每个角色。 在大多数情况下，希望所有资源都能被大多数角色读取，因此我们将重点关注该模型的写入访问权限。

#### 简单三层模型的写入权限

| | 网关类 | 网关 | 路由 | |-|-|-|-| | 基础设施提供商 | 是 | 是 | 是 | 集群运营商 | 否 | 是 | 是 | 应用开发人员 | 否 | 否 | 是 | 是

#### 高级 4 层模型的写入权限

| | 网关类 | 网关 | 路由 | |-|-||-| | 基础设施提供商 | 是 | 是 | 是 | 集群操作员 | 有时 | 是 | 是 | 应用程序管理员 | 否 | 在指定的命名空间中 | 在指定的命名空间中 | 应用程序开发人员 | 否 | 否 | 在指定的命名空间中 | 在指定的命名空间中

## 跨越 namespace 边界

Providers API 提供了跨越名称空间边界的新方法。 这些跨名称空间的功能相当强大，但需要谨慎使用，以避免意外暴露。 按照规定，每次允许跨越名称空间边界时，我们都需要在名称空间之间进行握手。 这可能有 2 种不同的方式：

#### 1.

路由可以连接到不同名称空间的 gateway。 要做到这一点，Gateway 所有者必须明确允许路由从其他名称空间绑定。 要做到这一点，可以在 Gateway 监听器中配置 allowedRoutes，如下所示：

```yaml
namespaces:
  from: Selector
  selector:
    matchExpressions:
    - key: kubernetes.io/metadata.name
      operator: In
      values:
      - foo
      - bar
```

这将允许来自 "foo "和 "bar "名称空间的路由附加到此 gateway 监听器。

#### 其他标签的风险

虽然可以用这个选择器来使用其他标签，但它并不那么安全。 虽然 `kubernetes.io/metadata.name` 标签在名称空间上会被一致设置为名称空间的名称，但其他标签却没有同样的保证。 如果你被引用了一个自定义标签，如 `env`，那么任何能够在你的集群内给名称空间贴标签的人实际上都能改变你的 gateway 所支持的名称空间集。

### 2.

在某些情况下，我们允许其他对象引用跨越名称空间边界，包括 gateway 引用 secret 和 Routes 引用后端（通常是服务）。 在这些情况下，所需的握手是通过 ReferenceGrant 资源完成的。 该资源存在于目标名称空间中，可用于允许其他名称空间的引用。

例如，下面的 ReferenceGrant 允许从 "prod "名称空间中的 HTTPRoutes 引用到与 ReferenceGrant 部署在同一名称空间中的服务。

```yaml
{% include 'standard/reference-grant.yaml' %}
```

有关 ReferenceGrant 的更多信息，请参阅我们的[该资源的详细文档](/api-types/referencegrant)。

### 高级概念：限制可被引用网关类的 namespace

一些基础设施提供商或集群运营商可能希望限制可以使用 GatewayClass 的 namespace。 目前，我们还没有在 API 中内置这方面的解决方案。 我们建议使用 Open Policy Agent 和 [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) 等策略代理来执行此类策略，以供参考。我们创建了一个 [配置示例](https://github.com/open-policy-agent/gatekeeper-library/pull/24)，可用于此目的。