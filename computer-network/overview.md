计算机网络 分组交换网络  计算机网络性能指标

OSI参考模型层次结构  TCP/IP参考模型层次结构


### 计算机网络基本概念

#### 计算机网络定义

1、计算机网络的诞生

从技术范畴来看，计算机网络是**计算机技术**与**通信技术**相互通融的产物。

2、计算机网络的定义

计算机网络是**互联的**、**自治的**计算机的集合

自治：互连的计算机系统彼此独立，不存在主从或者控制与被控制的关系。

互联：利用通信链路连接相互独立的计算机系统。

3、因特网

目前应用最广泛的计算机网络就是因特网（internet）

因特网服务提供商ISP

?>
1、计算机网络是___ 、____ 的计算机的集合。  
互连、自治  
2、目前最大的计算机网络称为( B )  
A、互联网
B、因特网
C、万维网
D、广域网

#### 协议的定义

1、网络协议

网络通信实体之间在数据交换过程中需要遵循的规则或约定。

例如：http、tcp、ip等

2、协议的三要素

语法: 定义实体之间交换信息的**格式与结构**。

语义: 定义实体之间交换信息中的**控制信息**。

时序: 同步、定义实体之间交换信息的**顺序**以及如何匹配或适应彼此的**速度**。

?>
1、定义实体间交换信息的顺序以及如何匹配或适应彼此速度的网络协议要素是时序。  
2、网络协议中定义实体之间交换信息格式与结构的协议要素是( A )  
A、语法
B、语义
C、模式
D、时序

#### 计算机网络的功能

计算机网络的核心功能是：**资源共享**

1、硬件资源共享: 计算资源(CPU)、存储资源、打印机与扫描仪I/O等。

例: 云存储、云计算

2、软件资源共享:SaaS

例: 大型办公软件、大型数据库系统

3、信息资源共享:信息检索，新闻阅读等。

?>
1、利用计算机网络可以实现的共享资源不包括( C )  
A、硬件资源
B、软件资源
C、技术资源
D、信息资源  
2、在目前的互联网环境下，软件共享的典型形式是( SaaS )

#### 计算机网络的分类

1、按覆盖范围分类

个域网：PAN 1-10m

局域网：LAN 10m-1km

城域网：MAN 5-50km

广域网：WAN 几十到几千米

2、按拓扑结构分类

**星形拓扑结构**：一个中央结点(集线器、交换机)、网络中的主机通过点对点通信链路与中央结点连接。(个域网、局域网)

优点：易于监控管理、故障诊断、隔离。

缺点：中央结点一旦故障，全网瘫痪。

**总线型拓扑结构**：采用一条广播信道作为公共传输介质(总线)。所有结点均与总线连接，结点间的通信均通过共享的总线进行。(早期局域网)

优点：结构简单，电缆数量少，易于扩展。

缺点：通信范围受限，容易产生冲突。

**环形拓扑结构**：利用通信链路将所有结点连接成一个闭合的环。(早期局域网、城域网)

优点：电缆长度短，可使用光纤，易于避免冲突。

缺点：某结点故障引起全网瘫痪，加新(撤出)结点麻烦。存在等待时间。

**网状拓扑结构**：网络中的结点通过多条链路与不同的结点直接相连接。(广域网、核心网络)

优点：网络可靠性高，一条或多条链路故障时，网络仍然可以联通。

缺点：网络结构复杂，成本高、选路协议复杂。

**树形拓扑结构**：可以看作是总线型拓扑或星形拓扑结构网络的扩展。(局域网)

优点：易于扩展，故障易隔离。

缺点：根结点要求高。

**混合拓扑结构**：由两种以上简单拓扑结构网络混合连接而成的网络。(绝大多数实际网络)

优点：易易于扩展，可以构建不同规模的网络，根据需要优选网络结构。

缺点：结构复杂，管理与维护复杂。

3、按交换方式分类

电路交换 报文交换 分组交换

4、按网络用户属性

公用网：面向公众开放的网络。例如电信网络。

私有网：某个组织(政府或者企业)出资建设专门面向该组织，不向公众开放。例如军事专用网络。

?>
1、比较多见于广域网、核心网络的拓扑结构是( A )  
A.网状拓扑结构
B.环形拓扑结构
C.树型拓扑结构
D.混合拓扑结构  
2、网络规模受限于中央结点端口数量的网络拓扑结构是( B )  
A.总线拓扑结构
B.星形拓扑结构
C.网状拓扑结构
D.环形拓扑结构  
3、按网络覆盖范围分类，计算机网络可以分为: 个域网、( 局域网 )、城域网和广域网。  
4、按网络用户属性，下列属于公用网的是( B )  
A.军事网络
B.电信网络
C.航空网络
D.银行网络

