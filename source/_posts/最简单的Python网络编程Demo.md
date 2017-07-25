---
title: 最简单的Python网络编程Demo
date: 2017-07-25 14:47:20
tags: Python
categories: 网络编程
description:
---




该Demo的大概流程如下：

1. 一台客户机从其标准输入（键盘）读入一行字符，并通过其套接字将该行发送到服务器。

2. 服务器从其连接套接字读取一行字符。

3. 服务器将该行字符前加一个`Server: `。

4. 服务器将修改后的字符串（行）通过连接套接字再发回给客户机。

5. 客户机从其套接字中读取修改后的行，然后将该行在其标准输出（监视器）上打印出来。

<!--more-->

## Server端

我们使用 socket 模块的 socket 函数来创建一个 socket 对象。socket 对象可以通过调用其他函数来设置一个 socket 服务。

现在我们可以通过调用 bind(hostname, port) 函数来指定服务的 port(端口)。

接着，我们调用 socket 对象的 accept 方法。该方法等待客户端的连接，并返回 connection 对象，表示已连接到客户端。

完整代码如下：
```python
# encoding: utf-8
import socket
import sys
import os


HOST = ''
PORT = 8888
BUFFER = 1024
ADDR = (HOST, PORT)

# 初始化套接字，AF_INET:IPv4, AF_INET6:IPv6，SOCK_STREAM:面向流的TCP协议
tcp_server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
tcp_server_socket.bind(ADDR)             # 绑定本机IP和任意端口(>1024)
tcp_server_socket.listen(5)              # 监听连接，等待客户端，等待连接的最大数目为1
print 'Waiting for connections...'

while True:     # TCP服务器端处理逻辑
    tcp_client_socket, client_addr = tcp_server_socket.accept()     # 接受新的连接请求，返回客户端的socket对象和客户端的地址
    print 'connected with %s: %s' % client_addr
    while True:
        data = tcp_client_socket.recv(BUFFER)                       # 接收客户端的数据
        print 'Client: %s' % data
        if not data:
            break
        tcp_client_socket.send('Server: %s' % data)                 # 发送给客户端的数据，这里直接在前面加了Server

tcp_server_socket.close()				# 关闭连接

```

## Client端

```python
import socket


HOST = '127.0.0.1'
POST = 8888
BUFSIZE = 1024
ADDR = (HOST, POST)
tcp_client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)   # 创建套接字
tcp_client_socket.connect(ADDR)                                         # 连接到server的IP与端口

while True:
    input = raw_input('>')                  # 从终端接收一个输入
    if input.lower() == 'quit':
        print 'Quit'
        break
    tcp_client_socket.send(input)           # 发送到server
    data = tcp_client_socket.recv(BUFSIZE)  # 从server接收
    print data
    if not data:
        break

tcp_client_socket.close()                   # 输入quit跳到此处关闭连接

```

<!--more-->