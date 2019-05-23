### 头文件与链接
1.     ```
       #include<WinSock2.h>
       #pragma comment(lib,"ws2_32.lib")
       ```

### WSAStartup和WSACleanup函数
1. WSAStartup函数用于初始化供进程调用的Winsock相关的dll，第一个参数是WORD类型的Winsock版本号，通常使用MAKEWORD来生成一个版本号，第二个参数是指向WSADATA结构体的指针，系统对windows sockets的描述会写入到这个结构体中。
2. WSAStartup函数如果调用成功将会返回0，否则就会返回五种错误代码之一，但是不能使用WSAGetLastError来获取错误代码。
3. WSACleanup函数释放对Winsock链接库的调用，无参数，返回值0表示正常退出，返回值SOCKET_ERROR表示异常，当异常时可以调用WSAGetLastError查看错误代码。需要注意的是，在多线程环境下，WSACleanup函数将终止所有线程的socket操作。

### socket和closesocket函数
1. socket函数是用于创建socket的函数，将创建指定传输服务的socket，第一个参数af表明地址簇类型，比如AF_UNSPEC(未指明)、AF_INET(IPv4)、AF_INET6(IPv6)等。第二个参数type指明socket的类型，比如SOCK_STREAM（流套接字，使用TCP协议）、SOCK_DGRAM（数据报套接字，使用UDP协议）、SOCK_RAW（原始套接字）等
2. socket函数第三个参数指明数据传输协议，该参数取决于af和type参数类型。protocol参数在Winsock2.h and Wsrm.h定义。通常使用如下3中协议：  
IPPROTO_TCP（TCP协议，使用条件，af是AF_INET or AF_INET6、type是SOCK_STREAM）  
IPPROTO_UDP（UDP协议，使用条件，af是AF_INET or AF_INET6、type是SOCK_DGRAM）  
IPPROTO_RM（PGM（Pragmatic General Multicast，实际通用组播协议）协议，使用条件，af是AF_INET 、type是SOCK_RDM）
3. socket函数如果调用成功将会返回socket的描述符（句柄），否则将返回INVALID_SOCKET，可以使用WSAGetLastError来获取错误代码。函数声明如下：
    ```
    SOCKET socket(
        _in    int af,
        _in    int type,
        _in    int protocol
    );
    ```
4. closesocket函数关闭socket，参数为需要关闭的SOCKET类型对象，如果无错误发生，函数返回0。否则，返回SOCKET_ERROR，可以使用WSAGetLastError来获取错误代码。。

### bind函数
1. bind函数将socket绑定到一个本地地址，通常用于服务器端,函数声明如下：
    ```
    int bind(
        _in     SOCKET s;
        _in     const struct sockaddr* name,
        _in     int namelen
    );
    ```
2. bind函数第一个参数s指定一个未绑定的socket，第二个参数name为指向sockaddr地址的指针，该结构体内含有IP地址的PORT端口号，第三个参数namelen为参数name的字节数
3. bind函数调用成功返回0，否则返回SOCKET_ERROR，可以使用WSAGetLastError来获取错误代码。
4. sockaddr结构体与unix底下同名结构体相同，并且由于其内部使用数组，赋值不方便，因此一般使用格式化后的结构体SOCKADDR_IN来赋值。SOCKADDR_IN里面又包含了in_addr 结构体，in_addr包含了网络地址信息。
5. 由于不同机器cpu字节序不同，网络字节序统一采用大端字节序方式。htons用于将unsigned short的数值从littele-endian转换为big-endian，由于short只有2字节，常用语port数值的转换。htonl用于将unsigned long的数值从littele-endian转换为big-endian，long有4字节，常用于ipv4地址的转换。
6. inet_addr用于将ipv4格式的字符串转换为unsigned long的数值。inet_ntoa用于将struct in_addr的地址转换为ipv4格式的字符串。
7. **INADDR_ANY，用INADDR_ANY来配置IP地址，意味着不需要知道当前服务器的IP地址。对于多网卡的服务器，INADDR_ANY允许你的服务接收一个服务器上所有网卡发来的数据。如果某个socket使用INADDR_ANY和8000端口，那么它将接收该所有网卡传来的数据。而其他socket将无法再使用8000端口。**

### listen函数
1. listen函数将某个socket置于监听状态，等待从这个socket发来的连接请求，通常用于服务器端。函数声明如下：
    ```
    int listen(
        _in    Socket s,
        _in    int backlog
    );
    ```
2. listen函数第一个参数为socket描述符，该socket是个未连接状态的socket，第二个参数backlog为挂起连接的最大长度，如果该值设置为SOMAXCONN，负责socket的底部服务提供商将设置该值为最大合理值，并没有该值的明确规定。
3. 调用成功将返回0，否则返回SOCKET_ERROR，可以使用WSAGetLastError来获取错误代码。

