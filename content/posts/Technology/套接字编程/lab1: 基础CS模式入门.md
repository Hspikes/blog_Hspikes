+++
date = '2025-09-03T17:33:00+08:00'
draft = false
title = 'lab1: 基础CS模式入门'
katex = true
+++

记录我在做《计算机网络：自顶向下方法》实验过程的一些问题以及掌握的知识。同时也包含了 python 语言的一些入门。

因为自己没有系统的去学习过 python 语言，所以在编程的过程中一边一葫芦画瓢，一边借助 Copilot 辅助。在以前用的相对偏少，这次频繁使用后深感现在生成式 AI 的强大，就连在复制实验提供的代码框架时，仅仅复制了一半的框架，自动补全另一半框架和实验提供框架完全一致。这个或许是数据库中包含，没什么了不起的。然而框架中的每一部分都可以轻松细化，并且根据你的要求对答如流，我觉得或许程序员的末日已经到了吧，要只争朝夕的提桶跑路了😭

Anyway, 回归正题，建立连接的流程不再赘述，忘了去看笔记，直接进入代码环节。

## Webserver 实现

### 框架代码

实验提供的框架代码如下:

```python
#import socket module
from socket import *
import sys # In order to terminate the program
serverSocket = socket(AF_INET, SOCK_STREAM)
#Prepare a sever socket
#Fill in start
#Fill in end
while True:
    #Establish the connection
    print('Ready to serve...')
    connectionSocket, addr = #Fill in start #Fill in end
    try:
        message = #Fill in start #Fill in end
        filename = message.split()[1]
        f = open(filename[1:])
        outputdata = f.read()
        #Send one HTTP header line into socket
        #Fill in start
        #Fill in end
        #Send the content of the requested file to the client
        for i in range(0, len(outputdata)):
            connectionSocket.send(outputdata[i].encode())
        connectionSocket.send("\r\n".encode())
        connectionSocket.close()
    except IOError:
        #Send response message for file not found
        #Fill in start
        #Fill in end
        #Close client socket
        #Fill in start
        #Fill in end
```

### 模块导入

最开始的部分类似于 C/C++ 的导入库，可以看到在这个代码中就包含了两种库的写法。

1. ```from socket import *```: 这里的 ```*``` 表示导入模块中的所有成员，**直接导入到当前命名空间**。
2. ```import socket```: 这种方式是将**模块整体导入命名空间，使用时要加上模块前缀名**，例如```accept```要写做```socket.accept```。

两者分别的优势非常显然，一个简洁，一个清晰避免冲突。根据需要使用就可以了。

### Socket 模块

只写一些常用的函数，一些常量在函数实例中会有解释。

1. ```socket()```: 用于创建一个套接字（socket）。
    - 语法：```socket(family, type, proto=0)```
    - family：地址族（如 ```AF_INET``` 表示 IPv4，```AF_INET6``` 表示 IPv6）。
    - type：套接字类型（如 ```SOCK_STREAM``` 表示 TCP，```SOCK_DGRAM``` 表示 UDP）。
    - proto：协议号，通常默认为 0。
2. ```bind()```: 将套接字绑定到指定的地址和端口。
    - 语法：```socket.bind(address)```
    - address 是一个元组 (host, port)。
    - 示例：```s.bind(('localhost', 8080))```
3. ```listen()```: 使套接字进入监听模式，等待客户端连接（仅适用于 TCP）。
    - 语法：```socket.listen(backlog)```
    - backlog：允许的最大连接数。
    - 示例：```s.listen(5)```
4. ```accept()```: 接受客户端连接，返回一个新的套接字和客户端地址。
    - 语法：```socket.accept()```
    - 示例：```connection, addr = s.accept()```
5. ```connect()```: 主动连接到服务器（仅适用于客户端）。
    - 语法：```socket.connect(address)```
    - 示例：```s.connect(('example.com', 80))```
6. ```send()```: 用于发送数据。
7. ```recv()```: 接收数据。
    - 语法：```socket.recv(bufsize)```
    - bufsize：接收缓冲区大小。
    - 示例：```data = s.recv(1024)```
8. ```close()```: 关闭套接字。
    - 语法：```socket.close()```

注意到除了创建套接字，其余的函数都需要类似类方法一般**指定对象调用**。

那么第一个补充块的内容就很简单了，我们只需要生成一个地址，然后绑定上就可以了。我这里的地址是随便编的，反正我本地的 8080 号端口确实没有被占用。

```python
server_address = ('0.0.0.0',8080)
serverSocket.bind(server_address)
serverSocket.listen(5)
```

### HTTP 报文解读

一个典型的 HTTP 请求报文可能如下：

```html
GET /filename.html HTTP/1.1
Host: localhost:8080

...
```

第一行是请求行，包含 HTTP 方法、请求文件以及协议版本。第二行首部行标识主机地址以及端口号。随后**一个空行表明头部行结束**。后续跟上数据。

相应报文则应当由如下格式：

```html
HTTP/1.1 200 OK
Content-Type: text/html

...
```

解读完全类似，包含**状态码以及状态码解释**，第二行为响应头部，注意就算是 404 不会返回任何数据仍然应当包含相应头部！

利用 python 方法实现对报文拆分：

1. ```split()```：按照指定的分隔符（默认为空格）将字符串分割成一个列表。
2. ```splitline()```：按照换行符（\n 或 \r\n）将字符串分割成一个列表。

