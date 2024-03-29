传输层功能、传输层寻址  
不可靠传输信道的表现，可靠数据传输原理  
停-等协议（ARQ）与滑动窗口协议（GBN、SR）  
用户数据报协议（UDP）  
TCP 特点及 TCP 连接管理（三次握手连接、四次挥手断开链接）  
TCP 可靠数据传输  
TCP 拥塞控制（慢启动、拥塞避免、快速重传、快速恢复）

### 传输层的基本服务

#### 传输层功能

1、传输层的核心任务：**应用进程之间提供端到端的逻辑通信服务**。

比如：你和你的朋友邮件通信、聊微信、聊 QQ。

2、传输层的功能

1）对应用层报文进行分段和重组 （了解）  
2）面向应用层实现复用与分解（第 2 节）  
3）实现端到端的流量控制（第 5 节）  
4）拥塞控制（第 5 节）  
5）传输层寻址（后续内容）  
6）对报文进行差错检测 （第 3 节提到，第五章详解）  
7）实现进程间的端到端可靠数据传输控制（第 3 节）

3、并不是所有的传输层协议都要实现所有的功能，大部分传输层协议只实现其中一部分的功能。

例如：internet 的传输层（TCP、UDP）

4、从传输层的角度看，通信的真正端点不是主机，而是在主机中运行的应用进程。端到端之间的通信是应用进程之间的通信。

例如：一个主机上，用户在使用浏览器浏览网页的同时、也在发邮件。 怎么确保各客户进程不出差错？复用与分解！（第二节）

#### 传输层寻址与端口

1、单个计算机中，不同应用进程用**进程标识符（进程 ID）**来区分

名称 PID  
Google Chrome 10710

不同计算机之间怎么区分应用进程？

传输层就是为了支持不同的主机，不同操作系统上的应用程序之间的通信，必须要使用**统一的寻址方式**对应用进程进行标识

TCP/IP 网络体系结构的解决方法：  
在传输层使用协议端口号，通常简称为端口  
在全网内利用**IP 地址+端口号**唯一标识一个通信端点

2、传输层端口号为 16 位整数，可以编写 65536 个（2 的 16 次方）

服务器端口号  
0-1023 熟知端口号  
1024-49151 登记端口号（IANA 登记、防止重复）

客服端口号  
49152-65535 短暂端口号

3、常用端口号

20、21 FTP 文件传输协议端口号  
25 SMTP 简单邮件传输协议端口号  
53 DNS 域名服务器端口号  
80 HTTP 超文本传输协议端口号  
110 POP3 第三版的邮局协议端口号

#### 无连接服务与面向连接服务

无连接服务： 数据传输之前：无需与对端进行任何信息交换（“握手”），直接构造传输层报文段并向接收端发送。（邮政系统的信件通信）

面向连接服务：数据传输之前：需要双方交换一些**控制信息**，建立逻辑连接，然后再传输数据，传输结束后还需要拆除连接。（电话通信）

### 传输层的复用与分解

#### 复用与分解的基本概念

1、多路复用与多路分解：支持众多应用进程共用同一传输层协议，能准确的将接收的数据交付给不同的应用进程，称为传输层的多路复用与多路分解。

2、多路复用：在源主机，传输层协议从不同的套接字收集应用进程发送的数据块，并为每个数据块封装上首部信息（包括用于分解的信息）构成报文段，然后将报文段传递给网络层。

同一主机，多个进程同时利用同一个传输层协议。

3、多路分解：在目的主机，传输层协议读取报文段中的字段，标识出接收套接字，进而通过该套接字，将传输层的报文段中的数据交付给正确的套接字。

对接收到报文段（封装了不同应用进程的数据）分解，交付给正确的进程

实现复用与分解的关键：传输层协议能够唯一识别一个套接字。

#### 无连接的多路复用与多路分解

Internet 传输层提供无连接服务的传输层协议是用户数据报协议

1、为 UDP 套接字分配端口号的两种方法：

1）创建一个 UDP 套接字时，传输层自动为该套接字分配一个端口号（1024-65535 之间），该端口号当前未被该主机中任何其他 UDP 套接字使用。

2）在创建一个 UDP 套接字后，通过调用 bind（）函数为该套接字绑定一个特定的端口号。

2、UDP 套接字二元组：<目的 IP 地址，目的端口号>

#### 面向连接的多路复用与多路分解

1、传输控制协议 ：Internet 提供面向连接服务的传输层协议。

