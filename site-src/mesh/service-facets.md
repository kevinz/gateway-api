<!-- TRANSLATED by md-translate -->
# 服务的不同方面

Kubernetes [Service](https://kubernetes.io/docs/concepts/services-networking/service/)资源比人们通常意识到的要复杂得多。当你创建一个服务时，通常集群机械会：

* 为服务本身分配一个全集群 IP 地址（其_集群 IP_）；
* 为服务分配一个 DNS 名称，解析到集群 IP（其 _DNS name_）；
* 将分配给服务选择器匹配的每个 Pod 的独立集群 IP 地址（_端点 IP_）收集到服务的端点或端点片中。
* 配置网络，使集群 IP 的流量在所有端点 IP 之间实现负载平衡。

遗憾的是，在考虑 gateway API 如何用于服务网格时，这些实现细节变得非常重要！

在 [GAMMA initiative](/concepts/gamma/) 的工作中，把 "服务 "看作由两个独立的_facets_（面）组成是很有用的：

* 服务的**前端**是集群 IP 和其 DNS 名称的组合。
其 DNS 名称的组合。
* 服务的**后端**是端点 IP 的集合。豆荚
不属于服务后端，但当然与端点 IP 密切相关。
与端点 IP 密切相关）。

面与面之间的区别至关重要，因为 [gateway](/api-types/gateway/) 和 [mesh](/mesh) 都需要决定提及特定服务的请求应被定向到该服务的前端还是后端：

* 将请求引导至服务前端（_服务路由_）后，由底层网络决定使用哪个端点 IP。
由相应的网络基础设施来决定使用哪个端点 IP（可能是 "kube-proxy"、"服务网格 "或其他什么东西）。
基础设施（可能是 "kube-proxy"、服务网格或其他
其他）。
* 将请求导向服务的后端（_endpoint routing_）通常是必要的。
通常需要将请求导向服务的后端（_端点路由_），以实现更高级的负载平衡决策（例如，实施粘性会话的网关）。
例如，实施粘性会话的 gateway）。

虽然服务路由可能最直接符合[Ana](/concepts/roles-and-personas#ana)的路由意识，但在[南北交通](/concepts/glossary#northsouth-traffic)和[东西交通](/concepts/glossary#eastwest-traffic)中使用 gateway API 时，端点路由的可预测性会更高。 GAMMA 计划](/concepts/gamma/)正在努力为这种用例正式制定指南。