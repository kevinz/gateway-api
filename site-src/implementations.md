<!-- TRANSLATED by md-translate -->
# 实施

本文档跟踪 gateway API 的下游实施和集成，并为其提供状态和资源参考。

我们鼓励 gateway API 的实施者和集成者更新本文档，提供有关其实施的状态信息、所涵盖的版本以及帮助用户入门的文档。

## gateway 控制器执行状态<a name="gateways"></a>

* [Acnodal EPIC]（#acnodal-epic）
* [亚马逊弹性 Kubernetes 服务](#amazon-elastic-kubernetes-service) (alpha)
* [Apache APISIX](#apisix) (beta)
* [Avi Kubernetes 操作员](#avi-kubernetes-operator) (技术预览版)
* [Azure 容器应用网关](#azure-application-gateway-for-containers) (预览版)
* [BIG-IP Kubernetes Gateway](#big-IP-kubernetes-gateway) (测试版)
* [Cilium](#cilium) （测试版）
* [轮廓](#contour) (测试版)
* [Easegress](#easegress) (GA)
* [Emissary-Ingress (Ambassador API Gateway)](#emissary-ingress-ambassador-api-gateway) (alpha)
* [特使网关](#envoy-gateway) (beta)
* [Flomesh 服务网格](#flomesh-service-mesh-fsm) (beta)
* [Gloo Gateway 2.0](#gloo-gateway) (beta)
* [谷歌 Kubernetes 引擎](#google-kubernetes-engine) (GA)
* [HAProxy ingress](#haproxy-ingress) (alpha)
* [HashiCorp Consul]（#hashicorp-consul）
* [Istio](#istio) (beta)
* [Kong](#kong) (GA)
* [隈](#kuma) (测试版)
* [LiteSpeed ingress 控制器](#litespeed-ingress-controller)
* [NGINX Gateway Fabric](#nginx-gateway-fabric) (GA)
* [STUNner](#stunner) (beta)
* [Traefik](#traefik) (alpha)
* [Tyk](#tyk) （进行中）
* [WSO2 APK](#wso2-apk) (GA)

### 服务网格实施状况<a name="meshes"></a>

* [Istio](#istio)（实验性）
* [Kuma](#kuma)（试验性）
* [Linkerd](#linkerd) （实验性）

## 整合<a name="integrations"></a>

* [证书管理器](#cert-manager) (alpha)
* [证书管理器](#cert-manager) (alpha)
* [argo-rollouts](#argo-rollouts) (alpha)
* [Knative](#knative) (alpha)
* [Kuadrant](#kuadrant) （进行中）

## 实施

在本节中，您可以找到博客文章、文档和其他 gateway API 参考资料的具体链接，以了解具体实施情况。

### Acnodal EPIC

[EPIC](https://www.epic-gateway.org/)是一个使用Kubernetes设计和构建的开源外部网关平台。它由网关集群、k8s网关控制器、独立的Linux网关控制器和网关服务管理器组成。它们共同创建了一个向集群用户提供网关服务的平台。 每个网关由运行在网关集群而非工作负载集群上的多个Envoy实例组成。网关服务管理器是一个简单的用户管理和用户界面，可用于为公共和私有集群实施网关即服务基础设施，并集成非k8s终端。

* [文档](https://www.epic-gateway.org/)
* [源码库](https://github.com/epic-gateway)

### 亚马逊弹性 Kubernetes 服务

[Amazon Elastic Kubernetes Service (EKS)](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html) 是一项托管服务，您可以使用它在 AWS 上运行 Kubernetes，而无需安装、操作和维护自己的 Kubernetes 控制平面或节点。EKS 的网关 API 通过 [AWS Gateway API Controller](https://github.com/aws/aws-application-networking-k8s) 实现，该控制器为 EKS 集群中的网关、HTTPRoute 提供 [Amazon VPC Lattice](https://aws.amazon.com/vpc/lattice/) 资源。

### APISIX

[Apache APISIX](https://apisix.apache.org/)是一个动态、实时、高性能的 API Gateway。APISIX 提供丰富的流量管理功能，如负载平衡、动态上游、金丝雀发布、断路、身份验证、可观察性等。

Apache APISIX 目前为其[Apache APISIX Ingress Controller](https://github.com/apache/apisix-ingress-controller)支持 gateway API `v1beta1` 版本的规范。

### Avi Kubernetes 操作员

[Avi Kubernetes Operator (AKO)](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/1.11/Avi-Kubernetes-Operator-Guide/GUID-1E86BD1A-899F-40C2-A931-29310B2B340F.html)利用 VMware NSX Advanced Load Balancer 提供 L4-L7 负载平衡。

从 AKO 版本 [v1.11.1](https://github.com/vmware/load-balancer-and-ingress-services-for-kubernetes)开始，支持 gateway API v0.6.2 版本。它实现了支持 GatewayClass、Gateway 和 HTTPRoute 对象的 v1beta1 版本的 Gateway API 规范。 AKO Gateway API 目前处于技术预览版中。

部署和使用 AKO Gateway API 的文档可在 [Avi Kubernetes Operator Gateway API](https://docs.vmware.com/en/VMware-NSX-Advanced-Load-Balancer/1.11/Avi-Kubernetes-Operator-Guide/GUID-84BD68AB-B96F-425C-8323-3A249D6AC8B2.html)上被引用。

#### Azure 容器应用网关

[Application Gateway for Containers](https://aka.ms/appgwcontainers/docs)是一个可管理的应用程序（第 7 层）负载平衡解决方案，为在 Azure 的 Kubernetes 集群中运行的工作负载提供动态流量管理功能。请按照[快速入门指南](https://learn.microsoft.com/azure/application-gateway/for-containers/quickstart-deploy-application-gateway-for-containers-alb-controller)部署 ALB 控制器并开始使用 Gateway API。

容器应用网关实现了网关 API 的 "v1beta1 "规范。

### BIG-IP Kubernetes Gateway

[BIG-IP Kubernetes Gateway](https://gateway-api.f5se.io/)是一个开源项目，提供了使用[F5 BIG-IP](https://f5.com)作为数据平面的 Gateway API 实现。它为企业提供了高性能的 Gateway API 实现。

我们正在积极支持 gateway API 的各种功能。 有关与 Gateway API 功能的兼容性，请参阅 [here](https://github.com/f5devcentral/bigip-kubernetes-gateway/blob/master/docs/gateway-api-compatibility.md)。有关本项目的任何问题，欢迎创建 [Issues](https://github.com/f5devcentral/bigip-kubernetes-gateway/issues) 或 [PR](https://github.com/f5devcentral/bigip-kubernetes-gateway/pulls)。此外，也欢迎您在 [slack channel](https://gateway-api.f5se.io/Support-and-contact/) 中与我们联系。

### 纤毛虫

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-Cilium-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/cilium.yaml)

[Cilium](https://cilium.io)是一种基于eBPF的网络、可观测性和安全解决方案，适用于Kubernetes和其他网络环境。它包括[Cilium Service Mesh](https://docs.cilium.io/en/stable/gettingstarted/#service-mesh)，这是一种高效的网状数据平面，可在[无侧车模式](https://isovalent.com/blog/post/cilium-service-mesh/)下运行，以显著提高性能，并避免侧车的操作复杂性。Cilium还支持侧车代理模式，为用户提供选择。从[Cilium 1.14](https://isovalent.com/blog/post/cilium-release-114/)开始，Cilium支持gateway API，并通过了v0.7.1的一致性测试。

Cilium 是开源项目，也是 CNCF 毕业生项目。

如果您有关于 Cilium Service Mesh 的问题，可以从 [Cilium Slack](https://cilium.io/slack) 上的 #service-mesh 频道开始。如果您想为开发工作做出贡献，可以查看 #development 频道或参加我们的 [每周开发人员会议](https://github.com/cilium/cilium#weekly-developer-meeting)。

### 轮廓

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v0.8.1-Contour-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v0.8.1/projectcontour-contour.yaml)

[Contour](https://projectcontour.io)是一个 CNCF 开源的基于 Envoy 的 Kubernetes ingress 控制器。

Contour [v1.27.0](https://github.com/projectcontour/contour/releases/tag/v1.27.0) 实现了 Gateway API v0.8.1，支持 v1alpha2 和 v1beta1 API 版本。支持所有 [Standard channel](https://gateway-api.sigs.k8s.io/concepts/versioning/#release-channels-eg-experimental-standard) 资源（GatewayClass、Gateway、HTTPRoute、ReferenceGrant）以及 TLSRoute、TCPRoute 和 GRPCRoute。Contour 的实现通过了 v0.8.1 发布中包含的所有核心和大部分扩展自 Gateway API 一致性测试。

有关如何部署和使用 Contour 的 gateway API 实现，请参阅 [Contour Gateway API Guide](https://projectcontour.io/docs/1.27/guides/gateway-api/)。

如需 Contour 实施方面的帮助和支持，请[创建问题](https://github.com/projectcontour/contour/issues/new/choose) 或在[Kubernetes slack 上的 #contour 频道](https://kubernetes.slack.com/archives/C8XRH2R4J) 寻求帮助。

_某些 "扩展 "功能尚未实现，[欢迎贡献！](https://github.com/projectcontour/contour/blob/main/CONTRIBUTING.md)._

### Easegress

[Easegress](https://megaease.com/easegress/)是一个云本地流量协调系统。

它可以作为一个复杂的现代网关、一个强大的分布式集群、一个灵活的流量协调器，甚至是一个可访问的服务网格。

Easegress 目前支持 [GatewayController](https://github.com/megaease/easegress/blob/main/docs/04.Cloud-Native/4.2.Gateway-API.md) 的 Gateway API `v1beta1` 版本规范。

###使者-ingress（大使 API 网关）

[Emissary-Ingress](https://www.getambassador.io/docs/edge-stack)（前身为 Ambassador API Gateway）是一个开源 CNCF 项目，它为 Kubernetes 提供了一个建立在 [Envoy Proxy](https://envoyproxy.io) 基础上的 ingress 控制器和 API gateway。有关与 Emissary 一起使用 Gateway API 的更多详情，请参阅 [此处](https://www.getambassador.io/docs/edge-stack/latest/topics/using/gateway-api/)。

### Envoy Gateway

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-EnvoyGateway-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/envoy-gateway.yaml)

[Envoy Gateway](https://gateway.envoyproxy.io/)是[Envoy](https://github.com/envoyproxy)的一个子项目，用于管理基于 Envoy 的应用网关。[这里](https://gateway.envoyproxy.io/v0.6.0/user/gatewayapi-support)概述了 Gateway API 支持的 API 和字段。使用 [quickstart](https://gateway.envoyproxy.io/v0.6.0/user/quickstart) 只需几个简单的步骤，就能让 Envoy Gateway 通过 Gateway API 运行。

### Flomesh 服务网格（FSM）

[Flomesh Service Mesh](https://github.com/flomesh-io/fsm)是一个社区驱动的轻量级服务网格，用于Kubernetes的东西向和南北向流量管理。 Flomesh使用[ebpf](https://www.kernel.org/doc/html/latest/bpf/index.html)进行第4层流量管理，使用[pipy](https://flomesh.io/pipy)代理进行第7层流量管理。 Flomesh捆绑了负载平衡器、跨集群服务注册/发现，并支持多集群联网。它支持 "ingress"（因此是一个 "ingress控制器"）和gateway API。

FSM 对 gateway API 的支持建立在 [Flomesh Gateway API](fgw) 的基础之上，目前支持 Kubernetes Gateway API 版本 [v0.7.1](https://github.com/kubernetes-sigs/gateway-api/releases/tag/v0.7.1) ，对 `v0.8.0` 的支持目前正在进行中。

* [FSM Kubernetes Gateway API 兼容性矩阵](https://github.com/flomesh-io/fsm/blob/main/docs/gateway-api-compatibility.md)
* [如何在 FSM 中使用 gateway API 支持](https://github.com/flomesh-io/fsm/blob/main/docs/tests/gateway-api/README.md)

#### Gloo Gateway

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-GlooGateway-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/gloo-gateway.yaml)

由 [Solo.io](https://www.solo.io) 提供的 [Gloo Gateway](https://docs.solo.io/gloo-gateway/v2) 是一个功能丰富、Kubernetes 原生 ingress 控制器和下一代 API 网关。Gloo Gateway 2.0 将 Gateway API 的全部功能和社区支持带到其现有的控制平面实现中。

### 谷歌 Kubernetes 引擎

[Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine)是谷歌云提供的一个托管Kubernetes平台。GKE通过[GKE Gateway controller](https://cloud.google.com/kubernetes-engine/docs/concepts/gateway-api)实现Gateway API，该控制器为GKE集群中的Pod提供谷歌云负载平衡器。

GKE Gateway 控制器支持权重流量分割、镜像、高级路由、多集群负载平衡等功能。 请参阅文档，部署[私有或公共 gateway](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-gateways)，以及[多集群 gateway](https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-multi-cluster-gateways)。

### HAProxy ingress

[HAProxy Ingress](https://haproxy-ingress.github.io/) 是 HAProxy 的社区驱动 ingress 控制器实现。

HAProxy Ingress v0.13 部分支持 gateway API 的 v1alpha1 规范。请参阅[控制器的 gateway API 文档](https://haproxy-ingress.github.io/docs/configuration/gateway-api/)，了解一致性和路线图。

### HashiCorp Consul

[Consul](https://consul.io)由[HashiCorp](https://www.hashicorp.com)推出，是一个用于多云网络的开源控制平面。单个Consul部署可跨越裸机、虚拟机和容器环境。

Consul 服务网格可在任何 Kubernetes 发行版上运行，连接多个集群，Consul CRD 提供 Kubernetes 原生工作流来管理网格中的流量模式和权限。[Consul API Gateway](https://www.consul.io/docs/api-gateway)支持用于管理南北流量的 Gateway API。

有关 gateway API 的支持版本和功能的最新信息，请参阅 [Consul API Gateway documentation](https://www.consul.io/docs/api-gateway)。

### Istio

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-Istio-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/istio-istio.yaml)

[Istio](https://istio.io)是一个开源的[服务网格](https://istio.io/latest/docs/concepts/what-is-istio/#what-is-a-service-mesh)和 gateway 实现。

Istio 的最小安装可用于为集群 ingress 流量控制提供完全兼容的 Kubernetes Gateway API 实现。 对于服务网格用户，Istio 还完全支持网格内的 [GAMMA initiative's](/concepts/gamma/) experimental Gateway API [support for east-west traffic management](/concepts/gamma/)。

Istio 的大部分文档，包括所有的[ingress 任务](https://istio.io/latest/docs/tasks/traffic-management/ingress/) 和几个 mesh 内部流量管理任务，都已经包含了使用 gateway API 或 Istio 配置 API 配置流量的并行说明。请查看[Gateway API 任务](https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/) 了解有关 Istio 中 Gateway API 实现的更多信息。

#### 香港

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-Kong%20Ingress%20Controller-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/kong-kubernetes-ingress-controller.yaml)

[Kong](https://konghq.com)是专为混合云和多云环境打造的开源 API gateway。

Kong 支持 [Kong Kubernetes Ingress Controller (KIC)](https://github.com/kong/kubernetes-ingress-controller)中的 gateway API，使用信息请参阅 [Gateway API Guide](https://docs.konghq.com/kubernetes-ingress-controller/latest/guides/using-gateway-api/)。

Kong 还支持 [Kong Gateway Operator](https://docs.konghq.com/gateway-operator/latest/) 中的 Gateway API。

如需有关 Kong 实现的帮助和支持，请随时[创建问题](https://github.com/Kong/kubernetes-ingress-controller/issues/new) 或[讨论](https://github.com/Kong/kubernetes-ingress-controller/discussions/new)。您也可以在[Kubernetes slack 上的 #kong 频道](https://kubernetes.slack.com/archives/CDCA87FRD)中寻求帮助。

### Kuma

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v0.8.0-Kuma-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v0.8.0/kumahq-kuma.yaml)

[Kuma](https://kuma.io)是一个开源服务网格。

Kuma 为 Kuma 内置的、基于 Envoy 的 Gateway 实施了 Gateway API 规范，并提供测试版稳定性保证。有关如何使用 Gateway API 设置 Kuma 内置网关的信息，请查阅 [Gateway API 文档](https://kuma.io/docs/latest/explore/gateway-api/)。

Kuma 2.3 及更高版本支持[GAMMA 计划的](/concepts/gamma/) 实验性 gateway API [支持网状结构内的东西向流量管理](/concepts/gamma/)。

### Linkerd

[Linkerd](https://linkerd.io/)是第一个通过CNCF认证的[服务网格](https://buoyant.io/service-mesh-manifesto)。它是唯一一个不基于Envoy的主要网格，而是依赖于专门构建的Rust微代理，为Kubernetes带来安全性、可观察性和可靠性，但并不复杂。

Linkerd 2.14 及更高版本支持[GAMMA 计划的](/concepts/gamma/) 实验性 gateway API [支持网状结构内的东西向流量管理](/concepts/gamma/)。

### LiteSpeed ingress 控制器

LiteSpeed Ingress Controller](https://litespeedtech.com/products/litespeed-web-adc/features/litespeed-ingress-controller)使用 LiteSpeed WebADC 控制器作为 Ingress Controller 和负载平衡器来管理 Kubernetes 集群上的流量。它实现了完整的核心 Gateway API，包括 Gateway、GatewayClass、HTTPRoute 和 ReferenceGrant 以及 cert-manager 的 Gateway 功能。Gateway 已完全集成到 LiteSpeed Ingress Controller 中。

* [产品文档](https://docs.litespeedtech.com/cloud/kubernetes/)。
* [gateway特定文档](https://docs.litespeedtech.com/cloud/kubernetes/gateway)。
* 全面支持请访问 [LiteSpeed 支持网站](https://www.litespeedtech.com/support)。

### NGINX Gateway Fabric

[![一致性](https://img.shields.io/badge/Gateway%20API%20Conformance%20v1.0.0-NGINX Gateway Fabric-green)](https://github.com/kubernetes-sigs/gateway-api/blob/main/conformance/reports/v1.0.0/nginxinc-nginx-gateway-fabric.yaml)

[NGINX Gateway Fabric](https://github.com/nginxinc/nginx-gateway-fabric)是一个开源项目，提供使用[NGINX](https://nginx.org/)作为数据平面的Gateway API实现。该项目的目标是实现核心Gateway API -- Gateway、GatewayClass、HTTPRoute、TCPRoute、TLSRoute和UDPRoute -- 为运行在Kubernetes上的应用程序配置HTTP或TCP/UDP负载平衡器、反向代理或API网关。 NGINX Gateway Fabric目前正在开发中，支持Gateway API的一个子集。

如果您对 NGINX Gateway Fabric 有任何建议或遇到问题，请在 GitHub 上[创建问题](https://github.com/nginxinc/nginx-gateway-fabric/issues/new) 或[讨论](https://github.com/nginxinc/nginx-gateway-fabric/discussions/new)。您也可以在 NGINX slack 上的[#nginx-gateway-fabric 频道](https://nginxcommunity.slack.com/channels/nginx-gateway-fabric)寻求帮助。

### STUNner

[STUNner](https://github.com/l7mp/stunner)是面向 Kubernetes 的开源云原生 WebRTC 媒体网关。STUNner 专门用于将 WebRTC 媒体流无缝接入 Kubernetes 集群，并简化了 NAT 穿越和动态媒体路由选择。同时，STUNner 还为大规模实时通信服务提供了更好的安全性和监控功能。 STUNner 数据平面向 WebRTC 客户端提供符合标准的 TURN 服务，而控制平面则支持网关 API 的子集。

STUNner 目前支持 gateway API 规范的 `v1alpha2` 版本。有关如何部署和使用 STUNner 进行 WebRTC 媒体摄取的信息，请查阅[安装指南](https://github.com/l7mp/stunner/blob/main/doc/INSTALL.md)。有关 STUNner 的所有问题、评论和错误报告，请直接发送到[STUNner 项目](https://github.com/l7mp/stunner)。

### 特拉菲克

[Traefik](https://traefik.io)是一个开源的云本地应用程序代理。

Traefik 目前支持网关 API 规范的版本 `v1alpha2` (`v0.4.x`)，请查看 [Kubernetes Gateway Documentation](https://doc.traefik.io/traefik/routing/providers/kubernetes-gateway/)，了解如何部署和使用 Traefik 的网关实现。

Traefik 目前正在努力实现 UDP 和 ReferenceGrant。 随着工作的进展，这里将提供最新状态和文档。

### Tyk

[Tyk Gateway](https://github.com/TykTechnologies/tyk)是一个云原生、开源的 API Gateway。

Tyk.io](https://tyk.io)团队正在努力实现 gateway API。您可以[在此](https://github.com/TykTechnologies/tyk-operator)跟踪该项目的进展情况。

### WSO2 APK

[WSO2 APK](https://apk.docs.wso2.com/en/latest/) 是专为 Kubernetes 环境定制的 API 管理解决方案，为企业管理其 API 提供无缝集成、灵活性和可扩展性。

WSO2 APK 实现了网关 API，包括网关和 HTTPRoute 功能。 此外，它还通过引用自定义资源 (CR) 为速率限制、身份验证/授权和分析/可观察性提供支持。

有关 gateway API 支持的版本和功能的最新信息，请参阅 [APK Gateway 文档](https://apk.docs.wso2.com/en/latest/catalogs/kubernetes-crds/)。如果您有任何疑问或想作出贡献，请随时创建 [issues or pull requests](https://github.com/wso2/apk)。加入我们的 [Discord 频道](https://discord.com/channels/955510916064092180/1113056079501332541) 与我们联系并参与讨论。

## 整合

在本节中，您可以找到博客文章、文档和其他 gateway API 参考资料的具体链接，以了解特定集成的情况。

### 旗手

[Flagger](https://flagger.app)是一款渐进式交付工具，可自动执行在 Kubernetes 上运行的应用程序的发布流程。

Flagger 可用于使用 gateway API 自动进行金丝雀部署和 A/B 测试。 它支持 gateway API 的 `v1alpha2` 和 `v1beta1` 规范。您可以引用 [本教程](https://docs.flagger.app/tutorials/gatewayapi-progressive-delivery)，在任何 gateway API 实现中使用 Flagger。

#### cert-manager

[cert-manager](https://cert-manager.io/)是一款在云本地环境中自动管理证书的工具。

cert-manager 可以为 gateway 资源生成 TLS 证书。 这是通过为 gateway 添加 Annotations 来配置的。 它目前支持 gateway API 的 `v1alpha2` 规范。你可以参考 [cert-manager docs](https://cert-manager.io/docs/usage/gateway/) 来试用它。

### Argo 推出

[Argo Rollouts](https://argo-rollouts.readthedocs.io/en/stable/)是Kubernetes的渐进式交付控制器。它支持蓝/绿和金丝雀等多种高级部署方法。Argo Rollouts通过[一个插件](https://github.com/argoproj-labs/rollouts-gatewayapi-trafficrouter-plugin/)支持gateway API。

### Knative

[Knative](https://knative.dev/)是一个基于 Kubernetes 构建的无服务器平台。Knative Serving 为运行无状态容器提供了一个简单的 API，具有自动管理 URL、修订之间的流量分割、基于请求的自动扩展（包括扩展到零）和自动 TLS 供应等功能。Knative Serving 通过插件架构支持多个 HTTP 路由器，其中包括一个[gateway API 插件](https://github.com/knative-sandbox/net-gateway-api)，由于并不支持 Knative 的所有功能，该插件目前还处于 alpha 阶段。

### Kuadrant

[Kuadrant](https://kuadrant.io/)是一个开源的多集群 gateway API 控制器，可与其他 gateway API Provider 集成并为其提供策略。

Kuadrant 支持网关 API，用于集中定义网关，并附加适用于多集群环境中所有网关实例的 DNS、TLS、Auth 和 Rate Limiting 等策略。 Kuadrant 将 Istio 作为相应的网关提供商，并计划与 Envoy Gateway 等其他网关提供商合作。

如需有关 Kuadrant 实施的帮助和支持，请随时[创建问题](https://github.com/Kuadrant/multicluster-gateway-controller/issues/new)或在[Kubernetes slack 上的 #kuadrant 频道](https://kubernetes.slack.com/archives/C05J0D0V525)中寻求帮助。