---
layout: post
title: 同步IO、异步IO、阻塞IO、非阻塞IO
date: 2019-05-05 15:30:02
---
# 概念剖析
* 同步
  所谓同步，就是在发出一个功能调用时，在没有得到结果之前，该调用就不返回。也就是必须一件事一件事地做，等前一件做完了才能做下一件事。
  例如B/S模式（同步）：提交请求->等待服务器处理->处理完毕返回，这个期间客户端浏览器不能干任何事。
* 异步
  异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的部件在完成后，通过状态、通知和回调来通知调用者。
  例如 ajax请求（异步）: 请求通过事件触发->服务器处理（这是浏览器仍然可以作其他事情）->处理完毕。
* 阻塞
  阻塞调用是指调用结果返回之前，当前线程会被挂起（线程进入非可执行状态，在这个状态下，cpu不会给线程分配时间片，即线程暂停运行）。函数只有在得到结果之后才会返回。
  有人也许会把阻塞调用和同步调用等同起来，实际上他是不同的。对于同步调用来说，很多时候当前线程还是激活的，只是从逻辑上当前函数没有返回,它还会抢占cpu去执行其他逻辑，也会主动检测io是否准备好。
* 非阻塞
  非阻塞和阻塞的概念相对应，指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回。
  
# IO模型
* 阻塞I/O（blocking I/O）
  使用recv的默认参数一直等数据直到拷贝到用户空间，这段时间内进程始终阻塞。
![阻塞I/O](/image/linux_io_1.png)
* 非阻塞I/O（nonblocking I/O）
  改变flags，让recv不管有没有获取到数据都返回，如果没有数据那么一段时间后再调用recv看看，如此循环。但是它只有检查有无数据的时候是非阻塞的，在数据到达的时候依然要等待复制数据到用户空间(等着水将水杯装满)，因此它还是同步IO。
![非阻塞I/O](/image/linux_io_2.png)
* IO复用（select，poll和epoll）
  这里在调用recv前先调用select或者poll，这2个系统调用都可以在内核准备好数据（网络数据到达内核）时告知用户进程，这个时候再调用recv一定是有数据的。因此这一过程中它是阻塞于select或poll，而没有阻塞于recv，有人将非阻塞IO定义成在读写操作时没有阻塞于系统调用的IO操作(不包括数据从内核复制到用户空间时的阻塞，因为这相对于网络IO来说确实很短暂)，如果按这样理解，这种IO模型也能称之为非阻塞IO模型，但是按POSIX来看，它也是同步IO。
![IO复用](/image/linux_io_3.png)
* 信号驱动IO
  通过调用sigaction注册信号函数，等内核数据准备好的时候系统中断当前程序，执行信号函数(在这里面调用recv)。
![信号驱动IO](/image/linux_io_4.png)
* 异步IO
  调用aio_read，让内核等数据准备好，并且复制到用户进程空间后执行事先指定好的函数。
![异步IO](/image/linux_io_5.png)

# 总结
1. 数据准备阶段
2. 内核空间复制回用户空间
*阻塞IO模型、非阻塞IO模型、IO复用模型(select/poll/epoll)、信号驱动IO模型都属于同步IO，因为阶段2是阻塞的（尽管时间很短）。只有异步IO模型是真真正正的异步IO，因为不管在阶段1还是阶段2都可以干别的事。*

# 参考
>https://www.cnblogs.com/chaser24/p/6112071.html
>https://www.cnblogs.com/euphie/p/6376508.html