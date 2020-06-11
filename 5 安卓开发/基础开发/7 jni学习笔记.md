## 一、jni概述

JNI(Java Native Interface)是Java语言提供的Java和C/C++相互沟通的机制。NDK是帮助开发者快速开发 C\C++动态库的一系列工具的集合，它集成了交叉编译器，并提供了相应的mk文件隔离 CPU、平台、ABI等差异。但是NDK的API支持的功能非常有限，包含有：C标准库(libc)、标准数学库(libm)、压缩库(libz)、Log库(liblog)。

JNI开发流程主要分为以下几步：

+ 通过`javac -source 1.6 -target 1.6 <javafile> -d <savedir>`将java文件(包含native方法)编译成class文件
+ 通过`javah -jin -classpath <classdir> -d <pkg.class> `将class文件编译生成native头文件
+ 通过`gcc -I$JAVA_HOME/include -I$JAVA_HOME/include/linux -fPIC -shared <cppfile> -o <soname> `编译
+ 通过`java -Djava.library.path=<sodir> -classpath <classdir> <pkg.class>`运行



## 二、jni注册

"注册"之意就是将java层的函数和native层的函数关联起来。有了这种关联后java就可以调用native函数。这种“关联”就是jni函数的注册。System.loadLibrary加载so文件的机制大体如下：

```
System.loadLibrary->
  Runtime.loadLibrary->(Java)
    nativeLoad->(C: java_lang_Runtime.cpp)
      Dalvik_java_lang_Runtime_nativeLoad->
        dvmLoadNativeCode-> (dalvik/vm/Native.cpp)
          1) dlopen(pathName, RTLD_LAZY)(把so文件mmap到进程空间，并把func等相关信息填充到soinfo中)
          2) dlsym(handle, "JNI_OnLoad")
          3) JNI_OnLoad->
               lookupSharedLibMethod(method)(根据signature在所有已经打开的.so中寻找此函数实现)
               dvmUseJNIBridge((Method*) method, fnPtr)
               ...
          4) ...
```

因此注册方法有两种，一种是在JNI_OnLoad时调用函数动态注册，一种是按照签名格式自动注册：

### 2.1 动态注册

Android平台提供了一个jniRegisterNativeMethods函数来完成注册工作：

```cpp
// 这才是真正注册的开始
int jniRegisterNativeMethods(JNIEnv* env,const char* className,
                            const JNINativeMethod* gMethods,int numMethods){
    jclass clazz = (*env)->FindClass(env,className);
    ...
    return ((*env)->RegisterNatives(env,clazz,gMethods,numMethods) < 0)?-1:0;// 注册核心
}

// 其中`JNINativeMethod`用来保存java和native的函数关系：
typedef struct {  
    const char* name;      // Java中函数的名字(不用携带包路径)
    const char* signature; // Java中函数的签名(格式采用jni类型)
    void* fnPtr;           // 指向native函数的函数指针
} JNINativeMethod;
```

如果so文件已经定义JNI_OnLoad函数，就会运行通过JNI_OnLoad函数：

```cpp
# include <JNIHelp.h>// 它内部已经包含了jni.h的头文件

static JNINativeMethod[] gMethods = {...};// 定义注册的函数
jnt JNI_OnLoad(JavaVM* vm, void* reserved){// JavaVM是虚拟机在JNI层的代表。每个Java进程只有一个实例
    JNIEnvi* env = NULL;
    jint result = -1;
    ... // 这里可以做其他的操作(具体解释见下面描述)
    if(jniRegisterNativeMethods(env,"...",gMethods,NELEN(gMethods)) < 0){
        return result;
    }
    return JNI_VERSION_1_4;
}
```

### 2.2 静态注册

如果so文件没有定义JNI_OnLoad函数，则直到需要调用的这下函数时，dvm调用dvmResolveNativeMethod进行动态解析。这种方法是根据函数名来查找对应的jni函数：

