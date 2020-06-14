# CORS漏洞详解

跨域资源共享CORS(Cross-Origin Resource Sharing)，是HTML5的一个新特性，已被所有浏览器支持，不同于古老的JSONP只能get请求。

## 背景

同源策略SOP(Same Origin Policy)是一种约定，它是浏览器最核心也最基本的安全功能。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现，**所谓同源是指域名、协议、端口相同**。

> 如果说SOP就是“禁止跨域请求”是不对的，**本质上SOP并不是禁止跨域请求，而是在请求后拦截了请求的回应**，所以CSRF不受同源策略限制。
>
> 不受同源策略限制的有：第一种是重定向、页面中的链接、页面中的表单提交是不会受到同源策略限制的。第二种是跨域资源的引入是可以的，但js不能读写加载的内容，如嵌入到页面中的`<script src="..."></script>`、`<img>`、`<link>`、`<iframe>`等。

受上面所讲的浏览器同源策略的影响，不是同源的脚本不能操作其他源下面的对象。想要操作另一个源下的对象是就需要跨域。跨域的实现方法有：降域`document.domain`、CORS跨域资源共享。

CORS是一个W3C标准，是一种放宽同源策略的机制，它允许浏览器向跨源服务器发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制，以使不同的网站可以跨域获取数据。CORS跨域获取资源的过程分两种：

### 简单跨域请求

简单跨域请求它有两个条件，只有同时满足以下两个条件时才是简单请求，否则为非简单请求：

+ 请求方法是三种方法之一：HEAD、GET、POST
+ HTTP的头信息不超出以下几种字段：`Accept`、`Accept-Language`、`Content-Language`、`Content-Type：application/x-www-form-urlencoded、 multipart/form-data、text/plain`、`Last-Event-ID`。

同时浏览器判断该请求为简单请求时，会在Request Heade中添加`Origin`字段，它表示我们的请求源。服务端在接收这个请求后，根据Origin这个字段的值，判断本次的请求的源是否在许可范围内。如果Origin指定的源不在许可范围内(即验证失败)会返回一个正常的HTTP回应，浏览器发现这个回应的头信息中的Access-Control-Allow-Origin字段不包含访问源就知道出错了，从而抛出同源检测异常的错误(注意这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200)。如果Origin指定的源在许可范围内(即验证通过)，服务端会在Response Header添加下面几个字段：

+ Access-Control-Allow-Origin：该字段是必须的。它要么是请求时`Origin`字段的值，要么是一个`*`(表示接受任意域名的请求)
+ Access-Control-Expose-Headers：该字段是可选的。CORS请求时，XMLHttpRequest对象的getResponseHeader方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段就必须在Access-Control-Expose-Headers里面指定
+ Access-Control-Allow-Credentials：该字段是可选的。它是一个布尔值，代表服务器是否允许客户端发送cookie。如果这个字段设置为true，那么跨域请求时cookie可以被包含在请求中。如果不允许客户端发送cookie，删除这个字段即可

### 非简单跨域请求

非简单跨域请求是那种对服务器有特殊要求的请求，比如请求方法是PUT或DELET，或`Content-Type`字段的类型是`application/json`。非简单请求的CORS请求，会在正式通信之前增加一次OPTIONS方法的预检请求。

浏览器先发送一个OPTIONS方法的预检请求(带有如下字段)：

+ Origin: 在CORS中专门作为Origin信息供后端比对，表明来源域
+ Access-Control-Request-Method: 接下来请求的方法，如PUT、DELETE
+ Access-Control-Request-Headers: 自定义的头部，所有用setRequestHeader方法设置的头部都将会以逗号隔开的形式包含在这个头中

然后如果服务器配置了CORS，会返回对应对的字段：

+ Access-Control-Allow-Origin:  允许进行跨域请求的域名
+ Access-Control-Allow-Methods: 允许进行跨域请求的方式
+ Access-Control-Allow-Headers: 允许进行跨区请求的头部

浏览器再根据上述响应判断是否发送非简单请求。发送完成后，服务器处理请求，会在返回结果中加上如下控制字段：

+ Access-Control-Allow-Origin: 允许跨域访问的域，可以是一个域的列表，也可以是通配符
+ Access-Control-Allow-Methods: 允许使用的请求方法，以逗号隔开
+ Access-Control-Allow-Headers: 允许自定义的头部，以逗号隔开且大小写不敏感
+ Access-Control-Allow-Credentials: 是否允许请求带有验证信息
+ Access-Control-Expose-Headers: 允许脚本访问的返回头，请求成功后，脚本可以在XMLHttpRequest中访问这些头的信息
+ Access-Control-Max-Age: 缓存此次请求的秒数。在这个时间范围内，所有同类型的请求都将不再发送预检请求而是直接使用此次返回的头作为判断依据，以此大幅优化请求次数

然后浏览器通过返回结果的这些控制字段来决定是将结果开放给客户端脚本读取还是屏蔽掉。如果服务器没有配置CORS，返回结果没有控制字段，浏览器会屏蔽脚本对返回信息的读取，并报出同源检测异常的错误！

## 流程

CORS完全是一个盲目的协议，只是通过HTTP头来控制的。那么CORS跨域资源共享漏洞是怎么发生的呢？由于程序员配置不当，Origin源不严格，从而造成跨域问题。

+ CORS服务端的Access-Control-Allow-Origin设置为了`*`，且Access-Control-Allow-Credentials设置为false，这样任何网站都可以获取该服务端的任何数据。
+ 有些网站的Access-Control-Allow-Origin的设置并不是固定的，而是根据用户跨域请求数据的Origin来定的。这时不管Access-Control-Allow-Credentials设置为了true还是false，任何网站都可以发起请求，并读取对这些请求的响应。意思就是任何一个网站都可以发送跨域请求来获得CORS服务端上的数据。

## 挖掘

CORS的漏洞主要看当我们发起的请求中带`Origin`头部字段时，服务器的返回包带有CORS的相关字段并且允许`Origin`的域访问。github上提供了一个关于扫描CORS配置漏洞的脚本`https://github.com/chenjj/CORScanner`。

