# HTTP协议详解

## 同源策略

同源策略SOP(Same Origin Policy)是一种约定，它是浏览器最核心也最基本的安全功能。可以说Web是构建在同源策略基础之上的，浏览器只是针对同源策略的一种实现，**所谓同源是指域名、协议、端口相同**。

说SOP就是“禁止跨域请求”是不对的，**SOP并不是禁止跨域请求，而是在请求后拦截了请求的回应**。跨域资源的引入如`<script>`、页面中的链接、页面中的表单提交、重定向等是不会受到同源策略限制的。

CORS跨域资源共享就是实现跨域的一种方法，它获取资源的过程分两种：

### 简单跨域请求

简单跨域请求它有两个条件，只有同时满足以下两个条件时才是简单跨域请求，否则为非简单跨域请求：

+ 请求方法是三种方法之一：HEAD、GET、POST
+ HTTP的头信息不超出以下几种字段：`Accept`、`Accept-Language`、`Content-Language`、`Content-Type：application/x-www-form-urlencoded、 multipart/form-data、text/plain`、`Last-Event-ID`。

同时浏览器判断该请求为简单请求时，会在Request Heade中添加`Origin`字段，它表示我们的请求源。服务端在接收这个请求后，根据Origin这个字段的值，判断本次的请求的源是否在许可范围内。如果Origin指定的源不在许可范围内(即验证失败)会返回一个正常的HTTP回应，浏览器发现这个回应的头信息中的Access-Control-Allow-Origin字段不包含访问源就知道出错了，从而抛出同源检测异常的错误(注意这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200)。如果Origin指定的源在许可范围内(即验证通过)，服务端会在Response Header添加下面几个字段：

+ Access-Control-Allow-Origin：该字段是必须的。它要么是请求时`Origin`字段的值，要么是一个`*`(表示接受任意域名的请求)
+ Access-Control-Expose-Headers：该字段是可选的。CORS请求时，XMLHttpRequest对象的getResponseHeader方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段就必须在Access-Control-Expose-Headers里面指定
+ Access-Control-Allow-Credentials：该字段是可选的。它是一个布尔值，代表服务器是否允许客户端发送cookie。如果这个字段设置为true，那么跨域请求时cookie可以被包含在请求中。如不允许客户端发送cookie，删除该字段即可

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

然后浏览器通过返回结果的这些控制字段来决定是将结果开放给客户端脚本读取还是屏蔽掉。如果服务器没有配置CORS导致返回结果没有控制字段，浏览器会屏蔽脚本对返回信息的读取，并报出同源检测异常的错误。

更详细的请查看这里[《HTTP协议详解》](https://www.cnblogs.com/EricaMIN1987_IT/p/3837436.html)