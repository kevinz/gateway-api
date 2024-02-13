<!-- TRANSLATED by md-translate -->
# HTTP 流量分割

HTTPRoute 资源](/api-types/httproute) 允许您指定权重，以便在不同的后端之间转移流量。 这对于在推出、临时变更或紧急情况下分割流量非常有用。 HTTPRoute`spec.rules.backendRefs` 接受路由规则将向其发送流量的后端列表。 这些后端的相对权重定义了它们之间的流量分割。 下面的 YAML 代码段显示了如何将两个服务列为一条路由规则的后端。 该路由规则将把 90% 的流量分割给 `foo-v1`，10% 的流量分割给 `foo-v2`。

交通分割](/镜像/simple-split.png)

```yaml
{% include 'standard/traffic-splitting/simple-split.yaml' %}
```

权重 "表示按比例分配流量（而不是百分比），因此单条路由规则中所有权重的总和就是所有后端的分母。"权重 "是一个可选参数，如果未指定，默认为 1。 如果路由规则只指定了一个后端，那么无论指定了什么权重（如果有的话），它都会隐式地接收 100% 的流量。

## Guide

本指南展示了一个服务的两个版本的部署情况。 流量拆分被引用来管理从 v1 到 v2 的流量逐步拆分。

本例假设部署了以下 gateway：

```yaml 
{% include 'standard/simple-gateway/gateway.yaml' %}
```

##金丝雀交通推出

At first, there may only be a single version of a Service that serves
production user traffic for `foo.example.com`. The following HTTPRoute has no
`weight` specified for `foo-v1`  or `foo-v2` so they will implicitly
receive 100% of the traffic matched by each of their route rules. A canary
route rule is used (matching the header `traffic=test`) to send synthetic test
traffic before splitting any production user traffic to `foo-v2`.
[Routing precedence](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule)
ensures that all traffic with the matching host and header 
(the most specific match) will be sent to `foo-v2`.

交通分流](/images/traffic-splitting-1.png)

```yaml
{% include 'standard/traffic-splitting/traffic-split-1.yaml' %}
```

## 蓝绿交通推广

在内部测试验证了 `foo-v2` 的成功响应后，最好将一小部分流量转移到新服务，以便逐步进行更真实的测试。 下面的 HTTPRoute 添加了 `foo-v2` 作为后端，并加上了权重。 权重加起来总共是 100，因此 `foo-v1` 接收 90/100=90% 的流量，而 `foo-v2` 接收 10/100=10% 的流量。

交通分流](/images/traffic-splitting-2.png)

```yaml
{% include 'standard/traffic-splitting/traffic-split-2.yaml' %}
```

## 完成推广

最后，如果所有信号都是正的，就可以将流量完全转移到 `foo-v2` 并完成推出。 将 `foo-v1` 的权重设置为 `0`，以便将其配置为接受零流量。

交通分流](/images/traffic-splitting-3.png)

```yaml
{% include 'standard/traffic-splitting/traffic-split-3.yaml' %}
```

此时，100% 的流量都会被路由到 `foo-v2`，推广工作已经完成。 如果 `foo-v2`由于任何原因出现错误，可以更新权重，将流量迅速转回 `foo-v1`。 一旦推广工作被视为最终结果，v1 就可以完全退出运行。