##select
###名称
> select, pselect, FD_CLR, FD_ISSET, FD_SET, FD_ZERO- 异布io处理

###内容简介
	/* 根据posix.1-2001*/
	#include <sys/select.h>
	/* 根据较早标准*/
	#include <sys/time.h>
	#include <sys/types.h>
	#include <unistd.h>
	int select(int nfds, fd_set *readfds, fd_set *writefds,
				fd_set *exceptfds, struct timeval *timeout);
	void FD_CLR(int fd, fd_set *set);
	int FD_ISSET(int fd, fd_set *set);
	void FD_SET(int fd, fd_set *set);
	void FD_ZERO(fd_set *set);

	#include <sys/select.h>

	int pselect(int nfds, fd_set *readfds, fd_set *writefds,
				fd_set *exceptfds, const struct timespec *timeout,
				const sigset_t *sigmask);

###描述
> `select()``pselect()`允许程序监控多个文件描述字，等待直到一个或多个fd为io操作为ready。一个fd如果可以非阻塞执行相关的io操作则认为ready

> `select()``pselect()`无区别的，除了以下三点：

1. `select()`使用一个结构体`timeal`（含有秒核毫秒）的timeout值，而pselect使用一个timespec（含有秒和纳秒）的时间值
2. select 更新`timeout`参数表示还剩多长事件，而pselect部改变参数
3. select 没有sigmask参数，其执行效果和pselect()使用null sigmask一样

> 监控着三个独立的文件描述字集合，在`readfds`中列的fd将被监控如果字符可读（严格意义来说，看读是否会阻塞，特殊的，fd仍然在eof），`writefds`中的fd集合被监控，看是否写会阻塞，在exceptfds查看异常。这些集合轮流修改来说明哪一个fd状态改变。每个fd集合可以为null说明没有fd在相关的一类事件中被监控。

> 提供了四个操作fdsets的宏。`FD_ZERO()`清空，`FD_SET``FD_CLR`分别代表添加或移除指定fd. `FD_ISSET()`测试一个fd是否在set中，在select返回时有用

* `nfds` 三个fdset中数量最大的值 加一
* `timeout` 指定阻塞等待时间，如果timeval两个值都为0，select立刻返回
* `sigmask` 指向信号量的指针，如果不是null，则pselect首先将sigmask替换当前的signal mask，执行select，然后回复原先的signal mask

> pselect的作用事等待一个信号或者一个fd为ready。避免条件竞争，先执行signal再执行sigmask

###返回值
返回三个集合中包含的fd总数

###错误
> EBADF EiNTR EINVAL ENOMEM

    
