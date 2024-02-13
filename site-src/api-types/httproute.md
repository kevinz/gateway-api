<!-- TRANSLATED by md-translate -->
# HTTPRoute

成功 "V0.5.0+ 版本中的标准通道"。

```
The `HTTPRoute` resource is Beta and part of the Standard Channel in `v0.5.0+`.
```

[HTTPRoute](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRoute)是一种 gateway API 类型，用于指定从 gateway 监听器到 API 对象（即服务）的 HTTP 请求的路由行为。

## Spec

HTTPRoute 的规范包括

* [ParentRefs](/reference/spec/#gateway.networking.k8s.io/v1beta1.ParentRef)- 定义此路由要连接到哪些 gateway。
* [Hostnames](/reference/spec/#gateway.networking.k8s.io/v1beta1.Hostname) （可选）- 定义主机名列表，用于匹配 HTTP 请求的 Host 头信息。
* [Rules](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule)- 定义对匹配的 HTTP 请求执行操作的规则列表。每个规则由 [match](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteMatch)、[filters](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteFilter)、[match]和[filters]组成。HTTPRouteFilter)（可选）、[backendRefs](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPBackendRef)（可选）和 [timeouts](/reference/spec/#gateway.networking.k8s.io/v1.HTTPRouteTimeouts) （可选）字段。

下面展示的 HTTPRoute 会将所有流量发送到一个服务：![httproute-basic-example](/images/httproute-basic-example.svg)

### 连接到 gateway

每个路由都包含一种引用父资源的方式，它希望附加到父资源上。 在大多数情况下，这将是 gateway，但这里也有一些灵活性，可以实现对其他类型父资源的支持。

下面的示例显示了路由如何连接到 `acme-lb` gateway：

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: httproute-example
spec:
  parentRefs:
  - name: acme-lb
```

请注意，目标 gateway 需要允许路由名称空间中的 HTTPRoutes 被附加，附加才能成功。

#### 主机名

主机名定义了与 HTTP 请求的 Host 头匹配的主机名列表。 当出现匹配时，HTTPRoute 将根据规则和过滤器（可选）选择执行请求路由。 主机名是网络主机的完全合格域名，由 [RFC 3986](https://tools.ietf.org/html/rfc3986) 定义。请注意以下与 RFC 中定义的 URI 的 "host "部分的偏差：

* 不允许使用 IP。
* 由于不允许使用端口，因此不使用 : 分隔符。

如果未指定主机名，则根据 HTTPRoute 规则和过滤器（可选）对流量进行路由。

下面的示例定义了主机名 "my.example.com"：

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: httproute-example
spec:
  hostnames:
  - my.example.com
```

### 规则

规则定义了根据条件匹配 HTTP 请求、执行附加处理步骤和将请求转发给 API 对象的语义。

#### 匹配

匹配定义了用于匹配 HTTP 请求的条件。 每个匹配都是独立的，也就是说，如果满足任何一个匹配条件，这条规则就会被匹配。

以下面的匹配配置为例：

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
...
spec:
  rules:
  - matches:
    - path:
        value: "/foo"
      headers:
      - name: "version"
        value: "2"
    - path:
        value: "/v2/foo"
```

请求必须满足以下任一条件，才能与本规则相匹配：

* 路径前缀为 /foo ** AND** 包含 "版本：2 "标头
* 路径前缀为 /v2/foo

如果未指定匹配项，默认情况下会使用"/"作为前缀路径匹配，从而匹配所有 HTTP 请求。

#### 过滤器（可选）

过滤器定义了在请求或响应生命周期内必须完成的处理步骤。 过滤器作为一个扩展点，可用于表达 gateway 实现中可能执行的附加处理。 一些例子包括请求或响应修改、实施身份验证策略、速率限制和流量整形。

下面的示例为带有主机标头 "my.filter.com "的 HTTP 请求添加了标头 "my-header: foo"。

```yaml
{% include 'standard/http-filter.yaml' %}
```

API 的一致性是根据过滤器类型来定义的。 目前还未说明多种行为排序的效果。 今后可能会根据 alpha 阶段的反馈意见进行修改。

一致性级别由过滤器类型定义：

* 实现者必须支持所有 "核心 "过滤器。
* 鼓励实现者支持 "扩展自 "过滤器。
* "特定于实现的 "过滤器没有跨实现的 API 保证。

多次指定核心过滤器具有未指定或特定实现的一致性。

除了 URLRewrite 和 RequestRedirect 筛选器不能组合使用外，所有筛选器都应相互兼容。 如果实现无法支持其他筛选器组合，则必须明确记录该限制。 如果指定了不兼容或不支持的筛选器，并导致 "已接受 "条件被设置为状态 "False"，则实现可使用 "IncompatibleFilters "原因来指定该配置错误。

#### BackendRefs（可选）

BackendRefs 定义了应将匹配请求发送到何处的 API 对象。 如果未指定，则规则不执行转发。 如果未指定且未指定可导致发送响应的过滤器，则返回 404 错误代码。

下面的示例将前缀为"/bar "的 HTTP 请求转发给端口为 "8080 "的服务 "my-service1"，将前缀为"/some/thing"、标头为 "magic: foo "的 HTTP 请求转发给端口为 "8080 "的服务 "my-service2"：

```yaml
{% include 'standard/basic-http.yaml' %}
```

下面的示例使用了 `weight` 字段，将指向 `foo.example.com` 的 HTTP 请求的 90% 转发到 "foo-v1 "服务，另外 10% 转发到 "foo-v2 "服务：

```yaml
{% include 'standard/traffic-splitting/traffic-split-2.yaml' %}
```

请参考 [backendRef](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPBackendRef) API 文档，了解有关 `权重`和其他字段的更多详情。

#### 超时（可选）

例如 "V1.0.0+ 版本中的实验频道"。

```
HTTPRoute timeouts are part of the Experimental Channel in `v1.0.0+`.
```

HTTPRoute 规则包含一个 "超时 "字段。 如果未指定，超时行为将根据具体实现而定。

HTTPRoute 规则中可配置两种超时：

1. request "是 gateway API 实现向客户端 HTTP 请求发送响应的超时。该超时的目的是尽可能覆盖整个请求-响应事务，尽管实现可以选择在收到整个请求流后开始超时，而不是在客户端启动事务后立即超时。
2. backendRequest "是从 gateway 向后端发出的单个请求的超时。该超时包括从网关第一次开始发送请求到后端收到完整响应的时间。如果 gateway 重试与后端连接，这一点会特别有用。

由于 `request` 超时包含了 `backendRequest` 超时，因此 `backendRequest` 的值不得大于 `request` 超时的值。

超时是可选的，其字段的类型为 [Duration](/geps/gep-2257/)。 零值超时（"0s"）必须解释为禁用超时。 有效的非零值超时必须 &gt;= 1ms。

下面的示例使用了 `request` 字段，如果客户端请求的完成时间超过 10 秒，该字段将导致超时。 该示例还定义了一个 2 秒的 `backendRequest` 字段，用于指定网关向后端服务 `timeout-svc` 发出的单个请求的超时时间：

```yaml
{% include 'experimental/http-route-timeouts/timeout-example.yaml' %}
```

更多详情请参考 [timeouts](/reference/spec/#gateway.networking.k8s.io/v1.HTTPRouteTimeouts) API 文档。

##### 后端协议

例如 "V1.0.0+ 版本中的实验频道"。

```
This concept is part of the Experimental Channel in `v1.0.0+`.
```

某些实现可能要求明确标注 [backendRef](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPBackendRef)，以便使用特定协议路由流量。 对于 Kubernetes 服务后端，这可以通过指定 [`appProtocol`](https://kubernetes.io/docs/concepts/services-networking/service/#application-protocol) 字段来实现。

## 状态

状态定义 HTTPRoute 的观察状态。

### 路由状态

RouteStatus 定义了所有路由类型都需要的观察状态。

#### 家长

父资源定义了与 HTTPRoute 相关联的 gateway（或其他父资源）列表，以及 HTTPRoute 相对于每个 gateway 的状态。 当 HTTPRoute 在 parentRefs 中添加对 Gateway 的引用时，管理 Gateway 的控制器应在第一次看到该路由时向该列表添加条目，并在修改路由时适当更新条目。

下面的示例表明 HTTPRoute "http-example "已被名称空间 "gw-example-ns "中的 gateway "gw-example "接受：

```yaml
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: http-example
...
status:
  parents:
  - parentRefs:
      name: gw-example
      namespace: gw-example-ns
    conditions:
    - type: Accepted
      status: "True"
```

## Merging

单个 gateway 资源可附加多个 HTTPRoutes。 重要的是，每个请求只能匹配一个 Route 规则。 有关冲突解决如何应用于合并的详细信息，请参阅 [API 规范](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule)。