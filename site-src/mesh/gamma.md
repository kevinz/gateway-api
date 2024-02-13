<!-- TRANSLATED by md-translate -->
# GAMMA 计划（服务网格 gateway API）

gateway API最初设计用于管理从集群外的客户端到集群内服务的流量--即[_ingress_]或[_north/south_](/concepts/glossary#northsouth-traffic)情况。 随着时间的推移，[service mesh](/concepts/glossary#service-mesh)用户的兴趣促使GAMMA（**G**ateway **A**PI for **M**esh **M**anagement and **A**dministration）计划于2022年创建，以定义如何将gateway API也引用到同一集群内的服务间或[_east/west_ traffic](/concepts/glossary#eastwest-traffic)。

GAMMA 计划是 gateway API 子项目中的一个专门工作流，而不是一个独立的子项目。 GAMMA 的目标是定义如何使用 gateway API 配置服务网格，目的是尽量减少对 gateway API 的改动，并始终保持 Gateway API [面向角色]（/概念/角色和人物）的性质。 此外，我们还努力倡导服务网格项目在实现 gateway API 时保持一致，无论其技术栈或代理如何。

## 交付成果

GAMMA 倡议的工作将在[网关增强提案](/geps/overview)中体现，这些提案将扩展或完善网关 API 规范，以涵盖网格和网格相邻用例。 迄今为止，这些变化相对较小（尽管有时会产生相对较大的影响！），我们希望这种情况继续下去。 网关 API 规范的治理仍完全由网关 API 子项目的维护者负责。

GAMMA 计划的理想最终结果是，服务网格用例成为 gateway API 的第一方关注点，届时将不再需要单独的计划。

## 捐款

我们欢迎所有级别的贡献者！为 gateway API 和 GAMMA 做出贡献有 [多种方式](/contributing/contributor-ladder)，包括技术性和非技术性贡献。

最简单的入门方法是参加 GAMMA 的例会，例会每两周举行一次，时间为周二，每次 1 小时（可在 [sig-network 日历](/contributing/community/#meetings)上找到），在太平洋时间下午 3 点和上午 8 点之间交替举行，以尽可能兼顾时区。 GAMMA 会议将由 [GAMMA 领导](https://github.com/kubernetes-sigs/gateway-api/blob/main/OWNERS_ALIASES#L23) 主持，并由一名志愿者记录。 社区成员可自由参加 GAMMA 和 gateway API 会议，但绝非必须参加。