### 计算机网络结构

#### 网络边缘
连接到网络上的主机或者端系统。

主机或者端系统:计算机、服务器、智能手机、智能传感器、智能家电等。

**连接到网络上的所有端系统构成了网络边缘。**


#### 接入网络

实现网络边缘的端系统与网络核心连接的网络。

1、电话拨号接入:利用电话网络接入。通过调制解调器进行数字信号与模拟信号的转换。56kbit/s。

2、非对称数字用户线路ADSL:

利用电话网络接入。   

基于频分多路复用技术(5.3节)。 

非对称(上行带宽小于下行带宽)。 

独享式接入。

3、混合光纤同轴电缆HFC接入网络:

利用有线电视网络接入的技术。 

基于频分多路复用技术。

非对称。 

共享式接入(多个用户共享上、下行带宽)。

4、局域网:典型的局域网技术是以太网、WiFi等。

5、移动接入网络:利用移动通信网络，如3G、4G、5G网络。

#### 网络核心

由通信链路互连的分组交换设备(路由器、交换机)构成的网络，
作用是实现网络边缘中主机之间的数据的中继与转发。

?>
1、利用有线电视网络实现接入的网络技术是( A )  
A.HFC接入
B.ADSL接入
C.移动接入
D.局域网接入  
2、大规模现代计算机网络结构中不包括的部分是( C )  
A.接入网络
B.网络核心
C.主服务器
D.网络边缘

### 数据交换技术

#### 数据交换的概念

1、计算机网络的目的:在网络边缘的主机之间实现相互的数据传输，信息交换。

2、如何实现？

通信链路: 主机数为N的网络中，需要N(N-1)/2条链路!
网络规模较大时，通过通信链路实现所有终端之间通信是不可行的。

**交换设备：**多通信端口，可以同时连接多个通信结点(主机或者交换设备)，进行通信的设备。

3、数据交换：实现在大规模网络核心上进行数据传输的技术基础。

常见的数据交换技术包括:电路交换、报文交换、分组交换。

电路交换网络、报文交换网络、分组交换网络

?>
计算机网络按照其所采用的数据交换技术可分为 ______网络、报文交换网络和分组交换网络。  
电路

#### 电路交换

1、电话网络是最早、最大的电路交换网络。

2、在电话交换网络中，首先需要通过中间交换结点为两台主机之间建立一条专用的通信线路，称为电路。

3、电路交换的步骤:1、建立电路。2、传输数据。3、拆除电路。

1)建立电路:物理链路带宽共享、频分多路复用、时分多路复用(第6章)。 

2)传输数据:数字数据或者模拟数据、单工或者全双工、独占。

3)拆除电路:释放物理链路，任意一方可以发起并完成拆除。

4、电路交换的优缺点:

优点:实时性高、时延和时延抖动小。 

缺点:突发数据传输，信道利用率低，传输速率单一。

适用于语音和视频类实时性强的业务。

?>
1、用于语音和视频这类实时性强的业务的交换技术是(电路交换 )。  
2、电路交换的三个步骤是:建立电路、( 传输数据 )、拆除电路。


#### 报文交换

1、什么是报文

报文:发送方把要发送的信息附加上发送/接收主机的地址和控制信息。

2、报文交换 

又称消息交换。发送方组装好报文，发给相邻报文交换机。
相邻报文交换机收到报文后检查无误，暂时存储报文，然后找出需要转发的下一个结点的地址，然后把报文给下一个结点的报文交换机。

3、交换方式

存储-转发的交换方式。

4、优缺点:

优点:信道利用率高。 

缺点:时延长，可能会报文过多而丢弃报文。

?>
在数据交换技术中，下列关于报文交换的描述错误的是( C )  
A.采用存储-转发的交换方式   
B.相对电路交换信道而言，报文交换线路利用率高   
C.适用于实时通信  
D.会出现丢弃报文的现象

#### 分组交换

1、分组交换（包交换）基本原理

将待传输的数据(报文)分割成较小的独立的数据块。每个数据
块附加地址、序号等信息构成数据分组。分组独立传输到目的地，到
目的再重组还原为报文。

**采用存储—转发交换方式，是计算机网络中使用最广泛的交换技术。**

