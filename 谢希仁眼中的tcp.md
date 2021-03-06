# 运输层

运输层提供应用进程间的逻辑通信。应用层将数据交给运输层后，由传输层将数据发送到对端。在应用层面看是运输层为其通信提供保障，但是在全局的视角看运输层会将数据包交给网络层、经过数据链路层、网络中的交换机，对端的网络层和运输层。

![1643420201349](D:\code\tcp\assets\1643420201349.png)



### 主要协议

- UDP：User Datagram Protocol，用户数据报协议
- TCP：Transmission Control Protocol，传输控制协议



主要应用层使用的传输层协议：

| 应用层协议 | 传输层协议 |
| ---------- | ---------- |
| DNS        | UDP        |
| DHCP       | UDP        |
| HTTP       | TCP        |
| FTP        | TCP        |
| TELNET     | TCP        |

这者的区别：

- 面向连接
  - UDP面向无连接，传输数据之前不需要建立连接
  - TCP面向连接，传输数据之前需要建立连接，传输完数据后释放连接



运输层端口的理解：

端口的本质是应用程序在运输层的地址。进程将数据发送给传输层，传输层需要：1）将发送的数据发送到指定的应用程序；2）接收数据将其发送到指定的进程。ip地址可以标识不同的主机。因此传输层需要将数据包发送、接收数据包到指定进程。由此端口诞生了。在书上提到了和进程号作为不同进程的标识做了对比，由于进程号的动态，导致其不太适合标识应用。

端口号为16位，则可用的端口为65535。可以分为三大类：

- 系统端口号：0-1023 。这些端口已经分配给常用的应用。

  | 应用程序 | 端口号 |
  | -------- | ------ |
  | FTP      | 21     |
  | TELNET   | 23     |
  | DNS      | 53     |
  | HTTP     | 80     |
  | HTTPS    | 443    |

- 登记端口号：1024-49151 。为用户的应用程序使用。

- 客户端使用端口：49152-65535 。又称短暂端口号，用于建立tcp连接时供客户端使用。

### UDP篇

### 特点

- 无连接：发送数据之前不需要建立连接

- 尽最大努力交付：不保证可靠交付

- 面向报文：upd将应用层的数据作为一个完成的报文发送出去，不做数据上的切分。如果报文太大，会在ip层进行分片，这降低了IP层效率。如果报文太短，传输的效率太低。需要在应用层面控制一个合适的报文长度

  ```java
   		DatagramSocket clientSocket=new DatagramSocket();//定义UDP数据报
          InetAddress IPAddress=InetAddress.getByName("localhost");//获取地址
          String sentence=inFromUser.readLine();
          byte[] sendData=new byte[sentence.length()];
          sendData=sentence.getBytes(); //面向字节流，要发送byte数据
          DatagramPacket sendPacket=new DatagramPacket(sendData,sendData.length,IPAddress,9876);//定义发送数据报包
          clientSocket.send(sendPacket);//利用socket发送数据报包
  ```

  从上面可以看到udp每次发送一个packet。

- 没有拥塞控制

- 支持一对一、一对多、多对一和多对多通信

- 首部只有8字节，开销较小

### 报文格式

每个udp报文分为两个部分：首部字段（8字节）+剩余的数据部分。

![1643430763114](D:\code\tcp\assets\1643430763114.png)

#### 首部字段

- 源端口：2个字节

- 目的端口：2个字节

- 长度：2个字节，报文的总长度。最小值为8（只有首部）

- 校验和：2个字节

  计算方法：

  - 数据预处理：在校验和中填充全0，在报文段前添加伪首部，填充数据字段使其为偶数字节。
  - 依次计算每16位的和
  - 将计算结果取反作为校验和

![1643457393714](D:\code\tcp\assets\1643457393714.png)

​	伪首部说明：第三字段为0，第四字段17表示为UDP，第五字段为UDP的总长度

### TCP

#### 特点

- 面向连接：传输数据前建立连接，传输完数据关闭连接

  这里的连接其实是tcp为了确保数据传输正确所做的传输准备：比如建立发送和接收缓冲区，协商数据段号。可以通过建立连接和关闭连接的过程推断得到。

- 连接是端到端的连接（一对一）
- 提供可靠交付的服务：传输的数据是无差错、不丢失、不重复、按序到达
- 全双工通信。tcp的两端都有收发缓冲区，应用层可以将需要发送的数据放到发送缓冲区或从接收缓冲区读取数据到应用层，tcp会在合适的时机将数据发送出去，接收到数据时tcp也会将数据放到接收缓冲区。只要缓冲区有空间，应用层就可以接收或者发送数据

