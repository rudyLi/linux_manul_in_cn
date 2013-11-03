##epoll
###名字
>  epoll io事件通知组件

###简介
> epoll是一个边缘触发和条件触发的接口，对于大量的监控文件标识符很容易扩展，以下系统调用可以用来创建和管理一个epoll实例：
* epollcreate创建一个epoll实例，返回一个与epoll实例相关的文件描述符
* 感兴趣的特定fd通过epoll_ctl注册，这时fd集合注册到一个有时叫做epoll集合的epoll实例
* 最后实际的等待是epoll_wait开始

###ET LT
> epoll事件分发接口可以处理条件触发LT和边缘触发ET，两种机制的不同描述如下：
1. 表示读的管道（rfd）的fd注册到 epoll实例中
2. 一个写的管道写了2kb数据
3. 触发对epollwait的回调并返回一个rfd（ready file descripter）
4. 管道读取1kb
5. 触发一次epollwait
> 如果rfd 文件标识符在注册到epollset时加了一个EPOLLET（边缘触发）标志，在第五步对wpollwait的调用将被挂起 尽管在输入缓存中有可用数据；同时写端可能一直在希望根据其发送的数据得到一个反馈。因为边缘触发只有在fd状态改变时才会发送时间所以在步骤5中调用者可能一直等待下去虽然输入缓存里面有所需的数据。在上述例子中，在rfd的事件会触发因为书写了2kb数据状态发生改变，步骤三执行时间。但是在步骤四中并没有读取所有的数据状态并未发生改变，步骤5则无限的阻塞下去。

> 采用EPOLLET的应用从应该采用非阻塞文件标识符来避免 由于存在一个阻塞读写而使一个处理多文件标识的任务无限阻塞下去。以边缘触发的方式使用epoll接口提供以下建议：
* `i` 非阻塞文件标识符
* `ii` 只有在read write返回eagain时一直等待一个事件

> 相反，epoll默认使用条件触发.即使是边缘触发epoll，多块数据的（收据）也可以保证发送多个事件。调用者有选项指定 EPOLLONESHOT，告诉epoll在epollwait接收到一个事件后禁用其相关的fd事件. 当你指定EPOLLONESHOT，你需要重新使用epoll_ctl 构造epoll,使用参数epollctlmod
以下的接口可以限制epoll消耗的内核内存大小：
	/proc/sys/fs/epoll/max_user_watches

> 指定注册到所有epoll实例的fd数目大小，大小是每个用户id多少，每个注册的fd在32位机上占用90字节，64位占用160。默认值是可用的低内存除以注册占用的bytes 

> 条件触发使用和poll使用完全一样 

> epoll_ctl(epollfd, EPOLL_CTL_ADD, listen_sock, &ev)当使用边缘触发时，出于性能的考虑， 可以指定(EPOLLIN|EPOLLOUT讲fd加到epoll（epoll_ctl_add）接口一次,不用不停的使用使用epoll_ctl 构造epoll,使用参数epollctlmod的方法切换
