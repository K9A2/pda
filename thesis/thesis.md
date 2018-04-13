# Extensive Evaluation of TCP Congestion Control Algorithms under Varied Network Environments

## 1. 绪论

### 1.1 论文研究背景

TCP（Transmission Control Protocol）协议是一种有连接的运输层协议，旨在为TCP/IP协议栈中的上层提供稳定、有序和不重复的数据传送服务。而在这种稳定可靠的数据传输服务中心中居于核心地位的，是用来控制一个数据包应该如何发送的TCP拥塞控制算法（Congestion Control Algorithm）[1]。拥塞控制算法能否有效地控制网络中的必定会出现的数据拥塞决定了TCP层所提供的服务质量高低。

而在过去的几十年里面，计算机网络所依赖的电子技术已经取得了巨大的进步，使得网络在带宽和误码率两方面有了明显改善。在这种情况下，Reno（New Reno）[2]、Vegas[3]和HighSpeed[4]等实现时间较早，但至今在现存系统中被广泛使用的拥塞控制算法是否能在现有的网络条件下有效地进行拥塞控制，值得我们去探讨。

同时，除了技术上的进步以外，不同的网络环境也对拥塞控制算法的有效性提出了巨大的挑战。比如，在数据中心网络（Data Center Network，DCN）的内部网络往往处于高带宽、低延迟和低误码率的理想状态；而蜂窝网络（Cellular Network）和无线局域网（Wireless Local Area Network，WLAN）的网络则常常与之相反；常用的广域网（Wide Area Network，WAN）和局域网（Local Area Network）的网络情况则相当多变。这些截然不同的网络环境对拥塞控制算法在不同场景下的有效性提出了巨大的挑战。因此，学界和工业界提出了各种拥塞控制算法以满足特定环境下对网络拥塞控制的要求，如DCN中常用的DC-TCP算法[5]以及在无线网络中常用的Westwood算法[6]等。然而，除了使用专用算法的DCN和蜂窝网络，WAN、LAN和WLAN等常用的网络类型是否存在一种通用（Generic）的拥塞控制算法也值得我们去探讨。

据相关文献资料[7]，在有线网络中，对用户的响应占据了繁忙时段网络总流量的三分之一。若要保证用户的请求能够在繁忙时段得到及时的相应，则必须对网路中的拥塞进行恰当的控制。而从网络管理员的角度来看，从源服务器到用户的网络可大致分为数据中心内部的私有网络（DCN）以及DCN网关到用户之间的公用网络。显然，后者所处的公用网络是不可控的。减轻数据中心网关到用户设备这段网络上的拥塞情况，提升目标服务的相应速度和稳定性将会对提高用户满意度产生积极影响。

### 1.2 论文主要研究内容

本次研究的主要内容是分别对有线和无线网络环境中的各种拥塞控制算法在不同的应用场景之下进行了全面的测试，并分析了各种算法在拥塞控制效果上的差异及其成因，并以此为基础提出了在不同网络场景中选择合适的拥塞控制算法的建议。本文的主要工作可简要描述如下：

* 全面调研了现有的针对TCP拥塞控制算法的测试研究，重点研究了这些研究中关于设计实验与部署测试工具的部分。同时，针对性地在前人工作的基础上引入了符合当前网络环境的实验工具与实验方法。
* 本次研究针对有线和无线两种网络形式，设计了几种具有针对性的测试场景，并对相应场景中常用的拥塞控制算法进行了全面的测试。
* 对测试结果的深入分析。同时针对过往的研究工作中所存在的一些不足之处提出了可能的解决方法。

### 1.3 论文的创新与贡献

文章通过对有线和无线网络中的拥塞控制算法进行了全面的测试与分析，对拥塞控制领域的相关研究做出了以下两条贡献：

* 提出了几条在不同网络环境下选择合适的拥塞控制算法的参考意见。
* 在基于现代网络设备的有线和无线网络场景中均取得了详实的实验资料，为以后的改进提供了依据。

### 1.4 论文各章内容安排

本文共分为六章，除首章绪论外，其余章节的主要内容简介如下：

