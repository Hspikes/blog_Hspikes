+++
date = '2025-09-04T12:28:00+08:00'
draft = true
title = 'lab1*: 基础CS文件传输'
katex = true
+++

对实验一的纯粹的实验性质程序加“一点点”小功能，可以更好的把玩一下。包含文件路径处理、多进程处理一系列简单改造。

## 支持不同文件类型

服务器现阶段返回的固定是 HTML 文件，改造成根据客户端的请求返回目录下的任意文件。

使用 Python 自带的 ```mimetypes``` 模块（Python 神了），根据文件扩展名自动推断 MIME 类型。在读取文件时不再使用 ```read``` 来读，不能正确处理二进制文件，换成 ```open```。发送的时候也应当避免使用 ```encode```，否则会损坏二进制数据。

改造后的文件处理部分：

```python
    content_type, _ = mimetypes.guess_type(filepath)
    if content_type is None:
        content_type = "application/octet-stream"  # 默认二进制

    # 读取文件（二进制）
    with open(filepath, "rb") as f:
        outputdata = f.read()
    ...
    header += f"Content-Type: {content_type}\r\n"
    ...
    connectionSocket.send(outputdata)
```

## 安全性

现阶段的文件访问没有做任何特别的处理，也就是说如果我在服务器上层放一张图片 HTTP.png，```http://localhost:8080/../HTTP.png```就可以直接访问到这个文件。这好像没什么但如果我是 ```/../../etc/passwd``` 你不炸了吗？

### 浏览器路径规范化与错误请求

但我在用浏览器测试的时候先发现这样一个问题，确实我发现了我访问 ```http://localhost:8080/../HTTP.png``` 但浏览器地址栏却是 ```http://localhost:8080/HTTP.png``` 。并且发现服务器端报出 ```Unexpected error: list index out of range``` 的错误，这涉及浏览器的一个机制：**浏览器的路径规范化**。

```localhost:8080/../HTTP.png``` 时，浏览器会**路径归一化(Normalization)**。```/../HTTP.png``` 会被规整为 ```/HTTP.png```，这是浏览器的“用户体验逻辑”，目的是防止用户在地址栏看到奇怪的 ```..```，也避免和文件系统“上级目录”混淆。这时你打出报文就会发现收到的报文其实是 ```GET /HTTP.png HTTP/1.1``` 。

那么为什么会报出 ```list index out of range``` 呢？能报这个错误的代码应当只有一句 ```filename = message.split()[1]```。 而在访问 ```/../HTTP.png``` 时，浏览器在地址栏输入一个“特殊 URL”时，可能会先发一次探测请求（甚至是异常格式，比如只发 ```GET```，没有路径），所以服务器就会解析崩溃，加上合理的健壮性判断即可。

```python
parts = message.split()
if len(parts) < 2:
    # 非法请求，直接返回 400 Bad Request
    header = "HTTP/1.1 400 Bad Request\r\n\r\n"
    connectionSocket.send(header.encode())
    connectionSocket.close()
    continue

filename = parts[1]
```

至此我就不再打算用浏览器测试了，浏览器自己有很多隐含的机制，我又不是在测试浏览器。我用自己写的客户端试了试，确实服务器可以收到 ```Request message: GET /../index.html HTTP/1.1``` 并且成功返回，下面我们就解决这个问题。

### 目录规范

HTTP 请求里的路径本质就是一个字符串，**目录穿越漏洞（Directory Traversal）**，真实 Web 服务器都会防御。关键在于把 URL 里的路径归一化（处理 ```.、..```、重复斜杠等情况）并且确保归一化后的路径仍然在你的根目录（document root）之下。

我们直接导入 GPT 给出的处理方案，并简单讲解。

```python
import os
from urllib.parse import unquote

# 定义允许访问的根目录（示例）
WEB_ROOT = os.path.abspath("./source")

def safe_path(request_path: str) -> str | None:
    # 浏览器或客户端可能把 .. 编码成 %2e%2e。在解析前应把百分号编码解回来再处理。
    request_path = unquote(request_path)
    # 去掉开头的 "/"
    rel_path = request_path.lstrip("/")
    # 归一化路径（会处理 .. 和 . 以及多余的斜杠）
    normalized = os.path.normpath(rel_path)
    # 拼接成绝对路径
    abs_path = os.path.abspath(os.path.join(WEB_ROOT, normalized))
    # 检查是否仍然在 WEB_ROOT 内
    if os.path.commonpath([WEB_ROOT, abs_path]) == WEB_ROOT:
        return abs_path
    else:
        return None
    
    filepath = safe_path(filename)
    if filepath is None:
        header = "HTTP/1.1 403 Forbidden\r\n\r\n"
        connectionSocket.send(header.encode())
        connectionSocket.close()
        return
```

注意 Python 的函数语法，还是比较别致的。```normpath``` 不会把相对路径“丢掉”开头的 ```..```，只合并冗余，而 ```os.path.abspath(...)``` 会解析 ```..```。最后再判断解析完成后的路径是否合规即可。

