# TCPIPDemo
Notes about TCPIP

趣谈网络协议：https://time.geekbang.org/column/85

7层协议图解：https://www.cnblogs.com/wanghuaijun/p/10092930.html

NS3路由模拟实验：https://blog.csdn.net/qq_39564555/article/details/98471596

## 1. 概述
### 1.1 分层
>网络协议通常分不同层次进行开发，每一层分别负责不同的通信功能。一个协议族，比如TCP/IP，是一组不同层次上的多个协议的组合，TCP/IP协议通常被认为是一个四层协议系统：

![image](https://user-images.githubusercontent.com/34849140/143688228-e51c3229-4be1-48d1-a784-c5119e653217.png)
- **链路层**也称为数据链路层或网络接口层，和系统中的设备驱动程序以及计算机中的网络接口卡一起处理与电缆的物理接口细节。
- **网络层**负责处理分组在网络中的活动，例如分组的选路。它包括IP网际协议、ICMP Internet互联网控制报文协议以及IGMP Internet组管理协议。
- **运输层**为两台主机上的应用程序提供端到端的通信，该协议族中有两个传输协议：**TCP传输控制协议**和**UDP用户数据报协议**，两者都是**以IP作为网络层协议**。
  - **TCP**为两台主机提供可靠请的数据通信，把应用程序交给它的数据分成合适的小块交给下面的网络层。TCP在不可靠的IP层上提供了一个可靠的运输层。为了提供这种可靠的服务，TCP采用了超时重传、发送和接收端到端的确认分组等机制。本书的17~22章节会详细讨论TCP的内部如何以不可靠的IP服务来提供一种可靠的运输层服务的细节。
  - **UDP**为应用层提供非常简单的服务，只是把称作数据报的分组从一台主机发送到另一台主机，不保证该数据报能到达另一端，可靠性由应用层来提供。本书的第11章讨论UDP、14章DNS、15章TFTP简单文本传输协议以及16章的BOOTP，介绍使用UDP的应用程序。SNMP也使用了UDP协议，放在了25章。
- **应用层**负责处理特定的应用程序细节，各种不同的TCP/IP协议族实现都会提供Telnet、SNMP等应用协议。
>网络层与运输层之间的区别似乎不是那么明显，为什么要把它们划分成两个不同的层次呢？这就要从**单个网络扩展到一组网络**说起。

![image](https://user-images.githubusercontent.com/34849140/144064189-081368f9-2f44-4a09-8d8a-44d8f87e8d06.png)

80年代，为了连接相同协议族却又孤立的系统组在一起形成互连网络，最简单的办法就是通过**路由器**进行连接，它的好处是为不同类型的物理网络提供连接：以太网、令牌环网以及点对点的链接等等。连接网络的另一个途径是使用网桥。**网桥是在链路层上对网络进行互连，而路由器则是在网络层上对网络进行互连**。网桥使得多个局域网LAN组合在一起，使其对上层来说就像一个局域网。

![image](https://user-images.githubusercontent.com/34849140/144064522-c0d2401c-ee8b-48dd-82c8-37a99df5d422.png)

>TCP/IP倾向于使用路由器而非网桥来连接网络，本书中着重介绍路由器。

**ICMP**是IP协议的附属协议，IP层用它来与其他主机或路由器交换错误和其他重要信息。它也可以被应用程序访问，7、8章节会介绍两个流行诊断工具Ping和Traceroute。

**IGMP**是Internet组管理协议，它用来把一个UDP数据报多播到多个主机，12章中会描述广播（把UDP数据报发送到某个指定网络上的所有主机）的特性。

**ARP和RARP**地址解析协议、逆地址解析协议是某些**网络接口**使用的特殊协议，用来转换IP层和网络接口层使用的地址，4、5章节会进行分析和介绍。

![image](https://user-images.githubusercontent.com/34849140/144064947-8b7d3ff9-cda5-45b4-ba11-a05cc3b4ec61.png)

### 1.2 互联网的地址
互联网上的每个接口必须有一个唯一的Internet地址，也称为IP地址，长32bit。IP地址具有以下5中结构：

![image](https://user-images.githubusercontent.com/34849140/144065435-1c555b32-388e-4c09-bc0d-3817490ebdfd.png)

负责为接入互联网的网络分配IP地址得管理机构称之为InternetNetworkIC互联网络信息中心，它只负责分配网络号，主机号的分配由系统管理员来负责。
### 1.3 封装
当应用程序用TCP传送数据时，数据被送入协议栈中，然后逐个通过每一层直到被当作一串比特流送入网络。其中每一层对收到的数据都要增加一些首部信息（有时还要增加尾部
信息）。TCP传给IP的数据单元称作TCP报文段或简称为TCP段（TCP segment）。IP传给网络接口层的数据单元称作IP数据报(IP datagram)，更准确的说法是分组（package）。**通过以太网传输的比特流称作帧(Frame)**。

![image](https://user-images.githubusercontent.com/34849140/144065814-b5cd2461-3576-4213-b583-f79f950fa648.png)

由于TCP、UDP、ICMP和IGMP都要向IP传送数据，因此IP必须在生成的IP首部中加入某种标识，以表明数据属于哪一层。为此IP在首部中存入一个长度为8 bit的数值，称作协议域。1表示为ICMP协议，2表示为IGMP协议，6表示为TCP协议，17表示为UDP协议。类似地，许多应用程序都可以使用TCP或UDP来传送数据。运输层协议在生成报文首部时要存入一个应用程序的标识符。**TCP和UDP都用一个16 bit的端口号来表示不同的应用程序**。TCP和UDP把源端口号和目的端口号分别存入报文首部中。

### 1.4 分用
>当目的主机收到一个以太网数据帧时，数据就开始从协议栈中由底向上升，同时去掉各层协议加上的报文首部。每层协议盒都要去检查报文首部中的协议标识，以确定接收数据的上层协议。这个过程称作分用（Demultiplexing）。

![image](https://user-images.githubusercontent.com/34849140/143732468-bd74b760-b6bb-4d9e-b042-6b404e403486.png)

为协议ICMP和IGMP定位一直是一件很棘手的事情。之前把它们与IP放在同一层上，那是因为事实上它们是I P的附属协议。但是在这里又把它们放在IP层的上面，这是**因为ICMP和IGMP报文都被封装在IP数据报中**。
对于ARP和RARP，我们也遇到类似的难题。在这里把它们放在以太网设备驱动程序的上方，这是因为**它们和IP数据报一样，都有各自的以太网数据帧类型**。但在之前我们又把ARP作为以太网设备驱动程序的一部分，放在IP层的下面，其原因在逻辑上是合理的。

### 1.5 端口号
>大部分网络应用程序在编写时都假设一端是客户、一端是服务器。服务器一般都通过知名端口号来识别，比如对每个TCP/IP实现来说，FTP服务器的TCP端口号都是21，每个Telnet服务器的TCP端口号都是23。这些知名端口号由Internet号分配机构来管理。客户端通常对它所使用的端口号不关心，只需要保证该端口号（客户端端口号又称为临时端口号）在本机上是唯一的就可以。

### 1.6 应用编程接口
>使用TCP/IP协议的应用程序通常采用两种应用编程接口（API）：socket和TLI（运输层接口：Transport Layer Interface）。前者有时称作“Berkeley socket”，表明它是从伯克利版发展
而来的。后者起初是由AT & T开发的。

### 1.7 测试网络
本书中所有例子运行的测试网络：

![image](https://user-images.githubusercontent.com/34849140/144066118-1e4e0bf5-46a9-411f-bec7-4c1776c19929.png)

## 2. 链路层
>本章中将详细讨论以太网链路层协议，两个串行接口链路层协议SLIP和PPP以及大多数实现都包含的环回驱动程序。以太网和SLIP是本书中大多数例子使用的链路层。

### 2.1 以太网链路层协议
>以太网是当今TCP/IP采用的主要局域网技术，它采用一种称作为**CSMA/CD（带冲突检测的载波侦听多路接入方法**的媒体接入方法。
- IEEE 802委员会公布了一个稍有不同的标准集，其中802.3针对整个CSMA/CD网络，802.4针对令牌总线网络，802.5针对令牌环网络。这三者的共同特性由802.2标准来定义，那就是802网络共有的逻辑链路控制LLC。IEEE 802网络的IP数据报封装是在RFC 1042[Postel and Reynolds 1988]中定义的。
- 在TCP/IP世界中，以太网IP数据报的封装是在RFC 894[Hornig 1984]中定义的。
两种帧格式都采用48 bit的目的地址和源地址（802.3允许使用16 bit的地址，但一般是48 bit地址），这就是我们在本书中所称的**硬件地址**。

![image](https://user-images.githubusercontent.com/34849140/144066712-6a77270a-291b-40a0-95af-acecf6b678b6.png)

其中的`CRC字段`用于帧内后续字节差错的循环冗余码检验（检验和）（它也被称为FCS或帧检验序列）。

### 2.2 SLP - 全称为串行线路IP
>它是对串行线路上对IP数据报进行封装的简单形式，适用于家庭中每台计算机几乎都有的RS-232串行端口和高速调制解调器接入Internet。
下面的规则描述了S L I P协议定义的帧格式：
- IP数据报以一个称作END（0xc0）的特殊字符结束。同时，为了防止数据报到来之前的线路噪声被当成数据报内容，大多数实现在数据报的开始处也传一个END字符（如果有线路噪声，那么END字符将结束这份错误的报文。这样当前的报文得以正确地传输，而前一个错误报文交给上层后，会发现其内容毫无意义而被丢弃）。
- 如果IP报文中某个字符为END，那么就要连续传输两个字节0xdb和0xdc来取代它。0xdb这个特殊字符被称作SLIP的ESC字符，但是它的值与ASCII码的ESC字符（0x1b）不同。
- 如果IP报文中某个字符为SLIP的ESC字符，那么就要连续传输两个字节0xdb和0xdd来取代它。
下图的例子就是含有一个END字符和一个ESC字符的IP报文。在这个例子中，在串行线路上传输的总字节数是原IP报文长度再加4个字节。

![image](https://user-images.githubusercontent.com/34849140/143899626-b8c66d69-62bf-4628-9e9c-9c74dc6f6e8f.png)

SLIP有一些缺点是：
- 数据帧中没有类型字段，如果一条串行线路用于SLIP那么它不能同时使用其他协议。
- SLIP没有在数据帧中加上检验和（类似于以太网中的CRC字段）。如果SL P传输的报文被线路噪声影响而发生错误，只能通过上层协议来发现（另一种方法是，新型的调制解调器可以检测并纠正错误报文）。这样，上层协议提供某种形式的CRC就显得很重要。在第3章和第17章中，我们将看到IP首部和TCP首部及其数据始终都有检验和。在第11章中，将看到UDP首部及其数据的检验和却是可选的。

### 2.3 PPP - 点对点协议
### 2.4 环回接口
大多数的产品都支持环回接口（Loopback Interface），以允许运行在同一台主机上的客户程序和服务器程序通过TCP/IP进行通信。A类网络号127就是为环回接口预留的。根据惯例，大多数系统把IP地址127.0.0.1分配给这个接口，并命名为localhost。一个传给环回接口的IP数据报不能在任何网络上出现。
### 2.5 MTU最大传输单元
以太网和802.3对数据帧的长度都有一个限制，其最大值分别是1500和1492字节。链路层的这个特性称作MTU，最大传输单元。不同类型的网络大多数都有一个上限。如果IP层有一个数据报要传，而且数据的长度比链路层的MTU还大，那么IP层就需要进行分片（fragmentation），把数据报分成若干片，这样每一片都小于MTU。我们将在11.5节讨论`IP分片的过程`。

## 3. IP - 网际协议
IP是TCP/IP协议族中最为核心的协议。所有的TCP、UDP、ICMP及IGMP数据都以IP数据报格式传输，IP提供不可靠、无连接的数据报传送服务。
- 不可靠的意思是它不能保证IP数据报能成功地到达目的地。IP仅提供最好
的传输服务。如果发生某种错误时，如某个路由器暂时用完了缓冲区，IP有一个简单的错误处理算法：丢弃该数据报，然后发送ICMP消息报给信源端。任何要求的可靠性必须由上层来提供（如TCP）。
- 无连接这个术语的意思是IP并不维护任何关于后续数据报的状态信息。每个数据报的处理是相互独立的。这也说明，IP数据报可以不按发送顺序接收。如果一信源向相同的信宿发送两个连续的数据报（先是A，然后是B），每个数据报都是独立地进行路由选择，可能`选择不同的路线`，因此B可能在A到达之前先到达。
- 
>在本章，我们将简要介绍IP首部中的各个字段，讨论IP路由选择和子网的有关内容。还要介绍两个有用的命令：ifconfig和netstat。

### 3.1 IP首部
![image](https://user-images.githubusercontent.com/34849140/143905383-5d92ec75-3686-42db-8214-69d07e06ae54.png)

4个字节的32 bit值以下面的次序传输：首先是0～7 bit，其次8～15 bit，然后16～23 bit，最后是24~31 bit。这种传输次序称作big endian字节序。由于TCP/IP首部中所有的二进制整数在网络中传输时都要求以这种次序，因此它又称作`网络字节序`。以其他形式存储二进制整数的机器，如little endian格式，则必须在`传输数据之前把首部转换成网络字节序`。

目前的协议版本号是4，因此IP有时也称作IPv4。

- `总长度字段`是指整个I P数据报的长度，以字节为单位。利用首部长度字段和总长度字段，就可以知道IP数据报中数据内容的起始位置和长度。由于该字段长16比特，所以IP数据报最长可达65535字节（超级通道MTU为65535。它的意思其实不是一个真正的MTU—它使用了最长的IP数据报）。当数据报被分片时，该字段的值也随着变化，这一点将在11.5节中进一步描述。总长度字段是IP首部中必要的内容，因为一些数据链路（如以太网）需要填充一些数据以达到最小长度。尽管以太网的最小帧长为46字节，但是IP数据可能会更短。如果没有总长度字段，那么IP层就不知道46字节中有多少是IP数据报的内容。
- `标识字段`唯一地标识主机发送的每一份数据报。通常每发送一份报文它的值就会加1。在11.5节介绍分片和重组时再详细讨论它。同样，在讨论分片时再来分析`标志字段和片偏移字段`。
- `TTL（Time-to-Live）生存时间字段`设置了数据报可以经过的最多路由器数。它指定了数据报的生存时间。TTL的初始值由源主机设置（通常为32或64），一旦经过一个处理它的路由器，它的值就减去1。当该字段的值为0时，数据报就被丢弃，并发送ICMP报文通知源主机。
- `首部检验和字段`是根据`IP首部计算的检验和码`。它不对首部后面的数据进行计算。ICMP、IGMP、UDP和TCP在它们各自的首部中均含有同时覆盖`首部和数据检验和码`。为了计算一份数据报的IP检验和，首先把检验和字段置为0。然后，对首部中每个16 bit进行二进制反码求和（整个首部看成是由一串16 bit的字组成），结果存在检验和字段中。当收到一份IP数据报后，同样对首部中每个16 bit进行二进制反码的求和。由于接收方在计算过程中包含了发送方存在首部中的检验和，因此，如果首部在传输过程中没有发生任何差错，那么接收方计算的结果应该为全1。`如果结果不是全1（即检验和错误），那么IP就丢弃收到的数据报。但是不生成差错报文，由上层去发现丢失的数据报并进行重传`。

### 3.2 IP网际协议的路由选择
>目的主机与源主机直接相连（如点对点链路）或都在一个共享网络上（以太网或令牌环网），那么IP数据报就直接送到目的主机上。否则，主机把数据报发往一默认的路由器上，由路由器来转发该数据报。
>IP层既可以配置成路由器的功能，也可以配置成主机的功能。当今的大多数多用户系统，包括几乎所有的Unix系统，都可以配置成一个路由器。

在一般的体制中，I P可以从TCP、UDP、ICMP和IGMP接收数据报（即在本地生成的数据报）并进行发送，或者从一个网络接口接收数据报（待转发的数据报）并进行发送。`IP层在内存中有一个路由表`。当收到一份数据报并进行发送时，它都要对该表搜索一次。当数据报来自某个网络接口时，IP首先检查目的IP地址是否为本机的IP地址之一或者IP广播地址。如果确实是这样，数据报就被送到由`IP首部协议字段所指定的协议模块`进行处理。如果数据报的目的不是这些地址，那么
- 如果IP层被设置为路由器的功能，那么就对数据报进行转发（也就是说，像下面对待发出的数据报一样处理）。
- 否则，数据报被丢弃。

路由表中的每一项都包含下面这些信息：
- `目的IP地址`。它既可以是一个完整的主机地址，也可以是一个网络地址，由该表目中的标志字段来指定（如下所述）。`主机地址有一个非0的主机号，以指定某一特定的主机，而网络地址中的主机号为0，以指定网络中的所有主机（如以太网，令牌环网）`。
- `下一站（或下一跳）路由器（next-hop router）的IP地址`，或者有直接连接的网络IP地址。下一站路由器是指一个在直接相连网络上的路由器，通过它可以转发数据报。下一站路由器不是最终的目的，但是它可以把传送给它的数据报转发到最终目的。
- `标志`。其中一个标志指明目的IP地址是网络地址还是主机地址，另一个标志指明下一站路由器是否为真正的下一站路由器，还是一个直接相连的接口（我们将在9.2节中详细介绍这些标志）。
- 为数据报的传输指定一个网络接口。

`完整主机地址匹配在网络号匹配之前执行。只有当它们都失败后才选择默认路由。默认路由，以及下一站路由器发送的ICMP间接报文（如果我们为数据报选择了错误的默认路由）是IP路由选择机制中功能强大的特性。`将在Chapter 9做详细介绍。

`为一个网络指定一个路由器，而不必为每个主机指定一个路由器，这是IP路由选择机制的另一个基本特性`。这样做可以极大地缩小路由表的规模，比如Internet上的路由器有只有几千个表目，而不会是超过100万个表目。
>EXAMPLE 1
当IP从某个上层收到这份数据报后，它搜索路由表，发现目的I P地址（140.252.13.33）在一个直接相连的网络上（以太网140.252.13.0）。于是，在表中找到匹配网络地址（在下一节中，我们将看到，由于以太网的子网掩码的存在，实际的网络地址是140.252.13.32，但是这并不影响这里所讨论的路由选择）。数据报被送到以太网驱动程序，然后作为一个以太网数据帧被送到sun主机上（见下图）。**IP数据报中的目的地址是sun的IP地址（140.252.13.33），而在链路层首部中的目的地址是48 bit的sun主机的以太网接口地址。这个48 bit的以太网地址是用ARP协议获得的**，我们将在下一章对此进行描述。

![image](https://user-images.githubusercontent.com/34849140/144076871-713d250b-ddb6-4178-9eb2-2fe62b919b09.png)

>EXAMPLE 2
主机bsdi有一份IP数据报要传到ftp.uu.net主机上，它的IP地址是192.48.96.9。经过的前三个路由器如下图。首先，主机bsdi搜索路由表，但是没有找到与主机地址或网络地址相匹配的表目，因此只能用默认的表目，把数据报传给下一站路由器，即主机sun。当数据报从bsdi被传到sun主机上以后，目的IP地址是最终的信宿机地址（192.48.96.9），但是链路层地址却是sun主机的以太网接口地址。这与上图不同，在那里数据报中的目的IP地址和目的链路层地址都指的是相同的主机（sun）。

当sun收到数据报后，它发现数据报的目的IP地址并不是本机的任一地址，而sun已被设置成具有路由器的功能，因此它把数据报进行转发。经过搜索路由表，选用了默认表目。根据sun的默认表目，它把数据报转发到下一站路由器netb，该路由器的地址是140.252.1.183。数据报是经过点对点SLIP链路被传送的，采用了最小封装格式。这里没有给出像以太网链路层数据帧那样的首部，因为在SLIP链路中没有那样的首部。当netb收到数据报后，它执行与sun主机相同的步骤：数据报的目的地址不是本机地址，而netb也被设置成具有路由器的功能，于是它也对该数据报进行转发。采用的也是默认路由表目，把数据报送到下一站路由器gateway（140.252.1.4）。位于以太网140.252.1上的主机netb用ARP获得对应于140.252.1.4的48 bit以太网地址。这个以太网地址就是链路层数据帧头上的目的地址。路由器gateway也执行与前面两个路由器相同的步骤。

![image](https://user-images.githubusercontent.com/34849140/144079642-339d29a3-5237-444c-828b-f62b6a2c255f.png)

### 3.3 子网寻址
>现在所有的主机都要求支持子网编址。**不是把IP地址看成由单纯的一个网络号和一个主机号组成，而是把主机号再分成一个子网号和一个主机号**。这样做的原因是因为A类和B类地址为主机号分配了太多的空间，事实上，在一个网络中人们并不安排这么多的主机。
>在InterNIC获得某类IP网络号后，就由当地的系统管理员来进行分配，由他（或她）来决定是否建立子网，以及分配多少比特给子网号和主机号。例如，这里有一个B类网络地址（140.252），在剩下的16 bit中，8bit用于子网号，8 bit用于主机号，就允许有254个子网，每个子网可以有254台主机。

![image](https://user-images.githubusercontent.com/34849140/144240395-88db4132-79fb-4fc3-9a98-ad8e2fa3a496.png)

**子网对外部路由器来说隐藏了内部网络组织（一个校园或公司内部）的细节**。在我们的网络例子中，所有的IP地址都有一个B类网络号140.252。但是其中有超过30个子网，多于400台主机分布在这些子网中。由一台路由器提供了Internet的接入，如下图：

![image](https://user-images.githubusercontent.com/34849140/144237762-2b33a4f0-5b36-43e8-8436-28b88f4b0dde.png)

在这个图中，我们把大多数的路由器编号为Rn，n是子网号。我们给出了连接这些子网的路由器，同时还包括了扉页前图中的九个系统。在图中，以太网用粗线表示，点对点链路用虚线表示。我们没有画出不同子网中的所有主机。例如，在子网140.252.3上，就超过50台主机，而在子网140.252.1上则超过100台主机。与30个C类地址相比，用一个包含30个子网的B类地址的好处是，它可以缩小Internet路由表的规模。B类地址140.252被划分为若干子网的事实对于所有子网以外的Internet路由器都是透明的。为了到达IP地址开始部分为140.252的主机，外部路由器只需要知道通往IP地址140.252.104.1的路径。这就是说，对于网络140.252只需一个路由表目，而如果采用30个C类地址，则需要30个路由表目。因此，**子网划分缩减了路由表的规模**。

**10.8动态选路协议-无类型域间选路小节中，会介绍一种新技术，即使用C类地址也可以缩减路由表的规模。**

子网对于子网内部的路由器是不透明的。一份来自Internet的数据报到达gateway，它的目的地址是140.252.57.1。路由器gateway需要知道子网号是57，然后把它送到kpno。同样，kpno必须把数据报送到R55，最后R55把它送到R57。

### 3.4 子网掩码
>任何主机在引导时进行的部分配置是指定主机IP地址。大多数系统把IP地址存在一个磁盘文件里供引导时读用。除了IP地址以外，**主机还需要知道有多少比特用于子网号及多少比特用于主机号。这是在引导过程中通过子网掩码来确定的**。这个掩码是一个32 bit的值，其中值为1的比特留给网络号和子网号，为0的比特留给主机号。下图是B类地址的两种不同的子网掩码格式。

![image](https://user-images.githubusercontent.com/34849140/144254216-08bb2b38-162e-4897-9d23-a9c79c10ee97.png)

第一个例子是noao.edu网络即上上图采用的子网划分方法，子网号和主机号都是8 bit宽。
第二个例子是一个B类地址划分成10 bit的子网号和6 bit的主机号。

给定IP地址和子网掩码以后，主机就可以确定IP数据报的目的是：
(1)本子网上的主机；
(2)本网络中其他子网中的主机；
(3)其他网络上的主机。

如果知道本机的IP地址，那么就知道它是否为A类、B类或C类地址(从IP地址的高位可以得知)，也就知道网络号和子网号之间的分界线。而根据子网掩码就可知道子网号与主机号之间的分界线。

>EXAMPLE
假设我们的主机地址是140.252.1.1（一个B类地址），而子网掩码为255.255.255.0（其中8 bit为子网号，8 bit为主机号）。
- 如果目的IP地址是140.252.4.5，那么我们就知道B类网络号是相同的（140.252），但是子网号是不同的（1和4）。用子网掩码在两个IP地址之间的比较如下图所示。

![3 81](https://user-images.githubusercontent.com/34849140/144263926-bf6283e7-4489-40e2-be72-d32707b3404d.png)

- 如果目的IP地址是140.252.1.22，那么B类网络号还是一样的（140.252），而且子网号也是一样的（1），但是主机号是不同的。
- 如果目的IP地址是192.43.235.6（一个C类地址），那么网络号是不同的，因而进一步的比较就不用再进行了。

**给定两个IP地址和子网掩码后，IP路由选择功能一直进行这样的比较。**

>下面这段摘录自：https://zhuanlan.zhihu.com/p/65226634

当网络号子网号不一致的时候（即上面的场景1），TCP/IP协议也会根据子网掩码判定两个网络中的主机处在不同的网络里。**要实现这两个网络之间的通信，则必须通过网关**。如果网络A中的主机发现数据包的目标主机不在本地网络中，就把数据包转发给它自己的网关，再由网关转发给网络B的网关，网络B的网关再转发给网络B的某个主机。**网关的IP地址是具有路由功能的设备的IP地址。**

### 3.5 一个子网的例子
图3-10

![image](https://user-images.githubusercontent.com/34849140/144445349-6c1cce3a-655f-48d0-ad9e-c261f2ac9774.png)

从路由器sun到上面的以太网之间的连接细节，实际上它们之间的连接是拨号SLIP。这个细节不影响本节中讨论的子网划分问题。我们在4.6节讨论**ARP代理**时将再回头讨论这个细节。

问题是我们在子网13中有两个分离的网络：一个以太网和一个点对点链路（硬件连接的SLIP链路）（点对点链接始终会带来问题，因为它一般在两端都需要IP地址）。将来或许会有更多的主机和网络，但是为了不让主机跨越不同的网络就得使用不同的子网号。我们的解决方法是把子网号从8 bit扩充到11 bit，把主机号从8 bit减为5 bit。这就叫作**变长子网**，因为140.252网络中的大多数子网都采用8 bit子网掩码，而我们的子网却采用11 bit的子网掩码。

作者子网中的IP地址结构如上图所示，11位子网号中的前8 bit始终是13。在剩下的3 bit中，我们用二进制001表示以太网（所以图中的子网是140.252.13.32，其中的32就是00100000），010表示点对点SLIP链路（子网是140.252.13.64，其中的32就是01000000）。这个变长子网掩码在140.252网络中不会给其他主机和路由器带来问题—只要目的是子网140.252.13的所有数据报都传给路由器sun（IP地址是140.252.1.29），如图下图所示。如果sun知道子网13中的主机有11 bit子网号，那么一切都好办了。

图3-11

![image](https://user-images.githubusercontent.com/34849140/144447210-407f3588-e128-462b-a4ed-35c432c79f75.png)

140.252.13子网中的所有接口的子网掩码是255.255.255.224，或0xffffffe0。这表明最右边的5 bit留给主机号，左边的27 bit留给网络号和子网号。

图3-12

![image](https://user-images.githubusercontent.com/34849140/144451610-de725386-150f-43cb-a3c1-7ce48ac9d680.png)

第1栏标为是“主机”，但是sun和bsdi也具有路由器的功能，因为它们是多接口的，可以把分组数据从一个接口转发到另一个接口。

>这个表中的最后一行是图3-10中的广播地址140.252.13.63：它是根据以太网子网号（140.252.13.32）和图3-11中的低5位置1（16＋8＋4＋2＋1=31）得来的（**我们在第12章中将看到，这个地址被称作以子网为目的的广播地址（subnet-directed broadcast address）**）。

## 4. ARP: 地址解析协议

数据链路如以太网或令牌环网都有自己的寻址机制（常常为48 bit地址），这是使用数据链路的任何网络层都必须遵从的。当一台主机把以太网数据帧发送到位于同一局域网上的另一台主机时，是根据48 bit的以
太网地址来确定目的接口的。设备驱动程序从不检查IP数据报中的目的IP地址。地址解析为这两种不同的地址形式提供映射：32 bit的IP地址和数据链路层使用的任何类型的地址。

![image](https://user-images.githubusercontent.com/34849140/144614367-a454f5d4-1f90-4b87-8cdc-1279057afc55.png)

### 4.1 例子

ARP为IP地址到对应的硬件地址之间提供动态映射。我们之所以用动态这个词是因为这个过程是自动完成的，一般应用程序用户或系统管理员不必关心。**RARP是被那些没有磁盘驱动器的系统使用（一般是无盘工作站或X终端），它需要系统管理员进行手工设置。我们在第5章对它进行讨论。**

>EXAMPLE

任何时候我们敲入下面这个形式的命令：
% ftp bsdi
都会进行以下这些步骤。这些步骤的序号如图4-2所示：
- 1.应用程序FTP客户端调用函数gethostbyname(3)把主机名（bsdi）转换成32 bit的IP地址。这个函数在DNS（域名系统）中称作解析器，我们将在第14章对它进行介绍。这个转换过程或者使用DNS，或者在较小网络中使用一个静态的主机文件（/etc/hosts）。
- 2.FTP客户端请求TCP用得到的IP地址建立连接。
- 3.TCP发送一个连接请求分段到远端的主机，即用上述IP地址发送一份IP数据报（在第18章我们将讨论完成这个过程的细节）。
- 4.如果目的主机在本地网络上（如以太网、令牌环网或点对点链接的另一端），那么IP数据报可以直接送到目的主机上。如果目的主机在一个远程网络上，那么就通过IP选路函数来确定位于本地网络上的下一站路由器地址，并让它转发IP数据报。在这两种情况下，IP数据报都是被送到位于本地网络上的一台主机或路由器。
- 5.假定是一个以太网，那么发送端主机必须把32 bit的IP地址变换成48 bit的以太网地址。**从逻辑Internet地址到对应的物理硬件地址需要进行翻译。这就是ARP的功能。ARP本来是用于广播网络的，有许多主机或路由器连在同一个网络上。**
- 6.ARP发送一份称作ARP请求的以太网数据帧给以太网上的每个主机。这个过程称作广播，如图4-2中的虚线所示。ARP请求数据帧中包含目的主机的IP地址（主机名为bsdi），其意思是“如果你是这个IP地址的拥有者，请回答你的硬件地址。”
- 7.目的主机的ARP层收到这份广播报文后，识别出这是发送端在寻问它的IP地址，于是发送一个ARP应答。这个ARP应答包含IP地址及对应的硬件地址。
- 8.收到ARP应答后，使ARP进行请求—应答交换的IP数据报现在就可以传送了。
- 9.发送IP数据报到目的主机。

图4-2：

![image](https://user-images.githubusercontent.com/34849140/144616034-1a370f63-7b65-4774-a18b-ca3e3a15bf92.png)

在ARP背后有一个基本概念，那就是网络接口有一个硬件地址（一个48 bit的值，标识不同的以太网或令牌环网络接口）。`在硬件层次上进行的数据帧交换必须有正确的接口地址`。但是，TCP/IP有自己的地址：32 bit的IP地址。知道主机的IP地址并不能让内核发送一帧数据给主机。`内核（如以太网驱动程序）必须知道目的端的硬件地址才能发送数据`。ARP的功能是在32 bit的IP地址和采用不同网络技术的硬件地址之间提供动态映射。

点对点链路不使用ARP。当设置这些链路时（一般在引导过程进行），必须告知内核链路每一端的IP地址。像以太网地址这样的硬件地址并不涉及。

### 4.2 ARP告诉缓存