2、与报文交换相比，分组交换方式的优点:

(1)交换设备存储容量要求低:只要能缓存一个小分组，而报文交换需要缓存整个报文。

(2)交换速度快:不需要访问外存，而报文交换，报文较大时可能 需要访问外存。 

(3)可靠传输效率高:重传出错的分组，不需要重传整个报文。 

(4)更加公平:交替排队，小报文分组较短时间到达。

3、分组长度的确定:

1)在其他条件相同的情况下，分组长度越长，延迟时间越长。 

2)通信链路的误码率是确定分组长度时另一个重点考虑因素。 

3)一般的分组长度:以16B — 4096B之间的2nB为标椎的分组长度。 例如:32B、64B、256B等。

?>
1、对于报文和分组交换方式来说，更为公平的交换方式是( 分组交换 )。  
2、报文交换和分组交换技术中，现代计算机网络不采用的是( 报文交换 )。  
3、在网络的交换方式中被称为包交换方式的是( D )   
A.电路交换
B.报文交换
C.虚拟交换
D.分组交换

### 计算机网络性能

#### 速率与带宽

1、速率

网络单位时间内传送的数据量，用以描述网络传输数据的快慢。

速率基本单位:bit/s(位每秒)

单位的换算: 1 Tbit/s = 10^3 Gbit/s = 10^6 Mbit/s = 10^9 Kbit/s = 10^12 bit/s


2、带宽

(1)在通信和信号处理领域，指的是信号的频带宽度，即信号成分的最高频率与最低频率之差。单位:Hz(赫兹)。

(2)在计算机网络领域，指的是一条链路或者信道的数据传输能力，也就是最高数据速率，单位:bit/s(位每秒)。

#### 时延

1、时延: 数据从网络中的一个结点(主机或者交换设备)到达另一个结点所需要的时间。

2、在分组交换网络中: 通常将连接两个结点的直接链路称为一个“跳步”，简称“跳”。

3、分组的每跳传输过程中4类时间延迟:

(1)结点处理时延。dc

交换设备检查分组是否有差错，确定如何转发分组的时间，还有可能修改分组的部分控制信息等。

此类延时较小，往往忽略。

(2)排队时延。dq 

分组在交换结点内被交换到输出链路，等待从输出链路发送到下一个结点的时间。

排队时延不确定。

(3)传输时延（发送时延）。dt 

分组在输出链路发送时，从发送第一位开始，到发送完最后一位需要的时间。

dt=L / R  
L:分组长度，单位:bit   
R:链路带宽(即速率)，单位:bit/s  

(4)传播时延。dp 

信号从发送端出来，经过一段物理链路到达接收端需要的时间。 

dp=D / V  
D:物理链路长度，单位:m  
V:信号传播速度，单位:m/s  

一个分组经过一个跳步需要的时间:dh = dc + dq + dt + dp

所有跳步的时间总和即为一个分组从源主机到达目的主机的时延。

?>
1、在讨论网络总时间延迟时常常被忽略的是( D )  
A.传输时延
B.分组排队时延
C.传播时延
D.结点处理时延  
2、信号从发送端发出，经过一定距离的物理链路到达接收端所需要的时间称为( D )  
A.处理时延
B.排队时延
C.传输时延 
D.传播时延  
3、物理链路长度D=500m，信号传播速度V=2500km/s， 则所求传播时延为( D )  
A.2*10^(-1)s   
B.2*10^(-2)s   
C.2*10^(-3)s   
D.2*10^(-4)s 

#### 延时带宽积

时延带宽积: 物理链路的传播时延与链路带宽的乘积，记为G。 表示一段链路可以容纳的数据位数，也称为:以位为单位的链路长度。

公式:G=传播时延×链路带宽=dp × R  
传播时延的单位:s  
带宽的单位: bit/s  
时延带宽积的单位:bit

?>
1、如果将物理链路看作传播数据的管道，则用来表示一段链路可容纳的数据位数的概念是( A )  
A.时延带宽积
B.排队时延
C.最大吞吐量 
D.链路带宽  
2、设信号传播速度V=2500km/s，链路长度D=500m，链路带宽 R=10Mbit/s，则该段链路的时延带宽积为( B )  
A.1500bit
B.2000bit
C.2500bit
D.4000bit

#### 丢包率

丢包率: 丢失分组和发送分组之比。反映网络的拥塞程度。  

Ns:发送分组数

Nr:接收分组数

