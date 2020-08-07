## 错误处理

C语言不提供对错误处理的直接支持，在发生错误时，大多数的C或UNIX函数调用返回1或NULL，同时会设置一个错误代码 **errno**，该错误代码是全局变量，表示在函数调用期间发生了错误。可在 errno.h 头文件中找到各种各样的错误代码。

所以C程序员可以通过检查返回值，然后根据返回值决定采取哪种适当的动作。开发人员应该在程序初始化时，把errno设置为0，这是一种良好的编程习惯。0值表示程序中没有错误。

+ `char *strerror(int errnum)`：用来依参数errnum的错误代码来查询其错误原因的描述字符串，然后将该字符串指针返回。位于string头文件中。
+ `void perror(const char *s)`：将上一个函数发生错误的原因输出到标准错误 (stderr)。参数s所指的字符串会先打印出，后面再加上错误原因字符串。位于stdio头文件中。
+ `int ferror(FILE *stream)`：检查参数stream所指定的文件流是否发生了错误情况，如有错误发生则返回非0值。



## I\\O函数

+ `int scanf(const char *format,...)`：根据参数format字符串来转换并格式化数据，输入源为键盘输入。格式转换的一般形式如:`%[*][size][l][h]type`。位于stdio头文件中。
+ `int sscanf(const char *str,const char *format,...)`：根据参数format字符串来转换并格式化数据，输入源为参数str的字符串，转换后的结果存于对应的参数内。位于stdio头文件中。
+ `int fscanf(FILE * stream ,const char *format,....)`：根据参数format字符串来转换并格式化数据，输入源为参数stream的文件流中读取字符串，转换后的结果存于对应的参数内。位于stdio头文件中。
+ `int printf(const char *format,...)`：根据参数format字符串来转换并格式化数据，将结果写到标准输出设备，直到出现字符串结束(’\0’)为止。格式转换的一般形式如:`%〔flags〕〔width〕〔.prec〕type`。位于stdio头文件中。
+ `int sprintf(char *str,const char *format,...)`：根据参数format字符串来转换并格式化数据，然后将结果复制到参数str所指的字符串数组，直到出现字符串结束(’\0’) 为止。使用此函数得留意堆栈溢出。位于stdio头文件中。
+ `int fprintf(FILE *stream, const char *format,...)`：根据参数format字符串来转换并格式化数据，然后将 结果输出到参数stream指定的文件中，直到出现字符串结束(‘\0 ‘)为止。位于stdio头文件中。



## 字符函数

+ `void * memset(void *s ,int c, size_t n)`：将参数s所指的内存区域前n个字节以参数c填入，然后返回指向s的指针。在编写程序时，在需要将某一数组初始化使用它时会很方便。头文件位于string中。

+ `char *strcpy(char *dest,const char *src)`：将参数src字符串拷贝到参数dest所指地址。头文件位于string中。

  `void *memcpy(void *dest,const void *src, size_t n)`：用来拷贝src所指的内存内容前n个字节到dest所指的内存地址上。与strcpy()不同的是它不会因为遇到字符串结束‘\0‘而结束。头文件位于string中。

+ `int strcmp(const char *s1,const char *s2)`：用来比较参数s1和s2字符串。字符串大小的比较是以ASCII码表上的顺序来决定。头文件位于string中。

  `int memcmp(const void *s1,const void *s2,size_t n)`：用来比较s1和s2所指的内存区间前n个字符。字符串大小的比较是以ASCII码表上的顺序来决定。头文件位于string中。

+ `char *strchr(const char *s,int c)`：用来从头找出开始找出参数s字符串中第一个出现的参数c地址，然后将该字符出现的地址返回。头文件位于string中。

  `void * memchr(const void *s,int c,size_t n)`：用来从头找出开始搜寻s所指的内存内容前n个字节，直到发现第一个值为c的字节，则返回指向该字节的指针。头文件位于string中。

+ `char * strtok(char *s,const char *delim)`：用来将字符串分割成一个个片段。参数s指向欲分割的字符串，参数delim则为分割字符串，当strtok()在参数s的字符串中发现到参数delim的分割字符时则会将该字符改为"\0"字符。在第 一次调用时，strtok()必需给予参数s字符串，往后的调用则将参数s设置成 NULL。每次调用成功则返回下一个分割后的字符串指针。头文件位于string中。

