<!-- TRANSLATED by md-translate -->
# 部署一个简单的 gateway

最简单的部署是由同一 Owner 一起部署网关和路由资源。 这代表了用于 ingress 的类似模式。 在本指南中，部署的网关和 HTTPRoute 可匹配所有 HTTP 流量，并将其导向名为 "foo-svc "的单一服务。

简单网关](/镜像/single-service-gateway.png)

```yaml  
{% include 'standard/simple-gateway/gateway.yaml' %}
```

Gateway 表示逻辑负载平衡器的实例化。 它的模板来自一个假定的 `acme-lb` GatewayClass。 Gateway 监听端口 80 上的 HTTP 流量。 这个特定的 GatewayClass 会自动分配一个 IP 地址，部署完成后将显示在 `Gateway.status`中。

路由资源使用 "ParentRefs"（父路由引用）指定它们要连接的 gateway。 只要 gateway 允许这种连接（默认情况下，来自同一 namespace 的路由是可信的），这将允许路由接收来自父网关的流量。"BackendRefs"（后端引用）定义流量将被发送到的后端。 更复杂的双向匹配和权限是可能的，并在其他指南中进行了说明。

下面的 HTTPRoute 定义了如何将来自 gateway 监听器的流量路由到后端。 由于没有指定主机路由或路径，该 HTTPRoute 将匹配到达负载平衡器 80 端口的所有 HTTP 流量，并将其发送到 `foo-svc` Pod。

```yaml  
{% include 'standard/simple-gateway/httproute.yaml' %}
```

虽然路由资源通常用于过滤流向许多不同后端（可能有不同的所有者）的流量，但本示例演示了使用单个服务后端的最简单路由。 本示例展示了服务所有者如何同时部署 gateway 和 HTTPRoute 以供其单独使用，从而让他们对服务的暴露方式拥有更多控制权和自主权。