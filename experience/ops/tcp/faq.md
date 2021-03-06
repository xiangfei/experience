
[参考](https://blog.csdn.net/qq_40337086/article/details/112443124)


## 为什么需要3次握手

### 三次握手才可`以阻止历史重复连接（主要原因）`

网络环境是错综复杂的，往往并不是如我们期望的一样:先发送的数据包，就先到达目标主机。可能会由于网络拥堵等乱七八糟的原因，会使得后发送的数据包先到达目标主机
那么这种情况下 TCP 三次握手是如何避免的呢？

![3次握手](/images/tcp_3_3.png)

但上面情况如果是两次握手的话，只要收到了响应就说明握手成功，客户端不会检测该响应ACK是否正确，这就发生了错误。并且当此前滞留的SYN报文姗姗来迟到达服务器时，这个报文本该是失效的，但是，两次握手的机制将会让客户端和服务器再次建立连接，这将导致不必要的错误和资源的浪费。

如果采用的是三次握手，就算是失效的报文传送过来了，服务端接受到了那条失效报文并且回复了确认报文，但是客户端不会再次发出确认。由于服务器收不到确认，就知道客户端并没有请求连接。


### 三次握手才`可以同步双方的初始序列号`

TCP 协议的通信双方， 都必须维护一个「序列号」， 序列号是可靠传输的一个关键因素，它的作用：

- 接收方可以去除重复的数据；

- 接收方可以根据数据包的序列号按序接收；

- 可以标识发送出去的数据包中， 哪些是已经被对方收到的；

可见，序列号在 TCP 连接中占据着非常重要的作用，所以当客户端发送携带「初始序列号」的 SYN 报文的时候，需要服务端回一个 ACK 应答报文，表示客户端的 SYN 报文已被服务端成功接收，那当服务端发送「初始序列号」给客户端的时候，依然也要得到客户端的应答回应，这样一来一回，才能确保双方的初始序列号能被可靠的同步。


![tcp_3_2](/images/tcp_3_2.png)

`两次握手只保证了客户端的初始序列号被服务端成功接收，没办法保证服务端的初始序列号被客户端确认接收。`

`四次握手当然能够可靠的同步双方的初始化序号，但由于第二步和第三步可以优化成一步，所以就成了「三次握手」，即四次握手没有必要，优化成了三次握手。`

### 三次握手才`可以避免重复建立连接`


如果只有「两次握手」，当客户端的 SYN 请求连接在网络中阻塞，客户端没有接收到 ACK 报文，就会重新发送 SYN ，由于没有第三次握手，服务器不清楚客户端是否收到了自己发送的建立连接的 ACK 确认信号，所以每收到一个 SYN 就会建立一个新的连接，服务器就会建立多个冗余的无效链接，造成不必要的资源浪费。





## 为什么需要4次挥手




## 数据包丢失了该怎么办？
### TCP 第一次握手的 SYN 丢包了，会发生什么？
重传 SYN 数据包，重传次数超过阈值后放弃

### TCP 第二次握手的 SYN、ACK 丢包了，会发生什么？
客户端 SYN 包没有收到ACK，所以会超时重传
服务端 SYN包也没有收到ACK， 也会超时重传。

### TCP 第三次握手的 ACK 包丢了，会发生什么？
服务端会一直重传 SYN、ACK 包，重传次数超过阈值后放弃
而客户端根据是否开启保活机制分为两种情况：

开启了保活机制的话，会经过 2 小时 11 分 15 秒发现一个「死亡」连接，于是客户端就会断开连接。

没有开启的话，则会一直重传该数据包，直到重传次数超过阈值后就会断开 TCP 连接。


## 初始序列号为什么随机产生？
`为了网络安全`
- 如果不是随机产生初始序列号，黑客将会以很容易的方式获取到你与其他主机之间通信的初始化序列号，并且伪造序列号进行攻击，这已经成为一种很常见的网络攻击手段

## 为什么 SYN 段不携带数据却要消耗一个序列号呢？

- 因为SYN 段需要对方的确认，所以需要占用一个序列号确保这个确认不会出现歧义。如果不占序列号的话，怎么知道这个确认是对数据包的确认还是对syn报文的确认呢？


## 每次握手可以确定哪些东西？

- 第一次握手：Client 什么都不能确认；Server 确认了对方发送正常，自己接收正常

- 第二次握手：Client 确认了自己发送、接收正常，对方发送、接收正常；

- 第三次握手：Server 确认了自己发送正常，对方接收正常

## 一个已经建立的 TCP 连接中，客户端中途宕机了，客户端恢复后，向服务端发送SYN包重新建立连接，此时服务端会怎么处理？

- 我们知道TCP 连接是由「四元组」唯一确认的。
- 然后这个场景中，客户端的IP、服务端IP、目的端口并没有变化
- 所以这个问题关键在于：本次连接的源端口是否和上一次连接的源端口相同。

`所以分两种情况：`

1. 不相同
- 此时服务端会认为是新的连接要建立，于是就会通过三次握手来建立新的连接。

> [!WARNING]那旧连接里的服务端会怎么样呢？ 
> - 如果服务端发送了数据包给客户端，由于客户端的连接已经被关闭了，此时客户的内核就会回 RST 报文，服务端收到后就会释放连接。
> - 如果服务端一直没有发送数据包给客户端，在超过一段时间后， TCP 保活机制就会启动，检测到客户端没有存活后，接着服务端就会释放掉该连接。

2. 相同

处于 establish 状态的服务端会回复一个携带了对上次报文的确认号和序列号，这个 ACK 被称之为 Challenge ACK。
接着，客户端收到这个 Challenge ACK，发现序列号并不是自己期望收到的，于是就会回 RST 报文，服务端收到后，就会释放掉该连接。

![tcp3_crash](/images/tcp_3_crash.png)

## 如何手动关闭一个TCP连接

`伪造一个能关闭 TCP 连接的 RST 报文`

RST报文满足条件`「四元组相同」`和`「序列号正好落在对方的滑动窗口内」`

`怎么伪造？`

直接伪造符合预期的序列号是比较困难，因为如果一个正在传输数据的 TCP 连接，滑动窗口时刻都在变化，因此很难伪造一个刚好落在对方滑动窗口内的序列号的 RST 报文。
办法还是有的，我们可以伪造一个四元组相同的 SYN 报文，来拿到“合法”的序列号！

`怎么拿到？`

如果处于 establish 状态的服务端，收到四元组相同的 SYN 报文后，会回复一个 Challenge ACK，这个 ACK 报文里的「确认号」，正好是服务端下一次想要接收的序列号，说白了，就是可以通过这一步拿到服务端下一次预期接收的序列号。
然后用这个确认号作为 RST 报文的序列号，发送给服务端，此时服务端会认为这个 RST 报文里的序列号是合法的，于是就会释放连接！



## 为什么客户端在TIME-WAIT阶段要等2MSL

为的是确认服务器端是否收到客户端发出的 ACK 确认报文，当客户端发出最后的 ACK 确认报文时，并不能确定服务器端能够收到该段报文。

所以客户端在发送完 ACK 确认报文之后，会设置一个时长为 2MSL 的计时器。

MSL 指的是 Maximum Segment Lifetime：一段 TCP 报文在传输过程中的最大生命周期。

2MSL 即是服务器端发出为 FIN 报文和客户端发出的 ACK 确认报文所能保持有效的最大时长。

服务器端在 1MSL 内没有收到客户端发出的 ACK 确认报文，就会再次向客户端发出 FIN 报文：

- 如果客户端在 2MSL 内，再次收到了来自服务器端的 FIN 报文，说明服务器端由于各种原因没有接收到客户端发出的 ACK 确认报文。客户端再次向服务器端发出 ACK 确认报文，计时器重置，重新开始 2MSL 的计时。

- 否则客户端在 2MSL 内没有再次收到来自服务器端的 FIN 报文，说明服务器端正常接收了 ACK 确认报文，客户端可以进入 CLOSED 阶段，完成“四次挥手”。


## 为什么需要四次挥手
其实是客户端和服务端的两次挥手，也就是客户端和服务端分别释放连接的过程.
再来回顾下四次挥手双方发 FIN 包的过程，就能理解为什么需要四次了。

- 关闭连接时，客户端向服务端发送 FIN 时，仅仅表示客户端不再发送数据了但是还能接收数据。
- 服务器收到客户端的 FIN 报文时，先回一个 ACK 应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送 FIN 报文给客户端来表示同意现在关闭连接
