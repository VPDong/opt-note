# Xposed学习笔记

Xposed框架的原理是修改系统文件，替换了`/system/bin/app_process`可执行文件，在启动Zygote时加载额外的jar文件(`/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar`)，并执行一些初始化操作(执行XposedBridge的main方法)。然后我们就可以在这个Zygote上下文中进行某些hook操作。

在之后的运作中，当其他系统模块(比如AMS)希望创建新进程时，只需向zygote进程发出请求，zygote进程监听到该请求后，会相应地fork出新的进程，于是这个新进程在初生之时，就先天具有了自己的Dalvik虚拟机以及系统资源。



## 一、教程

+ 在当前module的build.gradle中添加如下依赖：

  ```groovy
  compileOnly 'de.robv.android.xposed:api:82'
  compileOnly 'de.robv.android.xposed:api:82:sources'
  ```

+ 在AndroidManifest.xml声明相关信息：

  ```xml
  <!--xposed插件的声明-->
  <meta-data android:name="xposedmodule" android:value="true" />
  <!--xposed的最小版本-->
  <meta-data android:name="xposedminversion" android:value="30+" />
  <!--xposed插件的描述-->
  <meta-data android:name="xposeddescription" android:value="Xposed开发测试" />
  ```

+ 在assets中创建xposed_init文件，其中记录所有实现了Xposed功能接口的类的完整路径名，每一行书只写一个这样的路径：

  ```
  com.example.test.XposedApp
  ```

+ 编写上一步指定的实现类：

  ```java
  public class XposedApp implements IXposedHookLoadPackage {
      @Override
      public void handleLoadPackage(final XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
          if (!"com.example.test.app".equals(lpparam.packageName)) return;
          XposedBridge.log("HookLogic >> current package:" + lpparam.packageName);
          try {
              XposedHelpers.findAndHookMethod("com.example.test.MainActivity",
                      lpparam.classLoader, "onCreate", Bundle.class, new XC_MethodHook() {
                          @Override
                          protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                              super.beforeHookedMethod(param);
                          }
  
                          @Override
                          protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                              // 不能通过Class.forName()来获取Class，在跨应用时会失效
                              Class clz = lpparam.classLoader.loadClass("com.example.test.MainActivity");
                              Field field = clz.getDeclaredField("textView");
                              field.setAccessible(true);
                              // param.thisObject为执行该方法的对象，在这里指MainActivity
                              TextView textView = (TextView) field.get(param.thisObject);
                              textView.setText("Hello");
                          }
                      });
          } catch (Throwable t) {
              XposedBridge.log(t);
          }
      }
  }
  ```

注意：一个Xposed模块可以有多个不同的入口：既可在 Android 系统启动时被调用，也可在一个应用程序包被加载时被调用。想要在不同的时期被调用，则该功能类需要实现不同的接口。

+ `de.robv.android.xposed.IXposedHookZygoteInit`：实现了ZygoteInit阶段的hook能力，用于在Zygote进程启动之前执行相关代码，framework里的东西一般在这里改
+ `de.robv.android.xposed.IXposedHookLoadPackage`：实现了加载app阶段的hook能力，用于在app进程启动之前执行相关代码，app里的东西一般在这里改
+ `de.robv.android.xposed.IXposedHookInitPackageResources`：实现了加载app资源时的hook能力，用于修改app的一些资源，比如布局文件什么的



## 接口

具体的api信息请参考[【这里】](https://api.xposed.info/reference/packages.html)，未完待续。。。



