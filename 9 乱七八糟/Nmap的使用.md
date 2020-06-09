# Nmap的使用

Nmap包含四项基本功能：主机发现 (Host Discovery)、端口扫描 (Port Scanning)、版本侦测 (Version Detection)、操作系统侦测 (Operating System Detection)。另外nmap还提供强大的NSE(Nmap  Scripting Language)脚本引擎功能，脚本可以对基本功能进行补充和扩展。



## 主机发现

主机发现的原理与Ping命令类似：发送探测包到目标主机，如果收到回复则说明目标主机是开启的。Nmap支持十多种不同的主机探测方式，用户可以在不同的条件下灵活选用不同的方式来探测目标机。主机发现常用参数如下：

```
-sn: Ping Scan。进行主机发现，不进行端口扫描

-PO [protocollist]: 使用IP协议包探测对方主机是否开启
-PE/PP/PM: 使用ICMP echo、ICMP time stamp、ICMP address mask请求包方式进行发现主机
-PS/PA/PU/PY [portlist]: 使用TCP SYN/ACK或SCTP INIT/ECHO请求包方式进行发现主机
```

+ `nmap -sn <ip>`：在探测公网IP地址时，会依次发送四种不同类型的数据包(ICMP echo request、TCP SYN packet to port 443(https)、TCP ACK packet to port 80(http)、ICMP timestamp request)来探测目标主机是否在线，只要收到其中一个包的回复则说明目标机开启。使用四种不同类型的数据包可以避免因防火墙或丢包造成的判断错误；在探测内网IP地址时，会发送ARP请求包探测目标ip是否在线，如果有ARP回复包则说明在线。如果在线还可以得到其MAC地址，但是不会探测其开放的端口号。
+ `nmap -PE <ip>`：在探测公网IP地址时，ICMP Echo扫描简单来说是通过向目标发送ICMP Echo数据包来探测目标主机是否存活，但由于许多主机的防火墙会禁止这些报文，所以仅仅ICMP扫描通常是不够的；在探测内网IP地址时，会发送ARP请求包探测目标ip是否在线，如果有ARP回复包则说明在线。如果在线还会探测主机的端口的开启状态以及运行的服务。
+ `nmap -PP <ip>`：在探测公网IP地址时，ICMP time stamp时间戳扫描在大多数防火墙配置不当时可能会得到回复，可以此方式来判断目标主机是否存活。倘若目标主机在线，该命令还会探测其开放的端口以及运行的服务；在探测内网IP地址时，会发送ARP请求包探测目标ip是否在线，如果有ARP回复包则说明在线。如果在线还会探测主机的端口的开启状态以及运行的服务。
+ `nmap -PM <ip>`：在探测公网IP地址时，ICMP address mask地址掩码扫描会试图用备选的ICMP等级Ping指定主机，通常有不错穿透防火墙效果；在探测内网IP地址时，会发送ARP请求包探测目标ip是否在线，如果有ARP回复包则说明在线。如果在线还会探测主机的端口的开启状态以及运行的服务。



## 端口扫描

端口扫描是Nmap最基本最核心的功能，用于确定目标主机的TCP/UDP端口的开放情况。默认情况下Nmap会扫描1000个最有可能开放的TCP端口。Nmap通过探测将端口划分为6个状态：open、closed、filtered、unfiltered、open|filtered、closed|filtered 。端口扫描常用参数如下：

```
-T4： 指定扫描过程使用的时序，总有6个级别(0-5)，级别越高扫描速度越快，但也容易被防火墙或IDS检测并屏蔽掉，在网络通讯状况较好的情况下推荐使用T4
-A ：选项用于使用进攻性方式扫描

-p <port ranges>: 扫描指定的端口，如没指定则会按照nmap-services文件中指定的端口进行扫描。

-sS/sA/sT: 指定使用TCP SYN/ACK/Connect()的方式来对目标主机进行扫描
-sF/sX/sN/: 指定使用TCP FIN/Xmas/Null scans秘密扫描方式来协助探测对方的TCP端口状态
-sU: 指定使用UDP扫描方式确定目标主机的UDP端口状况

-script <script name>: 指定扫描脚本
```