+ `int strlen(const char *s)`：用来计算指定的字符串s的长度，不包括结束字符"\0"。头文件位于string中。



## 文件函数

+ `FILE *fopen(const char *path,const char *mode)`：参数path字符串包含欲打开的文件路径及文件名，参数 mode字符串则代表着流形态。头文件位于stdio中。
+ `int fclose(FILE *stream)`：用来关闭先前fopen()打开的文件，此动作会让缓冲区内的数据写入文件中，并释放系统所提供的文件资源。头文件位于stdio中。
+ `long ftell(FILE *stream)`：用来取得文件流目前的读写位置。头文件位于stdio中。
+ `int fseek(FILE *stream,long offset,int whence)`：用来移动文件流的读写位置。头文件位于stdio中。
+ `int fread(void *ptr,size_t size,size_t nmemb,FILE *stream)`：用来从文件流中读取数据。返回实际读取到的nmemb数目，如果此值比参数nmemb来得小，则代表可能读到了文件尾或有错误发生，这时必须用feof()或ferror()来决定发生什么情况。头文件位于stdio中。
+ `int fwrite(const void *ptr,size_t size,size_t nmemb,FILE *stream)`：用来将数据写入文件流中。返回实际写入的nmemb数目，如果此值比参数nmemb来得小，则代表可能读到了文件尾或有错误发生，这时必须用feof()或ferror()来决定发生什么情况。头文件位于stdio中。
+ `int fflush(FILE* stream)`：会强迫将缓冲区内的数据写回参数stream指定的文件中，如果参数stream为NULL，fflush()会将所有打开的文件数据更新。头文件位于stdio中。
+ `int setvbuf(FILE *stream,char *buf,int mode,size_t size)`：在打开文件流后且读取内容之前，调用它可以用来设置文件流的缓冲区。参数mode为缓冲模式，有下列几种:`_IONBF无缓冲`、`_IOLBF行缓冲`、`_IOFBF完全缓冲`。头文件位于stdio中。
+ `char *fgets(char *s,int size,FILE *stream)`：从所指的文件内读入字符并存到参数s所指的内存空间，直到出现换行字符、读到文件尾或是已读了size-1个字符为止，最后会加上NULL作为字符串结束。头文件位于stdio中。
+ `int fputs(const char *s,FILE *stream)`：将参数s所指的字符串写入到所指的文件内。头文件位于stdio中。
+ `int feof(FILE * stream)`：侦测是否读到文件尾。如已到文件尾则返回非零值，其他返回0。头文件位于stdio中。
+ `int ferror(FILE *stream)`：用来检查所指定的文件流是否发生了错误情况，如果文件流有错误发生则返回非0 值。头文件位于stdio中。
+ `int rename(const char *oldpath,const char *newpath)`：会将参数oldpath所指定的文件名称改为参数newpath所指的文件名称，若newpath所指定的文件已存在则会被删除。头文件位于stdio中。
+ `int remove(const char *path)`：会删除参数pathn指定的文件。如果参数path为一文件则调用unlink()处理；如果参数path为一目录则调用rmdir()来处理。头文件位于stdio中。
+ `void *mmap(void *start,size_t length,int prot,int flags,int fd,off_t offsize)`：建立文件内存映射，用来将某个文件内容映射到内存中，对该内存区域的存取即是直接对该文件内容的读写。参数start指向欲对应的内存起始地址(通常设为NULL，代表让系统自动选定地址，对应成功后该地址会返回)，参数length代表将文件中多大的部分对应到内存，参数prot代表映射区域的保护方式，参数flags会影响映射区域的各种特性，参数fd为open()返回的文件描述词(代表欲映射到内存的文件)，参数offset为文件映射的偏移量(通常设置为0，代表从文件最前方开始对应，offset必须是分页大小的整数倍)。通过`size_t getpagesize(void)`获取分页大小。若映射成功则返回映射区的内存起始地址。头文件位于unistd和sys/mman中。
+ `int munmap(void *start,size_t length)`：解除文件内存映射，用来取消参数start所指的映射内存起始地址，参数length则是欲取消的内存大小。当进程结束或利用exec相关函数来执行其他程序时，映射内存会自动解除(但关闭对应的文件描述词时不会解除映射)。头文件位于unistd和sys/mman中。



## 内存函数

