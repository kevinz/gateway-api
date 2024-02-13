<!-- TRANSLATED by md-translate -->
# 跨 namespace 路由

gateway API 的核心功能是支持跨 namespace 路由，这在多个用户或团队共享相应的网络基础设施时非常有用，但必须对控制和配置进行分段，以尽量减少访问和故障域。

网关和路由可以部署到不同的命名空间，路由可以跨命名空间连接到网关。 这样，用户访问控制就可以跨命名空间以不同方式应用于路由和网关，从而有效地将访问和控制细分到集群路由配置的不同部分。 路由跨命名空间连接到网关的能力受[_Route Attachment_](#cross-namespace-route-attachment)的支配。 本指南对路由连接进行了探讨，并演示了独立团队如何安全地共享同一个网关。

在本指南中，有两个独立的团队 _store_ 和 _site_，它们在同一个 Kubernetes 集群中的 `store-ns` 和 `site-ns` 名称空间中运行。 这些是它们的目标，以及它们如何使用 gateway API 资源来完成这些目标：

* 网站团队有两个应用程序：_home_ 和 _login_。团队希望

他们使用连接到同一 gateway 的独立 HTTPRoutes 来隔离路由配置，如金丝雀推出，但仍共享相同的 IP 地址、端口、DNS 域和 TLS 证书。

* 存储团队部署了名为 _store_ 的单一服务

存储在 `store-ns` 名称空间中，该名称空间也需要在相同的 IP 地址和域后面公开。

* Foobar 公司在 "foo.example.com "域名下运行，用于所有

由中央基础设施团队控制，在 "infra-ns "名称空间运行。

* 最后，安全团队控制着 `foo.example.com` 的证书。

通过单一共享 gateway 管理该证书，他们能够集中控制安全性，而无需直接涉及应用团队。

gateway API 资源之间的逻辑关系如下：

![跨命名空间路由](/镜像/cross-namespace-routing.svg)

## 跨 namespace 路由附件

[路由附加](/concepts/api-overview/#attaching-routes-to-gateways)是一个重要的概念，它决定了路由如何附加到网关并编排其路由规则。 当跨名称空间的路由共享一个或多个网关时，这一点尤为重要。 网关和路由的附加是双向的--只有网关所有者和路由所有者都同意这种关系，附加才能成功。 这种双向关系的存在有两个原因：

* 路由所有者不想通过路径过度暴露自己的应用程序，他们

不知道。

* 网关所有者不希望某些应用程序或团队在没有网关的情况下使用网关。

例如，内部服务不应通过互联网 gateway 访问。

网关支持 "附着限制"，这是网关侦听器上的字段，用于限制可以附着的路由。 网关支持名称空间和路由类型作为附着限制。 任何不符合附着限制的路由都无法附着到该网关。 同样，路由也可以通过路由的 "parentRef "字段明确引用它们想要附着的网关。 这些共同在信息设备所有者和应用程序所有者之间创建了一个握手机制，使他们能够独立定义如何通过网关公开应用程序。 这实际上是一种减少管理开销的策略。 应用程序所有者可以指定其应用程序应该使用哪些网关，信息设备所有者可以限制网关接受的名称空间和路由类型。

## 共享网关

基础设施团队将 `shared-gateway` gateway 部署到 `infra-ns` 名称空间：

```yaml
{% include 'standard/cross-namespace-routing/gateway.yaml' %}
```

上述 gateway 中的 `https` 监听器会匹配`foo.example.com`域的流量。 这样，基础架构团队就可以管理域的所有方面。 下面的 HTTPRoutes 无需指定域，如果未设置`hostname`，默认情况下会匹配所有流量。 这使得 HTTPRoutes 的管理更容易，因为它们可以与域无关，这在应用程序域不是固定不变的情况下很有帮助。

该 gateway 还会使用 `infra-ns` 名称空间中的 `foo-example-com` Secret 配置 HTTPS。 这样，基础架构团队就可以代表应用程序所有者集中管理 TLS。 `foo-example-com` 证书将终止所有流向其所附路由的流量，而 HTTPRoutes 本身无需进行任何 TLS 配置。

该 gateway 使用名称空间选择器来定义允许连接的 HTTPRoutes，这样基础架构团队就可以通过允许列出一组名称空间来限制谁或哪些应用程序可以使用该 gateway。

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: Gateway
spec:
  listeners:
  - allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            shared-gateway-access: "true"
...
```

只有标有 "shared-gateway-access: "true "的名称空间才能将其路由附加到 "shared-gateway"。 在以下名称空间中，如果 "no-external-access "名称空间中存在 HTTPRoute，且其 "parentRef "为 "infra-ns/shared-gateway"，那么网关将忽略该 HTTPRoute，因为它不符合附加约束（名称空间标签）。

```yaml
{% include 'standard/cross-namespace-routing/0-namespaces.yaml' %}
```

请注意，网关上的附着限制不是必需的，但如果集群中有许多不同的团队和 namespace，则附着限制是一种最佳做法。 在集群中所有应用程序都有权限附着到网关的环境中，则无需配置`listeners[].routes`字段，所有路由都可以自由使用网关。

## 路线附件

存储团队在 `store-ns` 名称空间中为 `store` 服务部署他们的路由：

```yaml
{% include 'standard/cross-namespace-routing/store-route.yaml' %}
```

该路由具有简单明了的路由逻辑，因为它只匹配 `/store` 流量，并将其发送到 `store` 服务。

网站团队现在为其应用程序部署路由。 他们在 `site-ns` 名称空间中部署了两个 HTTPRoutes：

* home "HTTPRoute作为默认路由规则，匹配所有流量

发送到现有路由规则未匹配的 `foo.example.com/*` 并将其发送到 `home` 服务。

* 登录 "HTTPRoute 会将 "foo.example.com/login "的流量路由至

它使用权重来细粒度地控制它们之间的流量分配。

这两个路由都被引用了相同的 gateway 附加配置，该配置将 `infra-ns` 名称空间中的 `gateway/shared-gateway` 指定为这些路由要附加到的唯一 gateway。

```yaml
{% include 'standard/cross-namespace-routing/site-route.yaml' %}
```

部署这三个 Routes 后，它们都将连接到 "shared-gateway "网关。 网关会将这些 Routes 合并为一个单一的路由规则平面列表。这些路由规则之间的[路由优先级](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule)由最特定的匹配决定，冲突则根据[冲突解决](/concepts/guidelines#conflicts)处理。 这为独立用户之间的路由规则合并提供了可预测和确定性。

得益于跨 namespace 路由，Foobar 公司可以更均匀地分配其基础设施的所有权，同时仍保留集中控制。 这为他们提供了两全其美的方案，所有方案都通过声明式开源 API 交付。