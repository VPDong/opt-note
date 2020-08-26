# CORS漏洞详解

跨域资源共享CORS(Cross-Origin Resource Sharing)，是HTML5的一个新特性，已被所有浏览器支持，不同于古老的JSONP只能get请求。

## 原理

发起一个请求时，request header里面有三个属性会涉及请求源信息，分别是Host、Referer、**Origin**。前端可能用不到这些值，但是后台业务系统会比较关心它们。

+ Host描述请求将被发送的目的地，包括域名和端口号。这个参数存在于任何类型请求中
+ Referer用来说明当前请求来源页面的地址(即表示当前页面是通过此来源页面里的链接进入的)，包括协议+域名+查询参数(注意不包含锚点信息)。这个参数适用于任何类型请求中。一般用来防盗链
+ **Origin**用来说明请求从哪里发起的，包括协议和域名。这个参数一般只存在于CORS跨域请求中

不是同源的脚本不能操作其他源下面的对象，想要操作另一个源下的对象是就需要CORS跨域。那么CORS跨域资源共享漏洞是怎么发生的呢？是因为由于对服务端配置不当，Origin源不严格，从而造成跨域问题：

+ CORS服务端的Access-Control-Allow-Origin设置为了`*`，且Access-Control-Allow-Credentials设置为false，这样任何网站都可以获取该服务端的任何数据。
+ 有些网站的Access-Control-Allow-Origin的设置并不是固定的，而是根据用户跨域请求数据的Origin来定的。这时不管Access-Control-Allow-Credentials设置为了true还是false，任何网站都可以发起请求，并读取对这些请求的响应

## 流程

利用方式一：发送钓鱼网站，流程上与CSRF很相似，区别在于发送请求使用的是JS(CSRF为HTML TAG)：

+ 假设用户登陆一个含有CORS配置网站`vuln.com`，同时又访问了攻击者提供的一个链接`evil.com`
+ `evil.com`的网站向`vuln.com`这个网站发起请求获取敏感数据，浏览器能否接收信息取决于`vuln.com`的配置
+ 如果`vuln.com`配置了`Access-Control-Allow-Origin`头且为预期，那么就会执行逻辑

利用方式二：如果origin为null，可以用iframe的沙箱技术`<iframe sandbox="allow-scripts allow-top-navigation allow-forms" src='data:text/html,<script>...</script>'></iframe>`

利用方式三：可与XSS漏洞结合，其威力更是无法想象的

## 挖掘

CORS的漏洞主要看当我们发起的请求中带`Origin`头部字段时，服务器的返回包带有CORS的相关字段并且允许`Origin`的域访问、或是null、或是`*`。github上提供了一个关于扫描CORS配置漏洞的脚本`https://github.com/chenjj/CORScanner`。

相关实践看这里[《CORS跨域漏洞的学习》](https://www.freebuf.com/column/194652.html)和[《如何利用CORS配置错误漏洞攻击比特币交易所》](https://xz.aliyun.com/t/2702)。