+ 第二章主要介绍了TCP拥塞控制的基本概念、所涉及到的算法和相关研究。
+ 第三章主要介绍本次实验的具体方案，包括测试的标准、网络拓扑、测试工具和网络场景等四方面。
+ 第四章主要介绍了在有线网络环境下进行的测试及其结论。
+ 第五章主要介绍了在无线网络环境下进行的测试及其结论。

结论于文末单独成章，对本文的工作进行了总结，并对下一步工作进行了展望。

### 1.5 术语说明

## 2. 相关研究综述

### 2.1 本章引论

### 2.2 TCP拥塞控制的基本概念

#### 2.2.1 拥塞的来源与拥塞窗口

由于TCP采用了有连接的方式来传输数据，所以每一条连接以及其上的数据就可以抽象地描述为一条TCP流。在实际的传输过程中，一条TCP流中的数据，从源节点到目的节点往往需要经过多次转发。而在这个转发的过程中，由于转发节点（瓶颈节点）的处理能力有限，需要把收到的数据进行缓存后，在处理能力许可的情况下再行处理。那么，在这个等待处理的过程中，流中的数据包将不可避免地在此处产生拥塞。如果数据的发送方没有采用拥塞避免机制，则仍会在网络出现拥塞之后向网络中注入大量数据包，进一步加重瓶颈节点的拥塞情况。而在启用了拥塞控制算的TCP链接中，发送方将会维护一个类似于滑动窗口的拥塞窗口（Congestion Window，cwnd），用来限制已发送但未确认的数据包的数量。

#### 2.2.2 慢开始、拥塞避免、快重传和快恢复

慢开始、拥塞避免（Congestion Avoidance）、快重传（Fast Retransmit）和快恢复（Fast Recovery）是TCP层进行拥塞控制的重要概念，在RFC 5681[21]中有详细描述。

慢开始（Slow start）指的是发送方以一个较小的拥塞窗口大小，比如1至10个MSS（Maximum Segment Size，最大分段大小）来发送数据并使得cwnd的值翻倍，以探测网络所能承载的最大数据量，并避免在未知网络具体情况的情况下就发送大量数据而使网络出现拥塞。具体地，每当发送方接收到一个数据包的确认时，就会使拥塞窗口的大小增加一个MSS，直到发送的数据超过了接收方的接收窗口（Receiver Window，rwnd）大小、超过了ssthresh的限制等异常情况，又或者网络中出现了拥塞，可简要描述如下：

* 在cwnd增长至与ssthresh相当时又或者超出了接收方的rwnd，就将用线性增长的方式来使得cwnd得以继续缓慢地增长。具体地就是在每经过RTT（Round Trip Time，往返时延）的时间就使cwnd增加一个MSS的大小。
* 若发送方察觉到网络中出现了拥塞，就将进入拥塞避免状态，并调用系统中指定的拥塞控制算法以降低网络中的数据包的数量，其具体操作由拥塞控制算法确定。而此种情况下的拥塞控制效果也是本次研究的重点。

在TCP的具体实现中，当接收方收到了一个失序抵达的数据包时，应当立刻向发送方回复一个重复的确认，以通知发送方此数据包未按预定顺序抵达，并提示应当收到的数据包的编号。通常而言，造成数据包失序抵达的原因可以是网络出现了拥塞、网络调换了数据包的顺序[22]又或者是网络复制了一个数据包等原因。对于发送方而言，收到重复的确认就可以认为是网络出现了拥塞。除了重复的确认以为，重传计时器超时也被认为是由于网络发生了拥塞而导致的。

而为了加快从拥塞中回复的流程，发送方往往采用“快恢复”机制来发送丢失的数据包。即发送方在通过三重ACK来确认一个数据包的丢失之后，立刻重传此数据包，而非等候重传计时器超时。

通常而言，慢启动机制将缓慢地增加cwnd的值以探测网络的最大负载。而在丢包重传的情况下，未丢失数据包之前的网络负责则是网络可以接受的。如果直接使用慢启动机制来是的cwnd减半则不利于发送方快速恢复到丢包之前的状态。所以，在快恢复机制把丢失的数据包重传之后，称为“快恢复”的机制将接替“快重传”机制的控制权直到下一个不重复的ACK抵达接收方。与慢开始（即拥塞窗口cwnd现在不设置为1）的不同之处是，cwnd值被设置为ssthresh减半后的数值，然后开始执行拥塞避免算法（即所谓的加法增大），使得拥塞窗口缓慢地线性增大。

