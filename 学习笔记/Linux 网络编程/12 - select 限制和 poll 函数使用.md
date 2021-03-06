## select 限制

- select 会修改传递进来的 fd_sets，导致它们不能被复用。即使你不需要做任何改变，例如当一个描述符接收到数据后还需要接收更多的数据，整个集合都需要再次重新构建或者使用 FD_COPY 来从备份中恢复。并且每次调用 select 时都需要这些操作。
- 找到是哪个描述符产生的事件，需要调用 FD_ISSET 遍历集合中的所有描述符。当你有 2000 个描述符时，并且只有一个产生了事件而且是最后一个，导致每次循环都会浪费大量 CPU 资源。
- 每次调用 select，都需要把 fd 集合从⽤用户态拷贝到内核态，这个开销在 fd 很多时会很大。
- 我刚刚提到了 2000 个描述符吗？好吧 select 并不能支持这个多个描述符。至少在 Linux 上所支持的最大描述符数量是 1024 个，它保存在 FD_SETSIZE 常量中。有些操作系统允许你在包含 sys/select.h 头文件之前重新定义 FD_SETSIZE 的值，但是这就失去了可移植性。而且 Linux 会忽略它，保持原有的限制不变。
- 当描述符在 select 中被监听时其他的线程不能修改它。假设你有一个管理线程检测到 sock1 等待输入数据的时间太长需要关闭它，以便重新利用 sock1 来服务其他工作线程。但是它还在 select 的监听集合中。如果此时这个套接字被关闭会发生什么？select 的 man 手册中有解释：如果 select 正在监听的套接字被其他线程关闭，结果是未定义的。
- 相同的问题，如果另外一个线程突然决定通过 sock1 发送数据，在等待 select 返回之前不能监听这个套接字的写事件。
- 选择监听的事件类型是有限的；例如，检查一个远程套接字是否关闭你只有两种方法：监听它的读事件或者尝试实际去读取这个套接字的数据来探测它是否关闭（当关闭时会返回 0）。如果你希望从这个套接字中读取数据这种方法是可行的，但是如果你是在发送文件完全不需要关心读事件该怎么办了？
- 当填充描述符集合时，select 会给你带来额外的负担，因为你需要计算描述符中的最大值并把它当作函数参数传递给 select。

下面演示如何修改最大文件描述符限制。

一个进程能打开的最大文件描述符限制，这可以通过调整内核参数进行设置。命令`ulimit -n`，修改为 2048 的命令是命令： `sudo ulimit -n 2048`。

通过代码实现：

```c
// 改变当前进程和它的子进程的值，且需要 root 权限，父进程不可以
struct rlimit rl;
getrlimit(RLIMIT_NOFILE, &rl);

printf("%d\n", (int)rl.rlim_max);

rl.rlim_cur = 2048;
rl.rlim_max = 2048;
setrlimit(RLIMIT_NOFILE, &rl);
```

注意 select 中 fd_set 集合容量的限制 FD_SETSIZE，修改它需要重新编译内核。

当然操作系统开发人员也会意识到这些缺陷，并且在设计 poll 口时解决了大部分问题，因此你会问，还有任何理由使用 select 吗？为什么不直接淘汰它了？其实还有两个理由使用它：

第一个原因是可移植性。select 已经存在很长时间了，你可以确定每个支持网络和非阻塞套接字的平台都会支持 select，而它可能还不支持 poll。另一种选择是你仍然使用 poll 然后在那些没有 poll 的平台上使用 select 来模拟它。

第二个原因非常奇特，select 的超时时间理论上可以精确到纳秒级别。而 poll 和 epoll 的精度只有毫秒级。这对于桌面或者服务器系统来说没有任何区别，因为它们不会运行在纳秒精度的时钟上，但是在某些与硬件交互的实时嵌入式平台上可能是需要的。

只有在上面提到的原因中你必须使用 select 没有其他选择。但是如果你编写的程序永远不会处理超过一定数量的连接（例如：200），此时 select 和 poll之 间选择不在于性能，而是取决于个人爱好或者其他原因。

## poll 函数使用

编写 poll 接口的主要目的就是为了解决 select 的缺陷，所以它具有以下优点：

- 它监听的描述符数量没有限制（跟内存大小有关），可以超过 1024。
- 它不会修改 pollfd 结构体中传递的数据，因此可以复用只需将产生事件的描述符对应的 revents 成员置 0。IEEE 规范中规定：“poll() 函数应该负责将每个 pollfd 
结构体中 revents 成员清 0，除非应用程序通过上面列出的事件设置对应的标记位来报告事件，poll() 函数应该判断对应的位是否为真来设置 revents 成员中对应的位”。
但是根据我的经验至少有一个平台没有遵循这个建议，Linux 中的 man 2 poll 就没有做出这样的保证。相比于 select 来说可以更好的控制事件。例如，
它可以检测对端套接字是否关闭而不需要监听它的读事件。

另外需要记住以下几个问题：

- 和 select 一样必须通过遍历描述符列表来查找哪些描述符产生了事件。更糟糕的是在内核空间也需要通过遍历来找到哪些套接字正在被监听，然后在重新遍历整个列表来设置事件。
- 和 select 一样它也不能在描述符被监听的状态下修改或者关闭套接字。

