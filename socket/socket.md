##socket
###名称
> socket 创建一个用于通信的终端

###简介
	#include <sys/types.h>
	#include <sys/socket.h>
	int socket(int domain, int type, int protocal);

###描述
> socket()创建一个用于通信的终端并返回一个描述字

> `domain` 参数指定一个连接域；这个选择一会用来通信的协议。这些协议簇在`<sys/socket.h>`中定义。可以理解的格式如下：

<table>
	<tr>
       <td>名称</td>
       <td>用途</td>
       <td>参考手册</td>
    </tr>
	<tr>
	<td>AF_UNIX, AF_LOCAL</td>
	<td>本地通信</td>
	<td>unix </td>
	</tr>
	<tr>
	<td>AF_INET</td>
	<td>IPv4协议</td>
	<td>ip</td>
	</tr>
	<tr>
	<td>AF_INET6</td>
	<td>IPv6</td>
	<td>ipv6 </td>
	</tr>
	<tr>
	<td>AF_NETLINK</td>
	<td>内核用户接口设备</td>
	<td>netlink</td>
	</tr>
</table>

> socket 有一个可标识的`type`，指定通信语义，定义如下：
* `SOCK_STREAM` 提供序列化的可靠的，双向的基于比特流的连接。支持带外数据传输
* `SOCK_DGRAM` 提供数据报（无链接，不可靠固定最大长度的消息）
* `SOCK_SEQPACKET` 提供序列化的可靠的双向连接，这中连接基于固定最大长度数据报的传输路径,用户需要对每个输入的系统调用读取整个包
* `SOCK_RAW` 提供原始网络协议访问
* `SOCK_RDM` 提供一个可靠数据报层，不保证数据到达的顺序
* `SOCK_PACKET` 在新的程序中绝对不能使用
> 自linux2.6.27以来，`type`第二个用途：进一步指定socket类型，type还可以包含按位与的以下值来修改socket（）的行为。
* `SOCK_NONBLOCK` 设置新打开的fd为`0_NONBLOCK`文件状态。使用这个标志可以保留额外的对`fcntl`获得同样的结果
* `SOCK_CLOEXEC` 在新打开的fd设置`close-on-exec`标志。查看open中O_CLOEXE标志到底起什么作用
> `protocal`具体指socket的协议.一般的在一直的协议族中只存在一个协议支持特定的socket， 在这种情况下可以设为0. 然而，有可能存在多个协议，在这种情况下，必须指定一个协议。所使用的协议数字在发生通信时指向“通信域”，查看`getprototent`如何将协议名称核协议数字对应起来的

> 类型为`SOCK_STREAM`的sockets是全双工比特流，类似于管道。并不保留记录边界。流式socket在发送与接受数据之前必须在连接状态。连接的另一个socket通过调用`connect`生成。一旦连接，数据通过`read`和`write`或者`send``recv`传送。当一个会话结束时，可能执行`close`。带外数据也可以通过`send`和`recv`发送

> 应用与`SOCK_STREAM`通信协议保证数据不会丢失和重复。如果在合理的时间长度内，对等协议缓存空间的数据并没有发送成功，连接认为死连接。当`SO_KEEPLIVE`在socket协议中使能，协议会检查在协议指定的行为内是否其它的端点还是活动的。如果一个进程接收或发送受损的数据流就会发送一个`SIG_PIPE`信号量，导致本地进程不处理此信号而退出。`SOCK_SEQPACKE`和`SOCK_STREAM`调用同样的系统调用.唯一的不同是read只返回请求数据的数量，在包中剩下的数据会丢弃。同样数据报的边界消息会保存下来。

> SOCK_DGRAM 和 SOCK_RAW 类型的socket允许发送在sendto(2)调用中命令的通信发送数据报。数据报一般有recvfrom（2）来接收，返回下一个报文的发送地址

> SOCK_PACKET是一种直接从驱动接受原始包的socket协议

> `fcntl` `F_SETOWN`操作可以用来当一个带外数据到来的时候指定一个进程或者进程组接受`SIGURG`信号，或者当`SOCK_STREAM`连接非预期的断掉接受`SIGPIP`信号。这个操作同样可以用来设置进程或进程组通过SIGIO来接受I/O和异步的I/O事件的通知.使用F_SETOWN 和参数为FIOSETOWN或 SIOCSPFRP的ioctl 是等效的.

> 当网络向协议模型发送一个错误情况的信号时(如对IP使用IGMP信息),那就对socket设置连接错误的标志位.在这个socket上的下个操作将会返回未定错误的错误码.对一些协议而言,有可能通过使能预socket错误队列的形式来取回错误的细节,在ip(7)参见 IP_RECVERR.

>socket的操作是有socket的level 操作选项的.这些选项在`<sys/socket.h>`中定义,函数 setsockopt(2) 和 getsockopt(2)分别来设置和获取选项.

###返回值
> 成功的话会返回新socket的文件描述符,错误的话返回-1,这时设置errno

###错误
* EACCES 无权限创建指定类型或者协议的socket
* EAFNOSUPPORT 程序不支持指定的地址族
*  EINVAL 未知协议,或者协议族无效，无效标志
* EMFILE 处理文件表溢出
* ENFILE 超出系统设定的打开文件的最大数目
*  ENOBUFS 容量不足
*  EPROTONOSUPPORT 协议类型或协议在此域下不支持

###查看
accept(2),  bind(2),  connect(2),  fcntl(2),  getpeername(2),  getsock‐name(2),   getsockopt(2),   ioctl(2),   listen(2),   read(2),  recv(2), select(2),  send(2),  shutdown(2),  socketpair(2),  write(2),   getpro‐toent(3), ip(7), socket(7), tcp(7), udp(7), unix(7)

