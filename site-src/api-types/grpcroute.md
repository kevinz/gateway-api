<!-- TRANSLATED by md-translate -->
# GRPCRoute

例如 "v0.6.0+ 版本中的实验频道"。

```
The `GRPCRoute` resource is Alpha and part of the Experimental Channel in
`v0.6.0+`. For more information on release channels, refer to the [related
documentation](/concepts/versioning).
```

[GRPCRoute](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCPRoute)是一种 gateway API 类型，用于指定 gRPC 请求从 gateway 监听器到 API 对象（即服务）的路由行为。

## 背景

虽然可以使用 "HTTPRoutes "或通过自定义的树外 CRD 来路由 gRPC，但从长远来看，这会导致生态系统支离破碎。

gRPC 是一个[被业界广泛引用的流行 RPC 框架](https://grpc.io/about/#whos-using-grpc-and-why)。该协议在 Kubernetes 项目中被普遍引用，是许多接口的基础，包括

* [CSI](https://github.com/container-storage-interface/spec/blob/5b0d4540158a260cb3347ef1c87ede8600afb9bf/spec.md)、
* [the CRI](https://github.com/kubernetes/cri-api/blob/49fe8b135f4556ea603b1b49470f8365b62f808e/README.md),
* [设备插件框架](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)

鉴于 gRPC 在应用层网络领域的重要性，尤其是对 Kubernetes 项目的重要性，我们决定不让生态系统出现不必要的分裂。

### 封装网络协议

一般来说，如果可以在较低层对封装协议进行路由，那么在满足以下条件的情况下，在较高层引入路由资源是可以接受的：

* 如果被迫在较低层路由，封装协议的用户将错过其生态系统的重要传统功能。
* 如果被迫在较低层路由，封装协议的用户将体验到降级的用户体验。
* 封装协议拥有庞大的用户群，尤其是在 Kubernetes 社区。

gRPC 符合所有这些标准，因此决定将 `GRPCRoute` 纳入 gateway API。

### Cross Serving

支持 GRPCRoute 的实现必须强制执行 `GRPCRoute`s 和 `HTTPRoute`s 之间主机名的唯一性。 如果 `HTTPRoute` 或 `GRPCRoute` 类型的路由 (A) 连接到了监听器，而该监听器已连接了另一种类型的路由 (B)，且 A 和 B 的主机名的交集不为空，则实现必须拒绝接受路由 A，也就是说，实现必须在相应的 RouteParentStatus 中引发状态为 "false "的 "已接受 "条件。

一般来说，建议为 gRPC 和非 gRPC HTTP 流量分别使用不同的主机名。 这符合 gRPC 社区的标准做法。 不过，如果必须在同一主机名上提供 HTTP 和 gRPC 服务（唯一的区别在于 URI），用户应为 gRPC 和 HTTP 使用 `HTTPRoute` 资源。 这将以改善 `GRPCRoute` 资源的用户体验为代价。

## Spec

GRPCRoute 的规格包括

* [ParentRefs](/reference/spec/#gateway.networking.k8s.io/v1beta1.ParentRef)- 定义此路由要连接到哪些 gateway。
* [Hostnames](/reference/spec/#gateway.networking.k8s.io/v1beta1.Hostname) （可选）- 定义主机名列表，用于匹配 gRPC 请求的主机头。
* [规则](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCRouteRule)- 定义对匹配的 gRPC 请求执行操作的规则列表。Each rule consists of [matches](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCRouteMatch), [filters](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCRouteFilter）（可选）和 [backendRefs](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCBackendRef) （可选）字段。

<!--- Editable SVG available at site-src/images/grpcroute-basic-example.svg -->

下面展示了一个将所有流量发送到一个服务的 GRPCRoute： ![grpcroute-basic-example](/images/grpcroute-basic-example.png)

### 连接到 gateway

每个路由都包含一种引用父资源的方式，它希望附加到父资源上。 在大多数情况下，这将是 gateway，但这里也有一些灵活性，可以实现对其他类型父资源的支持。

下面的示例显示了路由如何连接到 `acme-lb` gateway：

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GRPCRoute
metadata:
  name: grpcroute-example
spec:
  parentRefs:
  - name: acme-lb
```

请注意，目标 gateway 必须允许路由名称空间中的 GRPCRoutes 被附加，附加才能成功。

#### 主机名

主机名（Hostnames）定义了与 gRPC 请求的主机（Host）头匹配的主机名列表。 当出现匹配时，将根据规则和过滤器（可选）选择 GRPCRoute 来执行请求路由。 主机名是网络主机的完全合格域名，由 [RFC 3986](https://tools.ietf.org/html/rfc3986) 所定义。请注意以下与 RFC 中定义的 URI 的 "host "部分的偏差：

* 不允许使用 IP。
* 由于不允许使用端口，因此不使用 : 分隔符。

如果未指定主机名，则根据 GRPCRoute 规则和过滤器（可选）对流量进行路由。

下面的示例定义了主机名 "my.example.com"：

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GRPCRoute
metadata:
  name: grpcroute-example
spec:
  hostnames:
  - my.example.com
```

### 规则

规则定义了根据条件匹配 gRPC 请求、执行附加处理步骤和将请求转发给 API 对象的语义。

#### 匹配

匹配定义了用于匹配 gRPC 请求的条件。 每个匹配都是独立的，也就是说，如果满足任何一个匹配条件，该规则就会被匹配。

以下面的匹配配置为例：

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GRPCRoute
...
matches:
  - method:
      service: com.example.User
      method: Login
    headers:
      values:
        version: "2"
  - method:
      service: com.example.v2.User
      method: Login
```

请求必须满足以下任一条件，才能与本规则相匹配：

* com.example.User.Login "方法 ***和***包含 "version: 2 "标头。
* com.example.v2.User.Login "方法。

如果未指定匹配项，则默认匹配每个 gRPC 请求。

#### 过滤器（可选）

过滤器定义了在请求或响应生命周期内必须完成的处理步骤。 过滤器作为一个扩展点，可用于表达 gateway 实现中可能执行的附加处理。 一些例子包括请求或响应修改、实施身份验证策略、速率限制和流量整形。

下面的示例为带有主机标头 "my.filter.com "的 gRPC 请求添加了标头 "my-header: foo"。请注意，GRPCRoute 使用 HTTPRoute 过滤器来实现与 HTTPRoute 功能相同的功能，比如这样。

```yaml
{% include 'experimental/grpc-filter.yaml' %}
```

API 的一致性是根据过滤器类型来定义的。 目前还未说明多种行为排序的效果。 今后可能会根据 alpha 阶段的反馈意见进行修改。

一致性级别由过滤器类型定义：

* 支持 GRPCRoute 的实现必须支持所有 "核心 "过滤器。
* 我们鼓励实现者支持 "扩展自 "过滤器。
* "特定于实现的 "过滤器没有跨实现的 API 保证。

多次指定核心过滤器会产生未指定或自定义的一致性。

如果实现无法支持过滤器组合，则必须明确记录这一限制。 如果指定了不兼容或不支持的过滤器，并导致 "已接受 "条件被设置为状态 "false"，则实现可使用 "IncompatibleFilters "原因来指定此配置错误。

#### BackendRefs（可选）

BackendRefs 定义了应向其发送匹配请求的 API 对象。 如果未指定，则规则不执行转发。 如果未指定且未指定会导致发送响应的过滤器，则会返回 `UNIMPLEMENTED` 错误代码。

下面的示例将方法 `User.Login` 的 gRPC 请求转发给端口 `50051` 上的服务 "my-service1"，并将标头为 `magic: foo` 的方法 `Things.DoThing` 的 gRPC 请求转发给端口 `50051` 上的服务 "my-service2"：

```yaml
{% include 'experimental/basic-grpc.yaml' %}
```

下面的示例使用了 `weight` 字段，将 90% 发往 `foo.example.com` 的 gRPC 请求转发给 "foo-v1 "服务，另外 10% 转发给 "foo-v2 "服务：

```yaml
{% include 'experimental/traffic-splitting/grpc-traffic-split-2.yaml' %}
```

请参考 [backendRef](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCBackendRef) API 文档，了解有关`权重`和其他字段的更多详情。

## 状态

状态定义了 GRPCRoute 的观察状态。

### 路由状态

RouteStatus 定义了所有路由类型都需要的观察状态。

#### 家长

父资源定义了与 GRPCRoute 相关联的 gateway（或其他父资源）列表，以及 GRPCRoute 相对于这些 gateway 的状态。 当 GRPCRoute 在 parentRefs 中添加对 gateway 的引用时，管理 gateway 的控制器应在首次看到该路由时向该列表添加条目，并应在修改路由时适当更新条目。

## 示例

下面的示例表明 GRPCRoute "grpc-example "已被名称空间 "gw-example-ns "中的 gateway "gw-example "接受：

```yaml
apiVersion: gateway.networking.k8s.io/v1alpha2
kind: GRPCRoute
metadata:
  name: grpc-example
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

单个 gateway 资源可附加多个 GRPCRoutes。 重要的是，每个请求只能匹配一个 Route 规则。 有关冲突解决如何应用于合并的更多信息，请参阅 [API 规范](/reference/spec/#gateway.networking.k8s.io/v1alpha2.GRPCRouteRule)。