```java
// java代码
package com.android.test

import ...;

public class JNIJavaTest{
  public native void func(String arg);
}
```

```cpp
// 通过javah生成的native头文件
# include <jni.h>// 必须包含这个头文件，否则编译不通过

#ifndef
...
JNIEXPORT void JNICALL Java_com_android_test_JNIJavaTest_func(JNIEnv*,jobject,jstring);
...
#endif
```

从上述流程可以看出，静态注册要求jni函数的名字必须遵循特定的格式`JNIEXPORT <return-type> JNICALL Java_<pkg>_<classname>_<funcname>(JNIEnvi*,...)`



## 三、jni类型

在介绍之前，先看几个空定义`#define JNIIMPORT`、`#define JNIEXPORT`、`#define JNICALL`。

JavaVM是一个与进程相关的变量，是虚拟机在JNI层的代表(全进程只有一个JavaVM对象)。JNIEnv是一个与线程相关的变量，它的作用是调用java函数以及操作jobject对象等。调用JavaVM的AttachCurrentThread函数就可以得到这个线程的JNIEnv结构体，在退出前需要调用JavaVM的DetachCurrentThread函数来释放对应的资源。通过`(*env)->FindClass(env,className); `可以获取一个类的引用(需要对该引用进行判空处理)。以下是JNI中的类型：

<img src="https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/images/relational.png" alt="基本类型" style="zoom:50%;" />

<img src="https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/images/recommend.png" alt="引用类型" style="zoom:50%;"/>



## 四、jni操作

### 4.1 类型操作

#### jarray

参考[这里](https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/array.html)。JNI中的数组分为基本类型数组和对象数组，它们的处理方式是不一样的，基本类型数组中的所有元素都是JNI的基本数据类型而可以直接访问。而对象数组中的所有元素是一个类的实例或其它数组的引用，必须选择合适的JNI函数来访问和设置Java层的数组对象。JNI提供了如下方法进行操作:

+ `NewTypeArray(env,size,...)`：创建一个数组，需要进行判空处理。
+ `GetArrayLength(env,j_array)`：获取数组长度
+ `Get/SetXXXArrayRegion(env,j_array,...)`：将Java数组中的基本类型元素拷贝到C缓冲区中\直接设置Java数组中某基本类型元素。X取值为基本类型。
+ `Get/ReleaseXXXArrayElements(env,j_array,NULL)|Get/setObjectArrayElement(env,j_array,NULL)`：返回数组中指定位置的元素\释放(修改)数组中指定位置的元素。第三个参数表示返回的数组指针是原始数组(FALSE)，还是拷贝原始数据到临时缓冲区的指针(TRUE)。**在获取到的指针必须做校验，因为当原始数据在内存当中不是连续存放的情况下，JVM会复制所有原始数据到一个临时缓冲区，并返回这个临时缓冲区的指针。有可能在申请开辟临时缓冲区内存空间时，会内存不足导致申请失败，这时会返回 NULL**。
+ `Get/ReleasePrimitiveArrayCritical(env,j_array,NULL) `：直接获取\释放数组元素指针的函数，第三个参数表示返回的数组指针是原始数组(FALSE)，还是拷贝原始数据到临时缓冲区的指针(TRUE)。**特别的在于在访问数组对象时会暂停GC线程，所以这两个函数期间不能调用任何会让线程阻塞或等待JVM中其它线程的本地函数或JNI函数**。

#### jstring

参考[这里](https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/string.html)。字符串之间主要涉及到的问题是编码，因此很多方法都是针对与不同编码之间转换进行操作的：