j = (Ns - Nr) / Ns


#### 吞吐量

吞吐量: 在单位时间内源主机通过网络向目的主机实际送达的数据量，记为Thr。

单位:bit/s或B/s(字节每秒)

1B=8bit

吞吐量经常用于度量网络的实际数据传送能力。受带宽、连接复杂性、网络协议、拥塞程度等因素影响。

分组交换网络中，吞吐量约等于瓶颈链路的带宽，即: Thr=min(R1，R2....RN)

?>
1、在分组交换网中，若主机A向主机B传送数据只能经过由 3段链路组成的一条路径，
其速率分别是R1=500bit/s、R2=2Mbit/s、 R3=1Mbit/s，则A到B的最大吞吐量为( B )  
A.250bit/s
B.500bit/s
C.10^6bit/s
D.2*10^6bit/s  
2、设主机A和主机B由一条带宽为R=10^8bit/s、长度为 D=100m的链路互连，信号传播速率为V=250000km/s。
如果主机A从t=0时刻 开始向主机B发送长度为 L=1024bit的分组。  
试求:  
1.主机A和主机B间的链路传输时延dt。  
2.主机A发送该分组的传播时延dp。  
3.该分组从主机A到主机B的延迟T。(忽略结点处理时延和排队时延)   
4.在t=dt时刻，分组的第一位在何处。(说明原因)   
5.主机A与主机B间链路的时延带宽积G。  
解：  
1、dt = L / R = 1024 / 10^8 = 1.024 * 10^(-5)s    
2、dp = D / V = 100 / 2.5 * 10^8 = 4 * 10^(-7)s  
3、T = dt + dp = 1.064 * 10^(-5)s  
4、dp > dt, 分组的第一位在主机B处  
5、G = dp * R = 4 * 10^(-7) * 10^8 = 40bit  
              
### 计算机网络体系结构

#### 计算机网络分层体系结构

1、用户之间进行信息交换  

1)硬件:主机、链路、交换设备 

2)协议

2、协议最典型的划分方式就是采用**分层的**方式来组织协议。

3、分层的核心思路是上一层的功能建立在下一层的功能基础上，每一层内均要遵守一定的通信规则，即**协议**。

4、计算机网络体系结构:计算机网络所划分的**层次**以及**各层协议**的集合。
这种分层体系结构是按**功能**划分的，不是按实现方式划分的。


#### OSI参考模式

(一)OSI参考模型分层结构: 国际标准化组织(ISO): 开放系统互连(Open System Interconnection，OSI)参考模型。

应用层 (报文) -> 表示层 -> 会话层 -> 传输层 (数据段或报文) -> 网络层 （分组或包） -> 数据链路层 （帧） -> 物理层 （比特流或位流）

1、数据(PDU:协议数据单元)在垂直的层次中自上而下地逐层传递至物理层。 

2、实通信:物理层的两个端点进行物理通信。 

3、虚拟通信:对等层不直接进行通信。 

4、中间系统(路由器):通常只实现物理层、数据链路层和网络层功能。 

5、结点到结点层:物理层、数据链路层、网络层。 

端到端层:传输层、会话层、表示层、应用层。

6、七层简单了解

1)物理层:无结构的比特流传输。   
2)数据链路层:“帧”，有效的差错控制。  
3)网络层:数据转发与路由。  
4)传输层:第一个端到端的层次。复用/分解，端到端的可靠数据传输、连接控制、流量控制、拥塞机制等。  
5)会话层:用户与用户之间的连接。建立会话时核实双方身份、费 用、对话控制与管理等。  
6)表示层:处理应用实体之间交换数据的语法。  
7)应用层:提供网络服务(文件传送、电子邮件、P2P等)。
       

7、1-3层主要完成数据交换和数据传输，称为网络低层。 
5-6层完成信息处理服务的功能，称为网络高层。 
低层与高层之间由第4层(传输层)衔接。

(二)OSI参考模型相关术语
     
1、数据单元:在层的实体之间传送的比特组。  
协议数据单元(Protocol Data Unit ，PDU):对等层之间传输的数据单元。

2、服务访问点:相邻层间的服务是通过其接口面上的服务访问点( Service Access Point，SAP)进行的，每个SAP有唯一的地址号码。

3、服务原语:第N层向第(N+1)层提供服务，或第(N+1)层请求N层的服务。原语的图解形式如下图。

请求:用户实体请求服务做某种工作。 

指示:用户实体被告知某事发生。

