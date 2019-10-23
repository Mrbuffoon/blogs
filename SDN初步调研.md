# SDN简述
### SDN是什么(WHAT)
SDN(Software Defined Networking)，即软件定义网络，它并不是某项具体的技术或者某个具体的协议，而是一个思想，一个框架，一种网络设计理念。

SDN的核心理念是，希望应用软件可以参与对网络的控制管理，满足上层业务需求，通过自动化业务部署简化网络运维。其实说通俗一点，就是把“传统软硬件网络”给软件化、抽象化了，可以通过管理平台像配置应用程序一样对网络进行配置，而不需要像以前一样，逐个去手动配置每个节点的网络设备。

SDN一般具有以下几个特点：集中式控制、控制与转发分离、可编程、开放接口、虚拟化。

SDN基本架构主要是应用层、控制层、基础设施层。
示意图：
![图片](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWRlci5zaGltby5pbS9mL1hwdUk2SGRVOWg4TGV0T2oucG5nIXRodW1ibmFpbA?x-oss-process=image/format,png)

### SDN出现的背景(WHY)
早在1995年就已经提出软件定义网络的概念，直到目前 SDN 引起广泛关注，还要得益于网络需求侧翻天覆地的变化：云计算业务（服务器虚拟化技术为代表）成为主流，移动互联网催生的大数据技术日益普及，包括网络在内的资源快速配置、弹性扩容、按需调用需求强烈。传统模式的弊端显现：网络设备硬件、操作系统和网络应用三部分紧耦合在一起，组成一个封闭系统，这三部分相互依赖、每一部分的创新和演进都要求其余部分做出同样的升级。

越来越多的网络新协议和新算法使得网络控制平面变得越来越复杂，但是现在的网络用户却对网络的易用性有更高的要求，希望网络具有更多的可编程能力，从而自动化、智能化网络管理。网络发展目前还处于“管理复杂性”阶段，这样的架构严重阻碍了网络创新进程的开展。