```python
    connectionSocket, addr = serverSocket.accept()
    # 先调用 accept 建立连接
    try:
        message = connectionSocket.recv(2048).decode()
        print("Request message:", message)
        filename = message.split()[1]
        # 在命令行找到文件名，注意用 decode 对字符串解码
        f = open(filename[1:])
        outputdata = f.read()
        header = "HTTP/1.1 200 OK\r\n"
        header += "Content-Type: text/html\r\n"
        header += "\r\n"
        connectionSocket.send(header.encode())
        # 先发送头部，注意要用 encode 对字符串编码
```

注意到我们这里用了异常处理，最为这段中最为典型的异常肯定是 open 没有找到对应的文件，也就可以对应我们的 404，我们如法炮制补充 except 的情况。

```python
    except IOError:
        header = "HTTP/1.1 404 Not Found\r\n"
        header += "Content-Type: text/html\r\n"
        header += "\r\n"
        connectionSocket.send(header.encode())
        # 发个 404 回去
        connectionSocket.close()
```

这样下来最为基础的功能就完成了，我们可以简单测试，在终端运行 ```python3 webserver.py```。再在浏览器访问对应端口 ```localhost:8080/index.html```。我的 html 用的是网上随便薅下来的，确保和代码在同一个目录下，能够测试就可以了。

### 终止机制

下面部分就已经超出了基本的要求，是一些我自己实验中拓展内容。

如果你测试时频繁终止进程并重启你就会发现这样的问题，当我在终端上暴力的终止程序，使用 ctrl+shift+c 终端上进程确实停止运行了，然而我立刻再次运行则会报错：Traceback (most recent call last): File "/home/hspike/code/class/network/mylab/lab1/init_webserver.py", line 8, in \<module\> serverSocket.bind(server_address) OSError: [Errno 98] Address already in use

显然它的意思是我采用固定的 IP 和端口号仍在被占用，因为我暴力的终止导致了没有 close 导致了这个结果。但是**一段时间后不改变代码中的 IP 与端口号就能够正常运行了**，应该是操作系统介入释放了这个端口。

当你暴力终止程序（如按下 Ctrl+Shift+C），程序会立即停止运行，导致套接字没有被正确关闭。此时：**操作系统会将套接字标记为 TIME_WAIT 状态**。在 TIME_WAIT 状态下，操作系统会暂时保留该端口，以确保之前的连接完全关闭，避免数据包的延迟传输影响新连接。

操作系统会在一段时间后（通常是 30 秒到 2 分钟，具体取决于系统配置）自动释放端口。
这是 TCP 协议的机制，用于确保旧连接的数据包不会干扰新连接。

为了避免暴力终止程序导致资源未释放，可以捕获终止信号（如 SIGINT），并在程序退出前正确关闭套接字。也就是说**注册一个信号处理函数**。在 ICS 课上讲过信号处理异常控制流的机制，这里不再赘述，只展示 python 语法。注意需要 ```signal``` 库。

``` python
import signal

def graceful_shutdown(signum, frame):
    print("Shutting down server...")
    serverSocket.close()
    sys.exit(0)

signal.signal(signal.SIGINT, graceful_shutdown)
```

这样我们就优雅的终止进程了。

## Client 实现

Client 的实现就相对简单许多了，仿照 Server 的语法，代码如下：

```python
from socket import *
import sys
import signal

clientSocket = socket(AF_INET,SOCK_STREAM)
clientSocket.connect(('localhost',8080))

message = 'GET /index.html HTTP/1.1\r\nHost: localhost\r\n\r\n'
clientSocket.send(message.encode())
response = b""
while True:
    part = clientSocket.recv(4096)
    if not part:
        break
    response += part
print(response.decode())
```

我们不再做详细的讲解。

## Server 中断连接处理

这里提一个遇到的问题，在初版的 Client 我是这样写的：

```python
from socket import *
import sys
import signal

clientSocket = socket(AF_INET,SOCK_STREAM)
clientSocket.connect(('localhost',8080))

message = 'GET /index.html HTTP/1.1\r\nHost: localhost\r\n\r\n'
clientSocket.send(message.encode())
response = clientSocket.recv(4096)
print(response.decode())
```

我认为本地测试的 html 文件非常小，只用接收一次便可以了，然而运行结果出乎意料，报文竟然只到达了以下部分：

```html
HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html>
<!--STATUS OK--><htm
```

并且切换到服务器端发现服务器进程已经终止了，核心原因是**客户端只接受了很少部分报文便关闭连接，导致服务器 Send 函数抛出异常，进而导致了程序终止**。

显然客户端因为各种各样的原因中断连接很常见，不能客户端断了服务器就停摆吧，增加一个中断连接的异常处理。

```python
    except(ConnectionResetError, BrokenPipeError):
        # 这些异常表示用户中断了连接
        print("Connection with client was reset or closed.")
        connectionSocket.close()
```

这样就优雅的处理了这个问题。

但这里还有一个隐含的问题应当注意，代码的 4096 表示的是 4096 bytes，足足 4K。你应当意识到了问题，4K 表示一个纯文本的 html 文件错错有余，怎么会没有接收完呢？

原因在于**HTTP 相应报文的分块传输**，客户端只调用一次接收，所以并没有完整的接收到整个相应。

## To be done

这里完成了本地基础 C/S 交互，或许你意识到了这种方法借助于 TCP 提供的服务，甚至实现了**进程间的通信**。

进一步改进应当有两点：

1. 服务器端建立连接交由**子进程**处理：手段在 ICS 中介绍过，明天我会进一步实现这个功能。
2. 主机之间通信测试：暂时手上只有一个主机，明天会用另一台主机作为客户端，参与测试。