+ 端口指定(-p) ：要既扫描TCP又扫描UDP，则需要指定-sU及至少一个TCP扫描类型(-sS或-sT等），如果没给定协议限定符，端口号会被加到所有协议列表。如-p U:53,111,137,T:21-25,80,139,8080,S:9 (其中U代表UDP协议、T代表TCP协议、S代表SCTP协议)。
+ TCP SYN 扫描(-sS)：TCP SYN方式向目标主机的端口发送SYN包，如果收到SYN/ACK回复，则可判断端口是开放的；如果收到RST包，说明该端口是关闭的。如果没收到回复，可判断该端口被屏蔽。因该方式仅发送SYN包对目标主机的特定端口，不建立完整的TCP连接，所以相对比较隐蔽且效率高、适用范围广。**这是Nmap默认的扫描方式，通常被称作半连接扫描**。
+ TCP ACK 扫描(-sA)：TCP ACK方式向目标主机的端口发送ACK包，如果收到RST包，则该端口没有被防火墙屏蔽；没有收到RST包，则说明被屏蔽。该方式只能用于确定防火墙是否屏蔽某个端口，可以辅助TCP SYN的方式来判断目标主机防火墙的状况。
+ TCP connent 扫描(-sT)：TCP connect方式使用系统网络API向目标主机的端口发起连接，如无法连接，说明该端口关闭。该方式由于建立完整的TCP连接会在目标主机上留下记录信息且扫描速度比较慢，所以TCP connect是TCP SYN无法使用才考虑使用的方式。**这是Nmap补充的扫描方式，通常被称作全连接扫描**。
+ TCP FIN/Xmas/NULL 扫描(-sF/sX/sN)：因相对比较隐蔽，这三种扫描方式被称为秘密扫描。FIN扫描向目标主机的端口发送的TCP FIN包或Xmas tree包或NULL包，如果收到对方的RST回复包，则说明该端口是关闭的；没有收到RST包，则说明该端口可能是开放或被屏蔽了。其中Xmas tree包是指flags中FIN URG PUSH被置为1的TCP包；NULL包是指所有的flags都为0的TCP包。
+ UDP 扫描(-sU)：UDP扫描用于判断UDP端口的情况，向目标主机的UDP端口发送探测包，如果收到回复ICMP port unreachable，则说明该端口是关闭的；如没有收到回复，则说明该UDP端口可能是开放的或者屏蔽的。因此通过反向排除法的方式来判断哪些UDP端口是可能处于开放状态的。



## 版本侦测

Nmap提供的版本侦测具有如下的主要优点：如果检测到SSL则会调用openSSL继续侦测运行在SSL上的具体协议，如HTTPS/POP3S/IMAPS等；具有广泛的应用程序数据库，可识别几千种服务的签名，包含了180多种不同的协议等。具体使用参数如下：

```
-sV: 指定让Nmap进行版本侦测，如nmap -sV 10.96.10.246

--version-all: 尝试使用所有的probes进行侦测(intensity 9)
--version-light: 指定使用轻量侦测方式(intensity 2)
--version-intensity <level>: 指定版本侦测强度（0-9），默认为7。数值越高，探测出的服务越准确，但是运行时间会比较长。
```

版本侦测主要分为以下几个步骤：

+ 首先检查open与open|filtered状态的端口是否在排除端口列表内。如果在排除列表则将该端口剔除
+ 如果是UDP端口则直接使用nmap-services-probes中探测包进行探测匹配。根据结果对比分析出UDP应用服务类型。
+ 如果是TCP端口则尝试建立TCP连接。尝试等待片刻(通常6秒或更多)。通常在等待时间内会接收到目标机发送的“WelcomeBanner”信息：
  + nmap将接收到的Banner与nmap-services-probes中NULL probe中的签名进行对比，成功查找对应应用程序的名字与版本信息，然后返回。
  + nmap将接收到的Banner与nmap-services-probes中NULL probe中的签名进行对比，无法确定应用程序版本，nmap再尝试发送其他的探测包(即从nmap-services-probes中挑选合适的probe)，将probe得到回复包与数据库中的签名进行对比。
  + 如果反复探测都无法得出具体应用，那么打印出应用返回报文，让用户自行进一步判定。
+ 如果探测到应用程序是SSL，则调用openSSL进一步的侦查运行在SSL之上的具体的应用类型。
+ 如果探测到应用程序是SunRPC，则调用brute-force RPC grinder进一步探测具体服务。



## 系统侦测

Nmap拥有丰富的系统数据库nmap-os-db，目前可以识别2600多种操作系统与设备类型。具体使用参数如下：

```
-O: 指定Nmap进行OS侦测，如nmap -O 10.96.10.246

--osscan-limit: 限制Nmap只对确定的主机的进行OS探测(至少需确知该主机分别有一个open和closed的端口)
--osscan-guess: 大胆猜测对方的主机的系统类型。由此准确性会下降不少，但会尽可能多为用户提供潜在的操作系统
```

Nmap使用TCP/IP协议栈指纹来识别不同的操作系统和设备。在RFC规范中，有些地方对TCP/IP的实现并没有强制规定，由此不同的TCP/IP方案中可能都有自己的特定方式。Nmap主要是根据这些细节上的差异来判断操作系统的类型的。具体实现方式如下：

+ Nmap内部包含2600多已知系统的指纹特征(在文件nmap-os-db文件中)。将此指纹数据库作为进行指纹对比的样本库
+ 分别挑选一个open和closed的端口，向其发送经过精心设计的TCP/UDP/ICMP数据包，根据返回的数据包生成一份系统指纹
+ 将探测生成的指纹与nmap-os-db中指纹进行对比来查找匹配的系统。如果无法匹配则以概率形式列举出可能的系统



## 高级用法

防火墙与IDS规避为用于绕开防火墙与IDS的检测与屏蔽，以便能够更加详细地发现目标主机的状况。nmap提供了多种规避技巧通常可以从两个方面考虑规避方式：数据包的变换(Packet Change)和时序变换(Timing Change)

```
--mtu <val>: 指定使用分片、指定数据包的MTU
-g/--source-port <portnum>: 使用指定源端口
-e <iface>: 使用特定的网络接口
-S <IP_Address>: 伪装成其他IP地址
-D <decoy1,decoy2[,ME],...>: 用一组IP地址掩盖真实地址，其中ME填入自己的IP地址
--spoof-mac <mac address/prefix/vendor name>: 伪装MAC地址
--ip-options <options>: 使用指定的IP选项来发送数据包
--data-length <num>: 填充随机数据让数据包长度达到Num
--badsum: 使用错误的checksum来发送数据包(正常情况下，该类数据包被抛弃，如收到回复则说明回复来自防火墙或 IDS/IPS)
```

如`nmap -F -Pn -D 10.96.10.100,10.96.10.110,ME -e eth0 -g 5555 202.207.236.3`。具体用到技术如下：

+ 分片：将可疑的探测包进行分片处理(例如将TCP包拆分成多个IP包发送过去)，某些简单的防火墙为了加快处理速度可能不会进行重组检查，以此避开其检查
+ 指定源端口：某些目标主机只允许来自特定端口的数据包通过防火墙。例如FTP服务器的配置为允许源端口为21号的TCP包通过防火墙与FTP服务器通信，但是源端口为其他的数据包被屏蔽。所以在此类情况下，可以指定数据包的源端口
+ 扫描延时：某些防火墙针对发送过于频繁的数据包会进行严格的侦查，而且某些系统限制错误报文产生的频率。所以可以降低发包的频率和发包延时以此降低目标主机的审查强度
+ IP伪装：将自己发送的数据包中的IP地址伪装成其他主机的地址，从而目标机认为是其他主机与之通信。需要注意的是，如果希望接收到目标主机的回复包，那么伪装的IP需要位于统一局域网内。另外，如果既希望隐蔽自己的IP地址，又希望收到目标主机的回复包，那么可以尝试使用idle scan或匿名代理等网络技术
+ IP诱骗：在进行扫描时，将真实IP地址在和其他主机的IP地址混合使用(其他主机需要在线，否则目标主机将回复大量数据包到不存在的数主机，从而实质构成了DOS攻击)，以此让目标主机的防火墙或IDS追踪大量的不同IP地址的数据包，降低其追查到自身的概率。但是某些高级的IDS系统通过统计分析仍然可以追踪出扫描者真实的IP地址
+ 其他技术：nmap还提供其他多种规避技巧，如指定发送包的最小长度、指定发包的MTU、指定TTL、指定伪装的MAC地址，使用错误检查等



## 脚本引擎

NSE脚本引擎(Nmap Scripting Engine)是nmap最强大、最灵活的功能之一，允许用户自己编写脚本来执行自动化的操作或者扩展nmap的功能。nmap的脚本库的路径：`/usr/share/nmap/scripts`或`/xx/nmap/scripts/` ，该目录下的文件都是nse脚本。Nmap的脚本主要分为以下几类：

+ Default：使用-sC或-A选项扫描时默认的脚本，提供基本的脚本扫描能力
+ Discovery：对网络进行更多的信息搜集，如SMB枚举，SNMP查询等
+ External：利用第三方的数据库或资源 。如进行whois解析
+ Version：负责增强服务与版本扫描功能的脚本
+ Auth：负责处理鉴权证书(绕过鉴权)的脚本
+ Fuzzer：模糊测试脚本，发送异常的包到目标机，探测出潜在漏洞
+ Brute：针对常见的应用提供暴力破解方式，如HTTP/HTTPS
+ Dos：用于进行拒绝服务攻击
+ Exploit：利用已知的漏洞入侵系统
+ Intrusive：入侵性的脚本，此类脚本可能引发对方的IDS/IPS的记录或屏蔽
+ Safe：与Intrusive相反，属于安全性脚本
+ Malware：探测目标是否感染了病毒，开启后门等
+ Vuln：负责检查目标机是否有常见漏洞，如MS08-067

```
例如：
nmap -p 443 -script ssl-ccs-injection 192.168.10.34   #验证是否存在openssl CCS注入漏洞
nmap --max-parallelism 800 --script http-slowloris scanme.nmap.org  #可以探测该主机是否存在http拒绝服务攻击漏洞

nmap --script-vuln  192.168.1.1     #扫描是否有常见漏洞
nmap --script-brute 192.168.1.1     #nmap可对数据库、SMB、SNMP等进行简单密码的暴力破解
nmap -script smb-vuln-ms17-010 192.168.10.34          #可以探测该主机是否存在ms17_010漏洞
nmap -script mysql-empty-password 192.168.10.34       #验证mysql匿名访问
nmap -script http-iis-short-name-brute 192.168.10.34  #探测是否存在IIS短文件名漏洞
```