# SDN详述
### SDN各层拆解
* 控制层（ Control Layer ）
	控制层是 SDN 控制器管理网络的基础设施，可以根据需要灵活选择多种控制器。

	在这一层中，控制器中包含大量业务逻辑，以获取和维护不同类型的网络信息、状态详细信息、拓扑细节、统计详细信息等。

	由于 SDN 控制器是用于管理网络的，所以它必须具有用于现实世界网络使用情况的控制逻辑，如交换、路由、二层VPN、三层VPN、防火墙安全规则、DNS、DHCP和集群，**网络供应商和开源社区需要在自己的 SDN 控制器中实现自己的服务**。这些服务会向上层（应用层）公开自己的API（通常是基于 REST，这使网络管理员可以方便地使用应用程序上的 SDN 控制器的配置、管理和监控网络）。

	目前市场上的 SDN 控制器解决方案大致可以分为两类：大型网络设备厂商提供商业方案，例如 Cisco Open SDN controller, Juniper Contrail, Brocade SDN controller, 和来自 NEC 公司的 PFC SDN controller ；社区组织提供的开源方案，例如 OpenDaylight, Floodlight, Beacon, Ryu 等等。

	Commercial Solutions:

	[Cisco Open SDN Controller](https://www.cisco.com/c/en/us/products/cloud-systems-management/open-sdn-controller/index.html)

	[Juniper Contrail](https://www.juniper.net/us/en/products-services/sdn/contrail/)

	[Brocade SDN controller](http://www.luminanetworks.com/products/?utm_source=brocade&utm_medium=redirect&utm_campaign=launch)

	[PFC SDN controller(From NEC)](https://www.necam.com/sdn/Software/SDNController/)

	Open Source Solutions:

	[Beacon](https://www.sdxcentral.com/projects/beacon/)：由斯坦福大学开发，Java语言编写

	[Floodlight](http://www.projectfloodlight.org/floodlight/)：源于Beacon，Big Switch Networks开发，Java语言编写，Apache许可证

	[OpenDaylight](https://www.opendaylight.org/)：Linux基金会负责管理的开源项目，JAVA编写

	[Ryu](http://osrg.github.io/ryu/)：由 NTT 开发，Python 编写，能够与 OpenStack 平台整合，控制器API丰富

	[Mul](https://riboseyim.github.io/2017/05/12/SDN/#): 由 Kulcloud 开发，内核采用 C 语言实现的多线程架构

	[NodeFlow](https://riboseyim.github.io/2017/05/12/SDN/#): 由 Cisco 开发，基于 Node.js 的 [OpenFlow](https://riboseyim.github.io/2017/08/22/SDN-OpenFlow/) 控制器，JavaScript 编写

	[Trema](https://riboseyim.github.io/2017/05/12/SDN/#): 由 NEC 开发，Ruby/C 编写

	[NOX](https://riboseyim.github.io/2017/05/12/SDN/#): 由 Nicira 开发，C++/Python编写，业界第一款 [OpenFlow](https://riboseyim.github.io/2017/08/22/SDN-OpenFlow/) 控制器

	[POX](https://riboseyim.github.io/2017/05/12/SDN/#): 由 Nicira 开发，是 NOX 的纯 Python 实现版本，目的是提供跨平台部署的便利性

* 基础设施层（ Infrastructure Layer ）
	基础设施层，由各种网络设备构成。它可以是数据中心的一组网络交换机和路由器。控制层负责管理底层物理网络，物理层的实现可以是**支持 OpenFlow 的硬件交换机**，也可以是软件形态，例如 [Open vSwitch (OVS)](http://openvswitch.org/) 。OVS是一款基于开源技术实现的、能够与服务器虚拟化集成，具备交换机的功能，可以实现虚拟化组网。

* 应用层（ Application Layer ）
	应用层对于开发者来说是开放区域，鼓励开发尽可能多的创新应用。包括网络的可视化：拓扑结构、网络状态、网络统计等；网络自动化相关应用：网络配置管理，网络监控，网络故障排除，网络安全策略等。SDN 应用程序可以为企业和数据中心网络提供各种端到端的解决方案。

* 南向接口（ Southbound interface ）
	控制层到基础设施层（网络交换机）通讯需要经过南向接口，目前主要的协议是 OpenFlow , NetConf，OVSDB 。 OpenFlow 协议是事实上的国际行业标准，NOX 、Onix 、Floodlight 等都是基于 OpenFlow 控制协议的开源控制器。

* 北向接口（ Northbound interface ）
	北向接口：应用层通过 API 的方式与SDN 控制器通讯。与南向接口不同，现在北向接口还缺少业界公认的标准，实现方案思路有的从用户角度出发、有的从运营商角度出发、有的从产品能力角度出发。技术风格上，部分传统的网络设备厂商倾向于在现有的设备上提供编程接口供业务App调用，许多上层应用的开发者也比较倾向于采用 REST API 接口的形式。

### OpenFlow在SDN中的角色
在SDN解决方案中，openflow是其中非常关键的一个协议，它是一种具体的通信协议，而且基本上已经是事实上的行业标准了，当前业内SDN交互协议大多采用openflow。

很多人对openflow与SDN经常混淆，这主要是从历史上刊，两者确实是你中有我、我中有你。首先，作为一个开放的协议，OpenFlow 协议是众多 SDN 控制器解决方案的实现基础；另外，定义 SDN 概念和架构背后的许多重要人物开始在 OpenFlow 领域取得了突破，进而推动 SDN 概念走向成熟。

在传统的网络交换设备中，转发平面（通常采用专门的芯片以提高性能）与控制平面（分布地部署在网络的各个节点）是紧密耦合的，被集成实现在单独的设备中。当然，从另一个角度看这样的设计也有合理性，至少能提高单个节点的灵活性和容灾能力。但是众多厂商各自为政，更出于技术保密、保持市场的考虑，对于开放接口供用户调用、建立行业标准的兴趣不大。OpenFlow 协议的推出突破了传统壁垒，有利于增加用户侧的话语权，所以 Google、Facebook 等企业是 OpenFlow 协议最坚强的拥趸，他们的数据中心都在使用 OpenFlow 协议，并倡议发起成立 ONF 来推动这个技术。

OpenFlow 提供了一个在 SDN 控制器和网络设备（如交换机）之间通讯的标准协议。他允许由 SDN 控制器下发到转发规则（forwarding rules）、安全规则（security rules）到底层网络交换机，完成**路由决策、流量控制**。OpenFlow 协议相当于一种共同语言，所以SDN 控制器和交换机都需要实现OpenFlow 协议，以便他们能够理解 OpenFlow 消息（message）。

基于 OpenFlow 消息，该协议还可以支持**网络交换机监控**：为了监控网络交换机，OpenFlow 协议提供了从交换机抓取网络统计信息、事件消息的请求／响应报文，方便控制器获得从交换机一侧感知人工操作和失败信息的能力,包括流移除事件，端口状态 UP/DOWN 变化等。为了能够支持第三方厂商可以在 OpenFlow 交换机上执行特定的任务，OpenFlow 协议提供可扩展的自定义消息结构，允许控制器和交换机之间传递信息。那是怎样的 OpenFlow 被许多SDN应用程序用来提供简单的网络控制和管理解决方案。

OpenFlow 协议定义了多种消息来完成交换机和控制机通讯，例如：

* 连接设置消息（connection setup messages)
* 配置消息（configuration messages)
* 交换机统计信息消息（switch statistics messages)
* 连接监测消息（keep-alive messages)
* 异步事件消息（asynchronous events messages)
* 发生错误消息（error messages)

### SDN常见开源项目
以控制器、交换机、网络虚拟化等分类，列出众多开源项目，具体可参见下面链接：

[https://www.sdnlab.com/8091.html](https://www.sdnlab.com/8091.html)

# SDN能做什么
### 当前落地现状
SDN当前最大的一个应用场景是在网络虚拟化中的应用，其中尤其是在互联网数据中心的落地。硬件 SDN 的落地进展并不顺利，虽然也有一些落地案例，但是离大规模部署还有较远的路要走。

SDN的落地，归纳来讲，大致有两类驱动：一是业务层面灵活性的需求，二是转发层面灵活性的需求。

* 业务层面灵活性需求

这主要是强调可编程。通过开放的可编程接口，提供给用户原来无法获得的对网络配置管理和策略部署的灵活性控制。

比如，通过定制化的SDN交换机，使得用户可以在界面上按需调整自己的出口带宽。再比如某些场景下网络拓扑和策略是易变的，这也需要通过SDN来提供这种灵活性，实现网络拓扑的灵活配置甚至动态调整。

* 转发层面灵活性需求

这主要是针对一些非常特定的场景，主要是为了匹配或者修改特定字段，通常是传统交换机不支持的（其实芯片也许能支持，只是交换机系统没做）。比如用来做 DDoS 防攻击（日本 Sakura Internet 的应用），用来做负载均衡 + NAT，用来做 TAP 应用（价格是专业的 TAP 设备的至少 1/5），用来将 PPPoE 跟 IP 区分开并灵活控制等等。这类应用主要的灵活性体现在转发面上而不是控制面。

贴几个文档，阐述了当前SDN落地的几个主要场景。

[https://www.infoq.cn/article/sdn-practice-and-thinking-problem-plan](https://www.infoq.cn/article/sdn-practice-and-thinking-problem-plan)

[https://cloud.tencent.com/info/2ad72705d8b34a99a91ff4fd626225d8.html](https://cloud.tencent.com/info/2ad72705d8b34a99a91ff4fd626225d8.html)

### SDN支持的能力
最后梳理一下调研到的SDN目前能够支持的能力：

* 流量控制。可以根据业务需要定制策略，根据策略将流量转发到不同的链路上去。
* 转发控制。可以自定义一些转发策略（比如选取最大带宽网络），然后生成流表下发到底层设备，控制转发。
* 带宽控制。云计算领域，可以通过SDN能力来控制租户的网络带宽，实现按需收费。
* 网络拓扑管理。能够感知和控制网络拓扑结构，实现网络拓扑的灵活配置，快速部署。
* 网络隔离与访问控制。通过控制器，可以控制某些机器或网络之间相互隔离，不能互通，实现访问的控制。
* 安全策略配置。可以配置一些安全策略进行下发，提升网络的安全性。
* 网络设备监控。可以通过控制器相对集中地监控各种网络和设备状态。
* 安全防护（DDos攻击防护）。本质上还是通过引流，在检测到攻击后，会将攻击流量引到SDN交换机，然后进行数据包的相应改写导入清洗系统进行清洗。
* 负载均衡。本质上还是通过流量与转发控制，流表下发来实现四层上的负载均衡。