### accept函数
1. accept允许socket上的连接，通常用于服务器端，函数声明如下：  
    ```
    SOCKET accept(
        _in        SOCKET s,
        _out       struct sockaddr* addr,
        _in_out    int* addrlen
    );
    ```
2. 第一个参数s为listen函数用到的socket，accept函数将创建连接，第二个参数为指向通信层连接实体地址的指针，客户端连接地址会被写入到这个地址中，addr的格式取决于bind函数内地址簇的类型。addrlen为addr的长度指针。
3. 如果不发生错误，accept将返回一个新的SOCKET描述符，即新建连接的socket句柄。否则，将返回INVALID_SOCKET。传进去的addrlen应该是参数addr的长度，返回的addrlen是实际长度。

### connect函数
1. connect一般用于客户端请求服务端连接，函数声明如下：  
    ```
    int connect(
        _in     SOCKET s,
        _in     const struct sockaddr* name,
        _in     int namelen
    );
    ```
2. 第一个参数s为未连接的socket描述符，第二个参数name为待连接的地址，第三个参数namelen为name的长度。
3. accept函数调用返回值0表示正确，否则，将返回SOCKET_ERROR。如果是阻塞式的socket连接，返回值代表了连接正常与失败，可以使用WSAGetLastError来获取错误代码。。

### send、recv函数
1. send和recv函数用于发送和接收数据，send函数用于在已连接的socket上发送数据，recv用于在已连接或绑定的socket上接收数据，二者函数声明如下：  
    ```
    int send(
        _in      SOCKET s,
        _in      const char* buf,
        _in      int len,
        _in      int flags
    );
    int recv(
        _in      SOCKET s,
        _in      char* buf,
        _in      int len,
        _in      int flags
    );
    ```
2. 两个函数第一个参数s都是已连接的socket描述符，第二个参数buf是用于传输或写入数据的地址，第三个参数len为待发送或接收的数据长度，第四个参数flags为send(recv)函数的发送(接收)数据方式，MSDN给了以下几种发送方式：  
1：MSG_DONTROUTE（Specifies that the data should not be subject to routing. A Windows Sockets service provider can choose to ignore this flag.）  
2：MSG_OOB（Sends OOB data (stream-style socket such as SOCK_STREAM only）  
3：0
3. send的返回值标识已发送数据的长度，这个值可能比参数len小，这也意味着数据缓冲区没有全部发出去，要进行后续处理。返回SOCKET_ERROR标识send出错。  
recv的返回值标识已接收数据的长度。如果连接已关闭，返回值将是0。返回SOCKET_ERROR标识recv出错，可以使用WSAGetLastError来获取错误代码。。

### shutdown函数
1. shutdown函数的作用是禁止在socket上收发数据，函数声明如下：
    ```
    int shutdown(
        _in      SOCKET s,
        _in      int how
    );
    ```
2. 第一个参数s是socket描述符，指明需要禁止操作的socket，第二个参数how，描述了哪些操作将不被允许，how有以下几类取值：  
1：SD_RECEIVE表明关闭接收通道，在该socket上不能再接收数据，如果当前接收缓存中仍有未取出数据或者以后再有数据到达，则TCP会向发送端发送RST包，将连接重置。  
2：SD_SEND表明关闭发送通道，TCP会将发送缓存中的数据都发送完毕并在收到所有数据的ACK后向对端发送FIN包，表明本端没有更多数据发送。这个是一个优雅关闭过程。  
3：SD_BOTH则表示同时关闭接收通道和发送通道。
3. 调用成功则会返回0，不成功则会返回SOCKET_ERROR，shutdown并不关闭socket，只是禁止掉socket的recv或send行为。为保证数据收发完整性，在关闭socket之前应调用shutdown，可以使用WSAGetLastError来获取错误代码。。

### getsockname函数和getpeername函数
1. getsockname函数获取本地IP和PORT，getpeername函数获取对端IP和PORT，二者函数声明如下：
    ```
    int getsockname(
        _in       SOCKET s,
        _out      struct sockaddr* name,
        _in_out   int* namelen
    );
    int getpeername(
        _in       SOCKET s,
        _out      struct sockaddr* name,
        _in_out   int* namelen
    );
    ```
2. 两个函数第一个参数s都是socket描述符，第二个参数是待存放本地或对端地址信息的sockaddr结构体指针，第三个参数为结构体长度指针。
3. 调用成功返回0，出错则返回SOCKET_ERROR，可以使用WSAGetLastError来获取错误代码。
