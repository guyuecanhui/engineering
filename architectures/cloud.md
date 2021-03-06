# [云计算架构的几个设计原则](http://www.weidu8.net/wx/1017149467622557)


什么是架构？企业 IT 架构的设计不仅仅是注重某一个系统能力，更需要给企业一个可进化的、可持续发展、不断创新的平台。业务持续变更将成为“新常态”。架构师们既需要满足企业 IT 中高可靠、高安全、一致性、合规性要求，有需要满足创新 IT 所需要的灵活、快速、伸缩的挑战。

Gartner 给企业新的 IT 架构的标准答案是：双模 IT。既需要兼顾传统 IT 需求（稳定、合规、可靠）、有需要满足创新 IT 的需求（敏捷、试错、快速迭代）。

---

## 一、企业云架构设计基本原则
#### 原则一：不要与趋势为敌
在风口（趋势）面前，所有的阻力都显得会非常无力。当前云计算整体趋势：

1. 公有云进入快车道，逐渐成为主流；
2. 企业自建数据中心不断减少；
3. 公有云和企业 IT 会长期并存，形成混合云形态；

#### 原则二：混合云成为必然
* 私有云部分作为关键业务平台，承担敏感数据、合规性要求、交易型平台和企业核心平台。
* 公有云部分作为业务创新平台，承担交互类应用、大数据、移动化、数字化管理、生态合作及应用孵化等。

私有云和公有云之间可以进行平滑的负载迁移，在私有云高负载的情况下，部分业务可以平滑迁移到公有云部署；公有云业务随着企业管控要求，可以随时回归到私有云环境中；公有云和私有云可以进行混合型业务部署，私有云承担关键业务交易，公有云承担读写分离式的查询业务（类似于 12306）。

#### 原则三：打通任督二脉
从企业应用特点来看，企业 IT 系统的架构形态两极化发展：一种是状态要求高的交易型应用架构，为了一致性考虑设计；另一种是系统状态要求低的交互型应用，为了扩展性考虑设计。

* 交易型的强状态或重状态应用则往往对数据的一致性有很高的要求，重状态代表如Oracle RAC。
* 交互型的无状态或弱状态应用特点，通常业务链条都比较短，数据不需要强一致性保存，无状态代表如Sharding、Auto Scaling。

#### 原则四：开放自由的选择能力
作为云架构师，需要给企业选择和构建一个上下自如的云平台。可以把企业各类负载（数据库、中间件、虚拟化、数据同步等）平滑迁移上云，也可以在峰值过后，平滑地回退到企业自由数据中心中来。而不需要额外的应用代码修改，实现**虚拟化数据中心**的目标，这才是混合云架构的关键所在。

另一方面，企业对云计算有以下要求：

1. **数据主权：**要遵循合规性、法律和隐私方面的要求。敏感数据本地保存。符合其它安全保密规范要求。
2. **控制权：**对关键业务系统要保持 100% 的可控。有自己的防火墙、负载均衡器、硬件 VPN 等。非常高的 SLAs 要求。
3. **低延迟：**与后端 mainframes, databases, ERPs 等需要近乎零的低延迟链接。专用的基础设施可以提供更低的延迟。
4. **自主选择：**在本地运行云。按需在公有云或本地私有云中运行。无论公有云还是私有云，要有相同的体验。

#### 原则五：高可靠
在云计算的世界里，多数据中心的高可用架构可以借助多 Region 和 MAA (Max Availability Architecture) 架构结合方式来实现。在一个 Region 内，云厂商可以设置多个可用区（AD），在多个 AD 间提供高速的网络能力。云架构师可以轻松地选择部署多种可用性配置。例如：在不同 Region 间实现远距离的灾备部署，在一个 Region 中的不同 AD，可以部署零数据丢失的数据同步，保证当一个 AD 出现故障时，应用连续性和数据无丢失。在一个 AD 内，可以通过 RAC 类似的架构部署，来提供服务器的高可用保护能力。

## 二、云计算选择的标准
1. **性能 (Performance)：**企业需要更快的应用和分析，以更快的速度获取所需的信息。因此，卓越的性能和处理能力是企业选择云平台的关键。
2. **成本 (Cost)：**简化后的 IT 架构，能够帮助企业省下购买设备、软件和维护项目和费用，同时减少 IT 人力资源的投入。能够优化运维人力的调度，降低 IT 维护项目的人力投入，进而可将更多的资源放在促进业务需求和 IT 技术的结合，帮助企业创新。
3. **安全性 (Security)：**云计算平台在芯片、IaaS、PaaS、SaaS 各层都应部署完整的安全防护，安全控制将围绕着数据中心（物理基础设施）、访问安全、网络安全、存储安全、数据安全等多个方面展开。
4. **兼容性 (Compatibility)：**混合云是大势所趋，企业应能在不停机的状况下，自由地在云上和本地环境往返测试、部署，通过单一的管理工具就能同时管理云上和本地部署的环境。
5. **可用性 (Availability)：**云计算平台应能保证当灾难发生且一个数据中心出现问题时，另一个数据中心够实时运行，最大限度地确保企业业务的连续性。
6. **标准化 (Standard)：**云计算平台应引入业界标准，例如 SQL、Hadoop、NoSQ、Java、Ruby、Node.js、Linux、Docker 等。