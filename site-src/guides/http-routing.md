<!-- TRANSLATED by md-translate -->
# HTTP 路由

HTTPRoute 资源](/api-types/httproute) 允许您对 HTTP 流量进行匹配，并将其导向 Kubernetes 后端。 本指南展示了 HTTPRoute 如何对主机、报头和路径字段上的流量进行匹配，并将其转发到不同的 Kubernetes 服务。

下图描述了三个不同服务所需的流量：

* 指向 `foo.example.com/login` 的流量会被转发到 `foo-svc` 。
* 指向 `bar.example.com/*` 并带有 `env: canary` 标头的流量将被转发至

到 `bar-svc-canary`

* 指向 `bar.example.com/*` 而不带标头的流量将被转发到 `bar-svc` 。

HTTP路由](/镜像/http-routing.png)

虚线表示为配置此路由行为而部署的 gateway 资源。 有两个 HTTPRoute 资源在同一个 `prod-web` gateway 上创建路由规则。 这说明了一个 gateway 可以绑定多个路由，只要路由不冲突，就可以在 gateway 上合并。 有关路由合并的更多信息，请参阅 [HTTPRoute 文档](/api-types/httproute#merging)。

要接收来自 [gateway](/reference/spec/#gateway.networking.k8s.io/v1beta1.gateway) 的流量，就必须为`HTTPRoute`资源配置`ParentRefs`，该`ParentRefs`引用了它应连接的父网关。 下面的示例展示了如何配置`Gateway`和`HTTPRoute`的组合，以提供 HTTP 流量：

```yaml
{% include 'standard/http-routing/gateway.yaml' %}
```

HTTPRoute 可以匹配 [一组主机名](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteSpec)。 在 HTTPRoute 进行其他匹配之前，先匹配这些主机名。由于 `foo.example.com` 和 `bar.example.com` 是独立的主机，具有不同的路由要求，因此它们分别部署为自己的 HTTPRoute - `foo-route` 和 `bar-route`。

下面的 `foo-route` 将匹配 `foo.example.com` 的任何流量，并应用其路由规则将流量转发到正确的后端。 由于只指定了一个匹配项，因此只有 `foo.example.com/login/*` 流量会被转发。 此路由不会匹配任何其他不以 `/login` 开头的路径的流量。

```yaml
{% include 'standard/http-routing/foo-httproute.yaml' %}
```

同样，"bar-route "HTTPRoute 会匹配 "bar.example.com "的流量。该主机名的所有流量都将根据路由规则进行评估。 最特殊的匹配将优先，这意味着任何带有 "env: canary "标头的流量都将转发到 "bar-svc-canary"，如果标头缺失或不是 "canary"，则将转发到 "bar-svc"。

```yaml
{% include 'standard/http-routing/bar-httproute.yaml' %}
```