#### 2.2.3 AIMD

AIMD（Additive-Increase/Multiplicative-Decrease，加法增加、乘法减小）算法，是一种用来相对公平地分配链路带宽的反馈性算法，是一种对上节所属四种算法的高度抽象。其特点是，不存在拥塞时，使用加法来缓慢地增加拥塞窗口的大小；当网络出现拥塞是，乘法地减小拥塞窗口，以降低网络的拥塞程度。当网路中有的流都采用AIMD算法时，每一个流最终都将获得相等的带宽份额。通常而言，发送者会在每一个RTT长度的时间中让拥塞窗口增加一个MSS的大小。

### 2.3 本文所涉及到的TCP拥塞算法

#### 2.3.1 Vegas 1995

Vegas[3]是一个基于延迟的拥塞控制算法，即通过连续计算最新RTT值与基准RTT值的相对大小来确定合适的数据发送速率。当最新的RTT值与基准RTT值有较大幅度的增加时，就认为网络中将要出现拥塞，己方需要降低数据发送速率以缓解网络拥塞。当最新RTT值与基准RTT值相对下降时，认为网络空闲，己方需要增加数据发送速率。

#### 2.3.2 New Reno 1999

New Reno[2]是Reno[23]的最新改进版，主要针对后者所引入的快恢复算法作出了改进。在Reno的控制下，发送方会在重传计时器超时或者收到三个重复的确认的情况下重传一个数据包。这样的话，如果在多个数据包同时丢失，Reno就会频繁调用快重传算法[24]，使得拥塞窗口迅速减小[25]。New Reno算法则会在把所有从进入快恢复阶段开始的所有未被确认的数据包都被确认之后才会退出快恢复阶段，从而避免了以上问题。

#### 2.3.3 Westwood 2002

Westwood[26]是一种基于延迟和丢包的拥塞控制算法，其使用RTT以及此时间段内被发送的数据包大小作为计算可用最大带宽的依据。当数据包丢失时，拥塞窗口和慢启动门限会被设置得贴近当前带宽的估计值。这样就可以减小快恢复阶段所需要的时间。由于无线网络中常常丢包的现象，结合此算法恢复时间较短的特点，故此算法常用在无线网络中。

#### 2.3.4 HighSpeed 2003

HighSpeed[27]是一个基于丢包的拥塞控制算法，主要针对具有高BDP（Bandwidth Delay Product，带宽延迟积）特性的网络进行了优化。它的特点是使用加法因子和乘法因子以及当前的丢包率等来作为参数，以控制AIMD过程中拥塞窗口的大小。

#### 2.3.5 Scalable 2003

Scalable[28]是Reno经过简单修改之后的结果。在Scalable算法中，加法因子被修改为0.01而乘法因子被修改为0.875，以优化该算法在高速广域网中传输大量数据的性能表现。

#### 2.3.6 Veno 2003

Veno[35]是一种对无线网络中随机出现的丢包作出了针对性优化的拥塞控制算法同时考虑了延迟与丢包两种因素，并能够区分无线网络中经常出现的随机丢包现象和真正的网络拥塞。具体而言，就是使用了与Vegas相类似的算法来预测网络中是否将要出现拥塞；如果预测结果为肯定的，并且之后出现了丢包的现象，那么就认为这次丢包是由于网络拥塞引起的，需要进行拥塞避免操作；否则，认为此次丢包是由于网络随机错误引起的，直接重传此数据包。这样，就可以避免出现无谓的性能损失。

#### 2.3.7 BIC 2004

BIC[29]是一种基于丢包的拥塞控制算法，主要针对在高BDP网络环境下，通过动态计算当前拥塞窗口的大小所对应的最佳发送速率，以充分利用链路带宽并且维持算法之间的相对公平。其核心算法有两个：二次搜索增窗算法和加法增窗算法。其中，二次搜索增窗算法主要负责在拥塞窗口较小时在不同的流之间维持较好的公平性；加法增窗算法主要负责在拥塞窗口较大时

#### 2.3.8 H-TCP 2004

