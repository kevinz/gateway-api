<!-- TRANSLATED by md-translate -->
# gateway

成功 "V0.5.0+ 版本中的标准通道"。

```
The `Gateway` resource is Beta and part of the Standard Channel in `v0.5.0+`.
```

Gateway "与基础设施配置的生命周期为 1:1。 当用户创建一个 "Gateway "时，一些负载平衡基础设施将由 "GatewayClass "控制器进行供应或配置（详见下文）。"Gateway "是触发此 API 中操作的资源。 此 API 中的其他资源是配置片段，直到创建 Gateway 将资源连接在一起。

gateway` 规格定义如下：

* `GatewayClassName`- 定义该 gateway 被引用的 `GatewayClass` 对象的名称。
* 侦听器`- 定义侦听器的主机名、端口、协议、终止、TLS 设置以及可附加到侦听器的路由。
* `Addresses`- 定义为该网关请求的网络地址。

如果无法实现 gateway spec 中指定的所需配置，网关将处于出错状态，详细情况由状态条件提供。

#### 部署模式

根据 `GatewayClass` 的不同，创建 `Gateway` 可以执行以下任何操作：

* 被引用云 API 创建 LB 实例。
* 生成软件 LB 的新实例（在本集群或其他集群中）。
* 在已实例化的 LB 中添加配置说明，以处理新路由。
* 对 SDN 进行编程，以实现配置。
* 还有一些我们还没有想到的...

应用程序接口未指定将采取其中哪种操作。

### 网关状态

GatewayStatus"（网关状态）用于显示 "网关 "相对于 "spec"（规格）中所需状态的状态：

* 地址"- 列出实际绑定到 gateway 的 IP 地址。
* `Listeners`- 为 `spec` 中定义的每个唯一监听器提供状态。
* `Conditions`- 描述 gateway 的当前状态条件。

Conditions "和 "Listeners.conditions "都遵循 Kubernetes 中其他地方被引用的条件模式。 这是一个列表，其中包括条件类型、条件状态和此条件上次发生变化的时间。