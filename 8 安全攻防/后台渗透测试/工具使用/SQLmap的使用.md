# sqlmap的使用

sqlmap是一个自动化的SQL注入工具，其功能是扫描\发现并利用给定的URL进行SQL注入。它采用了以下5种独特的SQL注入技术：

+ T: Time-based blind SQL injection(基于时间延迟盲注)，即不能根据页面返回的内容判断任何信息，要用条件语句执行查看时间延迟语句是否已经执行(即页面返回时间是否增加)来判断
+ B: Boolean-based blind SQL injection(基于布尔型的盲注)，即可根据返回页面判断条件真假的注入
+ E: Error-based SQL injection(基于报错型的注入)，即页面会返回错误信息，或者把注入的语句的结果直接返回到页面中
+ U: UNION query SQL injection(基于联合查询的注入)，在可以使用Union的情况下注入
+ S: Stacked queries SQL injection(基于多语句查询的注入)，可以同时执行多条语句时的注入

sqlmap的强大的功能包括数据库指纹识别、数据库枚举、数据提取、访问目标文件系统，并在获取完全的操作权限时执行任意命令。



## 基本用法

```
-r: 从抓取的http的请求包中读取url进行注入测试
-u: 直接制定url进行注入测试
-p: 当有多个参数时指定注入测试的目标参数
    很多框架会使用URL重写技术，此时sqlmap就无法使用参数注入，但是可以在需要测试的参数后面加上*
    如：sqlmap -u "http://blog.spoock.com/2016/09/04*/sqli-bypass/"
--data=<"val">: 指定其post提交的数据
--cookie=<"val">: 当该网站需要登录时，需要指定cookie
--delay=3 --force-ssl: 当爆破HTTPS网站会出现超时的话，可以使用该参数组合

--current-db: 查看当前的数据库
--current-user: 查看数据库当前的用户
--users: 查看数据库的所有用户
--passwords: 查看数据库用户名的密码
--is-dba: 判断当前用户是否有管理员权限
--roles: 列出数据库所有管理员角色

-D/-T/-C: 指定爆出内容的限定范围(数据库/数据表/数据列)
--dbs: 爆出所有的数据库
--tables: 爆出限定范围下所有的数据表，限定范围是无限制、某数据库
--columns: 爆出限定范围下数据库中所有的列，限定范围是无限制、某数据库下的某数据表
--count: 获取某数据库下数据表或者某数据表下数据的个数
--dump-all: 爆出限定范围下的所有数据，限定范围可以是无限制、某数据库、某数据库下的某表
--dump: 爆出限定数据列的所有数据，可以用--start 1 --stop 100进一步限定范围
```

有时候使用 --passwords 不能获取到密码，当MySQL< 5.7时可以用`-D mysql -T user -C host,user,password --dump`，当MySQL>= 5.7时可以用`-D mysql -T user -C host,user,authentication_string --dump `。



## 高级用法

Sqlmap一共有5个探测等级(默认是1)。等级越高说明探测时使用的payload也越多。其中5级的payload最多，会自动破解出cookie、XFF等头部注入。这个参数会影响测试的注入点，GET和POST的数据都会进行测试，cookie在level为2时就会测试，User-Agent/Referer头在level为3时就会测试。在不确定哪个参数为注入点时，为了保证准确性，建议设置level为5。Sqlmap一共有3个危险等级，也就是说你认为这个网站存在几级的危险等级。和探测等级一个意思，在不确定的情况下建议设置为3级。

Sqlmap使用的payload在目录主要位于sqlmap/xml/payloads目录下。

```
--level=5 --risk=3 #探测等级5，平台危险等级3，都是最高级别
```

Sqlmap在默认情况下除了使用CHAR()函数做了处理来防止出现单引号，其他的并没有对注入的数据进行修改。我们可以使用--tamper参数对数据进行修改来绕过WAF等设备，其中的大部分脚本主要用正则模块替换攻击载荷字符编码的方式尝试绕过WAF的检测规则

Sqlmap目前官方提供几十个绕过脚本，主要位于sqlmap/tamper目录下。

```
--identify-waf: 检测是否有WAF

# 使用参数进行绕过
--flush-session: 清空会话，重构注入
--time-sec=3: 使用长的延时来避免触发WAF的机制，这方式比较耗时
--keep-alive: 保持连接，当出现[CRITICAL] connection dropped or unknown HTTP status错的时候，使用这个参数
--user-agent: 指定user-agent的值
--random-agent: 使用任意HTTP头进行绕过，尤其是在WAF配置不当的时候。它随机化的从txt/user-agents.txt中获取
--referer: 伪造referer字段
--mobile: 对移动端的服务器进行注入
--hex/--no-cast: 进行字符码转换
--hpp: 使用HTTP参数污染进行绕过，尤其是在ASP.NET/IIS平台上

--tor: 匿名注入
--ignore-proxy: 禁止使用系统的代理，直接连接进行注入
--proxy=<val>/--proxy-cred=<val>: 使用代理进行绕过
--retries: 当请求超时时，设定重新尝试连接次数，默认是3次
--tiemout: 设定超时时间，主要是设定一个请求超过多久被判定为超时。若设定为10.5表示是10.5秒，默认是30秒
--delay: 设定两次请求间的时间间隔。若设定为0.5则表示间隔时间是半秒，默认是没有延迟

# 指定脚本进行绕过
--tamper=“space2comment.py”: 用/**/代替空格
--tamper="space2comment.py,space2plus.py": 指定多个脚本进行过滤
```

Sqlmap除了检测sql注入点以外，还可以在一定条件下与服务器进行交互：

```
--sql-shell: 执行指定的sql语句，执行中会提示我们输入要查询的SQL语句，注意这里的SQL语句最后不要有分号，最后按q退出
--os-shell: 执行shell命令，获取目标服务器权限。在数据库为MySQL、PostgreSql或者SQL Server时，可以执行该选项
--os-pwn: 执行pwn命令，将目标权限弹到MSF上。在数据库为MySQL、PostgreSql或者SQL Server时，可以执行该选项

# 当数据库为MySQL、PostgreSQL或SQL Server，并且当前用户有读写权限时，可以读写指定文件(可以是文本文件或者二进制文件)
--file-read <path>: 读取目标服务器某路径下的文件
--file-write <path>:写入到目标服务器某路径下的某文件中
--file-dest <path>: 将本地的某路径下文件上传到目标服务器，命名为file-write指定的文件名
```

