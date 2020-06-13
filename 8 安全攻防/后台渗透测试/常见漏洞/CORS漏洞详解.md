# CORS漏洞详解

跨域资源共享CORS(Cross-Origin Resource Sharing)，是HTML5的一个新特性，已被所有浏览器支持，不同于古老的JSONP只能get请求。

## 流程

https://www.cnblogs.com/lailailai/p/4528092.html

## 挖掘

CORS的漏洞主要看当我们发起的请求中带`Origin`头部字段时，服务器的返回包带有CORS的相关字段并且允许`Origin`的域访问。github上提供了一个关于扫描CORS配置漏洞的脚本`https://github.com/chenjj/CORScanner`。