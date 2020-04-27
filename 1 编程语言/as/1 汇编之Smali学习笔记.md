## 一、文件

无论普通类、抽象类、接口类还是内部类，反编译的时候会为每个类单独生成一个smali文件。而内部类在生成smali文件时存在以下比较特殊的地方：

+ 编译器会在外部类中对私有成员和私有方法生成对应的可访问的静态方法
+ **编译器会把外部类的引用作为第一个参数插入到会内部类的构造器参数列表(保存在this$n，n表示引用的层数)中。类型前标注synthetic修饰符，表明是合成的而非开发者定义的**
+ 内部类访问外部类的私有变量和私有方法时，都要通过编译器对外部类生成的“合成方法”来间接访问

注解可以作用在类、变量、方法上，其内容既包含开发中常见的注解，也包含throws的定义(注解中的value定义了哪些异常)，更包含sdk仅对内的如MemberClass(用于定义该类有几个内部类)\EnclosingClass(表明该内部类作用于整个外部类，外部类在value中定义了)\EnclosingMethod(表明该内部类作用于外部类的某个方法，方法在value中定义了)。

```
// 基本信息
.class <访问权限> [修饰关键字] <类名> // 指定了当前类的类名
.super <父类名>                    // 指定了当前类的父类
.source <源文件名>                 // 指定了当前类的源文件名
.implements <接口名>               // 接口信息(如果有的话)
.annotation [注解属性] <注解类名>    // 注解信息(如果有的话)
  [注解字段=值]
.endannotation

// 字段信息(如果有的话)
#fields
.field <访问权限> [static] [修饰关键字] <字段名>:<字段类型>
  .annotation [注解属性] <注解类名>;// 注解信息(如果有的话)
    [注解字段=值]
  .end annotation
.end field

// 方法信息(如果有的话)
#methods
.method <访问权限> [static] [修饰关键字] <方法原型>
  .annotation [注解属性] <注解类名>; // 注解信息(如果有的话)
    [注解字段=值]
  .end annotation
  [.registers]  // 指定了使用的局部变量寄存器的个数
  [.parameter]  // 指定了方法的一个参数(可有多个)
  [.prologue]   // 指定了代码的开始处(混淆过的代码可能去掉了该指令)
  [.line]       // 指定了该处指令在源代码中的行号(混淆过的代码可能去掉了该指令)
  <代码体>       // 
.end method
```



## 二、基础

在 Dalvik 虚拟机字节码中寄存器的命名法中主要有2种:v命名法和p命名法。v命名法表示函数中用到的局部变量与参数，命名从v0开始而依次递增。p命名法表示函数的入参，命名从p0开始命名而依次递增。在调用非静态方法时需要传入该方法所在对象的引用，因此此时p0表示的是传入的隐式对象引用，从p1开始才是实际传入的入参。在调用静态方法时，由于静态方法不需要构建对象的引用，因此从p0开始就是实际传入的入参。

每个Dalvik寄存器都是32位大小，对于小于或者等于32位长度的类型来说，一个寄存器就可以存放该类型的值，而像J、D 等64位的类型，它们的值是使用相邻两个寄存器来存储的。每条指令使用的寄存器索引范围都有限制，除非指令通过标识len标明，都则一般均指为地址为256(一个字节大小)内的寄存器。



## 三、语法

### 3.1 格式

Smali指令的基本格式为：`<opcode>{<type>}{<cond>}{/<length>} {Vd} {,...}`

type：表示保存数据的类型，间接表示目的寄存器的使用个数

+ none：表示32位内的基本对象，间接表示使用一个寄存器存储数值，在get\put时需要细化类型boolean\char等
+ wide：表示64位内的基本对象(long、double类型)，间接表示使用两个寄存器进行存储
+ object：表示32位内的引用对象地址，间接表示使用一个寄存器存储地址，在const时需要细化string\class等。在invode时有interface\super\direct\virtual\static等

cond：表示对应指令特有的选项。如if中的gt\ge\eq\ne\le\lt\eqz等

leng：表示来源数据的地址长度，或来源寄存器的寻址范围

+ none：表示使用256地址内的寄存器
+ destNum：表示目的寄存器的地址位于2^Num内，dest为了理解而写(dest四个字母省略)
+ fromNum：表示来源寄存器的地址位于2^Num内，或者数据长度为num位

### 3.2 指令

定义：

+ `const{type}/{leng} vx,val`：存入数值到寄存器vx，其中tag要细分
+ `new-array vx,vy,类型`：根据类型新建一个数组，vy存入数组的长度，vx存入数组的引用
+ `new-instance vx,类型`：根据类型新建一个对象实例，并将实例的引用存入寄存器vx

访问：

+ `xput{type} vx,[vy],id(字段ID|索引)`：将vx的值作为tag型写入到vy指向实例的id中。静态字段没有vy，数组中vz为索引，x可选有s\i\a(静态实例、动态实例、数组)
+ `xget{type} vx,[vy],id(字段ID|索引)`：将位于vy指向实例的id的内容作为tag型读取到vx中。静态字段没有vy，数组中vz为索引，x可选有s\i\a(静态实例、动态实例、数组)

运算：

