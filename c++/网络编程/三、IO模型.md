## 一、IO模型

![img](https://pic1.zhimg.com/80/v2-b7bb0b2a45d4bf3ad546b1fcddc9766c_720w.jpg)

一个完整IO分为两阶段：[用户进程空间](https://www.zhihu.com/search?q=用户进程空间&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})<-->内核空间、内核空间<-->设备空间（磁盘、网络等）。

IO有内存IO、网络IO和磁盘IO三种，通常我们说的IO指的是后两者。

LINUX中进程无法直接操作I/O设备，其必须通过系统调用请求kernel来协助完成I/O动作；内核会为每个I/O设备维护一个[缓冲区](https://www.zhihu.com/search?q=缓冲区&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})。

一个输入操作通常包括两个阶段：

- 等待数据准备好
- 从内核向进程复制数据

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待数据到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

Unix 有五种 I/O 模型：

- 阻塞式 I/O
- 非阻塞式 I/O
- I/O 复用（select 和 poll）
- 信号驱动式 I/O（SIGIO）
- 异步 I/O（AIO）

## 二、阻塞式IO

进程发起IO系统调用后，进程被阻塞，转到内核空间处理，整个IO处理完毕后返回进程。操作成功则进程获取到数据。

<img src="https://pic1.zhimg.com/80/v2-7bd6b84c7ba0ebd2e59e90380f36a488_720w.jpg" alt="img" style="zoom:150%;" />

#### 1.典型应用：阻塞socket、Java BIO；

#### 2.特点：

- 进程阻塞挂起不消耗CPU资源，及时响应每个操作；
- 实现难度低、开发应用较容易；
- 适用并发量小的网络应用开发

不适用并发量大的应用：因为一个请求IO会阻塞进程，所以，得为每请求分配一个处理进程（线程）以及时响应，系统开销大。

## 三、非阻塞式IO

<img src="https://pic3.zhimg.com/80/v2-792824a0da2b9dd9ee6cd24ba448c206_720w.jpg" alt="img" style="zoom:150%;" />

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询（polling）。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率比较低。

#### 1.典型应用：socket是非阻塞的方式（设置为NONBLOCK）

#### 2.特点：

- [进程轮询](https://www.zhihu.com/search?q=进程轮询&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})（重复）调用，消耗CPU的资源；
- 实现难度低、开发应用相对阻塞IO模式较难；
- 适用并发量较小、且不需要及时响应的网络应用开发；

## 四、IO复用

<img src="https://pic1.zhimg.com/80/v2-0fa6b44cb9709be235b7e99c4066f984_720w.jpg" alt="img" style="zoom:150%;" />

多个的进程的IO可以注册到一个复用器（select）上，然后用一个进程调用该select， select会监听所有注册进来的IO；

如果select没有监听的IO在内核缓冲区都没有可读数据，select调用进程会被阻塞；而当任一IO在内核缓冲区中有可数据时，select调用就会返回；

而后[select调用进程](https://www.zhihu.com/search?q=select调用进程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})可以自己或通知另外的进程（注册进程）来再次发起读取IO，读取内核中准备好的数据。

可以看到，多个进程注册IO后，只有另一个select调用进程被阻塞。

#### 1、典型应用：select、poll、epoll三种方案，nginx都可以选择使用这三个方案;Java NIO;

#### 2、特点：

- 专一进程解决多个进程IO的阻塞问题，性能好；Reactor模式;
- 实现、开发应用难度较大；
- 适用高并发服务应用开发：一个进程（线程）响应多个请求；

#### 3、实现

select/poll/epoll 都是 I/O 多路复用的具体实现，select 出现的最早，之后是 poll，再是 epoll。

- select

  ```
  int select(int n, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
  ```

  select 允许应用程序监视一组文件描述符，等待一个或者多个描述符成为就绪状态，从而完成 I/O 操作。

  - fd_set 使用数组实现，数组大小使用 FD_SETSIZE 定义，所以只能监听少于 FD_SETSIZE 数量的描述符。有三种类型的描述符类型：readset、writeset、exceptset，分别对应读、写、异常条件的描述符集合。
  - timeout 为超时参数，调用 select 会一直阻塞直到有描述符的事件到达或者等待的时间超过 timeout。
  - 成功调用返回结果大于 0，出错返回结果为 -1，超时返回结果为 0。

  ```
  fd_set fd_in, fd_out;
  struct timeval tv;
  
  // Reset the sets
  FD_ZERO( &fd_in );
  FD_ZERO( &fd_out );
  
  // Monitor sock1 for input events
  FD_SET( sock1, &fd_in );
  
  // Monitor sock2 for output events
  FD_SET( sock2, &fd_out );
  
  // Find out which socket has the largest numeric value as select requires it
  int largest_sock = sock1 > sock2 ? sock1 : sock2;
  
  // Wait up to 10 seconds
  tv.tv_sec = 10;
  tv.tv_usec = 0;
  
  // Call the select
  int ret = select( largest_sock + 1, &fd_in, &fd_out, NULL, &tv );
  
  // Check if select actually succeed
  if ( ret == -1 )
      // report error and abort
  else if ( ret == 0 )
      // timeout; no event detected
  else
  {
      if ( FD_ISSET( sock1, &fd_in ) )
          // input event on sock1
  
      if ( FD_ISSET( sock2, &fd_out ) )
          // output event on sock2
  }
  ```

  

- poll

  ```
  int poll(struct pollfd *fds, unsigned int nfds, int timeout);
  ```

  poll 的功能与 select 类似，也是等待一组描述符中的一个成为就绪状态。

  poll 中的描述符是 pollfd 类型的数组，pollfd 的定义如下：

  ```
  struct pollfd {
                 int   fd;         /* file descriptor */
                 short events;     /* requested events */
                 short revents;    /* returned events */
             };
  ```

  ```
  // The structure for two events
  struct pollfd fds[2];
  
  // Monitor sock1 for input
  fds[0].fd = sock1;
  fds[0].events = POLLIN;
  
  // Monitor sock2 for output
  fds[1].fd = sock2;
  fds[1].events = POLLOUT;
  
  // Wait 10 seconds
  int ret = poll( &fds, 2, 10000 );
  // Check if poll actually succeed
  if ( ret == -1 )
      // report error and abort
  else if ( ret == 0 )
      // timeout; no event detected
  else
  {
      // If we detect the event, zero it out so we can reuse the structure
      if ( fds[0].revents & POLLIN )
          fds[0].revents = 0;
          // input event on sock1
  
      if ( fds[1].revents & POLLOUT )
          fds[1].revents = 0;
          // output event on sock2
  }
  ```

  

- epoll

  ```
  int epoll_create(int size);
  int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
  int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
  ```

  epoll_ctl() 用于向内核注册新的描述符或者是改变某个文件描述符的状态。已注册的描述符在内核中会被维护在一棵红黑树上，通过回调函数内核会将 I/O 准备好的描述符加入到一个链表中管理，进程调用 epoll_wait() 便可以得到事件完成的描述符。

  从上面的描述可以看出，epoll 只需要将描述符从进程缓冲区向内核缓冲区拷贝一次，并且进程不需要通过轮询来获得事件完成的描述符。

  epoll 仅适用于 Linux OS。epoll 比 select 和 poll 更加灵活而且没有描述符数量限制。

  epoll 对多线程编程更有友好，一个线程调用了 epoll_wait() 另一个线程关闭了同一个描述符也不会产生像 select 和 poll 的不确定情况。

  ```
  // Create the epoll descriptor. Only one is needed per app, and is used to monitor all sockets.
  // The function argument is ignored (it was not before, but now it is), so put your favorite number here
  int pollingfd = epoll_create( 0xCAFE );
  
  if ( pollingfd < 0 )
   // report error
  
  // Initialize the epoll structure in case more members are added in future
  struct epoll_event ev = { 0 };
  
  // Associate the connection class instance with the event. You can associate anything
  // you want, epoll does not use this information. We store a connection class pointer, pConnection1
  ev.data.ptr = pConnection1;
  
  // Monitor for input, and do not automatically rearm the descriptor after the event
  ev.events = EPOLLIN | EPOLLONESHOT;
  // Add the descriptor into the monitoring list. We can do it even if another thread is
  // waiting in epoll_wait - the descriptor will be properly added
  if ( epoll_ctl( epollfd, EPOLL_CTL_ADD, pConnection1->getSocket(), &ev ) != 0 )
      // report error
  
  // Wait for up to 20 events (assuming we have added maybe 200 sockets before that it may happen)
  struct epoll_event pevents[ 20 ];
  
  // Wait for 10 seconds, and retrieve less than 20 epoll_event and store them into epoll_event array
  int ready = epoll_wait( pollingfd, pevents, 20, 10000 );
  // Check if epoll actually succeed
  if ( ret == -1 )
      // report error and abort
  else if ( ret == 0 )
      // timeout; no event detected
  else
  {
      // Check if any events detected
      for ( int i = 0; i < ready; i++ )
      {
          if ( pevents[i].events & EPOLLIN )
          {
              // Get back our connection pointer
              Connection * c = (Connection*) pevents[i].data.ptr;
              c->handleReadEvent();
           }
      }
  }
  ```

  epoll 的描述符事件有两种触发模式：LT（level trigger）和 ET（edge trigger）。

  - #### LT 模式

    当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

  - #### ET 模式

    和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

#### 4.应用场景

- #### select 应用场景

  select 的 timeout 参数精度为微秒，而 poll 和 epoll 为毫秒，因此 select 更加适用于实时性要求比较高的场景，比如核反应堆的控制。

  select 可移植性更好，几乎被所有主流平台所支持。

- #### poll 应用场景

  poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

- #### epoll 应用场景

  只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接。

  需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

  需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且 epoll 的描述符存储在内核，不容易调试。



## 五、异步IO

<img src="https://pic4.zhimg.com/80/v2-c20e9f8596b3206ce5b1e66504704043_720w.jpg" alt="img" style="zoom:150%;" />

当进程发起一个IO操作，进程返回（不阻塞），但也不能返回果结；内核把整个IO处理完后，会通知进程结果。如果IO操作成功则进程直接获取到数据。

#### 1.典型应用：JAVA7 AIO、高性能服务器应用

#### 2.特点：

- 不阻塞，数据一步到位；Proactor模式；
- 需要操作系统的底层支持，LINUX 2.5 版本内核首现，2.6 版本产品的内核标准特性；
- 实现、开发应用难度大；
- 非常适合高性能高并发应用；

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

## 六、信号驱动IO

<img src="https://pic4.zhimg.com/80/v2-24608409a67c6e7f5e5d53cb9783d513_720w.jpg" alt="img" style="zoom:150%;" />

当进程发起一个IO操作，会向内核注册一个信号处理函数，然后进程返回不阻塞；当内核数据就绪时会发送一个信号给进程，进程便在信号处理函数中调用IO读取数据。相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

#### 1.特点：回调机制，实现、开发应用难度大；

## 七、IO模型比较

<img src="https://pic2.zhimg.com/80/v2-0a418d4da25432f7954139e28cc8b255_720w.jpg" alt="img" style="zoom:150%;" />

#### 1.阻塞IO调用和非阻塞IO调用、阻塞IO模型和非阻塞IO模型

这里的阻塞IO调用和非阻塞IO调用不是指阻塞IO模型和非阻塞IO模型：

- 阻塞IO调用 ：在用户进程（线程）中调用执行的时候，进程会等待该IO操作，而使得其他操作无法执行。
- 非阻塞IO调用：在用户进程中调用执行的时候，无论成功与否，该IO操作会立即返回，之后进程可以进行其他操作（当然如果是读取到数据，一般就接着进行数据处理）。

这个直接理解就好，进程（线程）IO调用会不会阻塞进程自己。所以这里两个概念是相对调用进程本身状态来讲的。

阻塞IO模型是一个阻塞IO调用，而非阻塞IO模型是多个非阻塞IO调用+一个阻塞IO调用，因为多个IO检查会立即返回错误，不会阻塞进程。

非阻塞IO模型对于阻塞IO模型来说区别就是，内核数据没准备好需要进程阻塞的时候，就返回一个错误，以使得进程不被阻塞。

#### 2.同步IO和异步IO

- 同步IO：导致请求进程阻塞，直到I/O操作完成。
- 异步IO：不导致请求进程阻塞。

扩展定义：

- 同步IO：用户进程发出IO调用，去获取IO设备数据，双方的数据要经过内核缓冲区同步，完全准备好后，再复制返回到用户进程。而复制返回到用户进程会导致请求进程阻塞，直到I/O操作完成。
- 异步IO：用户进程发出IO调用，去获取IO设备数据，并不需要同步，内核直接复制到进程，整个过程不导致[请求进程](https://www.zhihu.com/search?q=请求进程&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})阻塞。

阻塞IO模型、非阻塞IO模型、IO复用模型、信号驱动的IO模型者为[同步IO模型](https://www.zhihu.com/search?q=同步IO模型&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A180465312})，只有异步IO模型是异步IO。



参考文章：

https://zhuanlan.zhihu.com/p/180465312

http://www.cyc2018.xyz/计算机基础/Socket/Socket.html#epoll

https://zhuanlan.zhihu.com/p/127148459

https://zhuanlan.zhihu.com/p/127148459