+++
date = '2025-09-12T11:46:00+08:00'
draft = false
title = 'lab3: 邮件客户端'
katex = true
+++

这个实验主要是巩固邮件服务器以及客户端原理，并简要的了解 STMP 协议。实验时一定要选对实验用的邮箱服务器，不然可能遇到一些诡异的问题。不推荐校园邮箱、QQ、Google，这些要么会报诡异的错误，要么需要 SSL。

## SMTP 协议

SMTP 协议是一个常见的 Email 应用的协议，在 TCP 上在服务器与客户端之间传输报文，端口号默认固定为 ```port = 25```，报文**主要以 ASCII 码的形式传递**。

因为报文主要以 ASCII 码的形式传递，无法表示一般的二进制文件，也无法表示中文，要传输这些无法以 ASCII 码表示的报文，需要通过 Base64 将二进制编码为 ASCII 码，再通过 MIME 协议传输，我们实验中没有完成这部分，高情商的表示为以后留下提升空间。但我们仍用到了 Base64 编码来进行用户认证。

SMTP 协议一个典型的客户端与服务器交流流程如下：

1. ```connect```: 客户端发出连接请求，服务器回复 ```220``` 表示建立连接。

2. ```HELO```: 客户端发送报文 ```HELO```，服务器回应 ```250 OK```。这一步是可以省略的，但大部分客户端实现仍然会打招呼，礼貌总是好的。

3. ```AUTH```: 现在邮件服务器都需要认证身份，客户端发送 ```AUTH LOGIN``` 表示开始认证身份，在收到服务器的指示之后分别向服务器发送用户名与认证密钥。在现有的 SMTP 协议中要求**用户名与密钥均要使用 Base64 编码**，否则无法正确识别，这是为了避免密钥中带有 ```\n```一类特殊转义字符的缘故。这个设计显得十分诡异，但是是协议要求的缘故，现有的实现均是如此。

4. From-To: 在完成认证后，告知服务器邮件发送方以及接收方，发送方需要与认证身份匹配，交流成功均会收到 250 确认码。

5. Message: 向服务器发送 ```DATA``` 表明开始发送邮件，邮件必须以头部开始，头部要包含 ```from, to, subject and Content-type```, 正文部分以 ```\r\n.\r\n``` 结尾。发送结尾信息后，邮件服务器会发送 ```250``` 确认码。

6. ```QUIT```: 客户端发送表明连接结束，我用于实验的 163 邮箱退出时一句 Bye 都没有，世间的冷漠莫过于此。

## 代码实现

```python
from socket import * 
import base64 

def check_recv(csock, expmsg):
    recv = csock.recv(1024).decode()
    print(recv)
    if(recv[:3] != expmsg):
        print (expmsg + ' reply not received from server.')
        exit(1)
    return

endmsg = "\r\n.\r\n"
mailserver = ("smtp.163.com",25)
fromaddr = "***@163.com"
rcptaddr = "***@qq.com"
username = base64.b64encode("***@163.com".encode()).decode()
password = base64.b64encode("***".encode()).decode()

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)

msg = "\r\n I love computer network!"

clientSocket.connect(mailserver)
check_recv(clientSocket, '220')

heloCommand = 'HELO smtp.163.com\r\n' 
clientSocket.send(heloCommand.encode()) 
check_recv(clientSocket, '250')


clientSocket.send("AUTH LOGIN\r\n".encode())
check_recv(clientSocket, '334')

clientSocket.sendall((username + '\r\n').encode())
check_recv(clientSocket, '334')
clientSocket.sendall((password + '\r\n').encode())
check_recv(clientSocket, '235')

clientSocket.send((f"MAIL FROM:<{fromaddr}>" + '\r\n').encode())
check_recv(clientSocket, '250')

clientSocket.send((f"RCPT TO:<{rcptaddr}>" + '\r\n').encode())
check_recv(clientSocket, '250')

clientSocket.send('DATA\r\n'.encode())
check_recv(clientSocket, '354')


message = "from: hspikes <****@163.com>\r\n"
message += "to: hspikes <****@qq.com>\r\n"
message += "subject: Hello\r\n"
message += "Content-Type: text/plain\t\n"
message += msg

clientSocket.send(message.encode())
clientSocket.send(endmsg.encode())

check_recv(clientSocket, '250')

clientSocket.send('QUIT'.encode())
clientSocket.close()
```

可以看到我们用一个函数包装了接收服务器消息的流程，避免了过于繁琐的 ```if```。代码没有什么很特别的地方，基本就是按照 SMTP 标准的流程走一趟，注意到用户名和密钥的编码，收发以及头部的格式就好了。