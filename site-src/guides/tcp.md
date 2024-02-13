<!-- TRANSLATED by md-translate -->
信息 "实验频道"

```
The `TCPRoute` resource described below is currently only included in the
"Experimental" channel of Gateway API. For more information on release
channels, refer to the [related documentation](/concepts/versioning).
```

gateway API 设计用于与多种协议协同工作，[TCPRoute](/reference/spec/#gateway.networking.k8s.io/v1alpha2.TCPRoute) 就是这样一种路由，它允许管理 [TCP](https://datatracker.ietf.org/doc/html/rfc793) 流量。

在本例中，我们有一个 gateway 资源和两个 TCPRoute 资源，它们按照以下规则分配流量：

* gateway 8080 端口上的所有 TCP 流都会转发到 `my-foo-service` Kubernetes 服务的 6000 端口。
* gateway 8090 端口上的所有 TCP 流都会转发到 "my-bar-service "Kubernetes 服务的 6000 端口。

在此示例中，将在 [gateway](/reference/spec/#gateway.networking.k8s.io/v1alpha2.Gateway) 上应用两个 `TCP` 监听器，以便将它们路由到两个独立的后端 `TCPRoutes`，请注意，在 `Gateway` 上为 ` 监听器` 设置的 `protocol` 是 `TCP`：

```yaml
{% include 'experimental/basic-tcp.yaml' %}
```

在上例中，我们通过被引用`parentRefs`中的`sectionName`字段，将两个独立后端 TCP [Services](https://kubernetes.io/docs/concepts/services-networking/service/) 的流量分开：

```yaml
spec:
  parentRefs:
  - name: my-tcp-gateway
    sectionName: foo
```

这与 `Gateway` 中 `listeners` 中的 `name` 直接对应：

```yaml
listeners:
  - name: foo
    protocol: TCP
    port: 8080
  - name: bar
    protocol: TCP
    port: 8090
```

这样，每个 "TCPRoute "都会 "连接 "到 "gateway "上的不同端口，这样 "my-foo-service "服务就会接收来自集群外 "8080 "端口的流量，而 "my-bar-service "则接收 "8090 "端口的流量。