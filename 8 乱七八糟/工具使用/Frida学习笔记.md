# Frida学习笔记

Frida是一款轻量级HOOK框架，可用于android、ios、macos、windows等平台上。Frida通过其强大的仪器核心Gum(Gum是用C语言编写的)提供动态检测，因这种检测逻辑很容易发生变化，所以通常需要用脚本语言编写，这样在开发和维护它时会得到一个简短的反馈循环。在运行时内运行一段JavaScript，它可以完全访问Gum的API，允许读写内存、枚举加载库、遍历函数、挂钩函数等。

Frida分为两部分：一部分是运行在系统上的交互工具frida-client；一部分是运行在目标机器上的代码注入工具frida-serve，通过注入进程的方式来实现劫持应用函数。frida上层接口支持js、python、c等。



## 一、安装

下面以root过的安卓为例，通过`adb shell getprop ro.product.cpu.abi`获取abi型号`arm64-v8a`：

+ 安装frida-server：[下载](https://github.com/frida/frida/releases)运行在目标机上的frida-sever端(abi和version要对应)后，推送到`/data/local/tmp`后赋予执行权限运行
+ 安装frida-client：通过python安装frida-tools模块`pip install frida-tools`后，终端输入`frida-ps -U`查看手机进程



## 二、使用

### 2.1 hook java

```python
#!/usr/bin/env python

import frida  #导入frida模块
import sys    #导入sys模块

jscode = """ #从此处开始定义用来Hook的javascript代码
Java.perform(function() {
    var MainActivity = Java.use('com.example.MainActivity'); //获得MainActivity类
    MainActivity.func.implementation = function(arg1,arg2) { //Hook func函数，用js自己实现
        send('frida hook success'); //发送信息，用于回调python中的函数
        return 'Change String!'; //劫持返回值，修改为想要返回的字符串
    }
});
"""

def on_message(message, data): #js中执行send函数后要回调的函数
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

process = frida.get_usb_device().attach('com.example.testfrida') #得到设备并劫持进程(get_remote_device)
script = process.create_script(jscode) #创建js脚本
script.on('message',on_message) #加载回调函数
script.load() #加载脚本
sys.stdin.read()
```

运行frida-server后，然后执行该脚本`python xxx.py`即可。或者通过`frida -U -l xxx.js -f com.package.name `直接运行js脚本。

### 2.2 hook native

```python
#!/usr/bin/env python

import frida  #导入frida模块
import sys    #导入sys模块

jscode = """ #从此处开始定义用来Hook的javascript代码
Java.perform(function() {
    //指定要Hook的so文件名和要Hook的函数名,函数名就是上面IDA导出表中显示的那个函数名
    Interceptor.attach(Module.findExportByName("libxxx.so", "getString"), {
        onEnter: function(args) { //onEnter是进入该函数前要执行的代码，其中args是传入的参数
            // 一般so层函数第一个参数都是JniEnv，第二个是jclass，从第三个开始才是java层传入的参数
            send("hook frida start"); //发送信息，用于回调python中的函数
            send("args[2]=" + args[2]); //打印我们java层第一个传入的参数
            send("args[3]=" + args[3]); //打印我们java层传入的第二个参数
        },
        onLeave: function(retval) { //onLeave是该函数执行结束要执行的代码，其中retval参数即是返回值
            send("return:" + retval); //打印返回值
            var env = Java.vm.getEnv(); //获取env对象，也就是native函数的第一个参数
            var newRetVal = env.newStringUtf("tamper"); //构造一个newStringUtf对象
            retval.replace(newRetVal); //替换返回值
        }
    });
});
"""

def on_message(message, data): #js中执行send函数后要回调的函数
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

process = frida.get_usb_device().attach('com.example.testfrida') #得到设备并劫持进程(get_remote_device)
script = process.create_script(jscode) #创建js脚本
script.on('message',on_message) #加载回调函数
script.load() #加载脚本
sys.stdin.read()
```

运行frida-server后，然后执行该脚本`python xxx.py`即可。或者通过`frida -U -l xxx.js -f com.package.name `直接运行js脚本。



## 三、语法

### 3.1 基本hook语法

下面主要讲述`Java.use`用法，除此之外，还有`Java.choose(className, callbacks)`、`Java.cast(handle, klass)`、`Java.scheduleOnMainThread(fn)`等用法。

+ Hook生成对象使用$new，如：

  ```js
  const JavaString = Java.use('java.lang.String');
  var exampleString1 = JavaString.$new('Hello World, this is an example string in Java.');
  console.log('[+] exampleString1: ' + exampleString1);
  ```

+ Hook构造函数使用$init，如：

  ```js
  const StringBuilder = Java.use('java.lang.StringBuilder');
  StringBuilder.$init.overload('java.lang.String').implementation = function (arg1,arg2) {
      var partial = "";
      var result = this.$init(arg);
      if (arg !== null) {
           partial = arg.toString().replace('\n', '').slice(0,10);
      }
      console.log('new StringBuilder("' + partial + '");')
      return result;
  }
  ```

+ Hook重载函数使用overload，如：

  ```js
  var MainActivity = Java.use('com.example.MainActivity');
  MainActivity.func.overload("[B", "[B").implementation = function(arg1, arg2) {
      console.log('frida hook success');
      // todo
  }
  ```

+ Hook一般函数使用implementation，如：

  ```js
  var MainActivity = Java.use('com.example.MainActivity');
  MainActivity.func.implementation = function(arg1,arg2) {
      console.log('frida hook success');
      // todo
  }
  ```

+ Hook native 函数

  ```js
  Interceptor.attach(Module.findExportByName("xxx.so" , "xxxx"), {
      onEnter: function(args) {
          console.log('frida hook enter success');
          // todo
      },
      onLeave:function(retval){
          console.log('frida hook leave success');
          // todo
      }
  });
  ```

### 3.2 常用hook方法

+ 打印堆栈

  ```js
  const AndroidLog = Java.use("android.util.Log")
  const AndroidException = Java.use("java.lang.Exception")
  function printStackTrace(){
      console.log(AndroidLog.getStackTraceString(AndroidException.$new()));
  }
  ```

+ int转bytes

  ```js
  function int2bytes(arg) {
      var result = [];
      for (var i = 0; i < 2; i++) {
          result[i] = arg >> (8 - i * 8);
      }
      return result;
  }
  ```

+ bytes转String

  ```js
  function bytes2string(array){
      if (!array) {
          return '';
      }
  
      var result = "";
      for(var i = 0; i < array.length; ++i){
          result+= (String.fromCharCode(array[i]));
      }
      return result;
  }
  ```

+ string转bytes

  ```js
  function string2bytes(str){
      for (var i = 0,arr=[]; i < str.length;i++){
          arr.push(str.charCodeAt(i));
      }
      return new Uint8Array(arr);
  }
  ```

+ bytes转HexString

  ```js
  function bytes2HexString(uint8arr) {
      if (!uint8arr) {
          return '';
      }
  
      var result = '';
      for (var i = 0; i < uint8arr.length; i++) {
          var hex = (uint8arr[i] & 0xff).toString(16);
          hex = (hex.length === 1) ? '0' + hex : hex;
          result += hex;
      }
  
      return result.toUpperCase();
  }
  ```

+ HexString转bytes

  ```js
  function HexString2Bytes(str) {
      if (!str) {
          return new Uint8Array();
      }
  
      var a = [];
      for (var i = 0, len = str.length; i < len; i+=2) {
          a.push(parseInt(str.substr(i,2),16));
      }
  
      return new Uint8Array(a);
  }
  ```

+ ArrayBuffer转String

  ```js
  function ab2str(buf) {
      return String.fromCharCode.apply(null, new Uint8Array(buf));
  }
  ```

+ string转ArrayBuffer

  ```js
  function str2ab(str) {
      var buf = new ArrayBuffer(str.length * 2); // 每个字符占用2个字节
      var bufView = new Uint16Array(buf);
      for (var i = 0, strLen = str.length; i < strLen; i++) {
          bufView[i] = str.charCodeAt(i);
      }
      return buf;
  }
  ```



## 四、攻防

### 4.1 frida检测

...

### 4.2 frida躲避

...



---

+ [Frida官方文档](https://www.frida.re/docs/javascript-api/#java)
+ [JavaScript API(篇一)](https://zhuanlan.kanxue.com/article-342.htm)
+ [JavaScript API(篇二)](https://zhuanlan.kanxue.com/article-414.htm)
+ [Frida Hook Android 常用方法](https://blog.csdn.net/zhy025907/article/details/89512096)
+ [Hook神器Frida使用详解](https://blog.csdn.net/m0_46204016/article/details/104115893)
+ [这恐怕是学习Frida最详细的笔记了](https://blog.csdn.net/qq_34067821/article/details/107180885)

