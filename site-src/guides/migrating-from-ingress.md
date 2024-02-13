<!-- TRANSLATED by md-translate -->
# 从 ingress 迁移过来

gateway API 项目是[Ingress API](https://kubernetes.io/docs/concepts/services-networking/ingress/)的后续项目。不过，它不包括 ingress 资源（最接近的平行项目是 HTTPRoute）。因此，有必要将现有的 Ingress 资源一次性转换为相关的 Gateway API 资源。

本指南将帮助您完成转换：

* 解释为什么要改用 gateway API。
* 描述 Ingress API 和 Gateway API 之间的主要区别。
* 将 Ingress 功能映射到 Gateway API 功能。
* 展示将 Ingress 资源转换为 Gateway API 资源的示例。
* 提及自动转换的 [ingress2gateway](https://github.com/kubernetes-sigs/ingress2gateway)。

此外，由于 Ingress API 仅涵盖 HTTP/HTTPS 流量，本指南不包括 gateway API 对其他协议的支持。

### 改用 gateway API 的原因

Ingress API](https://kubernetes.io/docs/concepts/services-networking/ingress/)是 Kubernetes 为服务配置外部 HTTP/HTTPS 负载平衡的标准方式。它被 Kubernetes 用户广泛采用，并得到了供应商的大力支持，有许多实现（[Ingress 控制器](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)）可用。此外，一些云原生项目也与 Ingress API 集成，如[cert-manager](https://cert-manager.io/) 和[ExternalDNS](https://github.com/kubernetes-sigs/external-dns)。

不过，ingress API 有几个局限性：

* 功能有限。ingress API 仅支持 TLS 终止和简单的基于内容的 HTTP 流量请求路由。
* 依赖注解实现可扩展性_。Annotations 方法的可扩展性导致可移植性受限，因为每个实现都有自己支持的扩展，而这些扩展可能无法转换到任何其他实现。
* 权限模型不足_。ingress API 并不适合具有共享负载平衡基础设施的多团队集群。

gateway API 解决了这些限制，下一节将对此进行说明。

&gt; 阅读有关 gateway API 的 [设计目标](/#gateway-api-concepts) &gt; 的更多信息。

## ingress API 和 gateway API 的主要区别

ingress API 和 gateway API 有三大不同：

* 角色
* 可用功能
* 可扩展性方法（针对具体实施的功能）

### 个性

起初，Ingress API只有一种资源，即Ingress。 因此，它只有一个角色--用户--Ingress资源的所有者。 Ingress的功能让用户对应用程序如何暴露给其外部客户端有很大的控制权，包括TLS终止配置和负载平衡基础设施的供应（由一些Ingress控制器支持）。 这样的控制水平被称为自助服务模式。

与此同时，Ingress API 还包含了两个隐式角色来描述负责配置和管理 Ingress 控制器的人：由 Provider 管理的 Ingress 控制器的基础设施提供者和自托管 Ingress 控制器的集群操作员（或管理员）。随着 [IngressClass](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class) 资源的后期添加，基础设施提供者和集群操作员成为了该资源的所有者，因此也成为了 Ingress API 的显式角色。

gateway API 包括 [四个明确的角色](/concepts/security-model/#roles-and-personas)：应用程序开发人员、应用程序管理员、集群操作员和基础设施提供者。 这样，您就可以将用户角色的职责分给这些角色（除基础设施提供者外的所有角色），从而摆脱自助服务模式：

* 集群操作员/应用程序管理员定义外部客户端流量的入口点，包括 TLS 终止配置。
* 应用程序开发人员为连接到这些入口点的应用程序定义路由规则。

这样的拆分遵循了共同的组织结构，即多个团队共享相同的负载平衡基础架构。 同时，并不强制要求放弃自助服务模式--仍然可以配置单一的 RBAC 角色，履行应用程序开发人员、应用程序管理员和集群操作员的职责。

下表总结了 ingress API 和 gateway API 角色之间的映射关系：

| Ingress API Persona | Gateway API Persona | |-|-| | 用户 | 应用程序开发人员、应用程序管理员、集群操作员 | | 集群操作员 | 基础设施提供商 | 基础设施提供商 | 基础设施提供商

#### 可用功能

Ingress API 只具备基本功能：TLS 终止和基于内容的 HTTP 流量路由（基于主机头和请求的 URI）。为了提供更多功能，Ingress 控制器通过 Ingress 资源上的 [annotations](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/)来支持这些功能，这些annotations 是对 Ingress API 的特定实现扩展。

对可扩展性采用 Annotations 方法会给 ingress API 用户带来两个负面影响：

* 可移植性受限。由于许多功能都是通过 Annotations 提供的，因此在 ingress 控制器之间切换变得困难甚至不可能，因为必须将一种实现方式的注释转换为另一种实现方式的注释（另一种实现方式甚至可能不支持第一种实现方式的某些功能）。这就限制了 ingress API 的可移植性。
* API 的笨拙性。由于 Annotations 是键值字符串（而不是像 Ingress 资源规范那样的结构化方案），并且应用在资源的顶部（而不是规范的相关部分），因此 Ingress API 在使用时可能会变得很笨拙，尤其是当大量的 Annotations 被添加到 Ingress 资源时。

gateway API 支持 Ingress 资源的所有功能以及许多只有通过注解才能实现的功能。 因此，Gateway API 比 Ingress API 更具可移植性。 此外，正如下一节将介绍的那样，您根本不需要引用任何注解，这就解决了尴尬问题。

#### 可扩展性方法

ingress API 有两个扩展点：

* Ingress 资源上的 Annotations（上一节已有描述）
* [资源后端](https://kubernetes.io/docs/concepts/services-networking/ingress/#resource-backend)，即可以指定服务以外的后端

与 ingress API 相比，gateway API 功能丰富。 不过，要配置一些高级功能（如身份验证）或常见但不可跨数据平面移植的功能（如连接超时和健康检查），您需要依赖 gateway API 的扩展。

gateway API 有以下主要扩展点：

* gateway API 资源的一个特性（字段）可以引用配置该特性的 gateway 实现所特有的自定义资源。例如
    - [HTTPRouteFilter](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteFilter)
    可以通过 `extensionRef` 字段引用外部资源，从而
    配置特定于实现的过滤器。
    - [BackendObjectReference](/reference/spec/#gateway.networking.k8s.io/v1beta1.BackendObjectReference)
    支持服务以外的资源。
    - [SecretObjectReference](/reference/spec/#gateway.networking.k8s.io/v1beta1.SecretObjectReference)
    支持 Secret 以外的资源。
* _自定义实现_。对于某些特性，可由实现来定义如何支持它们。这些特性对应于特定于实现的（自定义）[一致性级别](/concepts/conformance/#2-support-levels)。例如
    - 的 `RegularExpression` 类型。
    HTTPPathMatch](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPPathMatch) 的 `RegularExpression` 类型。
* Gateway 实现可以定义称为 Policies 的自定义资源，用于公开身份验证等数据平面功能。gateway API 并不规定这些资源的细节。不过，它规定了标准用户体验。有关详细信息，请参阅[策略附件指南]（/reference/policy-attachment/）。与上述_外部引用_不同，gateway API 资源并不引用政策。相反，政策必须引用 gateway API 资源。

扩展点不包括对 gateway API 资源的 Annotations。 对于 API 的实现，强烈不建议采用这种方法。

## 将 ingress API 功能映射到 gateway API 功能

本节将把 ingress API 功能映射到相应的 gateway API 功能，主要涉及三个方面：

* 入口点
* TLS 终止
* 路由规则

#### 入口点

粗略地说，入口点是 IP 地址和端口的组合，外部客户端可通过它访问数据平面。

每个 Ingress 资源都有两个隐式入口点，一个用于 HTTP 流量，另一个用于 HTTPS 流量。 Ingress 控制器提供这些入口点。 通常情况下，所有 Ingress 资源都会共享这些入口点，或者每个 Ingress 资源都会获得专用入口点。

在 Gateway API 中，必须在 [Gateway](/api-types/gateway/) 资源中明确定义入口点。 例如，如果要让数据平面处理端口 80 上的 HTTP 流量，就需要为该流量定义一个 [监听器](/reference/spec/#gateway.networking.k8s.io/v1beta1.Listener)。 通常情况下，Gateway 实现会为每个 Gateway 资源提供一个专用数据平面。

gateway 资源归集群操作员和应用程序管理员所有。

### TLS 终止

ingress 资源通过[TLS 部分](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls) 支持 TLS 终止，TLS 证书和密钥存储在 Secret 中。

在 gateway API 中，TLS 终止是 [gateway 监听器] 的属性（/reference/spec/#gateway.networking.k8s.io/v1beta1.Listener），与 ingress 类似，TLS 证书和密钥也存储在 Secret 中。

由于监听器是 gateway 资源的一部分，因此集群操作员和应用程序管理员拥有自己的 TLS 终结。

#### 路由规则

ingress 资源的[基于路径的路由规则](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types) 直接映射到[HTTPRoute](/api-types/httproute/) 的[路由规则](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteRule)。

基于主机头的路由规则](https://kubernetes.io/docs/concepts/services-networking/ingress/#name-based-virtual-hosting) 映射到 HTTPRoute 的[hostnames](/reference/spec/#gateway.networking.k8s.io/v1beta1.Hostname)。不过请注意，在 ingress 中，每个主机名都有单独的路由规则，而在 HTTPRoute 中，路由规则适用于所有主机名。

&gt; 本指南将使用 gateway API 术语来引用 ingress 主机。

&gt; HTTPRoute 的 "主机名 "必须与 [gateway 监听器](/reference/spec/#gateway.networking.k8s.io/v1beta1.Listener) 的 "主机名 "匹配。 &gt; 否则，监听器将忽略未匹配的 &gt; 主机名的路由规则。 请参阅 [HTTPRoute 文档](/reference/spec/#gateway.networking.k8s.io/v1beta1.HTTPRouteSpec)。

HTTPRoutes 由应用程序开发人员拥有。

接下来的三节将介绍 ingress 路由规则的其他功能。

#### 规则合并与冲突解决

通常情况下，ingress 控制器会合并来自所有ingress 资源的路由规则（除非它们为每个ingress 资源提供一个数据平面），并解决规则之间的潜在冲突。 不过，合并和冲突解决都不是由 Ingress API 规定的，因此ingress 控制器可能会以不同的方式实现它们。

而 gateway API 则规定了如何合并规则和解决冲突：

* gateway 实现必须合并连接到监听器的所有 HTTPRoutes 的路由规则。
* 必须按照 [here](/concepts/guidelines/#conflicts) 的规定处理冲突。例如，路由规则中更具体的匹配会优先于不太具体的匹配。

#### 默认后端

Ingress [default backend](https://kubernetes.io/docs/concepts/services-networking/ingress/#default-backend)配置了一个后端，该后端将响应与该 Ingress 资源相关的所有未匹配 HTTP 请求。 Gateway API 没有直接对应的规则：必须明确定义这样的路由规则。例如，定义一条规则，将路径前缀为`/`的请求路由到与默认后端相对应的服务。

#### 选择要连接的数据平面

Ingress 资源必须指定一个 [class](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class)，以选择要使用的 Ingress 控制器。HTTPRoute 必须通过 [parentRef](/reference/spec/#gateway.networking.k8s.io/v1beta1.ParentRef) 指定要连接到哪个 gateway（或多个 gateway）。

### 具体实施的 ingress 功能（Annotations）

Ingress Annotations 配置特定于实现的功能。 因此，将它们转换为 gateway API 既取决于 Ingress 控制器，也取决于 gateway 实现。

幸运的是，通过 Annotations 支持的一些功能现在主要是 gateway API（HTTPRoute）的一部分：

* 请求重定向（包括 TLS 重定向）
* 请求/响应操纵
* 流量分割
* 基于头、查询参数或方法的路由选择

要转换这些功能，请查阅 gateway 实现文档，了解应使用哪个 [扩展点](#approach-to-extensibility)。

## 示例

本节举例说明如何将 ingress 资源转换为 gateway API 资源。

#### 假设

该示例包括以下假设：

* 所有资源都属于同一个 namespace。
* ingress 控制器：
    - 在集群中拥有相应的 IngressClass 资源 `prod`。
    - 通过
    example-ingress-controller.example.org/tls-redirect`注解支持 TLS 重定向功能。
* Gateway 实现：在集群中拥有相应的 GatewayClass 资源 `prod`。

此外，为简洁起见，省略了引用的 Secret 和 Services 以及 IngressClass 和 GatewayClass 的内容。

#### ingress 资源

下面的 ingress 定义了以下配置：

* 使用 `example-ingress-controller.example.org/tls-redirect` 注解，为任何针对 `foo.example.com` 和 `bar.example.com` 主机名的 HTTP 请求配置 TLS 重定向。
* 使用来自 Secret `example-com` 的 TLS 证书和密钥，为`foo.example.com`和`bar.example.com`主机名终止 TLS。
* 将对 `foo.example.com` 主机名的 HTTPS 请求（URI 前缀为 `/orders`）路由到 `foo-orders-app` 服务。
* 将对主机名为 `foo.example.com` 并带有任何其他前缀的 HTTPS 请求路由到 `foo-app` 服务。
* 将包含任何 URI 的 `bar.example.com` 主机名的 HTTPS 请求路由至 `bar-app` 服务。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    some-ingress-controller.example.org/tls-redirect: "True"
spec:
  ingressClassName: prod
  tls:
  - hosts:
    - foo.example.com
    - bar.example.com
    secretName: example-com
  rules:
  - host: foo.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: foo-app
            port:
              number: 80
      - path: /orders
        pathType: Prefix
        backend:
          service:
            name: foo-orders-app
            port:
              number: 80
  - host: bar.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: bar-app
            port:
              number: 80
```

接下来的三节将把 ingress 转换成 gateway API 资源。

#### 转换步骤 1 - 定义 gateway

以下是 gateway 资源：

* 属于网关类 `prod`。
* 提供负载平衡基础设施（这取决于 gateway 的实现）。
* 配置 HTTP 和 HTTPS 监听器（入口点），ngress 资源隐含地包含这些监听器：
    - 端口为 `80` 的 HTTP 监听器 `http
    - HTTPS 监听器 "https"，端口为 "443"，使用 TLS 终止，证书和密钥存储在
    证书和密钥存储在 "example-com "秘密中，该秘密与 Ingress
    中被引用的 Secret

此外，请注意这两个侦听器都允许来自同一 namespace 的所有 HTTPRoutes（这是默认设置），并将 HTTPRoute 主机名限制为 `example.com` 子域（允许 `foo.example.com` 等主机名，但不允许 `foo.kubernetes.io`）。

```yaml
{% include 'standard/simple-http-https/gateway.yaml' %}
```

#### 转换步骤 2 - 定义 HTTPRoutes

ingress 分成两个 HTTPRoutes -- 一个用于 `foo.example.com` 主机名，另一个用于 `bar.example.com` 主机名。

```yaml
{% include 'standard/simple-http-https/foo-route.yaml' %}
```

```yaml
{% include 'standard/simple-http-https/bar-route.yaml' %}
```

都是 HTTPRoutes：

* 连接到步骤 1 中 gateway 资源的`https`监听器。
* 为相应主机名定义与 ingress 规则中相同的路由规则。

#### 第 3 步 - 配置 TLS 重定向

下面的 HTTPRoute 配置了 TLS 重定向，ingress 资源通过 Annotations 配置了该重定向。 下面的 HTTPRoute：

* 附加到 gateway 的 `http` 监听器。
* 对任何指向 `foo.example.com` 或 `bar.example.com` 主机名的 HTTP 请求发出 TLS 重定向。

```yaml
{% include 'standard/simple-http-https/tls-redirect-route.yaml' %}
```

## 自动转换 ingress

Ingress to Gateway](https://github.com/kubernetes-sigs/ingress2gateway) 项目有助于将 Ingress 资源转换为 gateway API 资源，特别是 HTTPRoutes。转换结果应始终经过测试和验证。