H-TCP[34]是一种针对高速长距离网络传输以及兼容低速传统网络进行了针对性优化，并同时考虑丢包和延迟的拥塞控制算法。它以当前与上次拥塞事件之间的时间差来作为加法增窗函数的重要参数。当时间差小于某值时，加法增窗函数处于低速模式，其各项设定较为保守，以提高对低速网络的兼容性；当时间差大于某值时，它采用是检查作为加法增窗函数的参数，并激进地增大窗口大小，以提高在高速网络中的表现。

#### 2.3.9 Hybla 2004

Hybla[33]是一种针对高错误率和高延迟进行了针对性优化的拥塞控制算法。主要的特点在于，它使用了归一化之后的RTT以移除实时RTT对拥塞窗口的影响。

#### 2.3.10 YeAH 2007

YeAH[31]是一种同时考虑延迟和丢包的拥塞控制算法。它保留了Vegas用来探测网络状况的算法，并引入了快慢两种工作模式：快模式主要用于激进地提高窗口的大小以获取较高的带宽利用率；慢模式用来维持与其他拥塞控制算法的相对公平性。

#### 2.3.11 CUBIC 2008

CUBIC[30]是一种基于丢包的拥塞控制算法，针对BIC算法中较为激进的窗口调整策略和糟糕的算法间公平性作出了较大的改进。现已作为Linux 2.6以后的默认拥塞控制算法。简而言之，CUBIC使用了以距离上次丢包时间为主要参数的三次函数来动态地确定当前拥塞窗口的大小，以维持较好的可扩展性和稳定性。

#### 2.3.12 Illinois 2008

Illinois[32]是一种综合考虑了丢包和延迟的拥塞控制算法，针对高速长距离传输做了针对性优化。此算法是使用了丢包作为决定拥塞窗口增减的依据，并使用排队延迟（Queue Delay）来计算应该把拥塞窗口增加或者减小多少。

#### 2.3.13 BBR 2016

BBR[1]是一中基于BDP的拥塞控制算法，有效地解决了网络瓶颈处缓存过满的问题。该算法主要通过测量两个链路参数：往返传播时间（Rountd Trip Prapagation Time）和瓶颈带宽（Bottleneck Bandwidth），来保证发送方始终能以最大速度发送数据。

### 2.4 TCP拥塞算法性能测试相关研究

由于对TCP拥塞控制算法在不同网络环境下进行测试以检查其有效性十分重要，学界和工业界的研究人员均对此问题进行了大量研究。

Mario Hock等人在其研究中[8]对新提出并已经得到广泛应用的拥塞控制算法BBR在带宽（Throughput）、排队延迟（Queuing delay）、丢包率（Packet loss）和公平性（Fairness）等几个方面进行了独立而深入的测试与分析。由于他们已经对包含BBR在内的常用的拥塞控制算法的底层进行了深入分析，故本文便从上层应用的角度来分析拥塞控制算法的有效性。

Kevin Ong等人在其研究中[9]中对无线网络中的常用拥塞控制算法在进行了详细的测试。然而他们的文章中只有带宽和延迟两个测试项目，并不能完全反应算法的真实性能。

TAN Nguyen等人在其研究[11]中引入了NS-2模拟器以深入分析算法的性能差异。然而，作者的实验是基于DCN的，并不能反应这些算法在因特网中的表现。Thomas Lukaseder等人在其研究中[10]使用NetFPGA给测试网络引入可控的丢包率，并研究了New Reno、Scalable、HighSpeed、H-TCP、BIC和CUBIC等算法在不同丢包率下的性能差异，并分析差异的的成因。然而，作者的实验是基于国有的教育网，其结论同样并不适用与因特网。

论文发表时间较早的有Callegari等人的研究[12]，对Linux 2.6内核所包含的13种拥塞控制算法均进行了测试；以及LA Grieco等人所做的研究[13]，其中引入了模拟测试和场景测试的概念。他们的研究方案与结论对我进行本次研究给予了重要的参考。

## 3. 测试方案

### 3.1 性能标准

#### 3.1.1 RTT

#### 3.1.2 带宽

#### 3.1.3 公平性

#### 3.1.4 重传计数

### 3.2 测试工具

#### 3.2.1 iperf3