TCP 套接字四元组：<源 IP 地址，源端口号，目的 IP 地址，目的端口号>

### 停-等协议与滑动窗口协议

#### 可靠数据传输基本原理

1、internet 传输层的两个协议：TCP 、UDP。

TCP：可靠数据传输服务。将报文段交给 IP 传送，而 IP 只能提供“尽力”服务，也就是不可靠的服务。
必须采取措施才能使其在基于不可靠的网络层上实现可靠传输。

2、不可靠传输信道的不可靠性主要表现在以下几方面：

1）比特差错：1100——0110

2）乱序（先发的数据包后到达，后发的数据包先到达）：

发送：1、2、3、4、5 接收：2、1、5、4、3

3）数据丢失（中途丢失，不能达到目的地）：

发送：1、2、3、4、5 接收：1、3、4、5

3、基于不可靠信道实现可靠数据传输采取的措施（5 种）

1）差错检测：利用差错编码实现数据包传输过程中的比特差错检测 （甚至纠正）。

2）确认：接收方向发送方反馈接收状态。  
ACK（肯定确认）；NAK（否定确认）  
肯定确认：Positive Acknowledgement，正确接收数据。  
否定确认：Negative Acknowledgement，没有正确接收数据。

3）重传：发送方重新发送接收方没有正确接收的数据。  
发送方接收到 NAK，表示接收方没有正确接收数据，则将出错的数据重 新向接收方发送。

4）序号：确保数据按序提交。  
对数据包进行编号。可以避免由于重传引起的重复数据被提交的问题。

5）计时器：解决数据丢失问题。  
发送方发送数据后启动计时器，如超时还未收到接收方的确认。主动重发数据包，从而纠正数据丢失问题

4、有效、合理地综合应用上述措施，可以设计实现可靠数据传输的协议： 例如：停-等协议与滑动窗口协议。

#### 停-等协议

1、自动请求重传（ARQ）

差错检测、确认、重传、序号、计时器

最简单的 ARQ：停-等协议

2、停-等协议的特点： 每发送一个报文段后就停下来等待接收方的确认。

3、停-等协议的工作过程：  
1）发送方发送经过差错编码和编号的报文段，等待接收方的确认。  
2）接收方如果差错检测无误且序号正确，则接收报文段，并向发送方发送 ACK，否则丢弃报文段，并向发送方发送 NAK。  
3）发送方如果收到 ACK，则继续发送后续报文段，否则重发刚刚发送的报文段。

4、几点细节讨论  
1）差错控制： 报文段、ACK、NAK 数据包均需要进程差错编码以便进行差错控制。  
2）序列号： 只需要 1 位就够了（区分是新发的报文还是重传的报文）。  
3）ACK 和 NAK： 利用重复 ACK 代替 NAK。（对上一个正确接收的报文段再次进行确认）  
4）ACK 和 NAK 差错：有错推断。

#### 滑动窗口协议

停-等协议的不足：性能差、信道利用率低

1、流水线协议（管道协议）：允许发送方在没有收到确认前连续发送多个分组。从发送方向接收方传送的系列分组可以看成是填充到一条流水线（管道）中。

最典型的流水线协议：滑动窗口协议。

2、流水线协议实现可靠数据传输，做如下改进：  
1）增加分组序号（多位）。  
2）发送方和接收方可以缓存多个分组。

3、滑动窗口协议的工作特点：  
发送方依序按流水线方式发送分组，接收方接收分组，按序向上提交。  
发送方对于已发送未收到确认的分组，必须缓存，必要时重发。发送方可以连续发送多个未收到确认的分组（取决于缓存能力）。  
接收方对未按序到达的分组，必须缓存或者丢弃并确认（取决于缓存能力）。

4、滑动窗口协议的窗口：  
发送窗口（Ws）：发送方可以发送未被确认分组的最大数量；  
接收窗口（Wr）：接收方可以缓存的正确到达的分组的最大数量；

5、滑动窗口协议的示例

6、滑动窗口协议，根据窗口的大小，两种代表性的滑动窗口协议：

1）回退 N 步协议：GBN 协议（Go-Back-N）

2）选择重传协议：SR 协议（Selective Repeat）

1）GBN 协议  
发送窗口 WS≥1 ，接收窗口 Wr=1。  
发送端缓存能力高，可以在没有得到确认前发送多个分组。  
接收端缓存能力很低，只能接收 1 个按序到达的分组，不能缓存未按序到达的分组。