响应:用户实体表示对某事的响应。

证实:用户实体收到关于它的请求的答复。

4、面向连接的服务和无连接的服务

1)面向连接的服务(TCP): 通信双方在通信时，要事先建立一条通信线路，其过程有建立连接、使用连接(传送数据)和释放连接(拆除链路)三个过程。

2)无连接的服务(UDP):就是通信双方不需要事先建立一条通信线路，而是把每个带有目的地址的包(报文分组)送到线路上，
由系统选定路线进行传输。它不要求发送方和接收方之间的会话连接，不保证数据以相同的顺序到达。

?>
1、OSI参考模型中，在层的实体之间传送的比特组称为( B )  
A.地址
B.数据单元
C.端口
D.服务原语  
2、OSI参考模型中，应用层的协议数据单元(PDU)被称为( A )  
A.报文
B.比特流
C.分组
D.报文段  
3、计算机网络所划分的层次以及各层次协议的集合称为计算机( 网络体系结构 )  
4、提供网络服务，例如文件传送、电子邮件、P2P等服务的是OSI七层模型的( 网络层 )。


#### TCP/IP参考模型

1、最大、最重要的计算机网络—因特网的体系结构可以采用TCP/IP参考模型描述。

应用层（报文） ->  传输层（段） ->  网络互联层 （数据报） ->  网络接口层 (帧)

2、各层主要功能及主要协议

1)应用层:Internet上常见的一些网络应用在这一层。如WWW服务 (HTTP)，文件传输(FTP)，电子邮件(SMTP、POP3)等。面向连接的服务(TCP)、面向无连接的服务(UDP)。

2)传输层:应用层将用户数据按照特定的应用层协议封装好，接下来由传输层的协议负责把这些数据传输到接收方主机上对等的应用程序。TCP协议、UDP协议。

3)网络互联层:核心。解决把数据分组发往目的网络或主机的问题。路由选择，IP、ICMP、IGMP、BGP、OSPF、RIP协议等。

4)网络接口层:没有真正描述这一层的实现。具体的实现方法将随着网络类型的不同而不同。

?>
1、TCP/IP参考模型的核心层是( B )   
A.应用层
B.网络互联层
C.传输层
D.网络接口层  
2、在TCP/IP参考模型中，对应OSI参考模型的数据链路层和物理层的是( 网络接口层 )。    
3、下列不属于TCP/IP参考模型中网络互联层协议的是( A )。   
A.SNMP
B.ICMP
C.IP
D.BGP

#### 五层参考结构

结合OSI和TCP/IP参考模型， 提出综合理论需求和实际网络的五层参考模型。这是近年来、描述计算机网络中最常用、最接近实际网络的参考模型。

应用层 (报文) -> 传输层 (段) ->  网络层 (数据报) -> 物理层 (比特流)

?>
1、结合OSI和TCP/IP参考模型，提出综合理论需求和实际网络参考模型是( 五层参考模型 )。

### 计算机网络与因特网发展简史

#### 计算机网络与因特网发展简史

计算机网络的发展是随着分组交换技术的提出以及因特网的发展而逐渐发展起来的。

1、1967年发布了一个ARPAnet，ARPAnet是第一个分组交换计算机网络， 是当今因特网的祖先。

2、1969年底，ARPAnet建成由四个分组交换机互连的网络。

3、1972年底， ARPAnet已经发展到15个交换结点。

4、20世纪70年代早期与中期，除了ARPAnet之外，还诞生了许多其他分组交换网络，如ALOHAnet、Telenet。发展了三个因特网核心协议:TCP、 UDP、IP。

5、20世纪70年代末， ARPAnet已连接大约200台主机。因特网已现雏形。

6、20世纪80年代，因特网连接的主机数量达到100000台。 

7、20世纪90年代，因特网祖先ARPAnet已不复存在。

8、20世纪90年代后五年，因特网快速发展与变革的时期，企业和高校、 甚至个人开始接入因特网。

9、2000年开始，因特网进入爆发式发展期。

?>
1、被称为计算机网络技术发展里程碑的网络是( C ) 。  
A.Interact
B.无线局域网
C.ARPA网
D.多媒体网络  
2、下列关于ARPANET表述错误的是( A )  
A.ARPANET是一个开放式的标准化网络   
B.ARPANET被人们公认为分组交换网之父  
C.ARPANET是计算机网络技术发展中的一个里程碑  
D.ARPANET的主要目标是实现网内计算机能够资源共享


      
