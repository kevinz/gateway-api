<!-- TRANSLATED by md-translate -->
# HTTP 路径重定向和重写

[HTTPRoute资源](/api-types/httproute)可以向客户端发出重定向，或使用[过滤器](/api-types/httproute#filters-optional)重写上游发送的路径。 本指南展示了如何使用这些功能。

请注意，重定向和重写过滤器互不兼容。 规则不能同时被引用两种过滤器类型。

## 重定向

重定向会向客户端返回 HTTP 3XX 响应，指示其检索不同的资源。[`RequestRedirect`规则过滤器](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRequestRedirectFilter) 指示 gateway 对匹配已过滤 HTTPRoute 规则的请求发出重定向响应。

重定向过滤器可独立替换各种 URL 组件。 例如，要从 HTTP 向 HTTPS 发出永久重定向 (301)，请配置 `requestRedirect.statusCode=301` 和 `requestRedirect.scheme="https"：

```yaml
{% include 'standard/http-redirect-rewrite/httproute-redirect-http.yaml' %}
```

重定向会更改已配置的 URL 组件，使其与重定向配置相匹配，同时保留原始请求 URL 中的其他组件。 在此示例中，请求 `GET http://redirect.example/cinnamon` 将得到带有 `location: https://redirect.example/cinnamon` 头信息的 301 响应。主机名 (`redirect.example`)、路径 (`/ccinnamon`)和端口 (implicit) 保持不变。

### HTTP 到 HTTPS 重定向

要将 HTTP 流量重定向到 HTTPS，需要同时拥有 HTTP 和 HTTPS 监听器的 gateway。

```yaml
{% include 'standard/http-redirect-rewrite/gateway-redirect-http-https.yaml' %}
```

确保 gateway 安全有多种方法，本例中使用 Kubernetes Secret（"证书引用 "部分中的 "重定向示例"）。

我们需要一个 HTTPRoute 来连接 HTTP 监听器并将其重定向到 HTTPS。 在这里，我们将 `sectionName` 设置为 `http`，这样它就只能选择名为 `http` 的监听器。

```yaml
{% include 'standard/http-redirect-rewrite/httproute-redirect-http.yaml' %}
```

您还需要一个 HTTPRoute，它可以附加到 HTTPS 监听器，将 HTTPS 流量转发到应用程序后端。

```yaml
{% include 'standard/http-redirect-rewrite/httproute-redirect-https.yaml' %}
```

### 路径重定向

路径重定向使用 HTTP 路径修饰符来替换整个路径或路径前缀。 例如，下面的 HTTPRoute 将对路径以 `/cayenne` 开头的所有 `redirect.example` 请求发出 302 重定向，将其重定向到 `/paprika`：

```yaml
{% include 'standard/http-redirect-rewrite/httproute-redirect-full.yaml' %}
```

对 `https://redirect.example/cayenne/pinch` 和 `https://redirect.example/cayenne/teaspoon` 的请求都将收到一个重定向，其中包含一个 `location:https://redirect.example/paprika`。

另一种路径重定向类型 "ReplacePrefixMatch "只替换与 "matches.path.values "匹配的路径部分。 将上面的过滤器改为：

```yaml
{% include 'standard/http-redirect-rewrite/httproute-redirect-prefix.yaml' %}
```

将导致带有 `location: https://redirect.example/paprika/pinch` 和 `location: https://redirect.example/paprika/teaspoon` 响应头的重定向。

## 重写

重写会修改客户端请求的组件，然后将其代理到上游。 [`URLRewrite`过滤器](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPURLRewriteFilter)可以更改上游请求的主机名和/或路径。 例如，以下 HTTPRoute 将接受`https://rewrite.example/cardamom`请求，并将其发送到上游的`example-svc`，请求头中的`host: elsewhere.example`将改为`host: rewrite.example`。

```yaml
{% include 'standard/http-redirect-rewrite/httproute-rewrite.yaml' %}
```

路径重写还可以使用 HTTP 路径修饰符。下面的 HTTPRoute 将接收对 `https://rewrite.example/cardamom/smidgen` 的请求，并将对 `https://elsewhere.example/fennel` 的请求代理到上游的 `example-svc`。而使用 `type: ReplacePrefixMatch` 和 `replacePrefixMatch: /fennel` 则会请求上游的 `https://elsewhere.example/fennel/smidgen`。

```yaml
{% include 'standard/http-redirect-rewrite/httproute-rewritepath.yaml' %}
```