GBN 发送方响应的 3 类事件：  
1、上层调：窗口未满，用“下一个可以序号”编号并发送分组，否则拒绝发送新的数据。  
2、收到 1 个 ACKn。GBN 采用累积确认方式，即发送方收到 ACKn 时，表明接收方正确接收序号 n 以及序号小于 n 的所有分组。  
3、计时器超时。发送方只使用一个计时器，对“基序号”指向的分组 计时。如超时，重发当前发送窗口中所有已发送但未确认的分组，即 “回退 N 步”，因为接收方 Wr=1，无缓存能力。

GBN 的接收方操作：  
Wr=1，只能接收“基序号”所指向的分组。如接收方正确接收到序号为基序号，则发送一个 ACKn，
接收窗口滑动到序号 n+1 的位置。接收到的序号不是 n 或者分组差错等，则发送 ACKn-1

GBN 协议发送窗口示例：  
发送方如果收到 6-10 号的确认，则发送窗口便可滑动；如果计时器超时，重发 6-10 号分组，共 5 个分组。

GBN 协议总结：  
在差错较低的情况下，信道利用率会得到很大提高。如果信道误码率或者丢包率较高，导致大量重发，信道传输能力降低。  
GBN 适合低误码率、低丢包率、带宽高时延积信道，且对接收方缓存能力要求低

2）SR 协议  
选择重传（SR）通过让发送方仅重传那些未被接收确认（出错或 者丢失）的分组，避免了不必要的重传。  
发送窗口 WS ＞ 1，接收窗口 Wr ＞ 1。很多 SR 协议 WS 、Wr 大小相等。  
发送端缓存能力高。 接收端缓存能力高。

SR 发送方响应事件：  
1、上层调用，请求发送数据：检查“下一个可用序号”，位于发送窗口内则发送，否则缓存或者返回给上层。  
2、计时器超时。发送方对每个分组进行计时，超时则重发该分组。  
3、收到 ACKn。SR 协议对 n 进行判断。如 n 在当前窗口内，则标记已接收 （刚好是基序号，窗口向右滑动到最小未被确认序号处）；其他情形不做响应。

SR 接收方主要操作：  
1、正确接收到序号在接收窗口范围内的分组 PTKn，发送 ACKn，窗口滑动。  
2、正确接收到序号在接收窗口左侧的分组 PTKn，这些分组在之前已经正确接收并提交，丢弃 PTKn，并发送 ACKn，窗口不滑动。  
3、其他情况，直接丢弃分组，不做任何响应。

### 用户数据报协议（UDP）

#### UDP 基本知识

1、用户数据协议（User Datagram Protocol，UDP）：Internet 传输层协议，提供无连接、不可靠、数据报尽力传输服务。
通信进程间没有握手过程； UDP 没有拥塞控制机制； 实现了复用与分解以及简单的差错检测； DNS 是一个使用 UDP 的应用层协议的例子。

2、许多应用选择 UDP 的原因（UDP 特点）：  
1）应用进程容易控制发送什么数据以及何时发送。可能会出现分组的丢失和重复。  
2）无需建立连接。DNS 运行在 UDP 上的主要原因。  
3）无连接状态。UDP 无连接，无需维护连接状态。  
4）首部开销小，只有 8 个字节。TCP 报文段需要 20 字节的首部开销。

#### UDP 数据报结构

1、UDP 首部四个字段：每个字段长度都是 2 个字节，共 8 个字节。  
源端口号和目的端口号：UDP 实现复用和分解。  
长度：指示 UDP 报文段中的字节数（首部和数据的总和）。  
校验和：接收方使用来检测数据报是否出现差错

#### UDP 校验和

1、UDP 校验和：提供差错检测功能。  
UDP 的校验和用于检测 UDP 报文段从源到目的地传送过程中，其中的数据是否发生了改变。

2、UDP 校验和计算规则：  
1）所有参与运算的内容按 16 位对齐求和。  
2）求和过程中遇到溢出（即进位）都被回卷（即进位与和的最低位再相加）。  
3）最后得到的和取反码，就是 UDP 的校验和，填入 UDP 数据报的校验和字段。

3、UDP 校验和计算的内容包括 3 部分：UDP 伪首部、UDP 首部、应用数据

4、UDP 计算校验和示例：

5、UDP 校验和总结： UDP 提供差错检测，但是它没有差错恢复能力，只是简单地丢弃差错报文段，或者将受损的报文段交给应用程序并给出警告，由应用程序处理出错报文。