## 多进程服务器

这里差评，直接询问 GPT Python 里面启动多进程的方式，GPT 竟然不给出进程回收的机制，问他他还说我问的专业 ~~(也是把我夸的心花怒放了，感谢 ICS 课)~~。

和 ICS 课上的机制几乎一致，```fork``` 再子进程父进程分开处理。

```python
def handle_client(connectionSocket, addr):
    try:
        message = connectionSocket.recv(2048).decode()
        print("Request message:", message)
        # … 这里放你之前的文件处理逻辑 …
        connectionSocket.close()
    except Exception as e:
        print("Error:", e)
        connectionSocket.close()

signal.signal(signal.SIGCHLD, signal.SIG_IGN)

...

    connectionSocket, addr = serverSocket.accept()
    pid = os.fork()
    if pid == 0:  # 子进程
        serverSocket.close()  # 子进程里不用监听 socket
        handle_client(connectionSocket, addr)
        sys.exit(0)           # 子进程结束
    else:  # 父进程
        connectionSocket.close()  # 父进程里不用这个连接
```

至于回收子进程，我学到一个以前课上完全没提到过的机制，**自动回收**。

直接注册信号 ```signal.signal(signal.SIGCHLD, signal.SIG_IGN)```，忽略这个信号。如果父进程把 ```SIGCHLD``` 设置成忽略，Linux 内核就认为：“既然你不关心子进程的退出状态，我就不需要给你留着了。”于是，子进程退出时，内核会立刻自动回收，不会留下僵尸进程。

这个我仔细询问了 GPT，查阅了一些资料放置大模型幻觉，**是真的**！这个是现在 Linux 内核的惯例的处理，而不是 POSIX 的强制标准，在 ICS 的课上都没有讲过这个机制。

这里我们直接就用了这个机制而没有注册一个回收函数，即通过传统的 ```waitpid``` 的方式来进行回收，这种方式确实非常简便，但是就父进程失去了从子进程处得到信息并记录日志的机会，我们这里就不做的太复杂就这样了。

至此我们基本完成了服务器的改造，可以完成基本的文件传输功能。下面我们对客户端端做出配套的改造。

## 配套客户端

我们需要的效果是输入一个文件，直接到对应的服务器下去下载目标文件，那么必须要有一个输入。然后我们要将服务器发回的报文分段，将 Head 分离出来解读，如果报文正常，将 Body 部分存储下来。

我们直接采用命令行的输入方式，即把请求的文件直接作为脚本运行时的参数代入，Python 没有 main 函数的设计还是十分诡异。参数直接存储在变量 ```sys.argv``` 中，显然这是一个参数列表。

```python
if len(sys.argv) < 2:
    print(f"Usage: python3 {sys.argv[0]} <filename>")
    sys.exit(1)

filename = sys.argv[1]
```

按照先前的方式接收报文，在收到报文之后重要的就是要对报文进行分割解读。根据 HTTP 协议的标准格式，只需要找到两个连续的换行符即可把 Head 和 Body 分割出来。

```python
separator = b'\r\n\r\n'
header_end = response.find(separator)
if header_end == -1:
    print("Invalid HTTP response")
    sys.exit(1)

header_bytes = response[:header_end].decode(errors='replace')
body = response[header_end + 4:]
```

头部可以直接解码判断，输出在终端上判断情况。

```python
print("===HTTP Header===")
print(header_bytes)
```

如果头部表明文件正常传输了，那么我们就根据头部存储文件即可，否则报出相应的错误。

```python
content_type = "application/octet-stream"
for line in header_bytes.split("\r\n"):
    if line.lower().startswith("content-type:"):
        content_type = line.split(":", 1)[1].strip()
        break

status_line = header_bytes.split("\r\n")[0]
if not status_line.startswith("HTTP/1.1 200"):
    print("Request failed:", status_line)
    sys.exit(1)

with open(filename, "wb") as f:
    f.write(body)

print(f"Body saved to {filename}")
```

可以注意到文件的默认类型为 ```application/octet-stream``` 即二进制文件，然后根据首部行的内容分割出实际的文件类型，但其实我们这里不需要文件类型了，只是为了展示实际应当采用的分割策略。

接着就只用读到什么写入什么，注意判断一下异常即可。

```python
status_line = header_bytes.split("\r\n")[0]
if not status_line.startswith("HTTP/1.1 200"):
    print("Request failed:", status_line)
    sys.exit(1)

with open(filename, "wb") as f:
    f.write(body)

print(f"Body saved to {filename}")
```

## To be done

至此基本就能实现一个简单的文件传输了，但是实际测试就会发现：**传输效率实在不高**。一个 200 MB 的视频文件几乎传了 20 min 才完成，核心是收发包的过程很多处理很低效。如果以后还有机会的话就调一调这一部分。