iperf3[36]，由ESnet开发，是一个用来测试IP网络中TCP、UDP和SCTP协议下网络带宽的开源测试工具。每一个测试，iperf3都会提供相应的RTT、带宽和重传计数等信息，供用户判断当前网络状态。

#### 3.2.2 Mininet

Mininet[37]是一种可以创建由一些虚拟的终端节点、交换机、路由器连接而成的网络的仿真器，它采用轻量级的虚拟化技术使得系统可以和真实网络相媲美。在这个虚拟网络中，用户可以像使用真实的电脑一样进行操作：可以使用SSH登录、启动应用程序、向以太网端口发送数据包、观察数据包会被交换机和路由器接收并处理的过程等。有了这个网络，就可以灵活地为网络添加新的功能并进行相关测试，然后轻松部署到真实的硬件环境中。

#### 3.2.3 curl

curl[38]是利用URL语法在命令行方式下工作的开源文件传输工具，支持HTTP、FTP以及SCP等多种协议，能够方便地在不同的网络环境中使用。

### 3.3 网络拓扑

#### 3.3.1 有线网络部分

有线网络部分的测试安排了LAN、中国深圳到中国广州的WAN 1和美国Portland到中国北京的WAN 2等三条线路。各线路的带宽限制如下：

|线路名称|数据流起点|数据流终点|瓶颈带宽限制|
|-------|-------|---------|-------|
|LAN|暨南大学南海楼实验室|暨南大学数据中心|带宽由实验室100Mbps交换机限制|
|WAN 1|中国深圳阿里云机房|暨南大学数据中心|由阿里云方面的共享网关限制为1Mbps，但在实际测试中几乎所有样本点均落在1.5Mbps以内，突发传输速率可达3.5Mbps，最高平均速度约为1.3Mbps|
|WAN 2|美国Portland|中国北京阿里云机房|由阿里云方面的共享网关限制，实际测试中的最高平均速度约为15Mbps|

#### 3.3.2 无线网络部分

无线网络部分的测试安排在下图的无线网络中完成，其中包含一台TP-LINK TL-WDR6500无线路由器、一台作为服务器的台式机和两台作为客户端的笔记本电脑。在这个无线网络中，每一台PC机均使用无线网卡与无线路由器相连。根据实验的需要，三台PC机将同时使用2.4GHz或者5GHz连接路由器（例如，在2.4GHz的实验中，所有PC机都将使用2.4GHz频段连接无线路由器；其他设置由网卡与路由器协商确定）。

由于影响无线网络性能的因素甚多，本次实验挑选了频段和发射功率两个因素（即在2.4GHz和5GHz频段中均进行20mw、15mw、10mw和5mw四种发射功率的测试）。

### 3.4 测试类型

#### 3.4.1 基准性能测试

在对每一个场景进行全面测试之前，都会通过iperf3来对目标场景进行长时间的连续测试，以求全面地了解在该场景下用来作为基准的CUBIC（有线网络）算法和Westwood（无线网络）在RTT、带宽和重传计数等三个方面的性能表现。在基准测试中获得的RTT、带宽和重传计数等，其平均值将会作为基准线出现在之后的测试的结果图及其分析中。采样周期为1秒。在两类网络上的详细设定上的差异如下表：

|项目|有线网络|无线网络|
|----|------|-------|
|基准拥塞控制算法|CUBIC|Westwood|
|测试时长|24小时|1小时|

#### 3.4.2 TCP性能测试

在基准测试之后，将使用iperf3对所有场景中的TCP拥塞控制算法进行详细测试。主要的测试内容是在该场景下使用特定TCP拥塞控制算法来控制iperf3数据流，并以1秒为单位通过iperf3来记录该数据流的RTT和带宽等各项参数。在此测试过程中，将引入其他TCP流，以考察该TCP拥塞控制算法在有竞争的情况下的性能表现。除了发送测试数据的主机之外，其余主机的各项设定固定。在两类网络上的详细设定上的差异如下表：

|项目|有线网络|无线网络|
|----|-------|------|
|其他TCP流来源|不可控的公用网络中的其他数据流，其数据不能被记录|可控的使用iperf3引入的Westwood数据流，其数据会被记录|
|测试时长|共两组，每组测试一共进行六轮，每个算法在每一轮测试中均测试10分钟|共一组，每个频段的每一个发射功率均进行3轮测试，每个算法在每一轮测试中均测试10分钟|

