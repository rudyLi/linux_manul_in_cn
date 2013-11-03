## poll 
### 内容简介
	#include <poll.h>
    int poll(struct pollfd *fds, nfds_t nfds, int timeout

### 描述
> poll和select执行同样的工作，等待一个文件描述符集合的一个准备执行io操作

> 被监控的文件符集合通过fds变量指定，它是一个以下结构的数组

	struct pollfd {
	     int  fd; /*文件描述*/
		 short events; /*请求的事件*/
		 short revents; /* 返回的事件 */
		 };

> 函数调用必须在nfds中指定fds数组中元素的个数

> fd字段对于一个打开的文件包含一个文件描述符，如果这个值是负数，相应的事件字段将被忽略，revents字段返回0，（这样提供了一个简单的方法对于一个poll调用忽略一个文件描述，只要将其fd设为负数）

> events字段是输入参数，指定勒该应用对于该fd所关心的事件，如果该字段设为0，忽略掉该fd相关的所有事件以及revents为0

> revents 是输出字段，当事件发生时由内核填充。在revents返回的bits可以包含任何指定的事件，或者是POLLERR，POLLHUP，POLLNVAL中的一个（这三个bit值在事件中并没有任何意义，并且当相关的条件满足时，在revents中设定该值）

> 如果在任意的fd中并没有事件请求（包含无错），则poll事件将阻塞直到事件发生

> timeout 参数指定poll（）阻塞等待一个fd ready状态的毫米时间。这个间隔系统时钟向上舍入计算，由于内涵调度设置延长可能阻塞事件可能会超过指定值。如果指定timeout为负数，则意外着无限等待，如果为0，则poll立刻返回，即使没有fd在ready

> events 和 revents的数值在poll.h中有介绍：
	POLLIN 有数据要读
	POLLPRI 有紧急的数据要读（eg，[out_of_band](http://www.gnu.org/software/libc/manual/html_node/Out_002dof_002dBand-Data.html)in socket
	POLLOUT 现在写不会阻塞
	POLLRDHUP 流式socket关闭链接或者关闭半连接写
	POLLERR 错误条件（只有在输出时会发生）
	POLLHUP 挂起（只有在输出时）
	POLLNVAL 无效请求：fd没有打开（只有在输出时）
	如果按 _XOPEN_SOURCE 定义的编译，还有其他事件

> ppoll 允许一个应用安全的等待直到一个fd准备好或者捕捉到一个信号量

> 其他不同就是timeout的精度，以下就是ppoll（）调用
	ready = ppoll(&fds, nfds, timeout_ts, &sigmask);

> 等价于自动执行以下代码：
	sigset_t origmask;
	int timeout;
	timeout = (timeout_ts == NULL) ? -1 :
	          (timeout_ts.tv_sev * 1000 + timeout_ts.tv_nsec/1000000);
    sigprocmask(SIG_SETMASK, &sigmask, &origmask);
	ready = poll(&fds, nfds, timeout);
	sigprocmask(SIG_SETMASK, &origmask, NULL);

> 查看pselect2

> 如果`sigmask`指定为mull，则并不会执行任何符号量操作，只是事件精度的差别

> `timeout_ts`指定了阻塞时间的上限，这个参数事一个如下结构的指针
	struct timespec {
		long tv_sec; /*秒*/
		long tv_nsec; /*纳秒*/
	};

> 如果事件指定为null，则无限阻塞

###返回值
> 一旦成功，返回一个正数；这是一个含有非0`revents`字段的结构的数量，换句话说就是这些通知的与描述字相关的事件或错误。0表示这次调用超时或者没有fd准备好，如果返回-1，`errno`没有设置正确

###错误
* EFAULT 作为参数给定的这个数组并没有包含在调用程序的地址空间中
* EINTR	 在请求事件之前一个信号量发生了
* EINVAL `nfds`值超过`RLIMIT_MOFILE`值
* ENOMEM 没有空间分配fd table

	
