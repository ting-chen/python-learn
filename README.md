# python-learn

1. [PyCharm-Tips](#user-content-pycharm-tip)
2. [Socket](#user-content-Socket)

## pyCharm

### [**Tips**](pycharm-tip)

> 1. **Ctrl+N** : 通过class名称快速查找class定位
> 2. **Ctrl+G** : 快速跳转到指定行列
> 3. **Shift+Ctrl+Backspace** : 跳转到最后一次编辑的行
> 4. **Ctrl+Shift+T** : 快捷生成测试代码
> 5. **Ctrl+B** : 对于调用的查找定义处. 对于定义的查找调用处
> 6. **Ctrl+Alt+B** : 快速跳转到声明处
> 7. **Ctrl+F12** : 列出当前文件的全部变量以及函数声明
> 8. **Ctrl+Shift+Alt+2** : 列出当前文件的路径结构
> 9. **Ctrl+Alt+H** : 对函数快速编辑
> 10. **Shift+F6** : 快速重命名
> 11. **Ctrl+Q** : 查找注释
> 12. **Ctrl+Tab** : 切换鼠标焦点到不同的页面中

### [**Socket**](Socket)

> 1. socket. AF_UNIX
>
>    [^ AF_UNIX]: A Unix domain socket or IPC socket (inter-process communication socket) is a data communications endpoint for exchanging data between processes executing on the same host operating system. Valid socket types in the UNIX domain are: SOCK_STREAM (compare to TCP), for a stream-oriented socket; SOCK_DGRAM (compare to UDP), for a datagram-oriented socket that preserves message boundaries (as on most UNIX implementations, UNIX domain datagram sockets are always reliable and don't reorder datagrams); and SOCK_SEQPACKET (compare to SCTP), for a sequenced-packet socket that is connection-oriented, preserves message boundaries, and delivers messages in the order that they were sent.[1] The Unix domain socket facility is a standard component of POSIX operating systems 对内核压力大
>
>    
>
> 2. socket.AF_INET
>
>    [^AF_INET]: 指向网络通信，需要过底层协议层层往上传输
>
>    **(host, port)**
>
>    - host可以是域名
>    - host可以是主机ip地址
>
> 3. socket. AF_INET6
>
>    [^AF_INT6]: **(host, port, flowinfo, scopeid)** IPV6网络
>
>    - 使用了一个四元组(主机、端口、flowinfo、scopeid)，其中flowinfo和scopeid表示c中的结构sockaddr_in6中的sin6_flowinfo和sin6_scope_id成员。然而，请注意，scopeid的省略可能会导致操作scoped IPv6地址的问题
>
> 4. **INADDR_ANY** 对于IPv4来说指向任意地址（host）;**INADDR_BROADCAST** IPv4 指向任意端口
>
> 5. **tuple form** New in version 2.6:* Linux-only support for TIPC is also available using the `AF_TIPC` address family. TIPC is an open, non-IP based networked protocol designed for use in clustered computer environments. Addresses are represented by a tuple, and the fields depend on the address type. The general  is  ***(addr_type, v1, v2, v3 [, scope]***, where:
>
>    - *addr_type* is one of `TIPC_ADDR_NAMESEQ`, `TIPC_ADDR_NAME`, or `TIPC_ADDR_ID`.
>
>    - *scope* is one of `TIPC_ZONE_SCOPE`, `TIPC_CLUSTER_SCOPE`, and `TIPC_NODE_SCOPE`.
>
>    - If *addr_type* is `TIPC_ADDR_NAME`, then *v1* is the server type, *v2* is the port identifier, and *v3* should be 0.
>
>      If *addr_type* is `TIPC_ADDR_NAMESEQ`, then *v1* is the server type, *v2* is the lower port number, and *v3* is the upper port number. **接受范围值**
>
>      If *addr_type* is `TIPC_ADDR_ID`, then *v1* is the node, *v2* is the reference,**and *v3* should be set to 0.**
>
> 6. *exception*
>
>    1. **socket.error**
>
>       [^SocketErr]: either a `string` telling what went wrong or a pair `(errno, string)`
>
>    2. **socket.herror**
>
>       [^SocketHErr]: raised for `address`-related errors ,accompanying value is a pair `(h_errno, string)`
>
>    3. **socket.gaierror** 同herror
>
>    4. **socket.timeout**