+ `instance-of vx,vy,类型`：检查寄存器vy中的对象引用是否是类型对应的实例，是vx存入非0值，否则vx存入0
+ `cmp[l|g] vx,vy,vz`：比较vy和vz的值，并在vx存入int型返回值(根据[l|g]进行规则返回)
+ `move{type}/{leng} vx,vy`：将寄存器vy的内容赋值给寄存器vx。注意的是形式还有`move-result[tag] vx`
+ `add{type}/{leng} vx,vy,vz`：计算`vy+vz`并将结果作为tag型存入vx
+ `sub{type}/{leng} vx,vy,vz`：计算`vy-vz`并将结果作为tag型存入vx
+ `mul{type}/{leng} vx,vy,vz`：计算`vy*vz`并将结果作为tag型存入vx
+ `div{type}/{leng} vx,vy,vz`：计算`vy\vz`并将结果作为tag型存入vx
+ `rem{type}/{leng} vx,vy,vz`：计算`vy%vz`并将结果作为tag型存入vx
+ `and{type}/{leng} vx,vy,vz`：计算`vy&vz`并将结果作为tag型存入vx
+ `orr{type}/{leng}  vx,vy,vz`：计算`vy|vz`并将结果作为tag型存入vx
+ `xor{type}/{leng} vx,vy,vz`：计算`vy⊕vz`并将结果作为tag型存入vx
+ `shl{type}/{leng} vx,vy,vz`：计算`vy<<vz`并将结果作为tag型存入vx
+ `shr{type}/{leng} vx,vy,vz`：计算`vy>>vz`并将结果作为tag型存入vx

跳转：

+ `goto/{leng} 目标`：通过偏移量无条件跳转到目标
+ `if{cond} vx,vy,目标`：如果vx和vy符合cond定义，则跳转到目标。vx和vy是int型值
+ `packed-switch vx,索引表目标`：vx存储的是要查找具体case的值。查找时通过vx减去表case初始值得到target，进而确定执行代码地址。索引表case常量是连续的(一般用于连续整数)
+ `sparse-switch vx,查询表目标`：vx存储的是要查找具体case的值。查找时通过vx比较表case的数值得到target，进而确定执行代码地址。查询表case常量是不连续(一般用于枚举定义)
+ `invoke-{type}/[range] {参数},方法名`：调用方法
+ `execute-inline {参数},内联ID`：根据内联ID执行内联方法
+ `return{type} vx`：返回vx寄存器的值，当type为void时没有vx



## 四、语句

### 4.1 分支语句

分支语句有packed分支和sparse分支，不论哪一种，编译后的会变代码格式大体一样，只不过不同分支格式的寻址方式不太一样：

```
// Dalvik从指令起始字节起计算偏移量,计算偏移是以两个字节为单位
// 偏移量可以是正或负，负偏移量用二进制补码格式存储

// 索引表是有数据结构定义的
struct  packed-switch-payload {
    ushort ident; // 表类型,值固定为0x0100，代表packed-switch
    ushort size;  // case的个数
    int init_key; // 初始case的值，相减后得到具体case的target
    int[] targets;// 每个case代码地址相对switch指令处的偏移
};

// 查询表是有数据结构定义的
struct sparse-switch-payload {
    ushort ident; // 表类型,值固定为0x0200，代表sparse-switch
    ushort size;  // case的个数
    int[] keys;   // 每个case的值，比较后得到具体case的target
    int[] targets;// 每个case代码地址相对switch指令处的偏移
};
```

具体的代码分析可以参考：https://www.ituring.com.cn/book/miniarticle/26960

### 4.2 循环语句

循环语句有for循环和while循环，不论哪一种，编译后的会变代码格式大体一样：

+ 有循环开始的goto标签label
+ 条件判断、条件不成立的跳出逻辑、条件成立的执行逻辑(这一块的逻辑根据不同循环体顺序不太一样)
+ 状态变更后goto跳转到label

具体的代码分析可以参考：https://www.ituring.com.cn/book/miniarticle/26958

### 4.3 异常语句

在实际编写代码过程中，各种预想不到的结果都有可能出现。为了尽可能的捕捉到异常信息，有必要在代码中使用try/catch语句将可能发生问题的代码“包裹”起来。在smali文件中，try/catch语句有如下特点：

+ try语句块使用`try_start_n`开头的标号注明，以`try_end_n`开头的标号结束。第一个try语句的开头标号为`try_start_0`，结束标号为`try_end_0`。使用多个try语句块时标号名称后面的数值依次递增。

+ 在`try_end_n`标号下面使用`.catch`指令指定异常类型与catch的标号，格式如下：

  `.catch <异常类型> {<try起始标号> .. <try结束标号>} <catch标号>`，且子catch还会将父catch的类型带上

具体的代码分析可以参考：https://www.ituring.com.cn/book/miniarticle/26961



## 五、DEX

可在android源码中的/dalvik/libdex/DexFile.h找到其定义。可参考:https://www.cnblogs.com/csnd/p/11800659.html。手动分析过程可参考链接:https://bbs.pediy.com/thread-203417.htm。

![](https://bbs.pediy.com/upload/attach/201508/5A2QEVXBVNQSXE7.jpg)

















