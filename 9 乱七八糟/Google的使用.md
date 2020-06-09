# Google Hacking的使用

Google Hacking是利用谷歌搜索的强大，在浩瀚的互联网中搜索到我们需要的信息。轻量级的搜索可以搜素出一些遗留后门等；中量级的搜索出一些用户信息或源代码泄露等；重量级的则可能是mdb文件下载，网站配置密码，php远程文件包含漏洞等重要信息。

## 基本搜索

+ 通配符：`*`和`?`
+ 逻辑与：`+`，如`google + android`，搜索包含google且android的页面
+ 逻辑或：`|`，如`google | android`，搜索包含google或android的页面
+ 逻辑非：`-`，如`google -android`，搜索包含google且不包含android的页面

## 高级搜索

+ cache：将返回存储下来的历史页面。例如`cache:www.hackingspirits.com guest`将返回指定网站的缓存，并且正文中含有guest
+ info：返回指定站点的信息。例如`info:www.baidu.com`将返回百度的一些信息
+ site：指定访问的站点。例如`site:baidu.com inurl:Login`将只在baidu.com中查找url中含有Login的网页
+ (all)inurl：将返回url中含有关键词的网页。例如`inurl:Login`将返回url中含有Login的网页
+ (all)intitle：将返回标题中含有关键字的网页。例如`intitle:后台登录`将只返回标题中包含"后台登录"的网页
+ (all)intext：将返回正文中含有关键字的网页。例如`intext:后台登录`将只返回正文中包含"后台登录"的网页
+ filetype：指定访问的文件类型。例如`site:baidu.com filetype:pdf`将只返回baidu.com上文件类型为pdf的网页

而上面这些命令中用的最多的就是inurl，利用这个命令可以查到很多意想不到的东西。如查找SQL注入`inurl:?id=1`。

## 常见搜索

+ 查找网站后台
  + `site:xx.com inurl:login`
  + `site:xx.com intitle:后台`
  + `site:xx.com intext:管理`
+ 查看上传漏洞
  + `site:xx.com inurl:file`
  + `site:xx.com inurl:load`
+ 查看服务器使用的程序
  + `site:xx.com filetype:php`
  + `site:xx.com filetype:jsp`
  + `site:xx.com filetype:asp`
  + `site:xx.com filetype:aspx`

