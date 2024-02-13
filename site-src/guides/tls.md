<!-- TRANSLATED by md-translate -->
# TLS 配置

gateway API 允许以多种方式配置 TLS。 本文档列出了各种 TLS 设置，并给出了如何有效使用这些设置的一般指南。

信息 "实验频道"

```
The `TLSRoute` and `BackendTLSPolicy` resources described below are currently only included in the
"Experimental" channel of Gateway API. For more information on release
channels, refer to the [related documentation](/concepts/versioning).
```

## 客户端/服务器和 TLS

![概览](/镜像/tls-overview.svg)

对于 gateway 来说，涉及两个连接：

* **下游**：这是客户端与 gateway 之间的连接。
* **上游**：这是 gateway 与路由指定的后端资源之间的连接。这些后端资源通常是服务。

通过 gateway API，下游和上游连接的 TLS 配置可独立管理。

对于下游连接，根据监听器协议的不同，支持不同的 TLS 模式和路由类型。

| Listener Protocol | TLS Mode | Route Type Supported | |-------------------|-------------|---------------------| | TLS | Passthrough | TLSRoute | TLS | Terminate | TCPRoute | HTTPS | Terminate | HTTPRoute | GRPC | Terminate | GRPCRoute | TCPRoute | TCPRoute | TCPRoute | TCPRoute | TCPRoute | TCPRoute | GRPC

请注意，在 "直通 "TLS 模式下，由于客户端的 TLS 会话不会在 gateway 终止，而是加密后通过 gateway，因此 TLS 设置不会生效。

对于上游连接，使用 "BackendTLSPolicy"，监听协议和 TLS 模式都不适用于上游 TLS 配置。 对于 "HTTPRoute"，支持同时使用 "Terminate "TLS 模式和 "BackendTLSPolicy"。 将这两种模式结合起来使用，就能实现通常所说的连接终止，然后在 gateway 重新加密。

## 下游 TLS

下游 TLS 设置被引用到 gateway 层级的侦听器中。

### 监听器和 TLS

监听器以每个域或子域为单位公开 TLS 设置。 监听器的 TLS 设置适用于满足 "hostname "条件的所有域。

在下面的示例中，gateway 会为所有请求提供在 "default-cert "秘密资源中定义的 TLS 证书。 虽然示例引用的是 HTTPS 协议，但同样的功能也可以与 TLSRoutes 一起用于纯 TLS 协议。

```yaml
listeners:
- protocol: HTTPS # Other possible value is `TLS`
  port: 443
  tls:
    mode: Terminate # If protocol is `TLS`, `Passthrough` is a possible mode
    certificateRefs:
    - kind: Secret
      group: ""
      name: default-cert
```

### 示例

#### 使用不同证书的监听器

在本例中，gateway 被配置为为 `foo.example.com` 和 `bar.example.com` 域提供服务。 这些域的证书已在 gateway 中指定。

```yaml
{% include 'standard/tls-basic.yaml' %}
```

#### 通配符 TLS 监听器

在本例中，gateway 配置了`*.example.com`的通配符证书和`foo.example.com`的不同证书。由于特定匹配具有优先权，因此 gateway 将为对`foo.example.com`的请求提供`foo-example-com-cert`，而为所有其他请求提供`wildcard-example-com-cert`。

```yaml
{% include 'standard/wildcard-tls-gateway.yaml' %}
```

#### 跨 namespace 证书引用

在本例中，gateway 被配置为引用不同名称空间中的证书。 目标名称空间中创建的 ReferenceGrant 允许这样做。 如果没有该 ReferenceGrant，跨名称空间引用将无效。

```yaml
{% include 'standard/tls-cert-cross-namespace.yaml' %}
```

## 上游 TLS

上游 TLS 设置是使用通过目标引用附加到 "服务 "的实验性 "BackendTLSPolicy "来配置的。

该资源可用于描述 gateway 连接后端应使用的 SNI，以及如何验证后端 Pod 提供的证书。

### TargetRefs 和 TLS

BackendTLSPolicy 包含 "TargetRef "和 "TLS "的规范。 TargetRef 是必填项，用于标识 HTTPRoute 需要 TLS 的 "服务"。 TLS "配置包含必填的 "主机名 "以及 "CACertRefs "或 "WellKnownCACerts"。

主机名指的是 gateway 连接后端应使用的 SNI，必须与后端 pod 提供的证书相匹配。

CACertRefs 指的是一个或多个 PEM 编码的 TLS 证书。 如果没有要使用的特定证书，则必须将 WellKnownCACerts 设为 "System"（系统），以告诉 gateway 使用一组受信任的 CA 证书。 每个实现所使用的系统证书可能会有一些差异。 更多信息请参阅所选实现的文档。

信息 "限制"

```
- Cross-namespace certificate references are not allowed.
- Wildcard hostnames are not allowed.
```

### 示例

#### 使用系统证书

在本例中，"BackendTLSPolicy "被配置为使用系统证书与 TLS 加密的上游连接进行连接，其中支持 "dev "服务的 Pod 预计将为 "dev.example.com "提供有效证书。

```yaml
{% include 'experimental/v1alpha2/backendtlspolicy-system-certs.yaml' %}
```

#### 使用显式 CA 证书

在此示例中，"BackendTLSPolicy "被配置为使用配置映射 "auth-cert "中定义的证书来连接 TLS 加密的上游连接，其中支持 "auth "服务的 Pod 预计将提供 "auth.example.com "的有效证书。

```yaml
{% include 'experimental/v1alpha2/backendtlspolicy-ca-certs.yaml' %}
```

## 扩展

gateway TLS 配置提供了一个 "options "映射，用于为特定实施功能添加额外的 TLS 设置。 这里可引用的一些功能包括 TLS 版本限制或使用的密码。