+ `void *malloc(size_t size)`：在堆区配置内存空间，其大小由指定的size决定，并返回指向第一个元素的指针。头文件位于stdlib中。
+ `void *calloc(size_t nmemb，size_t size)`：在堆区配置内存空间，其大小由指定的nmemb*size决定，配置内存时会将内存内容初始化为0 ，并返回指向第一个元素的指针。头文件位于stdlib中。
+ `void *realloc(void *ptr, size_t newsize)`：把堆内存扩展到newsize。若size值比原分配的内存空间小，内存内容不会改变，且返回的指针为原来内存的首地址(即ptr)；若size值比原分配的内存空间大，则realloc不一定会返回原来的指针，原先的内容不会改变但新多出的内存则为设初始值。头文件位于stdlib中。
+ `void free(void *ptr)`：释放堆区内存。调用后ptr 所指的内存空间便会被收回。头文件位于stdlib中。



## 进程函数

#### 1 进程控制

+ ` pid_t fork/vfork(void)`：一个现有进程可以调用该函数创建一个新进程。创建的新进程被称为子进程。该函数被调用一次但返回两次。两次返回的唯一区别是子进程中返回0值而父进程中返回子进程ID。fork函数的子进程是父进程的副本，它将获得父进程数据空间、堆、栈等资源的**副本**，这意味着父子进程间不共享存储空间，所以父子进程谁先运行依赖于系统实现；而vfork创建的进程并不将父进程的地址空间完全复制到子进程中，所以它会保证子进程先运行，在调用exec或exit之前与父进程数据是共享的(如果在调用这两个函数之前子进程依赖于父进程的进一步动作则会导致死锁)，在调用exec或exit之后父进程才可能被调度运行。头文件位于unistd中。

  `一个非常重要的函数ptrace`：ptrace系统调用主要是父进程用来观察和控制子进程的执行过程、检查并替换子进程执行序列或者寄存器值的一种手段：

  ```
  long ptrace(
    enum __ptrace_request request,// 请求ptrace执行的操作
    pid_t pid,// 目标进程的ID
    void *addr,// 目标进程的地址值
    void *data// 作用则根据request的不同而变化
              // 如果需要向目标进程中写入数据，data存放的是需要写入的数据
              //  如果从目标进程中读数据，data将存放返回的数据
  );
  ```

  它主要用于实现断点调试和跟踪系统调用，Linux系统中的strace、gdb以及ida-pro等调试器都是通过ptrace系统调用实现。具体的信息可以参考[这里](https://www.cnblogs.com/mysky007/p/11047943.html)，或者查看《Linux源码分析-PTRACE》。

+ `int nice(int inc)`：用来改变进程的进程执行优先顺序。参数inc数值越大则优先顺序排在越后面，即表示进程执行会越慢。头文件位于unistd中。

  ```c
  // nice是getpriority()和setpriority()的一种组合形式
  int nice(int increamet)
  {
     int oldpro = getpriority(PRIO_PROCESS, getpid() );
     return setpriority(PRIO_PROCESS, getpid(), oldpro + increament );
  }
  ```

+ `pid_t getpid(void)`：用来取得目前进程的进程识别码。头文件位于unistd中。

+ `pid_t waitpid(pid_t pid,int *status,int options)`：等待子进程中断或结束。首先会暂时停止目前进程的执行，直到有信号来到或子进程结束。如果在调用wait()时子进程已经结束，则wait()会立即返回子进程结束状态值。头文件位于sys/wait中。

  参数pid为欲等待的子进程识别码：

  + pid<-1表示等待进程组识别码为pid绝对值的任何子进程
  + pid=-1表示等待任何子进程，相当于wait()
  + pid=0 表示等待进程组识别码与目前进程相同的任何子进程
  + pid>0 表示等待任何子进程识别码为pid的子进程

  参数option可以为0或下面的OR组合:

  + WNOHANG：如果没有任何已经结束的子进程则马上返回，不予以等待。
  + WUNTRACED：如果子进程进入暂停执行情况则马上返回，但结束 状态不予以理会。


#### 2 进程信号

+ `unsigned int alarm(unsigned int seconds)`：用来设置信号SIGALRM在经过参数seconds指定的秒数后传送给目前的进程。如果参数seconds为0，则之前设置的闹钟会被取消，并将剩下的时间返回。头文件位于unistd中。
+ `int pause(void)`：会令目前的进程暂停，直到被信号所中断。头文件位于unistd中。
+ `unsigned int sleep(unsigned int seconds)`：会令目前的进程暂停，直到达到参数seconds所指定的时间，或是被信号所中断。头文件位于unistd中。
+ `int kill(pid_t pid,int sig)`：可以用来送参数sig指定的信号给参数pid指定的进程。参数pid与waitpid函数一样。头文件位于signal中。
+ `void exit(int status)`：用来正常终结目前进程的执行，并把参数status返回给父进程，而进程所有的缓冲区数据会自动写回并关闭未关闭的文件。头文件位于stdlib中。

#### 3 进程命令

+ `int system(const char *string)`：会调用fork产生子进程，由子进程来调用/bin/sh-c string来执行参数string字符串所代表的命令，此命令执行完后随即返回原调用的进程。在调用system()期间SIGCHLD信号会被暂时搁置，SIGINT和SIGQUIT信号则会被忽略。头文件位于unistd中。
+ `exec系列的函数`：属于替换进程映像(不产生新的进程)。把原来的进程替换掉来执行路径(或文件)指向的程序，从文件的main开始运行，执行完exec函数后原来的程序不再执行。头文件位于unistd中。

*exec*族函数名称特点：

+ l* 命令行参数列表
+ v* 使用命令行参数数组
+ p* 搜素*file*时使用*path*变量，file可以是name而不是绝对路径
+ e* 使用环境变量数组，不使用进程原有的环境变量，设置新加载程序运行的环境变量

事实上，只有*execve*是真正的系统调用，其它五个函数最终都调用*execve*：

![](https://img-blog.csdn.net/20140917065519447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdG90b3R1enVvcXVhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

+ `int execvp(const char *file,char *const argv[])`：会从PATH环境变量所指的目录中查找符合参数file的文件名，找到后便执行该文件，然后将第二个参数argv传给该欲执行的文件。如`execvp(“ls”,argv)`。
+ `int execve(const char *file,char *const argv[],char *const envp[])`：第一个参数file代表文件绝对路径，第二个参数是利用数组指针来传递给执行文件，最后一个参数则为传递给执行文件的新环境变量数组。如`execve(“/bin/ls”,argv,envp)`。

使用时最好在在一系列的参数结尾处都要加上一个空地址参数指针，最好使用`(char*)0`。如：

```c
#include <unistd.h>
 
void main(){
    char * argv[] = {"ls", "-al", "/etc/passwd", (char *)0};
    char * envp[] = {"PATH=/bin", 0};
 
    execve("/bin/ls", argv, envp);
}
```



## 系统通信

#### 1 多路复用

+ `int select(int n,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,struct timeval *timeout)`：用来等待文件描述词状态的改变。参数n代表最大的文件描述词加 1，参数readfds、writefds和exceptfds称为描述词组，是用来回传该描述词的读或写或例外的状况。位于sys/time、sys/types、unistd头文件中。[**非常重要，看这里**](https://www.cnblogs.com/alantu2018/p/8612722.html)。

#### 2 pipe通信

+ `int pipe(int filedes[2])`：会建立管道， filedes[0]为管道里的读取端，filedes[1]则为管道的写入端。头文件位于unistd中。
+ `FILE *popen(const char *command,const char *type)`：会调用fork()产生子进程，然后从子进程中调用/bin/sh -c来执行参数command的指令。依照type值，popen()会建立管道连到子进程的标准输出设备或标准输入设备，然后返回一个文件指针。随后进程便可利用此文件指针来读取子进程的输出设备或是写入到子进程的标准输入设备中。此外除fclose()外所有使用文件指针(FILE*)操作的函数也都可以使用。头文件位于stdio中。
+ `int pclose(FILE *stream)`：用来关闭由popen所建立的管道及文件指针。参数stream为先前由popen()所返回的文件指针。头文件位于stdio中。

#### 3 socket通信

+ `int socket(int domain,int type,int protocol)`：用来建立一个新的socket，也就是向系统注册，通知系统建立一通信端口。参数domain指定使用何种的地址类型，完整的定义在/usr/include/bits/socket.h内。头文件位于sys/types和sys/socket中。
+ `int shutdown(int s,int how)`：用来终止参数s所指定的 socket 连线。参数s是连线中的socket处理代码，参数how有下列几种情况：how=0终止读取操作; how=1终止传送操作;how=2终止读取及传送操作。头文件位于sys/types和sys/socket中。
+ `int bind(int sockfd,struct sockaddr *my_addr,int addrlen)`：用来设置给参数sockfd的socket一个名称。此名称由参数my_addr指向sockaddr结构，对于不同的socket domain定义了一 个通用的数据结构。头文件位于sys/types和sys/socket中。
+ `int listen(int s,int backlog)`：用来等待参数s的socket连线。参数backlog指定同时能处理的最大连接要求，如果连接数目达此上限则client端将收到ECONNREFUSED的错误。listen并未开始接收连线，只是设置socket为listen模式，真正接收client端连线的是accept()。通常listen()会在socket()，bind()之后调用，接着才调用accept ()。头文件位于sys/types和sys/socket中。
+ `int accept(int s,struct sockaddr *addr,int *addrlen)`：用来接受参数s的socket连线。参数s的socket必需先经bind、listen函数处理过，当有连线进来时accept()会返回个新的socket处理代码，往后的数据传送与读取就是经由新的 socket 处理，而原来参数s的socket能继续使用accept()来接受新的连线要求。连线成功时参数addr所指的结构会被系统填入远程主机的地址数据，参数addrlen为scokaddr的结构长度。头文件位于sys/types和sys/socket中。
+ `int connect(int sockfd,struct sockaddr *serv_addr,int addrlen)`：用来将参数sockfd的socket连至参数 serv_addr指定的网络地址，参数addrlen为sockaddr的结构长度。对于不同的socket domain定义了一 个通用的数据结构。头文件位于sys/types和sys/socket中。
+ `int recv(int s,void *buf,int len,unsigned int flags)`：用来接收远端主机经指定的socket传来的数据，并把数据存到由参数buf指向的内存空间，参数len为可接收数据的最大长度。参数flags一般设0。头文件位于sys/types和sys/socket中。还有其他的recv系列函数，自行阅读学习。
+ `int send(int s,const void *msg,int len,unsigned int falgs)`：用来将数据由指定的socket传给对方主机。参数s为已建立好连接的socket。参数msg指向欲连线的数据内容，参数len则为数据长度。参数flags一般设0。头文件位于sys/types和sys/socket中。还有其他的send系列函数，自行阅读学习。



## 其他函数

+ `char *crypt (const char *key,const char *salt)`：使用DES算法将参数key所指的字符串加以编码，key字符串长度仅取前8个字符。参数salt为两个字符组成的字符串，由`a-z、A-Z、0-9，.和/`所组成，用来决定使用4096种不同内建表格的哪一个。函数执行成功后会返回指向编码过的字符串指针(参数key所指的字符串不会有所更动)。编码过的字符串长度为13个字符，前两个字符为参数salt代表的字符串。**使用gcc编译时需加-lcrypt**。位于unistd头文件中。
+ `void *lsearch(const void *key,const void *base,size_t *nmemb,size_t size, int (*compar) (const void*,const void*))`：利用线性搜索从通用的数组中查找数据。参数key指向欲查找的关键数据，参数base指向要被搜索的数组开头地址，参数nmemb代表数组中的元素数量，每一元素的大小则由参数size决定，最后一项参数 compar为一函数指针，这个函数用来判断两个元素是否相同，若传给compar的第一个参数所指的元素数据和第二个参数所指的元素数据不相同时则返回非0值，两个元素数据不相同则返回非0值。**如果lsearch()找不到关键数据时会主动把该项数据加入数组里**。位于stdlib头文件中。
+ `void *bsearch(const void *key,const void *base,size_t nmemb,size_t size,int (*compar) (const void*,const void*))`：利用二元搜索从排序好的数组中查找数据。参数key指向欲查找的关键数据，参数base指向要被搜索的数组开头地址，参数nmemb代表数组中的元素数量，每一元素的大小则由参数size决定，最后一项参数compar为一函数指针，这个函数用来判断两个元素之间的大小关系，若第一个参数所指的元素数据大于第二个参数所指的元素数据则必须回传大于0的值没，两个元素数据不相同则返回非0值。位于stdlib头文件中。



**参考资料**

参考：[《Linux C函数库参考手册》](https://www.linuxidc.com/Linux/2011-11/47913.htm)

