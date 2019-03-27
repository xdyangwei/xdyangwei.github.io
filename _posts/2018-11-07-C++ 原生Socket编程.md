## C++原生Socket编程
### Socket基础知识介绍
1. socket是使用标准Unix文件描述符（file descriptior）和其他程序通信的方式。
2. 既然是文件描述符，我们就可以使用read()和write()函数来进行套接字通信，但是使用send()和recv()能让我们更好的控制数据传输。
3. 两种常用套接字分别是“Stream Sockets”（流格式：SOCK_STREAM），另外一种是“Datagram Sockets”（数据包格式:SOCK_DGRAM）。
4. 流套接字是可靠的双向通讯的数据流，使用TCP协议，并且按顺序传输，无错误传递，有自己的错误控制，数据报套接字也称为无连接套接字，使用UDP协议（如果确实需要连接可以使用connect()）。
### Socket结构体
```
struct sockaddr {
　　unsigned short sa_family; /* 地址家族, AF_xxx */
　　char sa_data[14]; /*14字节协议地址*/
};
```
sa_family 能够是各种各样的类型，但是大家主流使用都是 "AF_INET"。 sa_data包含套接字中的目标地址和端口信息。为了处理struct sockaddr，程序员创造了一个并列的结构： `struct sockaddr_in ("in" 代表 "Internet"。)`
```
struct sockaddr_in {
　　short int sin_family; /* 通信类型 */
　　unsigned short int sin_port; /* 端口 */
　　struct in_addr sin_addr; /* Internet 地址 */
　　unsigned char sin_zero[8]; /* 与sockaddr结构的长度相同*/
};
```
用这个数据结构可以轻松处理套接字地址的基本元素。注意 sin_zero (它被加入到这个结构，并且长度和 struct sockaddr 一样) 应该使用函数 bzero() 或 memset() 来全部置零。 同时，这一重要的字节，一个指向 sockaddr_in结构体的指针也可以被指向结构体sockaddr并且代替它。这样的话即使 socket() 想要的是 struct sockaddr *，你仍然可以使用 struct sockaddr_in，并且在最后转换。同时，注意 sin_family 和 struct sockaddr 中的 sa_family 一致并能够设置为 "AF_INET"。最后，sin_port和 sin_addr 必须是网络字节顺序 (Network Byte Order)！
```
struct in_addr {
　　unsigned long s_addr;
};
```
### 本机网络字节序转换
1. 假设你想将 short 从本机字节顺序转 换为网络字节顺序。用 "h" 表示 "本机 (host)"，接着是 "to"，然后用 "n" 表 示 "网络 (network)"，最后用 "s" 表示 "short"： h-to-n-s, 或者 htons() ("Host to Network Short")。总共有：
htons()--"Host to Network Short"
htonl()--"Host to Network Long"
ntohs()--"Network to Host Short"
ntohl()--"Network to Host Long"
2. 为什么在数据结构 struct sockaddr_in 中， sin_addr 和 sin_port 需要转换为网络字节顺序，而sin_family 需不需要呢? 答案是： sin_addr 和 sin_port 分别封装在包的 IP 和 UDP 层。因此，它们必须要 是网络字节顺序。但是 sin_family 域只是被内核 (kernel) 使用来决定在数 据结构中包含什么类型的地址，所以它必须是本机字节顺序。同时， sin_family 没有发送到网络上，它们可以是本机字节顺序。
### IP地址转换
1. 假设你已经有了一个sockaddr_in结构体ina，你有一个IP地 址"132.241.5.10"要储存在其中，你就要用到函数inet_addr(),将IP地址从 点数格式转换成无符号长整型。使用方法如下：
```
ina.sin_addr.s_addr = inet_addr("132.241.5.10");
```
注意，inet_addr()返回的地址已经是网络字节格式，所以你无需再调用 函数htonl()。 我们现在发现上面的代码片断不是十分完整的，因为它没有错误检查。 显而易见，当inet_addr()发生错误时返回-1。记住这些二进制数字？(无符 号数)-1仅仅和IP地址255.255.255.255相符合！这可是广播地址！大错特 错！记住要先进行错误检查。
2. 将长整形转换为ip地址，也就是把in_addr结构体转换成点数格式的函数是
`inet_ntoa(ina.sin_addr)`需要注意的是inet_ntoa()将结构体in-addr作为一个参数，不是长整形。同样需要注意的是它返回的是一个指向一个字符的 指针。它是一个由inet_ntoa()控制的静态的固定的指针，所以每次调用 inet_ntoa()，它就将覆盖上次调用时所得的IP地址。假如你需要保存这个IP地址，使用strcopy()函数来指向你自己的字符指针。
### Socket系统调用
1. socket函数调用如下：
```
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```
首先，domain 应该设置成 "AF_INET"，就 象上面的数据结构struct sockaddr_in 中一样。然后，参数 type 告诉内核 是 SOCK_STREAM 类型还是 SOCK_DGRAM 类型。最后，把 protocol 设置为 "0"。(注意：有很多种 domain、type，我不可能一一列出了，请看 socket() 的 man帮助。当然，还有一个"更好"的方式去得到 protocol，同 时请查阅 getprotobyname() 的 man 帮助。) socket() 只是返回你以后在系统调用种可能用到的 socket 描述符，或 者在错误的时候返回-1。全局变量 errno 中将储存返回的错误值。(请参考 perror() 的 man 帮助。)    
2. bind()函数。
一旦你有一个套接字，你可能要将套接字和机器上的一定的端口关联 起来。(如果你想用listen()来侦听一定端口的数据，这是必要一步。如果你只想用 connect()，那么这个步 骤没有必要。
```
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```
sockfd 是调用 socket 返回的文件描述符。my_addr 是指向数据结构 struct sockaddr 的指针，它保存你的地址(即端口和 IP 地址) 信息。 addrlen 设置为 sizeof(struct sockaddr)。因系统的不同，包含的头文件也不尽相同。在处理自己的 IP 地址和/或端口的 时候，有些工作是可以自动处理的。
```
my_addr.sin_port = 0; /* 随机选择一个没有使用的端口 */
my_addr.sin_addr.s_addr = INADDR_ANY; /* 使用自己的IP地址 */
```
上述代码并没有将INADDR_ANY转换为网络字节顺序，其实INADDR_ANY就是0，当然为了更好的工作我们可以执行网络字节转换。
**不要采用小于 1024的端口号。所有小于1024的端口号都被系统保留！你可以选择从1024 到65535的端口(如果它们没有被别的程序使用的话)。**有时候其实我们并不需要用到bind()函数，如果你使用connect() 来和远程机器进行通讯，你不需要关心你的本地端口号，你只要简单的调用connect()就可以了，它会检查套接字是否绑定端口，如果没有，它会自己绑定一个没有使用的本地端口。   
3. connect()函数，connect()系统调用是这样的：
```
#include <sys/types.h>
#include <sys/socket.h>
int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```
sockfd 是系统调用 socket() 返回的套接字文件描述符。serv_addr是保存着目的地端口和IP 地址的数据结构struct sockaddr。addrlen设置为sizeof(struct sockaddr)。再一次，你应该检查connect()的返回值--它在错误的时候返回-1，并设置全局错误变量errno。同时，你可能看到，我没有调用 bind()。因为我不在乎本地的端口号。 我只关心我要去哪。内核将为我选择一个合适的端口号，而我们所连接的 地方也自动地获得这些信息。一切都不用担心。  
4. listen()函数。等待接入请求并且用各种方法处理它们。处 理过程分两步：首先，你听--listen()，然后，你接受--accept()。
```
int listen(int sockfd, int backlog);
```
sockfd 是调用 socket() 返回的套接字文件描述符。backlog 是在进入 队列中允许的连接数目。什么意思呢? 进入的连接是在队列中一直等待直 到你接受 (accept() )连接。它们的数目限制于队列的允许。 大多数系统的允许数目是20，你也可以设置为5到10。在你调用 listen() 前你或者要调用 bind() 或者让内 核随便选择一个端口。如果你想侦听进入的连接，那么系统调用的顺序可能是这样的：
```
socket();
bind();
listen();
/* accept() 应该在这 */
```
5. accept()函数。有人从很远的地方通过一个你在侦听 (listen()) 的端口连接 (connect()) 到你的机器。它的连接将加入到等待接受 (accept()) 的队列 中。你调用 accept() 告诉它你有空闲的连接。它将返回一个新的套接字文 件描述符！这样你就有两个套接字了，原来的一个还在侦听你的那个端口， 新的在准备发送(send())和接收(recv())数据。
```
#include <sys/socket.h>
int accept(int sockfd, void *addr, int *addrlen);
```
sockfd相当简单，是和listen()中一样的套接字描述符。addr是个指向局部的数据结构 sockaddr_in 的指针。这是要求接入的信息所要去的地方（你可以测定那个地址在那个端口呼叫你）。在它的地址传递给accept之前，addrlen是个局部的整形变量，设置为sizeof(struct sockaddr_in)。accept将不会将多余的字节给addr。如果你放入的少些，那么它会通过改变 addrlen的值反映出来。 
**注意，在系统调用 send() 和 recv() 中你应该使用新的套接字描述符 new_fd。如果你只想让一个连接进来，那么你可以使用 close() 去关闭原 来的文件描述符 sockfd 来避免同一个端口更多的连接。**   
6. send()andrecv()函数。这两个函数用于流式套接字或者数据报套接字的通讯。如果你喜欢使 用无连接的数据报套接字，你应该看一看下面关于sendto() 和 recvfrom() 的章节。
send() 是这样的：
```
int send(int sockfd, const void *msg, int len, int flags);
```
sockfd 是你想发送数据的套接字描述符(或者是调用 socket() 或者是 accept() 返回的。)msg 是指向你想发送的数据的指针。len 是数据的长度。 把 flags 设置为 0 就可以了。(详细的资料请看send()的 man page)。send() 返回实际发送的数据的字节数--它可能小于你要求发送的数 目！ 注意，有时候你告诉它要发送一堆数据可是它不能处理成功。它只是 发送它可能发送的数据，然后希望你能够发送其它的数据。记住，如果 send() 返回的数据和 len 不匹配，你就应该发送其它的数据。但是这里也 有个好消息：如果你要发送的包很小(小于大约 1K)，它可能处理让数据一 次发送完。最后要说得就是，它在错误的时候返回-1，并设置 errno。   
recv() 函数很相似：
```
int recv(int sockfd, void *buf, int len, unsigned int flags);
```
sockfd 是要读的套接字描述符。buf是要读的信息的缓冲。len是缓冲的最大长度。flags 可以设置为0。(请参考recv() 的 man page。) recv() 返回实际读入缓冲的数据的字节数。或者在错误的时候返回-1， 同时设置errno。  
7. 既然数据报套接字不是连接到远程主机的，那么在我们发送一个包之前需要什么信息呢? 不错，是目标地址！看看下面的：
```
int sendto(int sockfd, const void *msg, int len, unsigned int flags,
const struct sockaddr *to, int tolen);
```
 to 是个指向数据结构 struct sockaddr 的指针，它包含了目的地的 IP 地址和端口信息。tolen 可以简单地设置为 sizeof(struct sockaddr)。 和函数 send() 类似，sendto() 返回实际发送的字节数(它也可能小于 你想要发送的字节数！)，或者在错误的时候返回 -1。   
recvfrom() 的定义是这样的：
```
int recvfrom(int sockfd, void *buf, int len, unsigned int flags, 　
struct sockaddr *from, int *fromlen);
```
from 是一个指向局部数据结构 struct sockaddr 的指针，它的内容是源机器的 IP 地址和端口信息。fromlen 是个 int 型的局部指针，它的初始值为 sizeof(struct sockaddr)。函数调用返回后，fromlen 保存着实际储存在 from 中的地址的长度。recvfrom() 返回收到的字节长度，或者在发生错误后返回 -1。
**记住，如果你用 connect() 连接一个数据报套接字，你可以简单的调 用 send() 和 recv() 来满足你的要求。这个时候依然是数据报套接字，依 然使用 UDP，系统套接字接口会为你自动加上了目标和源的信息。**  
8. 关闭套接字描述符使用close函数`close(sockfd);`，如果你想在关闭套接字上有多一点的控制，可以使用函数shutdowm()。它允许你将一定方向上的通讯或者双向的通讯(就象close()一 样)关闭，你可以使用：`int shutdown(int sockfd, int how);`  
sockfd 是你想要关闭的套接字文件描述复。how 的值是下面的其中之一：  
0 – 不允许接受  
1 – 不允许发送  
2 – 不允许发送和接受(和 close() 一样)  
shutdown() 成功时返回 0，失败时返回 -1(同时设置 errno。) 如果在无连接的数据报套接字中使用shutdown()，那么只不过是让 send() 和 recv() 不能使用(记住你在数据报套接字中使用了 connect 后 是可以使用它们的)。  
9. getpeername()函数。函数 getpeername() 告诉你在连接的流式套接字上谁在另外一边。
```
#include <sys/socket.h>
int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
```
sockfd 是连接的流式套接字的描述符。addr 是一个指向结构 struct sockaddr (或者是 struct sockaddr_in) 的指针，它保存着连接的另一边的 信息。addrlen 是一个 int 型的指针，它初始化为 sizeof(struct sockaddr)。 函数在错误的时候返回 -1，设置相应的 errno。一旦你获得它们的地址，你可以使用 inet_ntoa() 或者 gethostbyaddr() 来打印或者获得更多的信息。但是你不能得到它的帐号。  
10. gethostname()函数。甚至比 getpeername()还简单的函数是 gethostname()。它返回你程序所运行的机器的主机名字。然后你可以使用gethostbyname() 以获得你的机器的IP 地址。
```
#include <unistd.h>
int gethostname(char *hostname, size_t size);
```
参数很简单：hostname 是一个字符数组指针，它将在函数返回时保存 主机名。size是hostname 数组的字节长度。
函数调用成功时返回 0，失败时返回 -1，并设置 errno。