+ `NewStringUTF`：构建一个新的java.lang.String字符串对象。这个新字符串会自动转换成Java支持的Unicode编码
+ `GetString(UTF)Length`：由于UTF-8编码的字符串以'\0'结尾，而Unicode字符串不是。如想获取指向Unicode编码的jstring字符串长度，在JNI中可通过这个函数获取
+ `GetStringRegion/GetStringUTFRegion`：表示获取Unicode和UTF-8编码字符串指定范围内的内容。这对函数会把源字符串复制到一个预先分配的缓冲区内
+ `Get/ReleaseStringChars(env,j_str,&isCopy)`|Get/ReleaseStringUTFChars(env,j_str,&isCopy)：用于获取\释放以Unicode(UTF-8)格式编码的字符串，需进行判空处理
+ `Get/ReleaseStringCritical`：可直接返回源字符串的指针。不过在这两个函数之间的本地代码不能调用任何会让线程阻塞或等待JVM中其它线程的本地函数或JNI函数。因为通过 GetStringCritical 得到的是一个指向 JVM 内部字符串的直接指针，获取这个直接指针后会导致暂停GC线程，如果其它线程触发GC继续运行的话，都会导致阻塞调用者。需要进行判空处理

#### jobject

参考[这里](https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/function2.html)。主要是用于c/c++进行回掉。通过理解下面的几个函数，感觉和java反射思路差不多：

+ `FindClass(env,"pkg.class")`：获取类的class引用，返回jaclass类型，要判空
+ `GetObjectClass(env,jobj) `：获取实例的class引用，返回jaclass类型，要判空
+ `NewObject(env,clazz,mid,args...)`：创建对象的实例，返回jobject类型，要判空
+ `Get(Static)MethodID(env, jcla, "fnName", "fnSign");`：获取类的方法ID，返回jmethodID类型，要判空
+ `Call(Static)XXXMethod(env,jobj|jcla,mid, args...)`：调用对象的实例\父类\静态方法
+ `Get(Static)FieldID(env,jcla,"fName","fType") `：从class引用中获取字段的ID
+ `Get/Set(Static)XXXField(env,jobj|jcla,fid,args...)`：获取\设置对象的实例\静态字段的值

### 4.2 垃圾回收

参考[这里](https://wiki.jikexueyuan.com/project/jni-ndk-developer-guide/recommend.html)。Java中对象最后是由垃圾回收器进行回收和释放内存的，当JNI中某方法保存了某对象引用而在其他时机再使用该对象时，此时该指针可能是一个野指针(因使用 thiz = this方式不会增加java对象的引用计数)。JNI有三种引用：

+ Local Reference：本地引用。特点是调用函数传入的jobject、各种JNI创建(FindClass\NewObject\NewCharArray)的对象，都会阻止GC回收所引用的对象，不能在本地函数中跨函数或跨线程使用。通过`DeleteLocalRef(env,jobj)`可手动本地引用的回收。*特别是在循环语句中(不然容易造成引用耗费，引用表最大空间为512个，如果超出这个范围JVM就会挂掉)*。
+ Global Reference：全局引用。特点是调用NewGlobalRef基于局部引用创建的对象，会阻GC回收所引用的对象，可以跨方法或跨线程使用。通过`DeleteGlobalRef(env,job)`可手动本地引用的回收。
+ Weak Global Reference：弱全局引用。特点是jni函数运行中，jobject可能会被垃圾回收。所以在使用之前需要调用`env->isSameObject`进行判断。

JNI提供了一系列函数来管理局部引用的生命周期。这些函数包括：EnsureLocalCapacity、PushLocalFrame、PopLocalFrame、NewLocalRef、DeleteLocalRef等。可自行查看相应函数的使用。

《Android JNI局部引用表溢出》这篇文章是一个JNI引用使用不当造成引用表溢出，最终导致程序崩溃的例子，可以上网搜查进行学习。

### 4.3 异常处理

JNI中的异常和Java不太一样。它在出现异常时不会中断本地函数的运行(但是不会继续执行，而是做一些资源清理工作)，直到从JNI层返回到Java层后，虚拟机才会抛出这个异常。JNIEnv提供了三个函数给予帮助：

+ ExceptionOccurd：用来判断是否发生异常
+ ExceptionClear：用来清除已经发生的异常
+ ThrowNew：用来向Java层抛出异常