#### 3.4.3 Web应用测试

在完成TCP性能测试之后，将使用curl进行文件下载测试。在此类测试之中，将在测试主机上使用Apache 2建立服务器，并且使用其他主机下载服务器中的文件。在有线网络和无线网络中将下载不同大小的文件，以保证每次下载操作的所需时间约为一分钟。下载操作将重复进行，然后计算总平均下载速度。在两类网络上的详细设定上的差异如下表：

|项目|有线网络|无线网络|
|----|-------|------|
|文件大小|5M|64M|
|重复次数|50次|20次|
|其他TCP流|不可控的公用网络数据流|另外一台使用Westwood算法的PC进行curl下载操作的数据流|

#### 3.4.4 公平性测试

在完成应用测试之后，由于有线网络环境较为复杂，无法保证网络中仅有测试数据流和对照数据流，故将在Mininet中按照基准测试中获得的网络特性参数（包括RTT、带宽和丢包率等）设定虚拟网络的特性参数，并按照哑铃型拓扑组建虚拟网络。拓扑中的两台发送端主机通过两台交换机组成的单瓶颈链路向处于瓶颈另一端的接收主机分别使用iperf3发送数据。而本次研究所用的无线网络拓扑简单，没有多余的数据流。故无线网络部分的公平性测试将直接在真实网络拓扑中进行。

## 4. 基于有线网络的性能测试结果与分析

### 4.1 基准性能测试

#### 4.1.1 LAN

对于LAN，其基准性能测试结果如下表：

|测试项目|最大值|最小值|平均值|中位数|方差|
|-------|------|----|------|-----|---|
|RTT（单位：秒）|176.89|2.57|9.68|8.14|48.75|
|带宽（单位：Mbps）|105.30|0.00|88.90|90.00|52.93|
|重传计数（单位：包/秒）|331.00|0.00|0.00|0.06|3.31|

#### 4.1.2 WAN 1

对于WAN 1，其基准性能测试结果如下表：

|测试项目|最大值|最小值|平均值|中位数|方差|
|-------|------|----|------|-----|---|
|RTT（单位：秒）|247.50|29.27|31.39|30.93|28.63|
|带宽（单位：Mbps）|3.50|0.00|1.33|1.03|0.11|
|重传计数（单位：包/秒）|130.00|0.00|9.56|9.00|3.13|

#### 4.1.3 WAN 2

对于WAN 2，其基准性能测试结果如下表：

|测试项目|最大值|最小值|平均值|中位数|方差|
|-------|------|----|------|-----|---|
|RTT（单位：秒）|247.50|29.27|31.39|30.93|28.63|
|带宽（单位：Mbps）|3.50|0.00|1.33|1.03|0.11|
|重传计数（单位：包/秒）|130.00|0.00|9.56|9.00|3.13|

### 4.2 TCP性能测试

#### 4.2.1 LAN

![lan-rtt.png](./lan-rtt.png)

![lan-throughput.png](./lan-throughput.png)

|算法，按平均RTT从小到大排序|平均RTT（单位：秒）|算法，按平均带宽从大到小排序|平均带宽（单位：Mbps）|
|----|-----------------|--|-------------------|
|Vegas|1.89|Scalable|89.18|
|BBR|4.10|H-TCP|89.13|
|YeAH|6.56|Hybla|89.10|
|Hybla|7.80|BIC|89.07|
|NewReno|8.18|CUBIC|88.99|
|Illinois|8.79|Illinois|88.96|
|BIC|8.93|HighSpeed|88.93|
|CUBIC|9.26|NewReno|88.90|
|H-TCP|9.39|YeAH|88.89|
|HighSpeed|9.70|BBR|87.94|
|Scalable|10.39|Vegas|67.90|

#### 4.2.2 WAN 1

![wan1-rtt.png](./wan1-rtt.png)

![wan1-throughput.png](./wan1-throughput.png)