但是请记住对于大多数客户端网络应用程序来说这些问题不会带来任何影响，除了 P2P 这种类型的应用程序可能同时打开数千个连接。这些问题甚至对于有些服务器
应用程序来说也没有任何影响。所以 poll 相对于 select 来说应该是你的默认选项，除非你有上面提到选择 select 的两个理由。如果是下面提到的这些情况，
相比于 epoll 你更应该选择 poll：

- 你需要在不止 Linux 一个平台上运行，而且不希望使用 epoll 的封装库。例如 libevent（epoll 是 Linux 平台上特有的）。
- 同一时刻你的应用程序监听的套接字少于 1000（这种情况下使用 epoll 不会得到任何益处）。
- 同一时刻你的应用程序监听的套接字大于 1000，但是这些连接都是非常短的连接（这种情况下使用 epoll 也不会得到任何益处，因为 epoll 所带来的加速都会被
添加新描述符到集合中时被抵消）。
- 你的应用程序没有被设计成在改变事件时而其他线程正在等待事件。

接口说明，

```c
struct pollfd
{
  int fd；      // 文件描述符
　short event； // 请求的事件
　short revent；// 返回的事件
};

#include <poll.h>
int poll(struct pollfd fd[], nfds_t nfds, int timeout);
```

revents 域是文件描述符的操作结果事件掩码，内核在调用返回时设置这个域。events 域中请求的任何事件都可能在 revents 域中返回。

```
POLLIN          有数据可读
POLLRDNORM      有普通数据可读
POLLRDBAND      有优先数据可读
POLLPRI         有紧迫数据可读
POLLOUT         写数据不会导致阻塞
POLLWRNORM      写普通数据不会导致阻塞
POLLWRBAND      写优先数据不会导致阻塞
POLLMSGSIGPOLL  消息可用
```

此外，revents 域中还可能返回下列事件：

```
POLLER    指定的文件描述符发生错误
POLLHUP   指定的文件描述符挂起事件
POLLNVAL  指定的文件描述符非法
```

这些事件在 events 域中无意义，因为它们在合适的时候总是会从 revents 中返回。

例如，要同时监视一个文件描述符是否可读和可写，我们可以设置 events 为 POLLIN |POLLOUT。在 poll 返回时，我们可以检查 revents 中的标志，对应于文件描述符请求的 events 结构体。如果 POLLIN 事件被设置，则文件描述符可以被读取而不阻塞。如果 POLLOUT 被设置，则文件描述符可以写入而不导致阻塞。这些标志并不是互斥的：它们可能被同时设置，表示这个文件描述符的读取和写入操作都会正常返回而不阻塞。

timeout 参数指定等待的毫秒数，无论 I/O 是否准备好，poll 都会返回。timeout 指定为负数值表示无限超时，使 poll() 一直挂起直到一个指定事件发生。timeout 为 0 表示 poll 调用立即返回并列出准备好 I/O 的文件描述符，但并不等待其它的事件。这种情况下，poll() 就像它的名字那样，一旦选举出来，立即返回。

poll 函数的返回值，

成功时，poll() 返回结构体中 revents 域不为 0 的文件描述符个数。如果在超时前没有任何事件发生，poll() 返回 0。失败时，poll() 返回 -1，并设置 errno 为下列值之一：
　　
```
EBADF       一个或多个结构体中指定的文件描述符无效
EFAULT      fd 指针指向的地址超出进程的地址空间
EINTR       请求的事件之前产生一个信号，调用可以重新发起
EINVAL      nfds 参数超出 PLIMIT_NOFILE 值
ENOMEM      可用内存不足，无法完成请求
```

使用案例，

```c
int startup(char *ip,int port)
{
    int listen_sock = socket(AF_INET,SOCK_STREAM,0);
    struct sockaddr_in local;
    local.sin_family = AF_INET;
    local.sin_addr.s_addr = inet_addr(ip);
    local.sin_port = port;
    bind(&listen_sock,(struct sockaddr*)&local,sizeof(local));
    if(listen(listen_sock,5) < 0)
    {
        perror("listen");
    }
    return listen_sock;
}

int main(int argc,char* argv[])
{
    struct pollfd pfd[1];
    int len = 1;

    .....
    pfd[0].fd = startup(ip, port);
    pfd[0].events = POLLIN;
    pfd[0].revents = 0;

    int done = 0;
    while(!done)
    {
        switch(poll(pfd,1,-1))
        {
            case 0:
                printf("timeout\n");
                break;
            case -1:
                perror("poll");
                break;
            default:
            {
                char buf[1024];
                if(pfd[0].revents & POLLIN)
                {
                    ssize_t _s = read(pfd[0].fd, buf, sizeof(buf) - 1);
                    if(_s > 0)
                    {
                        buf[_s] = '\0';
                        printf("echo:%s\n", buf);
                    }
                }
            }
            break;
        }
    }
}
```

## 参考

- <http://cxd2014.github.io/2018/01/10/epoll/>
