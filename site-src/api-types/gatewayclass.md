<!-- TRANSLATED by md-translate -->
# 网关类

成功 "V0.5.0+ 版本中的标准通道"。

```
The `GatewayClass` resource is Beta and part of the Standard Channel in `v0.5.0+`.
```

[GatewayClass](/reference/spec/#gateway.networking.k8s.io/v1beta1.GatewayClass)是基础架构 Providers 定义的集群范围资源。 该资源代表可实例化的 gateway 类。

&gt; 注：GatewayClass 与 &gt; [`networking.IngressClass` 资源](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class) 的功能相同。

```yaml
kind: GatewayClass
metadata:
  name: cluster-gateway
spec:
  controllerName: "example.net/gateway-controller"
```

我们预计基础设施 Provider 将为用户创建一个或多个 "GatewayClasses"。 它允许将实现 "Gateway "的机制（如控制器）与用户解耦。 例如，基础设施 Provider 可创建名为 "internet "和 "private "的两个 "GatewayClasses"，以反映定义面向互联网与私人内部应用程序的 "Gateway"。

```yaml
kind: GatewayClass
metadata:
  name: internet
  ...
---
kind: GatewayClass
metadata:
  name: private
  ...
```

类的用户不需要知道 `internet` 和 `private` 是如何实现的，而只需要了解与 `Gateway` 一起创建的类的结果属性。

### 网关类参数

作为类定义的一部分，"gateway "API 的 Provider 可能需要将参数传递给其控制器。 这可被 "GatewayClass.spec.parametersRef "字段引用：

```yaml
# GatewayClass for Gateways that define Internet-facing applications.
kind: GatewayClass
metadata:
  name: internet
spec:
  controllerName: "example.net/gateway-controller"
  parametersRef:
    group: example.net/v1alpha1
    kind: Config
    name: internet-gateway-config
---
apiVersion: example.net/v1alpha1
kind: Config
metadata:
  name: internet-gateway-config
spec:
  ip-address-pool: internet-vips
  ...
```

我们鼓励为 `GatewayClass.spec.parametersRef`使用自定义资源，但如果需要，实现者可以使用 ConfigMap。

### 网关类状态

Provider 必须验证 `GatewayClasses` 以确保配置的参数有效。 类的有效性将通过 `GatewayClass.status`向用户发出信号：

```yaml
kind: GatewayClass
...
status:
  conditions:
  - type: Accepted
    status: False
    ...
```

一个新的 `GatewayClass` 开始时，其 `Accepted` 条件将设为 `False`。 此时控制器尚未看到配置。 一旦控制器处理了配置，该条件将设为 `True`：

```yaml
kind: GatewayClass
...
status:
  conditions:
  - type: Accepted
    status: True
    ...
```

如果 `GatewayClass.spec` 中出现错误，条件将是非空的，并包含错误信息。

```yaml
kind: GatewayClass
...
status:
  conditions:
  - type: Accepted
    status: False
    Reason: BadFooBar
    Message: "foobar" is an FooBar.
```

### 选择网关类控制器

GatewayClass.spec.controller "字段决定了负责管理 "GatewayClass "的控制器实现。 该字段的格式不透明，是特定控制器的特有格式。 特定控制器字段选择的 GatewayClass 取决于集群中各种控制器如何解释该字段。

建议控制器作者/部署使用其管理控制下的域/路径组合（例如，管理以 "example.net "开头的所有 "控制器 "的控制器是 "example.net "域的所有者），使其选择具有唯一性，以避免冲突。

控制器版本可以通过在路径部分编码控制器的版本来实现。 一个示例方案可以是（类似于容器 URI）：

```text
example.net/gateway/v1   // Use version 1
example.net/gateway/v2.1 // Use version 2.1
example.net/gateway      // Use the default version
```