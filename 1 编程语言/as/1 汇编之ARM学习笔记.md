## 一、代码生成

这里以c语言为例，进行相关步骤的记录：

+ 预处理：去掉注释/展开头文件/宏替换/条件编译(但会保留#pragma编译器指令，因为编译器需要使用它们)

  通过`gcc -E hello.c -I <dir> -o hello.i`生成完整文件，`-I`指定头文件目录

+ **编译：检查词法\语法\语义，生成汇编**

  **通过`gcc -S hello.c -I <dir> -o hello.s`生成汇编文件，`-I`指定头文件目录**

+ 汇编：汇编代码转换机器码

  通过`gcc -c hello.c -o hello.o`生成二进制文件，等同于`as hello.s -o hello.o`

+ 链接：链接到一起生成可执行程序

  通过`gcc hello.c -o hello`生成可执行程序。等同于`ld hello.o ... -o hello`



## 二、基础知识

### 2.1 模式状态

ARM处理器支持用户模式和特权模式，具体有以下运行模式：

+ 用户模式(usr)：正常的程序执行状态(因此只学习该模式下指令)。*只能访问常规寄存器*
+ 系统模式(sys)：运行具有特权的操作系统任务。*只能访问常规寄存器*
+ 管理模式(svc)：操作系统使用的保护模式
+ 中断模式(irq\fiq)：用于通用的中断处理\用于高速数据传输或通道处理
+ 终止模式(abt\und)：用于虚拟存储及存储保护\当未定义的指令执行时进入该模式

ARM处理器在Thumb和ARM两种工作状态中随意切换。运行于Thumb状态时，执行的的16位字对齐的指令；运行于ARM状态时，执行的的32位字对齐的指令。

### 2.2 寄存器定义

ARM微处理器共有37个32位寄存器(其中31个为通用寄存器，6个为状态寄存器)，但是这些寄存器不能被同时访问，具体哪些寄存器是可以访问的，取决ARM处理器具体的运行模式和工作状态。**但在任何时候，通用寄存器R0~R14、程序计数器PC、一个状态寄存器CPSR都是可访问的**。

+ 未分组寄存器：R0-R7是不分组寄存器。这意味着在所有处理器模式下，访问的都是同一个物理寄存器。不分组寄存器没有被系统用于特别的用途，任何可采用通用寄存器的应用场合都可以使用未分组寄存器。ARM汇编中规定：R0寄存器用来存放函数调用的返回值；且R0-R3这4个寄存器用来传递函数的第1到第4个参数，超出的通过堆栈来传递

+ 已分组寄存器：R8-R14是已分组寄存器。**首先了解一个概念：每个函数所使用的栈空间是一个栈帧**。R11用作(栈)帧指针FP，指向当前函数栈帧的栈底；R12用作内部指针IP，是一个临时寄存器；R13用作栈指针SP，指向当前函数栈帧的栈顶；R14用作链接寄存器LP，指向函数的返回地址；R15被用作程序计数器PC，指向当前正在执行的指令的地址+8(因为在取地址和执行之间多了一个译码的阶段)。

  下面以一个例子图进行理解：栈的方向为向下增长，栈底在高地址而栈顶在低地址。fp、sp、lr、pc这四个寄存器是非常特殊的寄存器，它们记录了当前正在运行的函数一些重要信息。main stack frame为先前调用函数的栈帧，func1 stack frame为当前函数的栈帧。**在刚进入一个新的函数开始执行的时候，需要将上个函数入栈保存起来！参数入栈顺序是从右到左依次入栈(即使寄存器列表也是如此)，而参数的出栈顺序则是从左到右的出栈。**

  ![](https://img-blog.csdn.net/20160817105557618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

+ 状态寄存器：CPSR包含条件码标志、中断禁止位、当前处理器模式以及其他状态和控制信息，主要用于运行模式、运算指令的信息记录等。

  ![](https://img-blog.csdn.net/20161028225855208)



## 三、寻址方式

### 3.1 相对寻址

如`BL NEXT`以程序计数器PC的当前数值为基地址，指令中的地址标号作为偏移量，两者相加后的数值赋值给R0

### 3.2 立即寻址

如`MOV R0,#0x12`将十六进制0x12数值赋值给R0

如`MOV R0,R1`将寄存器R1的数值赋值给R0

如`MOV R0,R1,LSL #2`将寄存器R1的数值左移2位后的数值赋值给R0

### 3.3 间接寻址

如`LDR R0,[R1]`将寄存器R1的数值作为地址，取该地址存储单元的数值赋值给R0

如`LDR R0,[R1,#-4]`将寄存器R1的数值减去4作为地址，取该地址存储单元的数值赋值给R0

### 3.4 特别寻址

+ **堆栈寻址**：

  如`LDMFD SP!,{R1-R7,LR}`将数据出栈到R1-R7以及LR，多用于恢复子程序"现场"；

  如`STMFD SP!,{R1-R7,LR}`将R1-R7以及LR的数据入栈，多用于保存子程序"现场"

+ **存储寻址**

  如`LDMIA R0!,{R1-R3}`从寄存器R0指向的存储单元读取3个字到R1-R3寄存器中

  如`STMIA R0!,{R1-R3}`存储R1-R3寄存器的内容到寄存器R0指向的内存单元

+ **多寄存器寻址**：

  如`LDMIA R0,{R1,R2,R3}`将寄存器[R0]依次赋值给花括号中的寄存器，每次赋值完后R0根据规则运算，这里IA表示每次自增1字节，因此R1=[R0]\R2=[R0+#4]\R3=[R0+#8]



## 四、语法结构

### 4.1 指令格式

ARM指令的基本格式为：`<opcode>{mode}{type}{<cond>}{S} {Rd{!}} {,...}`

+ opcode为指令助记符，如MOV等，这也是理解记忆的重点
+ mode为执行模式，用于执行前后地址的处理
+ type为数据类型，用于标注操作的数据长度
+ cond为条件判断，用于决定是否执行指令
+ S为指令是否影响CPSR的值，如ADDS\SUBS等指令会影响
+ 感叹号!代表操作完的最终地址要写回到叹号附带的寄存器中，如LDM指令

```txt
// 常见的mode
  mode     含 义
  DB    decrease before，基址寄存器(不允许是R15)在执行指令之前减少
  DA    decrease after，基址寄存器(不允许是R15)在执行指令之后减少
  IB    increase before，基址寄存器(不允许是R15)在执行指令之前增加
  IA    increase after，基址寄存器(不允许是R15)在执行指令之后增加
  FD    full decrease，满递减堆栈，堆栈向低地址生长，SP指向最后一个入栈的有效数据项
  FA    full increase，满递增堆栈，堆栈向高地址生长，SP指向下一个要放入的空地址

// 常见的type
  type     含 义
   D       双字节
   B    无符号单字节
  SB    有符号单字节
   H    无符号半个字
  SH    有符号半个字

// 常见的cond
  条件码 助记符后缀        标 志                 含 义
  0000    EQ            Z置位                 相等
  0001    NE            Z清零                 不相等

  0010    CS/HS         C置位                 无符号数大于或等于
  0011    CC/LO         C清零                 无符号数小于
  
  0100    MI            N置位                 负数
  0101    PL            N清零                 正数或零

  0110    VS            V置位                 溢出
  0111    VC            V清零                 未溢出

  1000    HI            C置位Z清零             无符号数大于
  1001    LS            C清零Z置位             无符号数小于或等于

  1010    GE            N等于V                带符号数大于或等于
  1011    LT            N不等于V               带符号数小于
  1100    GT            Z清零且(N等于V)        带符号数大于
  1101    LE            Z置位或(N不等于V)       带符号数小于或等于

  1110    AL            忽略                  无条件执行
  1111    NV            忽略
```

### 4.2 指令内容

访问：

+ `PUSH/POP{cond} reglist`：将寄存器列表内容推入满递减栈/从满递减栈弹出数据到寄存器列表
+ `ADR{cond} Rd,label`：小范围的地址读取伪指令，它将基于PC相对偏移的地址值读取到寄存器中
+ `LDR{type}{cond} Rd,{Rd2,}label`：从label指向的内存中加载数据到Rd中。翻译为load to register
+ `LDM{mode}{cond} Rn{!} reglist`：从指定的存储单元加载多个数据到寄存器列表。翻译为load to multi register
+ `STR{type}{cond} Rd,{Rd2,}label`：将Rd的数据写入到label指向的内存中。翻译为store from register
+ `STM{mode}{cond} Rn{!} reglist`：将寄存器列表数据写入到指定的存储单元中。翻译为store from multi register

运算：

+ `CMP|CMN{cond} Rn,operand2`：将Rn值减去/加上op2的值，不保存结果，仅根据比较结果设置标志位

+ `TEQ|TST{cond} Rn,operand2`：将Rn值与op2的值进行异或/与，不保存结果，仅根据比较结果设置标志位

+ `MOV{cond}{S} Rd,operand2`：将8位的立即数或寄存器内容传给目标寄存器Rd

  `MVN`：mov not，为按mov操作完后目标寄存器数据接着按位取反

+ `ADD{cond}{S} Rd,Rn,operand2`：将寄存器Rn的数值加上ope2的值传给目标寄存器Rd。

  `ADC{cond}{S} Rd,Rn,operand2`：add cflag，为按add操作后再加上CPSR中C位的数值到目标寄存器

+ `SUB{cond}{S} Rd,Rn,operand2`：将寄存器Rn的数值减去ope2的值传给目标寄存器Rd。

  `SBC{cond}{S} Rd,Rn,operand2`：sub cflag，按sub操作后再减去CPSR中C位的数值到目标寄存器

  `RSB...`：reversal sub，将ope2的值减去寄存器Rn的数值传给目标寄存器Rd

  `RSC...`：reversal sub cflag，按rsb操作后再减去CPSR中C位的数值到目标寄存器

+ `{S|U}MUL{L|D}{cond}{S} Rd,{Rd1,}Rm,Rn`：将寄存器Rm的值与寄存器Rn的值相乘后传给目标寄存器Rd

  `{S|U}MLA{L|D}...`：mul add ra，按mul操作后再加上Ra寄存器的值的低32位到目标寄存器

  `{S|U}MLS{L|D}...`：mul sub ra，按mul操作后再减去Ra寄存器的值的低32位到目标寄存器

+ `{S|U}DIV{cond} Rd,Rm,Rn`：有\无符号除法指令

+ `LSL|LSR|ROL|ROR{cond}{S} Rd,Rn,operand2`：逻辑左右移/循环左右移

+ `AND|ORR|EOR{cond}{S} Rd,Rn,operand2`：逻辑与|或|异或。其中AND还有一种演变形式BIC位清除指令

跳转：

+ `B{L}{cond} label`：如果条件满足cond，跳转到寻址到label处运行。带L的话：先将寄存器PC赋值到LP，然后跳转到寻址到label处运行(多用于调用子程序)。翻译为break



## 四、语句解析



## 五、文件结构

```
  .section .rodata
  .align 2
  .arch armv7-a              @处理器架构
  .fpu softvfp               @协处理器类型
  .eabi_attribute 30,6       @接口属性
  .file "hello.c"            @源文件名称
.LC0:
  .text                      @段定义，也可用.section定义等
  .align 2                   @对齐方式，数值为2的次数方
  .ascii "Hello ARM!\000"    @声明字符串
  .global main               @声明全局符号
  .type main,%function       @指定符号类型，也可通过.long等缩减形式
main:
  @ args = 0,pretend = 0,frame = 8
  @ frame_needed = 1,uses_anpnymous_args = 0
  stmfd sp!,(fp,lr)
  add fp,sp,#4
  ...
.LPIC0:
  add r3,px,r3
  ...
.L4:
  .align 2
.L3:
  .section .note.GNU-satck,"",%progbits @定义.note.GNU-satck段，作用是禁止生成可执行堆栈
  .word .LC0-(.LPIC0+8)     @用来存放地址值
  .size main, .-main        @用来设定符号的大小
  .ident "GCC (GNU) 4.4.3"  @编译器标识符，无实际用途
```

ARM汇编中声明函数的方法如下：

```
  .global 函数名
  .type 函数名,%function
函数名:
  <...函数体...>
```

禁止生成可执行堆栈，是用来保护代码安全。可执行堆栈常常用来引发堆栈溢出之类的漏洞，关于这方面可以查阅软件漏洞研究方面的书籍。