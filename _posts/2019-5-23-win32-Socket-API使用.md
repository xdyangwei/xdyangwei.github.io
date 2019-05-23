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