- 面向字节流。TCP中的流表示流入和流出进程的字节序列。tcp将应用层的数据看做是无结构的字节流。tcp保证接收端的字节序列和发送端的字节序列一样（顺序，个数），但是不保证接收端的接收到的数据块和应用程序所发出的数据块具有对应的大小关系：这块的不太好理解，其实这个是无结构字节流的含义，假设发送端的应用程序总共发送10000byte的数据，可能是通过2次提交到tcp的发送缓冲区，第一次1个byte，第二次9999个byte。tcp会保证接收端收到10000byte数据，但是不保证接收端的应用程序接收的方式为第一次1个byte，第二个9999个byte，有可能接收端是一从接收10000个byte，或者其它的方式。这说明在tcp看来字节流中的每个字节是平等的，这个主要是为了和UDP面向报文进行区分，UDP是面向报文段的，报文段之间有边界，而tcp会根据自己的机制对发送端进行切分发送出去。详细参考[invalid s](https://www.zhihu.com/question/53960871/answer/137346451) 的回答

#### TCP的连接

tcp的连接的两个端点是socket，而socket=(ip:port)，所以tcp连接::={socket1,socket2}={(ip1:port1),(ip2:port2)}。此处的socket是一个抽象的概念，可以通过ip和port唯一确定。和我们进行网络编程时的socket是两个概念。

#### 可靠传输的原理

不做任何控制就能实现可靠传输需要满足的条件：

- 信道条件：信号经过传输信道后不产生差错
- 接收端条件：不管发送端以何种速率发送接收端都能接收并处理收到的数据

实际的信道中存在大量噪声会让信号以一定概率产生乱码，不同的接收端的内存尺寸的上限不同，这对要求发送端的速率需要根据不同的接收端调节，否则没有及时处理的数据会被新到的数据覆盖。

在实际的网络环境中为了实现数据的可靠传输需要对传输的过程进行控制，换句话说我们需要协议。

#### 最简单的可靠传输协议-停止等待协议

停止等待：每发送完一个分组就停止发送，等待对方的确认，等收到了分组的确认后再发送下一个分组，如果没有收到则等待一定时间后重新发送直到收到对端的确认。

分两种情况讨论：

A、B为发送端和接收端。

- 无差错情况

![1643524540675](D:\code\tcp\assets\1643524540675.png)

​	条件：信号能够无差错的发送到接收端，并且发送端的发送速率和接收端的处理速率匹配。

​	可见，发送端只有接收到对当前分组的确认才发送新的分组，接收端接收到分组后会发送对分组的确认。

- 有差错情况

  ![1643524893971](D:\code\tcp\assets\1643524893971.png)

  导致出现差错的情况可能为：

  - 发送的分组在信道中传输丢失
  - 发送端的分组发生差错，接收端通过计算校验和知道接收到的分组是错误的，接收端在这种情况下不会发送确认分组，和分组丢失情况的处理一样就是什么都不做（在谢的书中说是接收端收到错误的分组不会向发送端通知的，其实这种情况下是接收端是可以向发送端发送分组说明让其重传的，这样可以减少发送端的等待，至于原因笔者目前能想到的是：1.简化接收端的复杂度，这样发送端就不必区分发送分组是在信道中丢失换是发生错误。2.接收端可能收到的是一个早已废弃的数据包，如果接收端需要对接收的错误分组响应会增大接收端的负载）
  - 接收端突然崩溃，无法接收发送的分组
  - 接收端对分组的确认丢失

  这些情况导致的结果都是发送端收不到发送分组的确认。在等待一定时间后换是没有确认，发送端会重新发送分组直到收到对分组的确认。

  发送端实现超时重传需要做如下处理：

  - 已经发送但是没有被确认的分组需要保留，因为这些分组可能需要重新发送
  - 需要对待发送分组进行编号，这样就可以明确哪些分组发送成功，哪些分组需要重新发送。试想发送端需要发送的分组为1,2,3。如果是发送端的发送窗口为1，接收到确认信号后需要明确该确认分组对对当前窗口的确认换是已经被确认过的分组的确认，比如，发送端分组1已经发送并且收到确认，分组2已经发送但是没有确认。发送端可能收到2的确认分组，也有可能收到1的确认分组，因为分组1可能发送超时重传，接收端发出的确认分组在网络中滞留了一段时候后才达到发送端。
  - 超时时间应该比分组包的平均往返时间长些。如果短的话会造成不必要的重传，如果长的话会造成接收端过大的等待时间

##### 确认丢失

如果确认分组丢失，发送端收不到确认会进行超时重传，在这种情况下接收端可能会收到大量的重复分组。

接收端的处理如下：

1. 丢弃该重复分组
2. 向发送端发送确认

可见重复分组的发送导致重复确认，这样确保即使接收信道差发送端也能收到确认。发送端的超时重传会使得接收端一定能收到发送数据，接收端对重复分组的确认确保发送端接收到确认。从而整个连接发送的数据是可靠的。

##### 确实迟到

发送端收到重复确认只是收下后将其丢弃。

基于上面的这种方式实现的可靠传输称为自动重传请求ARQ（Automatic Repeat reQuest）。

#### ARQ的信道利用率

计算采用ARQ时信道利用率：

![1643530438826](D:\code\tcp\assets\1643530438826.png)

参数说明：

- Td为发送分组需要的时间，RTT为信号从发送端到接收端往返的时间，Ta为接收端发送确认需要的时间
- 分组和确认都能正确接收

则信道利用率U为：

![1643530675450](D:\code\tcp\assets\1643530675450.png)

例如：假定RTT=20ms，分组长度为1200bit，发送速率1Mbit/s，忽略Ta，则u=5.6%。可见信道的利用率是很低的。考虑到重传，更高的发送速率，u的值会更小。

注：发送速率的单位的bit，而不是byte。家里的宽带的单位也是bit，要得到byte需要除8 。

信道利用率低的主要是原因在于每次发送和确认一个分组，导致信道在RTT时间内闲置，如果一次可以发送多个分组会如何？如下

![1643531429491](D:\code\tcp\assets\1643531429491.png)

信道一直有数据传输其利用率会提升。

采用这用方式的协议成为**连续ARQ协议和滑动窗口协议**

#### 连续ARQ协议

##### 发送端

发送端可以一次发送多个分组，然后等待响应，没收到一个确认发送窗口就向前移动一个分组，可以发送新的分组了。

![1643531995458](D:\code\tcp\assets\1643531995458.png)

##### 接收端

采用累积确认，可以对按序到达的最后一个分组发送确认，表示到这个分组的所有分组已经收到。比如接收到分组1，2、3、4，可以只发送一个确认4表示收到1，2,3,4分组。

优势：容易实现，可以容忍一定的确认丢失。

劣势：无法反映所以接收到的分组的信息。比如：接收端接收到1,2,4，则发送确认2表示1,2收到，但是4收到这个信息丢失，发送端需要重新发送3,4，即使4已经收到。这种情况成为回退N。

#### TCP报文首部格式

![1643532776399](D:\code\tcp\assets\1643532776399.png)

tcp的分组可以分为首部、数据部分。首部可细分为20字节的固定长度+可变部分（选择、填充），可变部分的长度需要为32bit的整数倍，否则需要填充。

字段含义：

- 源端口、目的端口：发送端和接收端的端口，作为tcp套接字的一部分。占用长度16bit
- 序号：32bit。tcp中每个byte占用一个序号，当32bit的需要用完后从0重新编号。比如，tcp的报文段的序号为301，携带数据的长度为100byte，表示该分组的第一个byte的编号为301，总共有100byte的数据，最后一个byte的编号为400。则接收端发送的确认号为401，表示400编号之前的数据已经收到。正常情况下发送端下次发送数据的序号需要从401开始

- 确认序号：32bit。期望收到的下一个分组的第一个数据字节的编号。累计确认，表明该序号之前的分组已经收到。具体例子参考上面
- 数据偏移：4bit。数据起始处距离tcp首部的长度。注意其单位为4byte。最大整数为15，表示首部的长度最长为60byte。首部中的选型和填充最大为40byte
- 保留：6bit。保留位，目前默认为全0.
- 控制位：
  - 紧急URG：该位置为1表示该分组为紧急数据，需要立马发送出去。
  - 确认ACK：为位置为1表示确认字段生效。TCP规定，在连接建立后所有传送的报文段都必须把ACK置为1
  - 推送PSH：该位置1时生效。发送端碰到该位生效时立马将数据push出去，接收端接收到该数据立马交付给应用程序，而不必等到缓冲区满。
  - 复位RST：该位为1时生效。当tcp连接出现严重差错（主机崩溃或其它原因），必须释放连接。该位还可以拒绝一个非法的报文段和拒绝打开一个连接。
  - 同步SYN：该位为1时生效。当SYN=1、ACK=0表示这是一个连接请求报文，当SYN=1、ACK=1表示这个是一个连接接收报文。
  - 终止FIN：该位为1时生效。用来表示数据传送完毕，可以释放连接。

- 窗口：16bit。表示发送该报文方的接收窗口。该数值用来控制发送方的发送速率。例如，发送报文段确认区号是700，窗口为1000，表明接收方最大接收的字节数为1000，则发送方应该发送的报文序号在700-1699之间。**窗口值指明了允许对方发送的最大数据量，窗口值经常动态变化**

- 校验和：16bit。计算校验和基本和UDP一样，需要添加伪头部，计算的数据包括：伪头部、首部和数据。

- 紧急指针：16bit。在URG=1时才生效，表示本报文段紧急数据的字节数。

- 选型：最长为40byte。

  比较重要的选项如下：

  - 最大报文段长度：MSS（max segment size），容易让人产生误解，其实是报文段中数据部分的长度，不包括报文段首部的长度。
  - 窗口扩大：固定首部中的窗口的大小为16bit，则最大报文段是64kbyte，在卫星这种高延迟高带宽的环境下是不够的，需要更大的窗口。选型中的窗口扩大的最大值为14，则扩展后的窗口最大值为![1643551852585](D:\code\tcp\assets\1643551852585.png)
  - 时间戳：占用10byte，包含两个字段：时间戳字段（4字节）、时间戳回送回答字段（4字节）。发送方将当前时间写入时间戳字段，接收方在确认该报文段将时间戳字段复制到时间戳回送回答字段。时间戳字段的含义如下：
    - 计算分组往返时间RTT。
    - 防止序号绕回PAWS，处理TCP序号超过![1643768321482](D:\code\tcp\assets\1643768321482.png)的情况。通过时间戳可以过滤老旧的数据包

##### 如何利用选型中的时间戳计算RTT

主要是为了在后面计算超时时间设置时使用。

TCP可选项格式：

![1644276023228](D:\code\tcp\assets\1644276023228.png)

时间戳选项占10个字节=kind（1字节）+length（1字节）+info（8字节），其中kind=8，length=10，info由timestamp、timestamp echo两个字段组成，每个字段占4个字节。

示例演示：

发送端a，接收端b。a发送分组c时的时间戳字段为发送时间ta1。b接收到分组c，对c发送确认报文d时将分组c中的timestamp字段ta1复制到确认报文中的timestamp echo字段，将发送确认的时间tb写入到timestamp中。发送端a接收到报文段d，记录下当前的时间段ta2。从d中解析出timestamp echo的值ta1，那么：RTT=接收ack报文的时刻-发送报文的时刻=ta2-ta1.

#### TCP可靠传输的实现

##### 以字节为单位的滑动窗口

下图为一个常见的发送端的窗口展示：

![1644198872502](D:\code\tcp\assets\1644198872502.png)

前置条件：

- 滑动窗口的单位为字节
- 发送端接受到的ACK为31
- 发送窗口的大小为20
- 发送端表示A，接收端表示为B

###### 发送窗口

在没有收到B的确认的情况下，A可以将连续把窗口内的数据都发送出去。凡是已经发送过的数据，在未收到确认之前都必须暂时保留，以便在超时重传时使用

- 发送窗口受到接受端和拥塞的共同控制
- 发送窗口的后沿表示已经发送且收到确认
- 发送窗口的前沿表示不允许发送
- 发送窗口的后沿的移动方向：前移（收到确认）、不动（没有收到新的确认）
- 发送窗口的前沿的移动方向为：前移（收到确认）、不动（没有收到新的确认）和后移（可以但是不推荐）

#### 发送过程模拟

![1644234694870](D:\code\tcp\assets\1644234694870.png)

P1、P2、P3位置之间的关系：

- P3-P1=发送窗口

- P3-P2=可用窗口

- P2-P1=已经发送但是未收到确认的字节

##### 场景1：接收端接收到序号32-33，但是没有收到31

![1644234255951](D:\code\tcp\assets\1644234255951.png)

条件：

- A已经发送了31-41编号的字节
- B收到了32-33编号的字节，31编号在信道中丢失

发送端收到32、33分组时会发送ACK31的确认号，因为确认编号ACK N表示前N-1的编号已经收到。

##### 场景2：接收端接收到分组31-33

![1644234853602](D:\code\tcp\assets\1644234853602.png)

接收端发送ACK 34表示收到了31、32、33分组，接收窗口移动3个字节，接收窗口区间为34-53。发送端接收到ACK 34后发送窗口右移3个字节，发送窗口区间为34-53.

##### 场景3：发送端的可用窗口为0

![1644235312155](D:\code\tcp\assets\1644235312155.png)

发送端将发送窗口区间内的所有字节发送出去，但是没有收到确认，会停止发送。

##### 场景4：A一直没有收到B对31分组的确认

![1644235690686](D:\code\tcp\assets\1644235690686.png)

如果在指定时间内没有收到确认会触发超时重传，A重新设置超时计时器，重新发送31分组，重复上述过程直到收到B的确认为止。

#### 发送、接收缓存和发送、接收窗口的关系

![1644235579054](D:\code\tcp\assets\1644235579054.png)



前置条件：

- 发送、接收缓存其实是环形，这样可以复用内存。比如在发送缓存中已经被确认的字节已经没有用了，新的需要发送的数据可以直接覆盖

结论：

- 发送缓存的后沿和发送窗口的后沿重合，发送缓存的前沿大于发送窗口的前沿，也大于应用程序写入发送缓存中待发送数据的前沿，发送窗口的前沿小于待发送数据的前沿的。试想下在发送的数据已经完了时发送窗口的前沿和待发送数据的前沿是重合的。

- 程序写入发送缓存中的速度不能太快，太快的话发送缓存用完需要停止
- 接收窗口的前沿和接收缓存的前沿重合，这样可以确保尽可能提高信道的效率。接收缓存的后沿和为应用程序待读取的字节。
- 程序接收的速度不能太慢，如果太慢了则接收创建会越来越小直到为0
- tcp面向字节流是利用缓存作为中间媒介将应用程序写入/读取缓存中的数据和发送/接收数据到缓存的过程解耦，典型的生产者和消费者模式。

#### 累积确认碎碎念

接收方必须有累计确认的功能，可以采用：

- 延迟确认，延迟太大会触发重传，tcp有最大的延迟确认时间限制
- 在发送数据时捎带上确认

#### 超时重传时间的计算

超时重传的时间的时间不能太小，太小的话会造成不必要的重传，太大的话会造成信道长时间的利用率低，没有收到ACK导致滑动窗口长时间没有移动。实践中选用比RTT稍微大的值作为超时时间。问题来了：如何计算RTT？

##### 计算RTT的方式

TCP中使用自适应算法：记录报文段发出的时间ts，和收到对应确认的时刻te，报文段的往返时间RTT=te-ts。为了避免计算出来RTT过大或过小的变化，需要使用平滑处理后的时间RTTs。
$$
新的RTTs=\begin{cases}
RTT （第一次） \\
(1-\alpha)*旧的RTTs+\alpha*(新的RTT)
\end{cases}
$$
当α远远小于1时，新的RTT对RTTs影响较小（RTTs更新较慢），当α接近1时，新的RTT对RTTs影响较大（RTTs更新较快）。RFC6298建议α=1/8.

超时重传时间RTO（retransmissionTime-Out）应该大于RTTs。RFC6298建议RTO的计算公式为：
$$
RTO=RTTs+4*RTT_D
$$
$$RTT_D$$是RTT偏差的加权平均值。RFC6298建议$$RTT_D$$的计算公式为：
$$
新的RTT_D=\begin{cases}
RTT/2 （第一次） \\
(1-\beta)*旧的RTT_D+\beta*(RTTs-新的RTT)
\end{cases}
$$
推荐的$$\beta$$值为1/4.

##### 计算RTT存在的问题

注：此时计算RTT使用的方法上面提到的自适应算法RTT=te-ts。

问题：如何在在分组重传的情况下如何确定ts？

![1644282928608](D:\code\tcp\assets\1644282928608.png)

如果是对重传报文的确认被误当做原始报文的则会导致计算的RTT偏大，多次重传会导致RTT越来越大，降低了信道的传输效率。

如果是对原始报文的确认被误当做是超时报文的则会导致计算出来的RTT偏小，多次重传会导致RTT越来越小，会造成不必要的重传。

Karn提出的计算RTT的算法：在计算RTTs时只要报文段重传了就不采用其往返时间样本。但是如果在信道的时延都变大了，发送端在老的RTT内没有收到确认会重传报文，此时需要增大RTT，根据Karn的算法是不会更新RTTs的。为了应对这种场景需要修正Karn算法。

修正后的Karn算法：报文段每重传一次就将RTO变为之前的2倍。当报文段不再重传时使用RFC6298中的RTO计算方法。

#### 选择确认SACK

发送端是通过滑动窗口发送多个报文分组，接收端可能会接收到多个不连续的报文段，按照tcp的确认规则，此时会发送第一个未接收到的确认号的，这个会导致发送端会将已经接收到的分组也会重传。具体示例如下：

![1644320221590](D:\code\tcp\assets\1644320221590.png)

接收端接收到编号1-1000、1501-3000、3501-4500编号的分组，此时发送的确认报文为1001，表示1001之前的分组已经收到。而1501-3000、3501-4500这些已经收到的分组的信息发送端无法获得，会在超时重传中重新发送1501-3000、3501-4500分组，浪费了带宽资源。

然后出现了SACK（selective ACK），在TCP首部的选项中将已经收到的边界L1-R1、L2-R2这些信息通知给发送方，这样发送方在重传时可以只重传未收到的报文分组。

注意：tcp首部的选择最大是40字节。而表示一个字节块需要8个字节（开始位置和结束位置需要4个字节），一个字节标识该选项是SACK，一个字节表示选项的长度，这意味着表示4个块的信息需要34字节（32+1+1）。

### TCP的流量控制

流量控制：发送端的发送速率不要太快，要让接收方来得及接收。

#### 利用滑动窗口实现流量控制

通过示例来说明：
![1644323493351](D:\code\tcp\assets\1644323493351.png)

前置条件：

- 连接建立阶段确认的接收窗口为400
- 每个分组中的数据为100字节
- 确认方式为延迟一段时间的累积确认
- 发送窗口和接收窗口保持一致，实际中还受到信道中拥塞程度的影响

过程说明：

- 发送端接收到ack=201时，发送窗口被调整为300，可用窗口200（seq=201已经发送出去），所以seq=301，seq=401的分组可以发送出去，此后不再发送新的数据
- seq=201发生超时重传
- 接收到ack=501表示seq=201，seq=301，seq=401的分组已经收到，可见此时seq=301、seq=401的分组还为超时，或者是开启了SACK。此时发送窗口调整为100
- ack=601表示seq=501的分组收到，并且接收窗口为0表示不再接收新的数据
- 通过报文中接收窗口值控制了发送端的发送速率，实现了流量控制

#### 流量控制存在的死锁分析

当接收端的接收窗口为0时表示接收端不再接收数据。等到接收端的可以继续接收数据时发送新的rwnd报文到发送方，然后发送方又可以接着发送数据了。此时，如果接收端调整窗口的报文丢失，发送方和接收方会陷入死锁：发送方等待接收方新窗口值的报文，而接收方等待发送方的数据报文。

可以通过持续计时器解除死锁：发送端接收到rwnd=0的确认后，就启动持续定时器，在定时器到期就发送零窗口探测报文，接收方收到探测报文后发送新的接收窗口值的确认报文。如果发送方收到新的rwnd报文后继续发送数据，如果没有收到则重新设置持续定时器重复上述过程。

说明：即使接收端的接收窗口为0可以接收零窗口探测报文、确认报文和携带紧急数据的分组。

#### TCP的传输效率

上面所述的过程可以实现流量控制，但有一个点被忽视了：时机，发送端何时发送分组，接收端何时发送确认。

##### 发送端发送数据分组探究

造成tcp发送数据分组的时机：

- 发送缓存中的数据超过了MSS（最大报文段长度）
- 应用程序指明要求发送，tcp的push功能
- 发送方的计时器期限到了

但是上述扔就无法适用所有的场景，如果发送方是一个字节一个字节的发送，则每发送一个字节总共需要41字节（20字节tcp首部+20字节ip层首部），效率是及其低的。然后就出现了Nagle算法：若发送应用进程把发送的数据逐个字节的送到TCP的发送缓存，则发送方就把第一个数据字节先发送出去，把后面到达的数据字节缓存起来。当发送方收到对前一个分组的确认后才将发送缓存中的数据组装为一个tcp报文发送出去，可见大体过程遵循停-等协议，这样可以明显的减少带宽。

还有另外一种场景中发送端是每次发一个字节的，糊涂窗口综合征。如果接收端的接收缓存已满，接收窗口为0，此时接收缓存有多余的一个字节可以接收，然后发送rwnd=1给发送端。发送端发送一个字节的数据给接收方。接收方仍设置rwnd=1，这样每次发送端只能发送一个字节的数据。可以通过延迟一段时间后：可以接收一个MSS大小的报文，或者空闲的空间达到接收缓存的一半时再发送确认。

### TCP中的拥塞控制

拥塞：在某段时间内，若对网络中某一资源的需求超过了该资源所能提供的可用部分，网络的性能就要变坏。用等式表示如下：
$$
\sum对资源的需求>可用资源
$$


拥塞控制：防止过多的流量注入到网络中导致网络的处理能力

拥塞控制的难度：

- 影响因素较多，即使增加了某一个资源的供给，仍受到其它因素的控制。比如即使增加了网络中的带宽，可能会收到交换机处理速率的影响
- 拥塞常常趋于恶化。交换机没有足够的缓存空间保存分组，会将新的接收到的分组抛弃。这导致发送端报文的超时重传，造成更多的流量进入网络

##### 拥塞控制和流量控制的异同：

同：

- 都需要对发送端的速率进行控制

异：

- 流量控制是个端到端的问题，关注点是发送方和接收方。拥塞控制涉及到网络中的各个节点：信道的带宽，路由器和交换机的处理能力
- rwnd窗口是有接收端通知给发送方。cwnd需要由发送方根据拥塞算法计算得出

#### 输入网络中的流量和网络吞吐量的关系

![1644409434962](D:\code\tcp\assets\1644409434962.png)

特点：

- 理想的拥塞控制在没有发生拥塞时提供的负载和网络的吞吐量相等，提供多少的负载就有多少的吞吐量。达到系统的阈值后吞吐量不再提高
- 无拥塞控制时随着提供的负载增加，网络的吞吐量会明显的小于理想的吞吐量进入轻度拥塞。提供的负载继续增加直到超过了系统的阈值后，网络的吞吐量随提供负载的增加而下降，直到进入死锁。死锁状态时网络的吞吐量为0
- 实际的拥塞控制在轻度拥塞状态时吞吐量反而不如无拥塞控制的吞吐量，这主要是由于拥塞算法会进行额外控制，比如主动队列管理会随机抛弃分组造成发送端的超时重传。经过拥塞控制后吞吐量的阈值较高，并且不会进入死锁

##### 网络拥塞的开环控制和闭环控制

网络拥塞开环控制：在设计时考虑到影响造成拥塞控制的因素，避免这些因素造成拥塞。这个实际中也采用比如使用更高的宽带，对交换机和路由器纵向扩展，铺设新的网络。

网络拥塞的闭环控制：

1. 监测网络中何处、何时发送拥塞
2. 将拥塞信息发送到可以采取行动的地方
3. 调整网络系统的运行消除该处的拥塞

问题1：如何监测网络发生拥塞？

两类办法：

1. 采集网络中的指标：超时重传的分组数、平均分组时延
2. 路由器或交换机将拥塞状态写入到转发的分组中标识是否发送拥塞

### 拥塞控制方法

#### 基于窗口的拥塞控制

发送方维持一个拥塞窗口cwnd的状态变量，cwnd的大小取决于网络的拥塞程度。

发送方控制拥塞窗口的原则：只要网络中没有出现拥塞，拥塞窗口就可以再增大一些，以便将更多的分组发送出去。在出现拥塞或者有可能出现拥塞就将拥塞窗口减小些，缓解网络的拥塞。

在拥塞控制算法中都在体现这种原则。

如何监测网络拥塞？通过超时作为拥塞的标志。

拥塞控制的涉及到的算法总共有四种：慢启动、拥塞避免、快重传、快恢复。

以下说明问题的前置条件：

- 数据是单方面传送的，对方只传送确认报文
- 发送窗口只受到拥塞窗口限制

##### 慢启动

当主机开始发送数据时，并不清楚网络的拥塞状态，如果发送过多的数据到网络会加剧网络的拥塞。所以先发送小的数据进行探测，然后再逐步扩大拥塞窗口。每收到一个新报文段的确认后，可以增大拥塞窗口。$$拥塞窗口cwnd每次增加值=min(N，SMSS)$$。N为确认的字节数，SMSS为Sender Maximum Segment Size。

这是个悲观的设计，假定网络可能会出现拥塞。

##### 实例说明

![1644413643891](D:\code\tcp\assets\1644413643891.png)

上图为一个理想情况下的tcp传输过程。

轮次1中cwnd=1，收到M1后cwnd=2；轮次二中发送M2-M3，接收到M2、M3的确认后cwnd=4；轮次3中发送M4-M7，接收到M4-M7的确认后cwnd=8。

cwnd的大小和传输轮次呈指数级增加。可见慢启动只是cwnd初始值比较小，增长率换是挺大的。满足网络没有出现拥塞cwnd值总是增大的原则。

##### 存在的问题拥塞窗口过大会造成网络的拥塞

需要设置慢开始门限ssthresh变量值。

- 当cwnd<ssthresh时，使用慢启动算法
- 当cwnd>ssthresh时，使用拥塞避免算法
- 当cwnd=ssthresh时，使用慢启动算法或拥塞避免算法

通过ssthresh可以进行慢启动和拥塞避免算法的切换。

##### 拥塞避免

每经过一个传输伦次拥塞窗口增加1，不像慢启动那样拥塞窗口加倍的增加。由于没有发生拥塞，cwnd仍旧在增加，但是增长率减小了。

##### 快重传

目的是为了让发送方尽可能早的知道报文段的丢失。算法过程：接收方收到数据报文后立即发送确认，即使收到失序的报文也要发送确认报文。发送方收到连续3个重复确认就立即进行报文的重传，而不必等到超时的发送。没有超时会降低拥塞的发生避免拥塞窗口进入慢启动，提高了网路的传输效率。

##### 实例说明

![1644414790104](D:\code\tcp\assets\1644414790104.png)

接收方没有收到M3报文，仍旧需要在收到M4、M5、M6报文后立马发送对M2的确认。发送端收到连续对M2的确认说明M3丢失，会立即重传M3.

##### 快恢复

发送方连续收到3个重复的ACK，调整ssthresh=cwnd/2，同时设置新的cwnd值为ssthresh后开始执行拥塞避免算法。

##### 拥塞控制过程

![1644448632899](D:\code\tcp\assets\1644448632899.png)

先执行慢启动，cwnd从一个很低的值启动，随着传输轮次加倍增长。超过ssthresh阈值后执行拥塞避免算法，cwnd随着传输轮次线性增长。直到由于网络由于拥塞而出现数据报文出现超时，重新设置ssthresh=cwnd/2，然后执行慢启动算法，超过ssthresh后执行拥塞避免算法。发送端连续收到3个重复确认即3-ACK，判断此时网络可能出现拥塞，执行快重传算法，然后执行快恢复算法，设置ssthresh=cwnd/2，cwnd=ssthresh后执行拥塞避免算法。

当确认网络出现拥塞，即出现超时执行慢启动，当判断网络可能出现拥塞，即出现3-ACK执行快恢复，让窗口值从大的初始值开始增长，让信息有较高的传输效率。

##### 状态转换图

![1644452194567](D:\code\tcp\assets\1644452432454.png)

在执行慢启动和拥塞避免的过程中出现超时都会重新设置ssthresh=cwnd/2，cwnd=1 。在慢启动和拥塞避免过程中出现3-ACK，会执行快重传和快恢复。

##### 实际的发送窗口值

发送窗口值=Min(接收窗口rwnd，拥塞窗口cwnd)

说明：

- 当网络的性能较差，此时cwnd的值较小，发送窗口受到拥塞窗口的限制
- 当接收机的性能较差时，此时rwnd的值较小，发送窗口受到接收窗口的限制

### TCP的运输连接管理

tcp是一个面向连接的协议，通过连接可以可靠的协议。所以在传输数据的前后需要创建连接和销毁连接的过程。三次握手的说法不太准确，RFC973中是three way(message) handshake，三报文握手比较准确。所以此处采用三报文握手。

#### TCP连接的建立

##### 连接建立的过程需要做什么

- 需要确保对方的存在（需要收到对方的ACK）
- 双方协商一些参数，比如发送窗口等
- 运输层资源分配（收发缓存，发送数据的编号）

客户端：主动发起连接建立的进程

服务端：被动等待连接建立的进程

##### 三报文握手

![1644632313234](D:\code\tcp\assets\1644653381765.png)

前提条件：

- 信道的条件非常好，发送出去的报文对端一定能接收
- 每个报文段都有seq，哪怕是ACK报文

过程说明：

- 客户端主动发起连接建立的请求，服务端被动打开运行在LISTEN状态等待客户端的连接请求
- 客户端发送报文分组：SYN，seq=x，表示该报文为数据同步报文，发送序号从x开始。客户端进入SYN-SEND状态。SYN报文需要占用一个编号
- 服务端接收到发送端的SYN报文，也发送SYN，ACK报文，发送序号从y开始，确认号为x+1。服务端进入SYN-RECV状态
- 客户端接收到服务端的SYN，ACK报文后向服务端发送ACK报文，确认号为y+1 。发送成功后进入ESTABLISHED 状态
- 服务端收到ACK报文后也进入ESTABLISHED 状态
- 服务端和客户端都进入ESTABLISHED 状态后可以进行正常的数据传输

问题1：客户端和服务端的seq编号x、y是如何得来的？

x、y只是一个约定好随机数，主要是tcp是面向字节流，需要对发送的数据进行标号，这样就可以使用停-等协议了。

问题2：服务端确认客户端的ACK报文，必须发送SYN报文吗？

其实服务端可以将ACK和SYN分别发送出去，这样就成为四报文握手。但是这样没有意义，反而降低了传输效率。确认报文通常延迟发送或者和附带到数据报文中，所以服务端常常发送SYN、ACK报文

问题3：服务端接收到客户端的SYN报文后为什么进入立马ESTABLISHED状态？

为了避免历史遗留的SYN数据包被服务端接收到，然后立马建立连接。但是此时客户端并没有发送连接请求，浪费了服务端的宝贵资源。所以需要加个SYN-RECV状态。只要收到客户端的ACK确认报文后才服务端才建立连接

#### TCP连接的释放

![1644635849811](D:\code\tcp\assets\1644653677022.png)

前提条件：

- 信道的条件非常好，发送出去的报文对端一定能接收
- 每个报文段都有seq，哪怕是ACK报文
- 进程会调用关闭接口触发tcp发送FIN报文

过程说明：

- 客户端发送完数据后主动关闭，发送FIN，seq=x报文进入FIN-WAIT-1状态，表示不再发送数据。FIN报文也占用一个序号
- 服务端接收到发送端的FIN报文，发送响应ACK，seq=v，ack=x+1后进入CLOSE-WAIT状态，并通知应用程序不再接收数据。但是仍旧可以向客户端发送程序
- 客户端接收到服务端的ACK报文后进入FIN-WAIT-2状态，进入半关闭状态

- 服务端发送完数据后被动关闭，发送FIN，ACK，seq=y，ack=x+1报文后进入LAST-ACK报文
- 客户端接收到服务端发出的FIN报文进入TIME-WAIT状态，发送ACK，seq=x+1，ack=y+1报文并设置定时器为2MSL（max segment lifetime），表明客户端会在2MSL时间后自动关闭进入CLOSED状态。如果收到服务端的FIN重传报文会重新发送ACK报文和重新设置定时器为2MSL
- 服务端收到客户端的ACK报文后进入CLOSED状态

注意：

虽然发送端进入FIN-WAIT-2状态表示客户端不再发送数据，但是仍旧可以发送ACK分组。

问题1：服务端为啥发送FIN，seq=y，ack=x+1报文时需要携带ack=x+1序号，而这个序号其实已经确认过了，也就是说在关闭时发送端的FIN报文被确认了两次，哪怕是没有重传？

对于这个笔者现在还不太清楚

问题2：发送端在FIN-WAIT-2状态收到服务端的FIN报文时发送确认是否需要带上seq=x+1

目前按照笔者对书上以及wireshark中抓包，在ack时也需要seq编号，即使此时没有数据要发送了。对于这个目前笔者还不理解。

问题3：发送端为啥需要2MSL？

msl为网络层的概念，表示该报文在网络中最大的存活时间，查过这个时间段的报文会被网络节点丢弃。2倍的msl可以看做是如果没有发生重传则在这个时间段内一定能收到重传FIN报文。该状态有两个作用：1）确认服务端的FIN报文丢失时发送的重传报文。2）确保这个连接的所有交换的报文在整个网络中消失，防止对之后建立的连接产生影响。假设没有TIME-WAIT状态，发送端在关闭连接后在重新在该端口上和服务端建立新的连接，新连接发送的报文序号可能会和滞留在网络中的上次连接的报文序号相同，导致接收端无法分辨出这种差异造成不良的影响。这个额外的时间的副作用：对客户端其实是没有影响的，如果相同的应用程序启动会分配新的端口号；服务端会出现端口占用的错误，因为相同的程序没有额外配置的话占用的端口相同，如果关闭应用程序后立马重启端口仍旧被上次的连接占用从而出现端口占用。

#### TCP的有限状态机

![1644659191307](D:\code\tcp\assets\1644664599542.png)

说明：

- 省略了在SYN-RECV状态发送FIN进入FIN-WAIT-1状态，在SYN-RECV状态收到RST返回LISTEN状态，在SYN-SENT关闭或超时进入CLOSED状态
- 发送分为主动打开和被动打开，关闭分为主动关闭和被动关闭。服务端和客户端都可以同时主动打开，通常为服务端被动打开，客户端主动打开。服务端和客户端可以同时关闭，通常为客户端主动关闭，服务端被动关闭

- 红色为服务端被动打开，被动关闭流程变换过程。蓝色为客户端主动打开、主动关闭流程变换过程。黑色为可能会出现的流程变换过程

红色和蓝色的过程就是上面提到的连接建立和连接断开过程在此不再赘述。但是对于不太常见的状态：客户端和服务端同时打开、客户端和服务端同时关闭的过程研究下。

#### 客户端和服务端同时打开

客户端、服务端同时主动打开比较少见，但是确实存在这种可能性。实际场景中常为客户端访问服务端，即客户端主动打开，服务端被动打开。

![1644665360948](D:\code\tcp\assets\1644666372432.png)

对应的状态机上的流程：

![1644666005688](D:\code\tcp\assets\1644666005688.png)



说明：

- 红色表示为host1的发送报文以及host2的确认报文，黑色表示为host2的发送报文以及host1的确认报文
- 没有客户端和服务端的概念，因为两者同时是客户端和服务端

有意思的是从SYN-SENT中进入SYN-RECV时，即使在第一次发送SYN时seq序号已经发送了，但是在收到对端的SYN时仍旧发送SYN，ACK。可见在连接建立过程中SYN只占用一个序号。

#### 客户端和服务端同时关闭-先收到对方的FIN

![1644666995642](D:\code\tcp\assets\1644667123829.png)

![1644667455392](D:\code\tcp\assets\1644667455392.png)

说明：

- 红色表示为服务端的发送报文以及客户端的确认报文，黑色表示为客户端的发送报文以及服务端的确认报文

通过假如CLOSING状态表明发送FIN后收到对端FIN，即两端同时关闭的状态，注意和CLOSED状态区分。服务端和客户端都需要进入TIME-WAIT状态，等待2MSL时间后关闭连接。

#### 快速关闭-主动关闭后同时收到FIN，ACK

主动关闭方发送FIN后同时收到FIN，ACK报文后进入TIME-WAIT状态。注意此处和CLOSING状态的区别，已经收到了ACK了，可以看做跳过了FIN-WAIT-2状态，因为FIN-WAIT-2状态主要是为了接收对端的FIN报文。

![1644667918380](D:\code\tcp\assets\1644667918380.png)

参考文献

- 计算机网络第7版-谢希仁
- [TCP连接建立与关闭](https://blog.css8.cn/post/20868728.html) 



#### 课后习题

![1644668284779](D:\code\tcp\assets\1644668307330.png)

答：



![1644668335596](D:\code\tcp\assets\1644668335596.png)



![1644668350878](D:\code\tcp\assets\1644668350878.png)





![1644668371389](D:\code\tcp\assets\1644668371389.png)







![1644668386322](D:\code\tcp\assets\1644668386322.png)







![1644668393424](D:\code\tcp\assets\1644668393424.png)





![1644668401562](D:\code\tcp\assets\1644668401562.png)



![1644668407012](D:\code\tcp\assets\1644668407012.png)



![1644668413929](D:\code\tcp\assets\1644668413929.png)

 ![1644668449417](D:\code\tcp\assets\1644668449417.png)



![1644668438017](D:\code\tcp\assets\1644668438017.png)



![1644668461029](D:\code\tcp\assets\1644668461029.png)



![1644668469979](D:\code\tcp\assets\1644668469979.png)



![1644668479466](D:\code\tcp\assets\1644668479466.png)



![1644668487542](D:\code\tcp\assets\1644668487542.png)



![1644668496157](D:\code\tcp\assets\1644668496157.png)



![1644668506873](D:\code\tcp\assets\1644668506873.png)



![1644668516635](D:\code\tcp\assets\1644668516635.png)



![1644668527223](D:\code\tcp\assets\1644668527223.png)



![1644668535010](D:\code\tcp\assets\1644668535010.png)



![1644668542327](D:\code\tcp\assets\1644668542327.png)



![1644668550653](D:\code\tcp\assets\1644668550653.png)



![1644668560003](D:\code\tcp\assets\1644668560003.png)





![1644668567504](D:\code\tcp\assets\1644668567504.png)

![1644668579282](D:\code\tcp\assets\1644668579282.png)





![1644668588912](D:\code\tcp\assets\1644668588912.png)



![1644668595833](D:\code\tcp\assets\1644668595833.png)



![1644668602567](D:\code\tcp\assets\1644668602567.png)



![1644668609844](D:\code\tcp\assets\1644668609844.png)



![1644668617674](D:\code\tcp\assets\1644668617674.png)



![1644668627713](D:\code\tcp\assets\1644668627713.png)



![1644668635101](D:\code\tcp\assets\1644668635101.png)



![1644668642201](D:\code\tcp\assets\1644668642201.png)



![1644668650507](D:\code\tcp\assets\1644668650507.png)



![1644668659642](D:\code\tcp\assets\1644668659642.png)



![1644668669463](D:\code\tcp\assets\1644668669463.png)



![1644668676702](D:\code\tcp\assets\1644668676702.png)



![1644668686205](D:\code\tcp\assets\1644668686205.png)



![1644668694186](D:\code\tcp\assets\1644668694186.png)



![1644668700933](D:\code\tcp\assets\1644668700933.png)

![1644668708955](D:\code\tcp\assets\1644668708955.png)



![1644668719125](D:\code\tcp\assets\1644668719125.png)



![1644668730714](D:\code\tcp\assets\1644668730714.png)



![1644668737775](D:\code\tcp\assets\1644668737775.png)



![1644668745386](D:\code\tcp\assets\1644668745386.png)



![1644668751890](D:\code\tcp\assets\1644668751890.png)



![1644668759588](D:\code\tcp\assets\1644668759588.png)



![1644668765981](D:\code\tcp\assets\1644668765981.png)



![1644668772918](D:\code\tcp\assets\1644668772918.png)



![1644668781303](D:\code\tcp\assets\1644668781303.png)

![1644668788835](D:\code\tcp\assets\1644668788835.png)



![1644668796201](D:\code\tcp\assets\1644668796201.png)



![1644668804882](D:\code\tcp\assets\1644668804882.png)



![1644668812917](D:\code\tcp\assets\1644668812917.png)



![1644668821311](D:\code\tcp\assets\1644668821311.png)



![1644668830258](D:\code\tcp\assets\1644668830258.png)



![1644668837610](D:\code\tcp\assets\1644668837610.png)



![1644668844382](D:\code\tcp\assets\1644668844382.png)



![1644668851366](D:\code\tcp\assets\1644668851366.png)





![1644668859231](D:\code\tcp\assets\1644668859231.png)



![1644668867102](D:\code\tcp\assets\1644668867102.png)



![1644668874606](D:\code\tcp\assets\1644668874606.png)



![1644668881363](D:\code\tcp\assets\1644668881363.png)



![1644668889141](D:\code\tcp\assets\1644668889141.png)



![1644668897282](D:\code\tcp\assets\1644668897282.png)



![1644668904916](D:\code\tcp\assets\1644668904916.png)



![1644668912790](D:\code\tcp\assets\1644668912790.png)



![1644668919605](D:\code\tcp\assets\1644668919605.png)



![1644668927333](D:\code\tcp\assets\1644668927333.png)



![1644668934002](D:\code\tcp\assets\1644668934002.png)



![1644668940736](D:\code\tcp\assets\1644668940736.png)



![1644668948281](D:\code\tcp\assets\1644668948281.png)



![1644668956112](D:\code\tcp\assets\1644668956112.png)





![1644668965936](D:\code\tcp\assets\1644668965936.png)



![1644668972845](D:\code\tcp\assets\1644668972845.png)





![1644668979936](D:\code\tcp\assets\1644668979936.png)





![1644668987943](D:\code\tcp\assets\1644668987943.png)



![1644668995046](D:\code\tcp\assets\1644668995046.png)







#### 附加题

##### OSI 的七层模型分别是？各自的功能是什么？



##### 为什么需要三次握手？两次不行？



##### 为什么需要四次挥手？三次不行？



##### TCP与UDP有哪些区别？各自应用场景？



##### HTTP1.0，1.1，2.0 的版本区别



##### POST和GET有哪些区别？各自应用场景？

 

##### HTTP 哪些常用的状态码及使用场景？

 

##### HTTP状态码301和302的区别，都有哪些用途？

 

##### 在交互过程中如果数据传送完了，还不想断开连接怎么办，怎么维持？

 

##### HTTP 如何实现长连接？在什么时候会超时？

 

##### TCP 如何保证有效传输及拥塞控制原理

 

##### IP地址有哪些分类？

 

##### GET请求中URL编码的意义

 

##### 什么是SQL 注入？举个例子？

 

##### 谈一谈 XSS 攻击，举个例子？

 

##### 讲一下网络五层模型，每一层的职责？

 

##### 简单说下 HTTPS 和 HTTP 的区别

 

##### 对称加密与非对称加密的区别

 

##### 简单说下每一层对应的网络协议有哪些？

 

##### ARP 协议的工作原理？

 

##### TCP 的主要特点是什么？

 

##### UDP 的主要特点是什么？

 

##### TCP 和 UDP 分别对应的常见应用层协议有哪些？

 

##### 为什么 TIME-WAIT 状态必须等待 2MSL 的时间呢？

 

##### 保活计时器的作用？

 

##### TCP 协议是如何保证可靠传输的？

 

##### 谈谈你对停止等待协议的理解？

 

##### 谈谈你对 ARQ 协议的理解？

 

##### 谈谈你对滑动窗口的了解？

 

##### 谈下你对流量控制的理解？

 

##### 谈下你对 TCP 拥塞控制的理解？使用了哪些算法？

 

##### 什么是粘包？

 

##### TCP 黏包是怎么产生的？

 

##### 怎么解决拆包和粘包？

 

##### forward 和 redirect 的区别？

 

##### HTTP 方法有哪些？

 

##### 在浏览器中输入 URL 地址到显示主页的过程？

 

##### DNS 的解析过程？

 

##### 谈谈你对域名缓存的了解？

 

##### 谈下你对 HTTP 长连接和短连接的理解？分别应用于哪些场景？

 

##### HTTPS 的工作过程？

 

##### HTTP 和 HTTPS 的区别？

 

##### HTTPS 的优缺点？

 

##### 什么是数字签名？

 

##### 什么是数字证书？

 

##### Cookie 和 Session 有什么区别？

 

##### UDP 如何实现可靠传输？

 

##### Keep-Alive 和非 Keep-Alive 有什么区别？

 

##### HTTP 长连接短连接使用场景是什么

 

##### DNS 为什么用 UDP

 

##### 简单说下怎么实现 DNS 劫持

 

 

 

##### URI和 URL之间的区别

 

##### TIME_WAIT 状态会导致什么问题，怎么解决

 

##### 有很多 TIME-WAIT 状态如何解决

 

##### 简单说下 SYN FLOOD 是什么

 

##### ICMP 有哪些应用？

 

##### TCP 最大连接数限制

 

##### IP地址和MAC地址有什么区别？各自的用处？

 

##### IPV4 地址不够如何解决

 



#### 参考文献

- [帅地玩编程](https://www.iamshuaidi.com/4128.html)