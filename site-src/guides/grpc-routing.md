<!-- TRANSLATED by md-translate -->
# gRPC 路由

信息 "实验频道"

```
The `GRPCRoute` resource described below is currently only included in the
"Experimental" channel of Gateway API. For more information on release
channels, refer to the [related documentation](/concepts/versioning).
```

GRPCRoute 资源](/api-types/grpcroute) 可让您在 gRPC 流量上进行匹配，并将其导向 Kubernetes 后端。 本指南展示了 GRPCRoute 如何在主机、头和服务以及方法字段上匹配流量，并将其转发到不同的 Kubernetes 服务。

下图描述了三个不同服务所需的流量：

* 为使用 `com.Example.Login` 方法而访问 `foo.example.com` 的流量会被转发到 `foo-svc` 。
* 指向 `bar.example.com` 并带有 `env: canary` 标头的流量将被转发

为所有服务和方法的 "bar-svc-canary

* 指向 `bar.example.com` 而不带标头的流量将被转发到 `bar-svc` 以获取所有服务和方法

<!--- Editable source available at site-src/images/grpc-routing.png -->

gRPC 路由](/镜像/grpc-routing.png)

虚线表示为配置此路由行为而部署的 `Gateway` 资源。 有两个 `GRPCRoute` 资源在同一 `prod` Gateway 上创建路由规则。 这说明了如何将多个路由绑定到 Gateway，只要路由不冲突，就可以在 `Gateway` 上合并。 `GRPCRoute` 遵循相同的路由合并语义。 有关详细信息，请参阅 [文档](/api-types/httproute#merging)。

为了接收来自 [gateway](/reference/spec/#gateway.networking.k8s.io/v1beta1.gateway) 的流量，必须在 `GRPCRoute` 资源中配置 `ParentRefs` 以引用其应连接的父网关。 下面的示例显示了如何配置 `Gateway` 和 `GRPCRoute` 组合以提供 gRPC 流量：

```yaml
{% include 'experimental/v1alpha2/grpc-routing/gateway.yaml' %}
```

一个 `GRPCRoute` 可以匹配 [一组主机名](/reference/spec/#gateway.networking.k8s.io%2fv1alpha2.GRPCRouteSpec)。 在 GRPCRoute 中进行任何其他匹配之前，先匹配这些主机名。由于 `foo.example.com` 和 `bar.example.com` 是具有不同路由要求的独立主机，因此每个主机都部署为自己的 GRPCRoute - `foo-route` 和 `bar-route`。

下面的 `foo-route` 将匹配任何指向 `foo.example.com` 的流量，并应用其路由规则将流量转发到正确的后端。 由于只指定了一个匹配项，因此只有指向 `foo.example.com` 的 `com.example.User.Login` 方法的请求才会被转发。 任何其他方法的 RPC 都不会被此 Route 匹配。

```yaml
{% include 'experimental/v1alpha2/grpc-routing/foo-grpcroute.yaml' %}
```

同样，"bar-route "GRPCRoute 会匹配针对 "bar.example.com "的 RPC。针对该主机名的所有流量都将根据路由规则进行评估。 最特殊的匹配将优先，这意味着任何带有 "env: canary "标头的流量都将转发至 "bar-svc-canary"，如果标头缺失或没有 "canary "值，则将转发至 "bar-svc"。

```yaml
{% include 'experimental/v1alpha2/grpc-routing/bar-grpcroute.yaml' %}
```

要使用交互式客户端（如 [`grpcurl`](https://github.com/fullstorydev/grpcurl)），而无需在本地文件服务器上拷贝目标服务的协议缓冲区，就需要使用 [gRPC Reflection](https://github.com/grpc/grpc/blob/v1.49.1/doc/server-reflection.md)。要启用此功能，首先要确保在应用程序 pod 上有一个 gRPC 反射服务器在监听，然后将反射方法添加到您的 `GRPCRoute` 中。这在开发和暂存环境中可能会很有用，但只有在考虑了安全影响后，才应在生产环境中启用此功能。

```yaml
{% include 'experimental/v1alpha2/grpc-routing/reflection-grpcroute.yaml' %}
```