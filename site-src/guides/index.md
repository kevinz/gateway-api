<!-- TRANSLATED by md-translate -->
# 开始使用 gateway API

**1.** **[安装 gateway 控制器](#installing-a-gateway-controller)** _OR_ **[手动安装 gateway API CRDs](#installing-gateway-api)**。

然后

**2.** **试用其中一个可用的指南：**

* [Simple Gateway](/guides/simple-gateway)（一个很好的入门工具）
* [HTTP 路由](/guides/http-routing)
* [HTTP 重定向和重写](/guides/http-redirect-rewrite)
* [HTTP流量分割](/guides/traffic-splitting)
* [跨 namespace 路由](/guides/multiple-ns)
* [配置 TLS](/guides/tls)
* [TCP 路由](/guides/tcp)
* [gRPC 路由](/guides/grpc-routing)
* [从 ingress 迁移](/guides/migrating-from-ingress)

## 安装 gateway 控制器

有[多个项目]（/实现）支持 Gateway API。 通过在 Kubernetes 集群中安装 Gateway 控制器，您可以尝试上述指南。 这将证明您的 Gateway 资源（以及您的 Gateway 资源所代表的网络基础设施）实际上正在实现所需的路由配置。 请注意，许多 Gateway 控制器设置都会为您安装和移除 Gateway API 捆绑程序。

## 安装 gateway 应用程序接口

Gateway API 捆绑包代表与 gateway API 版本相关的 CRD 集。 每个发布版本包括两个具有不同稳定性的通道：

#### 安装标准通道

标准发布通道包括已升级到 GA 或 beta 版的所有资源，包括 GatewayClass、Gateway、HTTPRoute 和 ReferenceGrant。 要安装此通道，请运行以下 kubectl 命令：

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/standard-install.yaml
```

#### 安装实验通道

实验发布通道包括标准发布通道中的所有内容以及一些实验资源和字段。 其中包括 TCPRoute、TLSRoute、UDPRoute 和 GRPCRoute。

请注意，未来发布的 API 可能会对实验性资源和字段进行破坏性修改。 例如，任何实验性资源或字段都可能在未来发布的版本中被移除。 有关实验性通道的更多信息，请参阅我们的[版本文档](/concepts/versioning/)。

要安装实验通道，请运行以下 kubectl 命令：

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.0.0/experimental-install.yaml
```

#### 清理

卸载完成后，您可以卸载 Gateway API CRD，将上述命令中的 "引用 "替换为 "删除"。 如果这些资源正在使用中，或者是由 Gateway 控制器安装的，则不要卸载它们。 这将卸载整个集群的 Gateway API 资源。 如果这些资源可能正在被其他人引用，则不要卸载，因为这会破坏使用这些资源的任何程序。

### 关于 CRD 管理的更多信息

本指南仅提供了如何开始使用 gateway API 的高级概述。 有关管理 gateway API CRD 主题的更多信息，请参阅我们的[CRD 管理指南](/guides/crd-management)。