|算法，按平均RTT从小到大排序|平均RTT（单位：秒）|算法，按平均带宽从大到小排序|平均带宽（单位：Mbps）|
|----|-----------------|--|-------------------|
|Scalable|30.25|H-TCP|1.02|
|Illinois|30.42|NewReno|1.02|
|HighSpeed|30.87|Hybla|1.02|
|YeAH|30.87|HighSpeed|1.02|
|Hybla|30.93|Vegas|1.02|
|BIC|31.09|Illinois|1.02|
|NewReno|31.20|BIC|1.02|
|BBR|31.23|CUBIC|1.02|
|CUBIC|31.42|Scalable|1.02|
|H-TCP|31.68|BBR|1.02|
|Vegas|31.79|YeAH|1.02|

#### 4.2.2 WAN 2

![wan2-rtt.png](./wan2-rtt.png)

![wan2-throughput.png](./wan2-throughput.png)

|算法，按平均RTT从小到大排序|平均RTT（单位：秒）|算法，按平均带宽从大到小排序|平均带宽（单位：Mbps）|
|----|-----------------|--|-------------------|
|YeAH|244.26|Illinois|56.19|
|Illinois|244.62|Hybla|52.68|
|H-TCP|245.72|YeAH|52.19|
|NewReno|246.52|Scalable|49.01|
|Scalable|246.78|BIC|46.95|
|Hybla|247.09|H-TCP|45.56|
|Vegas|247.61|BBR|44.88|
|BBR|247.93|CUBIC|43.10|
|CUBIC|248.40|NewReno|37.16|
|BIC|248.93|HighSpeed|36.92|
|HighSpeed|612.90|Vegas|29.00|

### 4.3 网络应用测试

### 4.4 算法间公平性测试

### 4.5 本章小结

## 5. 基于无线网络的性能测试结果与分析

### 5.1 基准性能测试

### 5.2 TCP性能测试

### 5.3 网络应用测试

### 5.4 算法间公平性测试

### 5.5 本章小结

## 6. 结论

## 参考文献

