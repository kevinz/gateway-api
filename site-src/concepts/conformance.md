<!-- TRANSLATED by md-translate -->
# 一致性

该应用程序接口涵盖了广泛的功能和用例，并得到了广泛的实施。 这种既有大量功能集又有多种实施方式的组合需要明确的一致性定义和测试，以确保无论在哪里使用该应用程序接口，都能提供一致的体验。

在考虑 gateway API 一致性时，有三个重要概念：

## 1. 发布通道

在 gateway API 中，发布通道用于表示字段或资源的稳定性。 API 的 "标准 "通道包括已升级到 "测试版 "的字段和资源。 API 的 "实验 "通道包括 "标准 "通道中的所有内容，以及仍可能以破坏性方式**或完全删除**的实验字段和资源。 有关此概念的更多信息，请引用我们的 [versioning](/concepts/versioning) 文档。

## 2.

遗憾的是，有些应用程序接口的实现无法支持所定义的所有功能，为此，应用程序接口为每个功能定义了相应的支持级别：

* **核心***功能将是可移植的，我们希望所有实现都有一个合理的路线图，以支持这类应用程序接口。
* **扩展自**功能是指那些可移植但在不同实现中不被普遍支持的功能。支持该功能的实现将具有相同的行为和语义。预计一些路线图功能最终会迁移到核心功能中。扩展功能将成为 API 类型和模式的一部分。
* **特定于实施的***功能是指那些不可移植且特定于供应商的功能。除非通过通用扩展点，否则特定于实现的功能将不具有 API 类型和模式。

核心集和扩展集中的行为和功能将通过行为驱动的一致性测试来定义和验证。 一致性测试将不涵盖特定于实现的功能。

通过在 API 规范中加入扩展功能并使之标准化，我们希望能够在不影响整体 API 支持的情况下，在不同的实现中汇聚可移植的 API 子集。 缺乏普遍支持不会成为开发可移植功能集的障碍。 规范标准化将使我们更容易在支持普及时最终升级到 Core。

### 重叠支持水平

特定字段的支持级别有可能重叠。 出现这种情况时，应解释所表达的最低支持级别。 例如，一个相同的结构体可能被嵌入到两个不同的地方。 在其中一个地方，该结构体被认为具有核心支持，而另一个地方只包含扩展支持。 该结构体中的字段可以表达单独的核心和扩展支持级别，但这些级别不得被解释为超出了它们所嵌入的父结构体的支持级别。

举个更具体的例子，HTTPRoute 对在规则中定义的过滤器提供核心支持，而在 BackendRef 中定义的过滤器则提供扩展支持。 这些过滤器可分别为每个字段定义支持级别。 在解释重叠的支持级别时，应解释最小值。 这意味着，如果某个字段的支持级别是核心级别，但在过滤器中附加的支持级别是扩展级别，则解释的支持级别必须是扩展级别。

## 3. 一致性测试

gateway API 包含一系列一致性测试，这些测试会创建一系列具有指定 gatewayClass 的网关和路由，并测试其实现是否符合 API 规范。

目前，一致性测试涵盖了标准通道中的大部分核心功能，以及一些扩展功能。

#### 运行测试

一致性测试主要有两套对比测试：

* 网关相关测试（也可视为 ingress 测试）
* 服务网格相关测试

对于 "Gateway "测试，您必须启用 "Gateway "测试功能，然后选择运行任何其他特定测试（如 "HTTPRoute"）。 对于与 Mesh 相关的测试，您必须启用 "Mesh"。

我们将分别介绍每种用例，但如果您的实现同时实现了这两种用例，也可以将它们结合起来。 还有一些选项与整个测试套件有关，无论您运行的是哪种测试。

#### gateway 测试

默认情况下，面向 `Gateway` 的一致性测试将在集群中安装名为 `gateway-conformance` 的 GatewayClass，测试将针对该类运行。 通常情况下，您会使用不同的类，可以使用 `-gateway-class` flag 和相应的测试命令来指定该类。 检查您的实例，查看要使用的 `gateway-class` 名称。 您还必须启用 `Gateway` 支持，并为您的实现所支持的任何 `*Routes` 提供测试支持。

下面将运行与 `Gateway`、`HTTPRoute` 和 `ReferenceGrant` 有关的所有测试：

```shell
go test ./conformance/... -args \
    -gateway-class=my-gateway-class \
    -supported-features=Gateway,HTTPRoute
```

其他有用的 flag 可以在 [conformance flags](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/utils/flags/flags.go) 中被引用。

#### 网目测试

只需启用 "网格 "功能，即可运行网格测试：

```shell
go test ./conformance/... -args -supported-features=Mesh
```

如果您的网格还包括使用 API（如 `HTTPRoute`）的 ingress 支持，您可以通过启用 `Gateway` 功能和任何相关 API 功能（如 `HTTPRoute`），在同一测试运行中运行相关测试：

```shell
go test ./conformance/... -args -supported-features=Mesh,Gateway,HTTPRoute
```

#### 命名空间标签和 Annotations

如果需要在用于测试的 namespace 上设置标签，可以使用 `-namespace-labels` 标志传递一个或多个 `name=value` 标签，以便在测试 namespace 上设置标签。 同样，可以使用 `-namespace-annotations` 指定要应用于测试 namespace 的注释。 对于网格测试，如果实现需要在承载网格工作负载的 namespace 上设置标签（例如，启用 sidecar 注入），则可以使用此标志。

例如，在测试 Linkerd 时，您可以运行

```shell
go test ./conformance/... -args \
   ...
   -namespace-annotations=linkerd.io/inject=enabled
```

以便将测试 namespace 正确注入网格。

#### 不包括测试

默认情况下，"gateway "和 "ReferenceGrant "功能是启用的。 您无需使用"-supported-features"（支持的功能）flag 明确列出这些功能。 不过，如果您不想运行这些功能，则需要使用"-exempt-features"（豁免的功能）flag 禁用它们。 例如，只运行 "Mesh "测试，而不运行其他测试：

```shell
go test ./conformance/... -args \
    -supported-features=Mesh \
    -exempt-features=Gateway,ReferenceGrant
```

#### 套房级选项

在运行任何类型的测试时，您可能不希望测试套件在测试完成后清理测试资源（例如，这样您就可以在测试失败时检查集群状态）。 您可以使用以下方法跳过清理：

```shell
go test ./conformance/... -args -cleanup-base-resources=false
```

使用测试的 `ShortName` 来运行一个非常具体的测试可能会有帮助（尤其是在实施特定功能时）：

```shell
go test ./conformance/... --run TestConformance/<ShortName>
```

## 促进一致性

许多实施者都会将一致性测试作为其完整的 e2e 测试套件的一部分来运行。 提供一致性测试意味着实施者可以分担测试开发方面的投资，并确保我们提供一致的体验。

所有与一致性相关的代码都放在项目的"/conformance "目录下。 测试定义放在"/conformance/tests "目录下，每个测试都包含一对文件。 YAML 文件包含运行测试时要应用的配置清单。 Go 文件包含确认实现是否正确处理这些配置清单的代码。

与一致性相关的问题是[标有 "区域/一致性"](https://github.com/kubernetes-sigs/gateway-api/issues?q=is%3Aissue+is%3Aopen+label%3Aarea%2Fconformance)。这些问题通常包括添加新测试以提高我们的测试覆盖率，或修复现有测试中的缺陷或限制。