1. Cardwell N, Cheng Y, Gunn C S, et al. BBR: Congestion-based congestion control[J]. Queue, 2016, 14(5): 50.
2. Floyd S, Gurtov A, Henderson T. The NewReno modification to TCP's fast recovery algorithm[J]. 2004.
3. Brakmo L S, Peterson L L. TCP Vegas: End to end congestion avoidance on a global Internet[J]. IEEE Journal on selected Areas in communications, 1995, 13(8): 1465-1480.
4. Floyd S. HighSpeed TCP for large congestion windows[J]. 2003.
5. Nguyen T A N, Gangadhar S, Sterbenz J P G. Performance Evaluation of TCP Congestion Control Algorithms in Data Center Networks[C]//Proceedings of the 11th International Conference on Future Internet Technologies. ACM, 2016: 21-28.
6. Gerla M, Sanadidi M Y, Wang R, et al. TCP Westwood: Congestion window control using bandwidth estimation[C]//Global Telecommunications Conference, 2001. GLOBECOM'01. IEEE. IEEE, 2001, 3: 1698-1702.
7. Feknous M, Houdoin T, Le Guyader B, et al. Internet traffic analysis: A case study from two major European operators[C]//Computers and Communication (ISCC), 2014 IEEE Symposium on. IEEE, 2014: 1-7.
8. Hock M, Bless R, Zitterbart M. Experimental evaluation of BBR congestion control[C]//Network Protocols (ICNP), 2017 IEEE 25th International Conference on. IEEE, 2017: 1-10.
9. Ong K, Murray D, McGill T. Large-Sample comparison of TCP congestion control mechanisms over wireless networks[C]//Advanced Information Networking and Applications Workshops (WAINA), 2016 30th International Conference on. IEEE,Brakmo L S, Peterson L L. TCP Vegas: End to end congestion avoidance on a global Internet[J]. IEEE Journal on selected Areas in communications, 1995, 13(8): 1465-1480. 2016: 420-426.
10. Lukaseder T, Bradatsch L, Erb B, et al. A comparison of TCP congestion control algorithms in 10G networks[C]//Local Computer Networks (LCN), 2016 IEEE 41st Conference on. IEEE, 2016: 706-714.
11. Nguyen T A N, Gangadhar S, Sterbenz J P G. Performance Evaluation of TCP Congestion Control Algorithms in Data Center Networks[C]//Proceedings of the 11th International Conference on Future Internet Technologies. ACM, 2016: 21-28.
12. Callegari C, Giordano S, Pagano M, et al. Behavior analysis of TCP Linux variants[J]. Computer Networks, 2012, 56(1): 462-476.
13. Grieco L A, Mascolo S. Performance evaluation and comparison of Westwood+, New Reno, and Vegas TCP congestion control[J]. ACM SIGCOMM Computer Communication Review, 2004, 34(2): 25-38.
14. RFC 793, https://tools.ietf.org/html/rfc793
15. RFC 2001, https://tools.ietf.org/html/rfc2001
16. Jacobson V. Congestion avoidance and control[C]//ACM SIGCOMM computer communication review. ACM, 1988, 18(4): 314-329.
17. Floyd S, Gurtov A, Henderson T. The NewReno modification to TCP's fast recovery algorithm[J]. 2004.
18. Brakmo L S, Peterson L L. TCP Vegas: End to end congestion avoidance on a global Internet[J]. IEEE Journal on selected Areas in communications, 1995, 13(8): 1465-1480.
19. Ha S, Rhee I, Xu L. CUBIC: a new TCP-friendly high-speed TCP variant[J]. ACM SIGOPS operating systems review, 2008, 42(5): 64-74.
20. Xu L, Harfoush K, Rhee I. Binary increase congestion control (BIC) for fast long-distance networks[C]//INFOCOM 2004. Twenty-third AnnualJoint Conference of the IEEE Computer and Communications Societies. IEEE, 2004, 4: 2514-2524.
21. RFC 5681, https://tools.ietf.org/html/rfc5681
22. Paxson V. End-to-end Internet packet dynamics[C]//ACM SIGCOMM Computer Communication Review. ACM, 1997, 27(4): 139-152.
23. Fall K, Floyd S. Simulation-based comparisons of Tahoe, Reno and SACK TCP[J]. ACM SIGCOMM Computer Communication Review, 1996, 26(3): 5-21.
24. Hoe J C. Start-up dynamics of TCP's congestion control and avoidance schemes[D]. Massachusetts Institute of Technology, 1995.
25. Floyd S. TCP and successive fast retransmits[R]. Technical report, October 1994. ftp://ftp. ee. lbl. gov/papers/fastretrans. ps, 1995.
26. Gerla M, Sanadidi M Y, Wang R, et al. TCP Westwood: Congestion window control using bandwidth estimation[C]//Global Telecommunications Conference, 2001. GLOBECOM'01. IEEE. IEEE, 2001, 3: 1698-1702.
27. RFC 3649, https://buildbot.tools.ietf.org/html/rfc3649
28. Kelly T. Scalable TCP: Improving performance in highspeed wide area networks[J]. ACM SIGCOMM computer communication Review, 2003, 33(2): 83-91.
29. Xu L, Harfoush K, Rhee I. Binary increase congestion control (BIC) for fast long-distance networks[C]//INFOCOM 2004. Twenty-third AnnualJoint Conference of the IEEE Computer and Communications Societies. IEEE, 2004, 4: 2514-2524.
30. Ha S, Rhee I, Xu L. CUBIC: a new TCP-friendly high-speed TCP variant[J]. ACM SIGOPS operating systems review, 2008, 42(5): 64-74.
31. Baiocchi A, Castellani A P, Vacirca F. YeAH-TCP: yet another highspeed TCP[C]//Proc. PFLDnet. 2007, 7: 37-42.
32. Liu S, Başar T, Srikant R. TCP-Illinois: A loss-and delay-based congestion control algorithm for high-speed networks[J]. Performance Evaluation, 2008, 65(6-7): 417-440.
33. Caini C, Firrincieli R. TCP Hybla: a TCP enhancement for heterogeneous networks[J]. International journal of satellite communications and networking, 2004, 22(5): 547-566.
34. Leith D, Shorten R. H-TCP: TCP for high-speed and long-distance networks[C]//Proceedings of PFLDnet. 2004, 2004.
35. Fu C P, Liew S C. TCP Veno: TCP enhancement for transmission over wireless access networks[J]. IEEE Journal on selected areas in communications, 2003, 21(2): 216-228.
36. esnet/iperf, https://github.com/esnet/iperf
37. Mininet Team, http://mininet.org/
38. curl, https://github.com/curl/curl