## Linux多进程开发

### 一、进程概述

#### 1、进程和程序

（1）程序：程序是包含一系列信息的文件，这些信息描述了如何在运行时创建一个进程：

- 二进制格式标识：每个程序文件都包含用于描述可执行文件格式的元信息。内核利用此信息来解释文件中的其他信息。

- 机器语言指令：对程序算法进行编码。

- 程序入口地址：表示程序开始执行时的起始指令位置。

- 数据：程序文件包含的变量初始值和程序使用的字面变量值。

- 符号表及重定位表：描述程序中函数和变量的位置及名称。这些表格有多重用途，其中包括调试和运行时的符号解析（动态链接）。

- 共享库和动态连接信息：程序文件所包含的一些字段，列出了程序运行时需要使用的共享库，以及加载共享库的动态连接器的路径名。

- 其他信息：程序文件还包含许多其他信息，用以描述如何创建进程。

（2）进程：进程是正在运行的程序的实例。是一个具有一定独立功能的程序关于某个数据集合的一次运行活动。他是操作系统动态执行的基本单元，在传统的操作系统中，进程既是基本的分配单位，也是基本的执行单位。

- 可以用一个程序来创建多个进程，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。
- 从内核的角度看，进程由用户内存空间和一系列内核数据成员组成，其中用户内核空间包含了程序代码及代码所用的变量，二内核数据结构则用于维护进程状态信息。
- 记录在内核数据结构中的信息包括许多与进程相关的标识号（IDs)、虚拟内存表、打开文件的描述符表、信号传递及处理的有关信息、进程资源使用及限制、当前工作目录和大量的其他信息。

#### 2、单道、多道程序设计

1. 单道程序，即在计算机内存中只允许一个程序运行。
2. 多道程序设计技术是在计算机内存中同时存放几道相互独立的程序，是他们在管理程序的控制下，相互穿插运行，两个或两个以上程序在计算机系统中同处于开始到结束之间的状态，这些程序共享计算机系统的资源。引入多道程序设计技术的根本目的是为了提高CPU的利用率。

- 对于一个单CPU系统来说，程序同时处于运行状态只是一种宏观的概念，他们虽然都已经开始运行，但就围观而言，任意时刻，CPU上运行的程序只有一个。
- 在多道程序设计模型中，多个进程轮流使用CPU。

#### 3、时间片

- 时间片（timeslice)又称为“量子”或“处理器片”,是操作系统分配给每个正在运行的进程微观上的一段CPU时间。
- 一段时间片通常为5 ms到800 ms。
- 时间片有操作系统内核的调度程序分配给每个进程。首先，内核会给每个进程分配相等的初始时间片，然后每个进程轮番地执行相应的时间，当所有进程都处于时间片耗尽的状态时，内核会重新为每个进程计算并分配时间片，如此往复。

#### 4、并行和并发

1. 并行（parallel）：指在同一时刻，有多条指令在多个处理器上同时执行。
2. 并发（concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，是的宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分为若干段，是多个进程快速交替执行。

#### 5、进程控制块

为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分配一个PCB(Processing Control Block)进程控制块，维护进程相关的信息，Linux内核的进程控制块是task_struct 结构体。

task_struct结构体所在位置：/usr/src/linux-headers-xxx/include/linux/sched.h

task_struct中一些重要的成员：

- 进程id:系统中每个进程的唯一id,用pid_t类型表示，是一个非负整数
- 进程状态：包括就绪、运行、挂起、停止等状态
- 进程切换时需保存和恢复的一些CPU寄存器
- 描述虚拟地址空间的信息
- 描述控制终端的信息
- 当前工作目录
- umask掩码
- 文件描述符表，包含很多指向file结构体的指针
- 和信号相关的信息
- 用户id和组id
- 会话（Session）和进程组
- 进程可以使用的资源上限（Resource Limit)

### 二、进程状态转换

#### 1、进程的状态

- 进程的状态反映进程执行过程的变化。这些状态随着进程的执行和外界条件的变化而转换。

  - 三态模型：在三态模型中，进程状态分为三个基本状态，即就绪态、运行态、阻塞态。

    ![](D:\WebServer\NoteBook\第二章\image\进程的状态-三态模型.png)

    - 运行态：进程占有处理器正在运行

    - 就绪态：进程具备运行条件，等待系统分配处理器以便运行。

      当进程已分配到除CPU以外的所有必要资源后，只要再获得CPU，便可以立即执行。

      在一个系统中处于就绪状态的进程可能有多个，通常将它们排成一个队列，成为就绪队列。			

    - 阻塞态：又称为等待态或睡眠态，之进程不具备运行条件，正在等待某个事件的完成。

  - 五态模型：包括新建态、就绪态、运行态、阻塞态和终止态

    ![](D:\WebServer\NoteBook\第二章\image\进程的状态-五态模型.png)

    - 新建态：进程刚被创建时的状态，尚未进入就绪队列

    - 终止态：进程完成任务到达正常结束点，或出现无法克服的错误而异常终止，或被操作系统及有终止权的进程所终止时所处的状态。

      进入终止态的进程以后不再执行，但依然保留在操作系统中等待善后。	

      一旦其他进程完成了对终止态进程的信息抽取后，操作系统将删除该进程。

#### 2、进程相关的命令

1. 查看进程 

   - 命令 :ps

   ```shell
   ps aux / ajx
   ```

   ​	a:显示终端上的所有进程，包括其他用户的进程

   ​	u:显示进程的详细信息

   ​	x:显示没有控制终端的进程

   ​	j:列出与作业控制相关的信息

   - STAT参数意义

   | 参数 | 意义                              |
   | ---- | --------------------------------- |
   | D    | 不可中断                          |
   | R    | 正在运行，或在队列中的进程        |
   | S    | 处于休眠状态                      |
   | T    | 停止或被追踪                      |
   | Z    | 僵尸进程                          |
   | W    | 进入内存交换（从内核2.6开始无效） |
   | X    | 死掉的进程                        |
   | <    | 高优先级                          |
   | N    | 低优先级                          |
   | s    | 包含子进程                        |
   | +    | 位于前台的进程组                  |

2. 实时显示进程动态

   命令：top

   可以在使用top命令时加上 -d来指定显示信息更新的时间间隔，在top命令执行后，可以按以下按键对显示的结果进行排序：

   | 按键 | 排序方式                 |
   | ---- | ------------------------ |
   | M    | 根据内存使用量排序       |
   | P    | 根据CPU占有率排序        |
   | T    | 根据进程运行时间长短排序 |
   | U    | 根据用户名来筛选进程     |
   | K    | 输入指定的pid杀死进程    |

3. 杀死进程

   命令：kill

   ```shell
   kill [-signal] pid
   ```

   kill -l  列出所有信号

   kill -SIGKILL 进程ID

   kill -9 进程ID

   killall name 根据进程名杀死进程

#### 3、进程号和相关函数

- 每个进程都由进程号来标识，其类型为pid_t(整型)，进程好的范围：0~32767。

  每个进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。

- 任何进程（除init进程）都是由另一个进程创建，该进程为被创建进程的父进程，对应的进程号称为父进程号（PPID)。

- 进程组是一个或多个进程的集合。他们之间相互关联，进程组可以接受同一终端的各种信号，关联的进程有一个进程组号（PGID）。默认情况下，当前进程号会当做当前的进程组号。

- 进程号和进程组相关函数：

  - ```c
    pid_t getpid(void);
    ```

  - ```c
    pid_t getppid(void);
    ```

  - ```c
    pid_t getpgid(pid_t pid);
    ```

#### 4、进程创建

系统允许一个进程创建新进程，新进程即为子进程，子进程还可以创建新的子进程，形成进程树结构模型。

- fork函数

  ```c
  #include<sys/types.h>
  #include<unistd.h>
  
  pid_t fork(void);
  ```

  作用：用于创建子进程

  返回值：fork()会返回两次。一次是在父进程中，一次是在子进程中。

  - 成功：子进程中返回0，父进程中返回子进程ID
  - 失败：返回-1

  失败的两个主要原因：

  ​	1、当前系统的进程数已经达到了系统规定的上限，这是errno会被设置为EAGAIN

  ​	2、系统内存不足，这时errno会被设置为ENOMEN

利用fork函数创建子进程：

```c
#include<sys/types.h>
#include<unistd.h>
#include<stdio.h>

int main(){
    //创建子进程
    pid_t pid = fork();

    //判断是父进程还是子进程
    if(pid>0){
        //当前是父进程：返回值大于0，返回的是创建的子进程的进程号
        printf("pid:%d\n",pid);
        printf("I am parent process,pid:%d,ppid:%d\n",getpid(),getppid());
    }else if(pid==0){
        //当前是子进程：返回值为0
        printf("I am child process,pid:%d,ppid:%d\n",getpid(),getppid());
    }else{
        //创建失败：返回值为-1
        perror("fork");
        return -1;
    }

    for(int i=0;i<5;++i){
        printf("i:%d\n",i);
        sleep(1);
    }

    return 0;
}
```

#### 5、父子进程虚拟地址空间情况

实际上，Linux的fork()使用是通过写时拷贝（copy-on-write)实现。

写时拷贝是一种可以推迟甚至避免拷贝数据的技术。

内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。

只用在需要写入的时候才会复制地址空间，从而使各个进程拥有各自的地址空间。

也就是说，资源的复制是在需要写入的时候才会进行，在此之前，只有以只读的方式共享。

注意：fork之后父子进程共享文件，fork产生的子进程与父进程相同的文件描述符指向相同的文件表，引用计数增加，共享文件偏移指针。

#### 6、父子进程关系及GDB多进程调试

1. 父子进程之间的关系

   - 区别：

     - fork()函数的返回值不同

       ​	父进程中，返回子进程的ID

       ​	子进程中，返回值为0

     - PCB中的一些数据不同

       ​	当前进程的id、pid

       ​	当前进程的父进程的id、pid

       ​	信号集

   - 共同点：

     - 某些状态下（子进程刚被创建出来，还没有执行任何写数据的操作时），以下信息相同:

       ​	-用户区的数据

     ​		   -文件描述符表

2. 父子进程对变量是不是共享的？

   - **读时共享**：子进程被创建，两个进程都没有做任何写的操作时是共享的。
   - **写时拷贝**：只有当某个进程进行写操作时，内核才会为进程复制资源，分配独立的空间。

3. GDB多进程调试

   - 使用GDB调试的时候，GDB默认只能跟踪一个进程，默认是跟踪父进程。可以在fork函数调用之前，通过指令设置GDB调试工具跟踪父进程或子进程。

     - 设置调试父进程或者子进程：```set follow-fork-mode [parent(默认) | child]```

     - 设置调试模式：```set detach-on-fork [on | off]```

       默认为on,表示调试当前进程的时候，其他进程继续运行；

       如果为off，调试当前进程的时候，其他进程被GDB挂起。

     - 查看调试的进程：```info inferiors```
     - 切换当前调试的进程：```inferior num```
     - 使进程脱离GDB调试：```detach inferiors num```

### 三、exec函数族

#### 1、exec函数族介绍

- exec函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，换句话说，就是在调用进程内部执行一个可执行文件。

- exec函数族的函数执行成功后不会返回。因为调用进程的实体，包括代码段、数据段和堆栈等都已经被新的内容取代，只留下进程ID等一些表面上的信息。只有函数调用失败了，它们才会返回-1，从原程序的调用点从上往下执行。

#### 2、exec函数族

![](D:\WebServer\NoteBook\第二章\image\exec函数族.png)

1. execl

   ```c
   #include<unistd.h>
   
   int execl(const char *path,const char *arg,...);
   ```

   - 参数：

     ​	-path:需要指定的执行的文件的路径或名称（推荐使用绝对路径）

     ​	-arg:是执行可执行文件所需要的参数列表

     ​			第一个参数一般没有什么作用，为了方便面，一般写的是执行的程序的名称。

     ​			从第二个参数开始往后，就是程序执行所需要的参数列表。

     ​			参数最后需要以NULL结束（哨兵)

   - 返回值：

     ​	失败：返回-1,并设置errno

     ​	成功：没有返回值

     

2. execlp：会到环境变量中查找指定的可执行文件，如果找到就执行，找不到就执行不成功

   ```c
   #include<unistd.h>
   
   int execl(const char *file,const char *arg,...);
   ```

   - 参数：

     ​	-file:需要指定的执行的文件名

     ​	-arg:是执行可执行文件所需要的参数列表

     ​			第一个参数一般没有什么作用，为了方便面，一般写的是执行的程序的名称。

     ​			从第二个参数开始往后，就是程序执行所需要的参数列表。

     ​			参数最后需要以NULL结束（哨兵)

   - 返回值：

     ​	失败：返回-1,并设置errno

     ​	成功：没有返回值

     

3. execv

   ```c
   #include<unistd.h>
   
   int execl(const char *path,const char *argv[]);
   ```

   argv是需要的参数的一个字符串数组，例如：

   ```c
   char *argv[]={"ps","aux",NULL};
   execv("/bin/ps",argv);
   ```

   

### 四、进程控制

#### 1、进程退出

- 进程退出流程

![进程退出流程](D:\WebServer\NoteBook\第二章\image\进程退出.png)

- exit函数和_exit函数

  - exit：标准C库进程退出函数

    ```c
    #include<stdlib.h>
    
    void exit(int status);
    ```

  - _exit： Linux系统调用，进程退出函数

    ```c
    #include<unistd.h>
    void _exit(int status);
    ```

    - 参数：-status:是进程退出时的一个状态信息。父进程回收子进程资源时可以获取到。

    注意：exit函数会先刷新I/O缓冲区并关闭文件描述符，再调用系统_exit函数；

    ​			而_exit函数不会刷新缓冲区。

    举个简单的例子：

    代码一：

    ```c
    #include<stdlib.h>
    #include<stdio.h>
    #include<unistd.h>
    
    int main(){
    	printf("hello\n");
    	printf("world");
    	
    	exit(0);
    }
    //程序执行结果：hello\nworld
    ```

    代码二：

    ```c
    #include<stdlib.h>
    #include<stdio.h>
    #include<unistd.h>
    
    int main(){
    	printf("hello\n");
    	printf("world");
    	
    	_exit(0);
    }
    //程序执行结果：hello
    ```

    分析：printf()函数属于C标准输出函数，它会将内容先保存到缓冲区。

    ​			第一次调用printf函数后有一个换行符，换行符会刷新缓冲区，此时输出hello；

    ​			而第二次调用printf函数时没有换行符，因此缓冲区并不会被刷新，那么world还在缓冲区中，此时：

    ​		针对代码一：调用exit函数，会刷新缓冲区，输出world,然后调用系统的_exit函数，程序退出；

    ​		针对代码二：调用_exit函数，不会刷新缓冲区，程序直接退出，缓冲区中的内容被释放，因此不会输出world。

#### 2、孤儿进程（Orphan Process)

- 父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为孤儿进程。
- 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为init,而init进程会循环地wait()它的已经退出的子进程。这样，当一个孤儿进程结束生命周期时，init进程就会为其做善后工作。因此，孤儿进程并不会有什么危害。

#### 3、僵尸进程（Zombie Process)

- 每个进程结束后，都会释放自己地址空间中的用户数据，内核区的PCB没有办法自己释放掉，需要父进程去释放。
- 进程终止时，父进程尚未回收，子进程残留资源（PCB)仍存放在内核中，就会变成僵尸进程。
- 僵尸进程不能被kill -9命令杀死。
- 系统的进程号时有限的，而僵尸进程的进程号不会被释放，那么，如果有大量僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即僵尸进程的危害，应当避免。

#### 4、进程回收

- 每个进程退出时，内核会释放该进程的所有资源，包括打开的文件、占用的内存等。但是仍会为其保留一些信息（主要是进程控制块PCB的信息，包括进程号、退出状态、运行时间等）。

- 父进程可以通过调用wait和waitpid得到他的退出状态同时彻底清除这个进程。

  wait()和waitpid()功能一样，区别在于：

  ​	wait函数会阻塞；

  ​	waitpid函数可以设置不阻塞，还能指定等待哪个子进程结束。

​		注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环。

1. wait函数

   ```c
   #include<sys/types.h>
   #include<sys/wait.h>
   
   pit_t wait(int *wstatus);
   ```

   - 功能:等待任意一个子进程结束，如果任意一个子进程结束，此函数会回收子进程的资源。

     调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或收到一个不能被忽略的信号。

   - 参数：-wstatus:进程退出的状态信息你，传入的是一个int类型的地址，传出参数。

     退出信息相关的宏函数：

     | 宏函数               | 含义                                         |
     | -------------------- | -------------------------------------------- |
     | WIFEXITED(status)    | 非0，程序正常退出                            |
     | WEXITSTATUS(status)  | 如果宏为真，获取进程退出的状态（exit的参数） |
     | WIFSIGNALED(status)  | 非0，进程异常终止                            |
     | WTERMSIG(status)     | 如果宏为真，获取进程终止的信号编号           |
     | WIFSTOPPED(status)   | 非0，进程处于暂停状态                        |
     | WSTOPSIG(status)     | 如果宏为真，获取使进程暂停的信号编号         |
     | WIFCONTINUED(status) | 非0，进程暂停后已经继续运行                  |

   - 返回值：

     - 成功：返回被回收的子进程的id
     - 失败：返回-1（所有子进程都结束，调用函数返回失败）

```C
#include<sys/types.h>
#include<sys/wait.h>
#include<unistd.h>
#include<stdio.h>

int main(){

    //有一个父进程，创建了5个子进程（兄弟）

    pid_t pid;

    for(int i=0;i<5;++i){
        pid=fork();
        if(pid==0){
            break;
        }
    }

    if(pid>0){
        //父进程
        while(1){
            printf("I am parent process,pid:%d\n",getpid());

            int ret=wait(NULL);
            printf("child die,pid:%d\n",ret);

            sleep(1);
        }
    }else if(pid==0){
        //子进程
        while(1){
            printf("I am child process,pid:%d\n",getpid());
            sleep(1);
        }
    }else{
        perror("fork");
        return -1;
    }

    return 0;
}
```



1. waitpid函数

   ```c
   #include<sys/types.h>
   #include<sys/wait.h>
   
   pit_t wait(pid_t pid,int *wstatus,int options);
   ```

   - 功能：回收指定进程号的子进程，可以设置是否阻塞

   - 参数：

     ​		-pid:

     | pid  | 含义                                               |
     | ---- | -------------------------------------------------- |
     | >0   | 某个子进程的pid                                    |
     | =0   | 回收当前进程组的所有子进程                         |
     | =-1  | 回收所有的子进程，相当于wait() 最常用()            |
     | <-1  | 某个进程组的组id的绝对值，回收指定进程组中的子进程 |

     ​		-wstatus:与wait中的wstatus相同

     ​		-options:设置阻塞或非阻塞

     ​				0：阻塞

     ​				WNOHANG:非阻塞

   - 返回值：

     ​		>0:返回子进程的id;

     ​		=0: options=WNOHANG,表示还有子进程没有退出

     ​		=-1:错误，或者没有子进程了

### 五、进程间通信（IPC: Inter Processes Communication)

#### 1、概念

- 进程是一个独立的资源分配单元，不同进程（主要是用户进程）之间的资源是独立的，不能再一个进程中直接访问另一个进程的资源。
- 但是进程不是孤立的，不同进程间需要进行信息交互和状态的传递等，因此需要进程间通信。

- 进程间通信的目的：
  - 数据传输：一个进程需要将他的数据发送给另一个进程。
  - 通知事件：一个进程需要向另一个或一组进程发送消息，通知它发生了某种事件（如进程终止时要通知子进程）。
  - 资源共享：多个进程之间共享同样的资源。（需要内核提供互斥和同步机制）
  - 进程控制：有些进程希望完全控制力另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够即使知道它的状态改变。

#### 2、Linux进程间通信的方式

- 同一主机进程间通信
  - Unix进程间通信方式
    - 匿名管道
    - 有名管道
    - 信号
  - System V和POSIX进程间通信方式
    - 消息队列
    - 共享内存
    - 内存映射
    - 信号量
- 不同主机（网络）进程间通信：Socket

#####	匿名管道（PIPE)

管道也叫无名（匿名）管道，它是Unix系统IPC的最古老方式，所有Unix系统都支持这种通信方式。

- 管道的特点：

  （1）管道其实是一个在内核内存中维护的缓冲器，这个缓冲区的存储能力是有限的，不同的操作系统大小不一定相同。

  （2）管道拥有文件的特质：可以对管道进行读、写操作，匿名管道没有文件实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。

  （3）一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。

  （4）通过管道传递的数据是顺序的，从管道中读取出来的字节的顺序和他们被写入管道的顺序是完全一样的

  （5）在管道中的数据是单向传递的，一端用于写，另一端用于读，管道时半双工的。

  （6）从管道读取数据是一次性操作，数据一旦被读走，他就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用lseek()来随机访问数据。

  （7）匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系的进程）之间使用

- 管道的数据结构：循环队列

- 管道的使用

  - 创建匿名管道

    ```c
    #include<unistd.h>
    
    int pipe(int pipefd[2]);
    ```

    - 作用：创建一个匿名管道，用来进程间通信

    - 参数：int pipefd[2]是一个传出参数

      ​			pipefd[0]对应的是管道的读端；

      ​			pipefd[1]对应的是管道的写端。

    - 返回值：

      ​			成功：返回0

      ​			失败：返回-1

      注意：（1）匿名管道只能用于具有关系的进程之间的通信；

      ​			（2）管道默认是阻塞的：

      ​						如果管道中没有数据，read阻塞；

      ​						如果管道满了，write阻塞。

    利用匿名管道实现父子进程之间的通信：

    ```c
    #include<unistd.h>
    #include<stdio.h>
    #include<sys/types.h>
    #include<stdlib.h>
    #include<string.h>
    
    //子进程发送数据给父进程，父进程读取到数据输出
    int main(){
    
        //fork之前创建管道
        int pipefd[2];
        int ret=pipe(pipefd);
        if(ret==-1){
            perror("pipe");
            exit(0);
        }
    
        //创建子进程
        pid_t pid=fork();
        if(pid>0){
            //父进程
            printf("I am child process,pid:%d",getpid());
            char buf[1024]={0};
            while(1){
                //从管道的读取端读数据
                 int len=read(pipefd[0],buf,sizeof(buf));
                printf("parent receive :%s,pid:%d\n",buf,getpid());
    
                //写数据
                char *str="hello,I am parent";
                write(pipefd[1],str,strlen(str));
                sleep(1);
            }
    
        }else if(pid==0){
            //子进程
            printf("I am child process,pid:%d",getpid());
    
            char buf[1024]={0};
            while(1){
                //写数据
                char *str="hello,I am child";
                write(pipefd[1],str,strlen(str));
                sleep(1);
    
                int len=read(pipefd[0],buf,sizeof(buf));
                printf("child receive :%s,pid:%d\n",buf,getpid());
            }
        }else{
            //出错
            perror("fork");
            exit(0);
        }
    
        return 0;
    }
    ```

  - 查看管道缓冲大小的命令

    ```shell
    ulimit -a
    ```

  - 查看管道缓冲大小的函数

    ```c
    #include<unistd.h>
    
    long fpathconf(int fd,int name);
    ```

  利用fpathconf查看管道缓冲大小

  ```c
  #include<unistd.h>
  #include<sys/types.h>
  #include<stdlib.h>
  #include<stdio.h>
  
  int main(){
  
      int pipefd[2];
      int ret=pipe(pipefd);
      if(ret==-1){
          perror("pipe");
          exit(0);
      }
  
      //获取管道的大小
      long size=fpathconf(pipefd[0],_PC_PIPE_BUF);
  
      printf("pipe size:%ld\n",size);
  
      return 0;
  }
  ```

  

匿名管道通信案例：实现ps aux | grep xxx	父子进程间通信

```c
/*
    父子进程间通信
    子进程：ps aux,子进程结束后，将数据发送给父进程，
            将标准输出重定向到管道的写端
    父进程：获取到数据，过滤
*/

#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
#include<stdlib.h>
#include<wait.h>
#include<string.h>

int main(){

    //创建一个管道
    int fd[2];
    int ret=pipe(fd);
    if(ret==-1){
        perror("pipe");
        exit(0);
    }

    //创建子进程
    pid_t pid=fork();
    if(pid>0){
        //父进程
        //关闭写端
        close(fd[1]);
        //从管道中读数据
        char buf[1024]={0};

        int len=-1;
        while((len=read(fd[0],buf,sizeof(buf)-1))>0){
            //过滤数据输出
            printf("%s",buf);
            //清空数据
            memset(buf,0,1024);
        }
        wait(NULL);

    }else if(pid==0){
        //子进程
        //关闭读端
        close(fd[0]);

        //文件重定向 std_fileno->fd[1]
        dup2(fd[1],STDOUT_FILENO);

        //执行ps aux
        execlp("ps","ps","aux",NULL);
        perror("execlp");
        exit(0);

    }else{
        perror("fork");
        exit(0);
    }
    return 0;
}
```

- 管道读写的特点

使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）：

（1）所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0），有进程从管道的读端读数据，那么管道中剩余的数据被读取后，再次read会返回0，类似于读到文件末尾。

（2）如果有指向管道写端的文件描述符没有关闭（管道写端引用计数大于0），而持有管道写端的进程也没有往管道中写数据，这个时候有进程从管道中读取数据，那么管道中剩余的数据被读取后，再次read会阻塞，直到管道中有数据可以读了才会读取数据并返回。

（3）如果所有指向管道读端的文件描述符都关闭了（管道读端引用计数为0），这个时候有进程向管道中写数据，那么该进程会收到一个信号SIGPIPE，通常会导致程序异常终止。

（4）如果有指向管道独断的文件描述符没有关闭（管道读端引用计数大于0），而持有管道独断的进程也没有从管道中读数据，这时有进程向管道中写数据，那么在管道被写满时再次write会被阻塞，直到管道中有空位置才能再次写入数据并返回。

总结：

读管道：

​		-管道中有数据，read返回实际读到的字节数。

​		-管道中无数据：

​				写端被完全关闭，read返回0（相当于读到文件的末尾）

​				写端没有完全关闭，read阻塞等待

写管道：

​		-管道读端被完全关闭，进程异常终止（进程收到SIGPIPE信号）

​		-管道读端未完全关闭：

​				管道已满：write阻塞

​				管道未满：write将数据写入，并返回实际写入的字节数

- 设置管道非阻塞

  fcntl函数

  ```c
  int fd[2];
  int ret=pipe(fd);
  
  //获取原来的flag
  int flags=fcntl(fd[0],F_GETFL);
  //修改flag的值
  flags |= O_NONBLOCK;
  //设置新的flag
  fcntl(fd[0],F_SETFL,flags);
  ```



##### 有名管道(FIFO)

- 相关概念

  - 匿名管道由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道（FIFO)，也叫命名管道，FIFO文件。
  - 有名管道（FIFO)不同于匿名管道之处在于它提供了一个路径名与之相关联，以FIFO的文件形式存在于文件系统中，并且打开方式与打开一个普通文件是一样的，这样即使与FIFO的创建进程不存在亲缘关系的进程，只要访问该路径，就能彼此通过FIFO互相通信，因此，通过FIFO不相关的进程也能交换数据。
  - 一旦打开了FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的I/O系统调用了。与管道一样，FIFO也有一个读端和写端，并且从管道中读取数据的顺序与写入的顺序是一样的。
  - 有名管道和匿名管道的区别：
    - FIFO在文件系统中作为一个特殊文件存在，但FIFO中的内容却存放在内存中。
    - 当使用FIFO的进程退出后，FIFO文件将继续保存在文件系统中以便以后使用。
    - FIFO有名字，不相关的进程可以通过打开有名管道进行通信。

- 有名管道的使用

  - 通过命令创建有名管道```mkfifo 名字```

  - 通过函数创建有名管道

    ```c
    #include<sys/types.h>
    #include<sys/stat.h>
    int mkfifo(const char *pathname,mode_t mode);
    ```

    - 参数：

      ​	-pathname:管道名称的路径

      ​	-mode:文件的权限（和open函数的mode是一样的），是一个八进制的数

    - 返回值：成功返回0；失败返回-1，并设置errno

    使用mkfifo创建一个FIFO，就可以使用open打开它，常见的文件I/O函数都可用于FIFO。

    FIFO严格遵守先进先出，对管道及FIFO的读总是从开始处返回数据，对他们的写则是把数据添加到末尾。它们不支持lseek()等文件定位操作。

  - 注意事项

    - 一个为只读而打开一个管道的进程会阻塞，直到另外一个进程为只写打开管道
    - 一个为只写而打开一个管道的进程会阻塞，直到另外一个进程为只读打开管道

利用有名管道实现进程间通信（一个进程读数据，另一个进程写数据）：

read.c:读数据

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

//从管道中读数据
int main(){

    //打开管道文件
    int fd=open("test",O_RDONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }

    //读数据
    while(1){
        char buf[1024]={0};
        int len=read(fd,buf,sizeof(buf));
        if(len==0);{
            printf("写端断开了连接...\n");
            exit(0);
        }
        printf("recv buf: %s\n",buf);
    }

    return 0;
}
```

write.c：写数据

```c
#include<unistd.h>
#include<stdio.h>
#include<stdlib.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<string.h>

//想管道中写数据
int main(){

    //判断文件是否存在
    int ret=access("test",F_OK);
    if(ret==-1){
        printf("管道不存在，创建管道\n");

        //创建管道
        ret=mkfifo("test",0664);

        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }

    //以只写的方式打开管道
    int fd=open("test",O_WRONLY);
    if(fd==-1){
        perror("open");
        exit(0);
    }

    //写数据
    for(int i=0;i<100;++i){
        char buf[1024];
        sprintf(buf,"hello,%d\n",i);
        printf("write data : %s\n",buf);
        write(fd,buf,strlen(buf));
        sleep(1);
    }

    close(fd);
    return 0;
}
```

利用有名管道实现聊天功能：

chatA.c

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<stdlib.h>
#include<fcntl.h>
#include<string.h>

int main(){
    //1.判断有名管道是否存在
    int ret=access("fifo1",F_OK);
    if(ret==-1){
        //文件不存在
        printf("管道不存在，创建对应的有名管道\n");
        ret=mkfifo("fifo1",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }

    ret=access("fifo2",F_OK);
    if(ret==-1){
        //文件不存在
        printf("管道不存在，创建对应的有名管道\n");
        ret=mkfifo("fifo2",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }

    //2.以只写的方式打开管道1
    int fdw=open("fifo1",O_WRONLY);
    if(fdw==-1){
        perror("open");
        exit(0);
    }
    printf("打开管道fifo1成功，等待写入数据...\n");

    //3.以只读的方式打开管道2
    int fdr=open("fifo2",O_RDONLY);
    if(fdr==-1){
        perror("open");
        exit(0);
    }
    printf("打开管道fifo2成功，等待读取...\n");

    //4.循环的写、读数据
    char buf[128];
    pid_t pid=fork();

    if(pid>0){
        while(1){
            //清空数据
            memset(buf,0,128);
            //获取标准输入的数据
            fgets(buf,128,stdin);
            //写数据
            ret=write(fdw,buf,strlen(buf));
            if(ret==-1){
              perror("write");
              exit(0);
            }
        }
    }else if(pid==0){
        while(1){
            //读数据
            memset(buf,0,128);
            ret=read(fdr,buf,128);
            if(ret==-1){
                perror("read");
                exit(0);
            }
            printf("buf:%s\n",buf);
        }
    }

    //5.关闭文件描述符
    close(fdw);
    close(fdr);

    return 0;
}
```

chatB.c

```c
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<stdlib.h>
#include<fcntl.h>
#include<string.h>

int main(){
    //1.判断有名管道是否存在
    int ret=access("fifo1",F_OK);
    if(ret==-1){
        //文件不存在
        printf("管道不存在，创建对应的有名管道\n");
        ret=mkfifo("fifo1",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }

    ret=access("fifo2",F_OK);
    if(ret==-1){
        //文件不存在
        printf("管道不存在，创建对应的有名管道\n");
        ret=mkfifo("fifo2",0664);
        if(ret==-1){
            perror("mkfifo");
            exit(0);
        }
    }

    //2.以只读的方式打开管道1
    int fdr=open("fifo1",O_RDONLY);
    if(fdr==-1){
        perror("open");
        exit(0);
    }
    printf("打开管道fifo1成功，等待读取...\n");

    //3.以只写的方式打开管道2
    int fdw=open("fifo2",O_WRONLY);
    if(fdw==-1){
        perror("open");
        exit(0);
    }
    printf("打开管道fifo2成功，等待写入数据...\n");

    //4.循环的读、写数据

    char buf[128];
    pid_t pid=fork();
    if(pid>0){
        while(1){
            //读数据
            memset(buf,0,128);
            ret=read(fdr,buf,128);
            if(ret==-1){
                perror("read");
                exit(0);
            }
            printf("buf:%s\n",buf);
        }
    }else if(pid==0){
        while(1){
            //清空数据
            memset(buf,0,128);
            //获取标准输入的数据
            fgets(buf,128,stdin);
            //写数据
            ret=write(fdw,buf,strlen(buf));
            if(ret==-1){
                perror("write");
                exit(0);
            }
        }    
    }

    //5.关闭文件描述符
    close(fdw);
    close(fdr);

    return 0;
}
```

##### 内存映射(Memory-mmaped I/O)

内存映射是将磁盘文件的数据映射到内存，用户通过修改内存就能修改磁盘文件。

![内存映射原理图](D:\WebServer\NoteBook\第二章\image\内存映射.jpg)

- 内存映射相关系统调用

  - mmap函数

    ```c
    #include<sys/mman.h>
    void *mmap(void *addr,size_t length,int prot,int flags,int fd,off_t offset);
    ```

    - 功能：将一个文件或设备的数据映射到内存中

    - 参数：

      ​	-addr:地址，一般设置为NULL，有内核指定

      ​	-length:要映射的数据的长度，不能为0。建议使用文件的长度。

      ​				获取文件长度：lseek stat

      ​	-prot:对申请的内存映射区的操作权限，可以设置为：

      ​				-PROT_EXEC：可执行权限

      ​				-PROT_READ：读权限

      ​				-PROT_WRITE：写权限

      ​				-PROT_NONE：无权限

      ​			要操作映射内存，必须要有读权限。

      ​	-flags:

      ​			-MAP_SHARED:映射区的数据会自动和磁盘文件同部，进程间通信必须设置这个选项

      ​			-MAP_PRIVATE:不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件（copy on write)

      ​			-MAP_ANONYMOUS:匿名映射

      ​	-fd:需要映射的那个文件的文件描述符

      ​			通过open得到，open的是一个磁盘文件。

      ​			注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。

      ​	-off_t:偏移量，一般不用。必须指定为4k的整数倍，0表示不偏移。

    - 返回值：

      ​	成功：返回创建的内存的首地址

      ​	失败：返回MAP_FAILED(其实就是void* -1),并设置errno

  - munmap函数
  
    ```c
    #include<sys/mman.h>
    int munmap(void *addr,size_t length);
    ```

    - 功能：shifangneicunyings

    - 参数：

      ​	-addr:要释放的内存的首地址
  
      ​	-length:要释放的内存的大小，要和mmap函数中的length参数的值一样

​	使用内存映射实现进程间通信：

1. 有关系的进程（父子进程）

   （1）还没有子进程时，通过唯一的父进程先创建内存映射区；

   （2）有了内存映射区后创建子进程

   （3）父子进程共享创建的内存映射区

   ```c
   #include<stdio.h>
   #include<sys/mman.h>
   #include<fcntl.h>
   #include<sys/types.h>
   #include<sys/wait.h>
   #include<unistd.h>
   #include<string.h>
   #include<stdlib.h>
   
   int main(){
   
       //1.打开一个文件
       int fd=open("test.txt",O_RDWR);
       //获取文件的大小
       int size=lseek(fd,0,SEEK_END);
   
       //2.创建内存映射区
       void *ptr=mmap(NULL,size,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
       if(ptr==MAP_FAILED){
           perror("mmap");
           exit(0);
       }
   
       //3.创建子进程
       pid_t pid=fork();
       if(pid>0){
           //父进程
           wait(NULL);
           char buf[64];
           strcpy(buf,(char*)ptr);
           printf("read data :%s\n",buf);
           
       }else if(pid==0){
           //子进程
           strcpy((char*)ptr,"hello,son!");
       }
       //关闭内存映射区
       munmap(ptr,size);
       return 0;
   }
   ```

   

2. 没有关系的进程间通信

   （1）准备一个大小不是0的磁盘文件

   （2）进程1：通过磁盘文件创建内存映射区，并得到一个操作这块内存的指针

   ​		  进程2：通过磁盘文件创建内存映射区，并得到一个操作这块内存的指针

   （3)使用内存映射区通信

   注意：内存映射区通信不会阻塞

- 内存映射的注意事项

  - 对mmap的返回值（ptr)直接做++操作，而没有保存，munmap会出错

    ```c
    void *ptr=mmap(...);
    ptr++;//可以对其进行++操作
    munmap(ptr,len);//错误，要保存地址
    ```

  - prot参数的权限应少于或等于open()函数的权限，否则会出错。

    建议open()函数中的权限和prot参数的权限保持一致。

  ​		例如：open时的权限为O_RDONLY,mmap时prot参数指定PROT_READ | PROT_WRITE会出错，mmap会返回MAP_FAILED

  - 偏移量必须是4k的整数倍，否则会返回MAP_FAILED

  - mmap调用失败的情况：

    - 第一个参数：length=0

    - 第三个参数：prot

      ​	-只指定了写权限

      ​	-prot参数指定的权限与open函数指定的权限冲突

  - 可以通过open的时候O_CREAT一个新文件来创建映射区，但是创建的文件大小不能为0，否则会调用失败，可以用lseek或者truncate函数对新的文件进行扩展

  - mmap后关闭文件描述符，对mmap映射没有影响

    ```c
    int fd=open("xxx");
    mmap(...,fd,0);
    close(fd);
    
    //mmap映射区还存在，只是创建映射区的fd被关闭，没有任何影响。
    ```

- 复制文件

  内存映射处理用于进程间通信，还可以进行文件复制

  思路：

  ​      （1）对原文件的内容进行内存映射

  ​      （2）创建一个新文件（拓展该文件）

  ​      （3）把新文件的数据映射到内存中

  ​      （4）通过内存拷贝将第一个文件的内存数据拷贝到新的文件内存中

  ​      （5）释放资源

  代码实例：

  ```c
  #include<stdio.h>
  #include<stdlib.h>
  #include<sys/mman.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<unistd.h>
  #include<fcntl.h>
  #include<string.h>
  
  int main(){
      //1.对原始文件进行内存映射
      int fd=open("english.txt",O_RDWR);
      if(fd==-1){
          perror("open");
          exit(0);
      }
  
      //获取原始文件的大小
      int size=lseek(fd,0,SEEK_END);
  
      //2.创建新文件（拓展该文件）
      int fd1=open("cpy.txt",O_RDWR | O_CREAT,0664);
      if(fd==-1){
          perror("open");
          exit(0);
      }
  
      //对新文件进行拓展
      truncate("cpy.txt",size);
      write(fd1," ",1);
  
      //3.分别做内存映射
      void *ptr=mmap(NULL,size,PROT_READ |PROT_WRITE,MAP_SHARED,fd,0);
      void *ptr1=mmap(NULL,size,PROT_READ |PROT_WRITE,MAP_SHARED,fd1,0);
  
      if(ptr==MAP_FAILED){
          perror("mmap");
          exit(0);
      }
  
      if(ptr1==MAP_FAILED){
          perror("mmap");
          exit(0);
      }
  
      //4.内存拷贝
      memcpy(ptr1,ptr,size);
  
      //5.释放资源
      munmap(ptr1,size);
      munmap(ptr,size);
  
      close(fd1);
      close(fd);
  }
  ```

- 匿名映射:不需要文件实体进程的内存映射

  通过匿名映射实现进程间的通信：

  ```c
  #include<stdio.h>
  #include<stdlib.h>
  #include<sys/mman.h>
  #include<sys/types.h>
  #include<sys/stat.h>
  #include<unistd.h>
  #include<fcntl.h>
  #include<string.h>
  #include<wait.h>
  
  int main(){
  
      //1.创建匿名内存映射区
      int len=4096;
      void *ptr=mmap(NULL,len,PROT_READ | PROT_WRITE,MAP_SHARED | MAP_ANONYMOUS,-1,0);
      if(ptr==MAP_FAILED){
          perror("mmap");
          exit(0);
      }
  
      //2.父子进程间通信
      pid_t pid=fork();
      if(pid>0){
          //父进程
          strcpy((char*)ptr,"hello,world");
          wait(NULL);
      }else if(pid==0){
          //子进程
          sleep(1);
          printf("I am child,pid:%d,info:%s\n",getgid(),(char*)ptr);
      }
  
      //3.释放内存映射区
      int ret=munmap(ptr,len);
      if(ret==-1){
          perror("munmap");
          exit(0);
      }
  
      return 0;
  }
  ```


##### 信号

1. 信号概述

   - 信号有时也称为软件中断，是事件发生时对进程的通知机制，他只在软件层次上对中断机制的一种模拟，是一种异步通信方式。信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件。
   - 信号通常源于内核，引发内核为进程产生信号的各类时间如下：
     - 对于前台进程，用户可以通过输入特殊终端字符来给它发送信号。比如Ctrl+C通常会给进程发送一个中断信号。
     - 硬件发生异常，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应的信号给相关进程。比如执行了一条异常的机器语言指令，或者引用了无法访问的内存区域。
     - 系统状态变化，比如alarm定时器到期将引起SIGALRM信号，进程执行的CPU时间超限，或者该进程的某个子进程退出。
     - 运行kill命令或调用kill函数
   - 使用信号的主要目的：
     - 让进程知道已经发生了一个特地的事情
     - 强迫进程执行她自己代码中的信号处理程序
   - 信号的特点：
     - 简单
     - 不能携带大量信息
     - 满足某个特定条件才发送
     - 优先级比较高
   - 查看系统的信号：```kill -l```

2. Linux信号一览表

   | 编号  | 信号名称          | 对应事件                                                     | 默认动作                   |
   | ----- | ----------------- | ------------------------------------------------------------ | -------------------------- |
   | 1     | SIGHUP            | 用户退出shell时，该shell启动的所有进程将收到这个信号         | 终止进程                   |
   | 2     | SIGINT            | 当用户按下<Ctrl+C>组合键时，用户终端向正在运行中的由该终端启动的进程发出此信号 | 终止进程                   |
   | 3     | SIGQUIT           | 当用户按下<Ctrl+\\>组合键时产生该信号，用户终端向正在运行中的由该终端启动的进程发出此信号 | 终止进程                   |
   | 4     | SIGILL            | CPU检查到某进程执行了非法指令                                | 终止进程并产生core文件     |
   | 5     | SIGTRAP           | 该信号由断点指令或其他trap指令产生                           | 终止进程并产生core文件     |
   | 6     | SIGABRT           | 调用abort函数时产生该信号                                    | 终止进程并产生core文件     |
   | 7     | SIGBUS            | 非法访问内存地址，包括内存对齐出错                           | 终止进程并产生core文件     |
   | 8     | SIGFPE            | 在发生致命的运算错误时发出。不仅包括浮点运算错误，还包括溢出及除数为0等所有的算法错误 | 终止进程并产生core文件     |
   | 9     | SIGKILL           | 无条件终止程序。该信号不能别忽略、处理和阻塞                 | 终止进程，可以杀死任何进程 |
   | 10    | SIGUSR1           | 用户定义的信号。即程序员可以在程序中定义并使用该信号         | 终止进程                   |
   | 11    | SIGSEGV           | 表示进程进行了无效内存访问（段错误）                         | 终止进程并产生core文件     |
   | 12    | SIGUSR2           | 另外一个用户自定义信号，程序员可以在程序中定义并使用该信号   | 终止进程                   |
   | 13    | SIGPIPE           | Broken pipe，向一个没有读端的管道写数据                      | 终止进程                   |
   | 14    | SIGALRM           | 定时器超时，超时的时间由系统调用alarm设置                    | 终止程序                   |
   | 15    | SIGTERM           | 程序结束信号，与SIGKILL不同的是，该信号可以被阻塞和终止。通常用来表示程序正常退出。执行shell命令kill时，缺省产生这个信号 | 终止程序                   |
   | 16    | SIGSTKFLT         | Linux早期版本出现的信号,先仍保留向后兼容                     | 终止进程                   |
   | 17    | SIGCHLD           | 子进程结束时，父进程会收到这个信号                           | 忽略这个信号               |
   | 18    | SIGCONT           | 如果进程已停止，则使其继续运行                               | 继续/忽略                  |
   | 19    | SIGSTOP           | 停止（暂停）进程的执行。信号不能被忽略、处理和阻塞           | 为终止进程                 |
   | 20    | SIGTSTP           | 停止终端交互进程的运行。按下<Ctrl+Z>组合键时发出这个信号     | 暂停进程                   |
   | 21    | SIGTTIN           | 后台进程读终端控制台                                         | 暂停进程                   |
   | 22    | SIGTTOU           | 该信号类似于SIGTTIN，在后台进程要向终端输出数据时发生        | 暂停进程                   |
   | 23    | SIGGURG           | 套接字上有紧急数据时，向当前正在运行的进程发出信号，报告有紧急数据到达。如网络带外数据到达 | 忽略该信号                 |
   | 24    | SIGXCPU           | 进程执行时间超过了分配给该进程的CPU时间，系统产生该信号并发送给该进程 | 终止进程                   |
   | 25    | SIGXFSZ           | 超过文件的最大长度设置                                       | 终止进程                   |
   | 26    | SIGVTALRM         | 虚拟时钟超时时产生该信号。类似于SIGALRM，但是该信号只计算该进程占用CPU的使用时间 | 终止进程                   |
   | 27    | SIGPROF           | 类似于SIGVTALRM,它不光包括该进程占用CPU时间，还包括执行系统调用时间 | 终止进程                   |
   | 28    | SIGWINCH          | 窗口变化大小时发出                                           | 忽略该信号                 |
   | 29    | SIGIO             | 此信号向进程指示发出了一个异步IO事件                         | 忽略该信号                 |
   | 30    | SIGPWR            | 关机                                                         | 终止进程                   |
   | 31    | SIGSYS            | 无效的系统调用                                               | 终止进程并产生core文件     |
   | 34~64 | SIGRTMIN~SIGRTMAX | Linux的实时信号，它们没有固定的含义（可以由用户自定义）      | 终止进程                   |

   

3. 信号的5种默认处理动作

   - 查看信号的详细信息：```man 7 signal```
   - 信号的5种默认处理动作
     - Term	终止进程
     - Ign        当前进程忽略掉这个信号
     - Core     终止进程并生成一个Core文件，Core文件用于保存进程异常退出的错误信息
     - Stop     暂停当前进程
     - Cont     继续执行当前被暂停的进程
   - 信号的几种状态：产生、未决、递达
   - SIGKILL和SIGSTOP信号不能被捕捉、阻塞或忽略，只能执行默认动作

4. 信号相关的函数

   - kill函数

     ```c
     #include<signal.h>
     int kill(pid_t pid,int sig);
     ```

     - 功能：给任何的进程或进程组 pid,发送任何的信号 sig

     - 参数：

       ​	-pid:需要发送给进程的id

       ​			pid>0:将信号发送给指定的进程

       ​			pid=0:将信号发送给当前的进程组

       ​			pid=-1:将信号发送给每一个有权限接收这个信号的进程

       ​			pid<-1:这个pid等于某个进程组的ID取反

       ​	-sig:需要发送给的信号的编号或时宏值，0表示不发送任何信号

     

     父进程中利用kill函数杀死子进程的程序：

     ```c
     #include<stdio.h>
     #include<sys/types.h>
     #include<signal.h>
     #include<unistd.h>
     
     int main(){
     
     	pid_t pid=fork();
     	if(pid>0){
     		//父进程
     		printf("parent process\n");
     		sleep(2);
     		printf("kill child process\n");
     		kill(pid,SIGINT);
     	}else if(pid==0){
     		//子进程
     		for(int i=0;i<5;++i){
     			printf("child process\n");
     			sleep(1);
     		}
     	}
     	
     	return 0;
     }
     ```

     

   - raise函数

     ```c
     #include<signal.h>
     int raise(int sig);
     ```

     - 功能：给当前进程发送信号

     - 参数：-sig:要发送的信号

     - 返回值：

       ​		成功返回0

       ​		失败返回非0

       ​		

   - abort函数

     ```c
     #include<signal.h>
     int abort(void);
     ```

     - 功能：发送SIGABRT信号给当前进程，杀死当前进程

   - alarm函数

     ```c
     #include<unistd.h>
     unsigned int alarm(unsigned int seconds);
     ```

     - 功能：设置定时器。调用函数开始倒计时，当倒计时为0时，函数会给当前进程发送一个SIGALARM信号。该函数是不阻塞的。

       SIGALARM：默认终止当前的进程，每个进程有且仅有一个定时器。如果多次调用alarm函数，前面调用的函数设置的时间会失效。

     - 参数：-seconds:倒计时时长，单位：秒。如果参数为0，定时器无效可以通过alarm(0)取消定时器。

     - 返回值：

       ​		如果前面没有调用此函数，则返回0；

       ​		如果前面调用过该函数，则返回前一个倒计时剩余时间。

     ```c
     #include<stdio.h>
     #include<unistd.h>
     
     int main(){
     
         printf("start...\n");
         unsigned int seconds=alarm(5);   //这里返回0
         
         printf("seconds=%d",seconds);   
         sleep(2);
     
         seconds=alarm(2);
         printf("seconds=%d",seconds);   //这里返回3
     
         while(1){}
     
         return 0;
     }
     /*
         程序运行结果：
         seconds=0
         seconds=3(2秒后输出)
         CLOCK（4秒后输出）
     */
     ```

     

   - setitimer函数

     ```c
     #include<sys/time.h>
     int setitimer(int which,const struct itimerval *new_value,struct itimerval *old_value);
     ```

     - 功能：设置定时器，可以替代alarm函数。精度为us,可以实现周期性的定时。

     - 参数：

       ​	-which:定时器以什么时间计时

       ​			ITIMER_REAL:真实时间，时间到达发送SIGALRM信号（常用）

       ​			ITIMER_VIRTUAL:用户时间，时间到达发送SIGVTALRM信号

       ​			ITIMER_PROF:以该进程在用户态和内核态下所消耗的时间来计算，时间到达发送SIGPROF信号

       ​	-new_value:设置定时器的属性

       ```c
       //定时器结构体
       struct itimerval{
       	struct timerval it_interval;	//每个阶段的时间，间隔时间
       	struct timerval it_value;		//延迟多长时间执行定时器
       };
       //时间结构体
       struct timerval{
       	time_t 		tv_sec;				//秒数
       	suseconds	tv_usec;			//微秒
       }
       ```

       ​	-old_value:记录上一次的定时的时间参数，一般不使用，指定为NULL

       - 返回值：

         ​	成功：返回0

         ​	失败：返回-1，并设置errno

5. 信号捕获

   - signal函数

     ```c
     #include<signal.h>
     typedef void (*sighandler_t) (int);
     sighandler_t signal(int signum,sighandler_t handler);
     ```

     - 功能：设置某个信号的捕捉行为

       SIGKILL和SIGSTOP不能被捕捉，也不能被忽略

     - 参数：

       ​		-signum:要捕捉的信号

       ​		-handler:捕捉到的信号要如何处理

       ​				-SIG_IGN：忽略信号

       ​				-SIG_DFL：使用信号默认的行为

       ​				-回调函数：这个函数是由内核调用的，程序员只负责写，定义捕捉到的信号如何处理信号。```typedef void (*sighandler_t) (int);```

     - 返回值：

       ​	成功：返回上一次注册的信号处理函数的地址。第一次调用返回NULL。

       ​	失败：返回SIG_ERR，设置错误号

   ```c
   #include<sys/types.h>
   #include<signal.h>
   #include<stdio.h>
   #include<stdlib.h>
   #include<sys/time.h>
   
   void myalarm(int num){
       printf("捕捉到的信号的编号是：%d\n",num);
       printf("xxxxxxx\n");
   }
   
   int main(){
   
       //注册信号捕捉
       //信号捕捉一定要在设置定时器之前
       //void (*sighandler_t)(int);函数指针，int类型的参数表示捕捉到的信号的值
       signal(SIGALRM,myalarm);
   
       struct itimerval new_value;
   
       //设置间隔时间
       new_value.it_interval.tv_sec=2;
       new_value.it_interval.tv_usec=0;
   
       //设置延迟时间，3秒后开始第一次定时
       new_value.it_value.tv_sec=3;
       new_value.it_value.tv_usec=0;
   
       int ret=setitimer(ITIMER_REAL,&new_value,NULL);
       printf("定时器开始了...\n");
   
       if(ret==-1){
           perror("setitimer");
           exit(0);
       }
   
       getchar();
   
       return 0;
   }
   ```

   - sigaction函数

     ```c
     int sigaction(int signum,const struct sigaction *act,struct sigaction *oldact);
     ```

     - 功能：检查或者改变信号的处理。信号捕捉

     - 参数：

       ​		-signum:需要捕捉的信号的编号或宏值

       ​		-act:捕捉到信号之后的处理动作

       ​		-oldact:上一次信号捕捉相关的设置，一般设置为NULL

     - 返回值：成功返回0，失败返回-1

     ```c
     struct sigaction{
     	void (*sa_handler)(int);//函数指针，指向的函数就是信号捕捉到之后的处理函数
         
     	void (*sa_sigaction)(int,siginfo *,void *);//不常用
         
     	sigset_t sa_mask;//临时阻塞信号集，信号捕捉函数执行过程中，临时阻塞信号
         
     	int sa_flags;/*使用哪一个信号处理对捕捉到的信号进行处理
     				  *这个值可以是0，表示sa_handler;
     				  *也可以是SA_SIGINFO,表示使用sa_sigaction
     				  */
         
     	void (*sa_restorer)(void);//已经废弃了
     }
     ```

     

6. 信号集

   - 信号集是一个用来管理和操作信号的数据结构，它是多个信号的集合，其系统数据结构类型为```sigset_t```

   - 在PCB中有两个非常重要的信号集。一个是“阻塞信号集”，另一个是“未决信号集"。这两个信号集都是内核使用位图机制实现的，但操作系统不允许我们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数来对PCB中的这两个信号集进行操作。

   - 信号的”未决“是一种状态，指的是从信号的产生到信号被处理前的这一段时间

   - 信号的”阻塞“是一个开关动作，指的是阻止信号被处理，但不是阻止信号的产生

   - 信号的阻塞就是让系统暂时保留信号，留待以后发送。由于另外有办法让系统忽略信号，所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。

   - 未决信号集和阻塞信号集的工作原理

     - 用户通过键盘Ctrl+C产生2号信号SIGINT（信号被创建）

     - 此时信号刚产生，但还未被处理（即未决状态）

       ​	-内核中会将所有未处理的信号存储在一个集合中（未决信号集）

       ​	-SIGINT信号会被存储在第二个标志位上	

       ​			-如果这个标志位的值位0，表示信号已被处理，不是未决状态

       ​			-如果这个标志为的值位1，表示信号未被处理，处于未决状态

     - 处于未决状态的信号是需要被处理的，在处理之前需要和另一个信号集（阻塞信号集）进行比较

       ​	-阻塞信号及默认不阻塞任何信号

       ​	-如果想要阻塞某个信号需要调用系统API

     - 在处理的时候和阻塞信号集中的标志位进行比较，看是不是对该信号设置阻塞了

       ​	-如果没有阻塞，这个信号就会被处理

       ​	-如果设置了阻塞，那么这个信号继续处于未决状态，直到阻塞解除，这个信号就被处理

   - 相关函数（对自定义的信号集进行操作）

     头文件```#include<signal.h>```

     （1）sigemptyset函数
     
     ```c
     int sigemptyset(sigset_t *set);
     ```
     
     - 功能：清空信号集中的数据，将信号集中的所有标志位置设置为0
     - 参数：set，传出参数，需要操作的信号集
     - 返回值：成功返回0，失败返回-1

     （2）sigfillset函数
     
     ```c
     int sigfillset(sigset_t *set);
     ```
     
     - 功能：将信号集中所有的标志位置1
     - 参数：set，传出参数，需要操作的信号集
     - 返回值：成功返回0，失败返回-1

     （3）sigaddset函数
     
     ```c
     int sigaddset(sigset_t *set,int signum);
     ```

     - 功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号

     - 参数：

       ​		-set:传出参数，需要操作的信号集

       ​		-signum:需要设置阻塞的那个信号

     - 返回值：成功返回0，失败返回-1

     （4）sigdelset函数
     
     ```c
     int sigdelset(sigset_t *set,int signum);
     ```

     - 功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号

     - 参数：

       ​		-set:传出参数，需要操作的信号集

       ​		-signum:需要设置不阻塞的那个信号

     - 返回值：成功返回0，失败返回-1

     （5）sigismember函数
     
     ```c
     int sigismember(const sigset_t *set,int signum);
     ```

     - 功能：判断某个信号是否阻塞

     - 参数：

       ​		-set:需要操作的信号集

       ​		-signum:需要判断的那个信号

     - 返回值：

       ​		1： signum被阻塞

       ​		0： signum不被阻塞
     
       ​		-1：调用失败
       
       ```c
       #include<signal.h>
       #include<stdlib.h>
       #include<stdio.h>
       
       int main(){
           //创建一个信号集
           sigset_t set;
       
           //清空信号集的内容
           sigemptyset(&set);
       
           //判断SIGINT是否在信号集中set的
           int ret=sigismember(&set,SIGINT);
           if(ret==0){
               printf("SIGINT不阻塞\n");
           }else if(ret==1){
               printf("SIGINT阻塞\n");
           }else{
               perror("sigismember");
               exit(0);
           }
       
       
           //添加几个信号到信号集中的
           sigaddset(&set,SIGINT);
           sigaddset(&set,SIGQUIT);
       
           //判断SIGINT是否在信号集中set的
           ret=sigismember(&set,SIGINT);
           if(ret==0){
               printf("SIGINT不阻塞\n");
           }else if(ret==1){
               printf("SIGINT阻塞\n");
           }else{
               perror("sigismember");
               exit(0);
           }
       
           //判断SIGQUIT是否在信号集中set的
           ret=sigismember(&set,SIGQUIT);
           if(ret==0){
               printf("SIGINT不阻塞\n");
           }else if(ret==1){
               printf("SIGINT阻塞\n");
           }else{
               perror("sigismember");
               exit(0);
           }
       
           sigdelset(&set,SIGQUIT);
           //判断SIGQUIT是否在信号集中set的
           ret=sigismember(&set,SIGQUIT);
           if(ret==0){
               printf("SIGINT不阻塞\n");
           }else if(ret==1){
               printf("SIGINT阻塞\n");
           }else{
               perror("sigismember");
               exit(0);
           }
       
           return 0;
       }
       ```
       
       
     
     （6）sigprocmask函数
     
     ```c
     int sigprocmask(int how,const sigset_t *set,sigset_t *oldset);
     ```
     
     - 功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
     
     - 参数：
     
       ​		-how:如何对内核阻塞信号金进行处理
     
       ​				SIG_BLOCK:将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变。假设内核中默认的阻塞信号是mask,mask | set
     
       ​				SIG_UNBLOCK:根据用于设置的数据，对内核中的数据进行接触阻塞。相当于mask &= ~set
     
       ​				SIG_SETMASK：覆盖内核中原来的值
     
       ​		-set:已经初始化好的用户自定义数据集
     
       ​		-oldset:保存设置之前内核中的阻塞信号集的状态，可以是NULL
     
     - 返回值：
     
       ​		成功返回0；
     
       ​		失败返回-1，并设置错误号：EFAULT、EINVAL
     
     （7）sigpending函数
     
     ```c
     int sigpending(sigset_t *set);
     ```
     
     - 功能：获取内核中的未决信号集
     - 参数：set,传出参数，保存内核中未决信号集的信息
     - 返回值：成功返回0，失败返回-1

7. 内核实现信号捕捉的过程

   ![](D:\WebServer\NoteBook\第二章\image\内核实现信号捕捉的过程.png)

8. SIGCHLD信号

   - SIGCHLD信号产生的条件：
     - 子进程终止时
     - 子进程收到SIGSTOP信号停止时
     - 子进程处于停止状态，接收到SIGCONT后唤醒时
   - 使用SIGCHLD信号可以解决僵尸进程的问题

利用SIGCHLD解决僵尸进程的例子：

```c
#include<signal.h>
#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<sys/wait.h>

void myFun(int num){
    printf("捕捉到的信号：%d\n",num);

    //回收子进程PCB的资源
    // wait(NULL);
    while(1){
        int ret=waitpid(-1,NULL,WNOHANG);
        if(ret>0){
            printf("child die,pid:%d\n",ret);
        }else if (ret==0){
            //说明还有子进程
            break;
        }else if(ret==-1){
            //说明没有子进程了
            break;
        }
    }
}

int main(){
    
    //提前设置好阻塞信号集，阻塞SIGCHLD，因为有可能子进程很快结束，父进程还没有注册完信号捕捉
    sigset_t set;
    sigemptyset(&set);
    sigaddset(&set,SIGCHLD);
    sigprocmask(SIG_BLOCK,&set,NULL);

    pid_t pid;
    for(int i=0;i<20;++i){
        //创建一些子进程
        pid=fork();
        if(pid==0){
            break;
        }
    }

    if(pid>0){
        //父进程
        //捕捉子进程死亡时发送的SIGCHLD信号
        struct sigaction act;
        act.sa_flags=0;
        act.sa_handler=myFun;
        sigemptyset(&act.sa_mask);

        sigaction(SIGCHLD,&act,NULL);
        
        //注册完信号捕捉以后，解除阻塞
        sigprocmask(SIG_UNBLOCK,&set,NULL);

        while(1){
            printf("parent process id:%d\n",getpid());
            sleep(1);
        }   
    }else if(pid==0){
        //子进程
        printf("child process pid:%d\n",getpid());
    }

    return 0;
}
```

##### 共享内存

1. 概述

   - 共享内存允许两个或多个进程共享物理内存的同一块区域（通常被称为段）。由于一个共享内存段会成为一个进程用户空间的一部分，因此这种IPC机制无需内核介入。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其他所有共享同一个段的进程可用。
   - 共享内存是效率最高的IPC技术
   - 

2. 共享内存使用步骤

   - 调用shmget()创建一个新的共享内存段或取得一个既有的共享内存段的标识符（即有其他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的共享内存标识符。

   - 使用shmat()来附上这段共享内存，使该段成为调用进程的虚拟内存的一部分。
   - 此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存，程序需要使用由shmat()调用返回的addr值，它是一个指向进程的虚拟地址空间中该共享内存段起点的指针。
   - 调用shmdt()来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存了。这一步是可选的，并且在进程终止时会自动完成这一步。
   - 调用shmctl()来删除共享内存段。只有当当前所有附加内存段的进程都与之分离之后内存段才会销毁。

3. 相关函数

   头文件：

   ```c
   #include<sys/ipc.h>
   #include<sys/shm.h>
   ```

   - shmget函数

     ```c
     int shmget(key_t key,size_t size,int shmflg);
     ```

     - 功能：创建一个新的共享内存段，或者获取一个既有的共享内存段标识

       ​			新创建的内存段中的数据都会被初始化为0

     - 参数：

       ​		-key: key_t类型是一个整型，通过这个找到或创建共享内存

       ​					一般使用16进制表示，非0值

       ​		-size: 共享内存的大小，不能为0

       ​		-shmflg:共享内存的属性

       ​					-访问权限

       ​					-附加属性：创建/判断共享内存是否存在

       ​							创建：IPC_CREAT

       ​							判断共享内存段是否存在：IPC_EXCL,需要和IPC_CREAT一起使用：```IPC_CREAT | IPC_EXCL | 0664```

     - 返回值：

       ​		成功：返回共享内存的引用的ID(后面对共享内存的操作都是通过这个值来进行的)

       ​		失败：返回-1，并设置errno

   - shmat函数

     ```c
     void *shmat(int shmid,const void *shmaddr,int shmflg);
     ```

     - 功能：和当前的进程关联

     - 参数：

       ​		-shmid:共享内存的标识(ID),由shmget获得

       ​		-shmaddr:申请的共享内存的起始地址，指定为NULL，由内核指定

       ​		-shmflg:对共享内存的操作

       ​				SHM_RDONLY:读权限，要操作共享内存必须有读权限

       ​				0：设置为0，系统会默认为读和写权限

     - 返回值：

       ​		成功：返回共享内存的首地址

       ​		失败：返回（*void)-1

   - shmdt函数

     ```c
     int shmdt(const void *shmaddr);
     ```

     - 功能：解除当前进程和共享内存的关联
     - 参数：shmaddr:共享内存的首地址
     - 返回值：成功返回0，失败返回-1

   - shmctl函数

     ```c
     int shmctl(int shmid,int cmd,struct shmid_ds *buf);
     ```

     - 功能：删除共享内存，共享内存要删除才会消失，创建共享内存的进程被销毁了对共享内存没有任何影响

     - 参数：

       ​		-shmid:共享内存的ID

       ​		-cmd:要做的操作

       ​				IPC_STAT:获取共享内存当前的状态

       ​				IPC_SET:设置共享内存的状态

       ​				IPC_RMID:标记共享内存被摧毁

       ​		-buf:需要设置或获取的共享内存的属性信息

       ​				IPC_STAT: buf存储数据

       ​				IPC_SET: buf中需要初始化数据，设置到内核中

       ​				IPC_RMID:没有用，NULL

       ```c
       struct shmid_ds {
                      struct ipc_perm shm_perm;    /* Ownership and permissions */
                      size_t          shm_segsz;   /* Size of segment (bytes) */
                      time_t          shm_atime;   /* Last attach time */
                      time_t          shm_dtime;   /* Last detach time */
                      time_t          shm_ctime;   /* Last change time */
                      pid_t           shm_cpid;    /* PID of creator */
                      pid_t           shm_lpid;    /* PID of last shmat(2)/shmdt(2) */
                      shmatt_t        shm_nattch;  /* No. of current attaches */
                      ...
                  };
       
       ```

   

   利用共享内存实现进程间的通信：

   shm_write.c

   ```c
   #include<stdio.h>
   #include<sys/ipc.h>
   #include<sys/shm.h>
   #include<string.h>
   #include<stdlib.h>
   
   int main(){
   
       //创建共享内存
       int shmid = shmget(100,4096,IPC_CREAT | 0664);
       if(shmid==-1){
           perror("shmget");
           exit(-1);
       }
       printf("shmid:%d\n",shmid);
   
       //和当前进程关联
       void *ptr = shmat(shmid,NULL,0);
   
       char *str="hello,world";
   
       //写数据
       memcpy(ptr,str,strlen(str)+1);
   
       printf("按任意键继续...\n");
       getchar();
   
       //解除关联
       shmdt(ptr);
   
       //删除共享内存
       shmctl(shmid,IPC_RMID,NULL);
   
       return 0;
   }
   ```

   shm_read.c

   ```c
   #include<stdio.h>
   #include<sys/ipc.h>
   #include<sys/shm.h>
   #include<string.h>
   #include<stdlib.h>
   
   int main(){
   
       //获取一个共享内存
       int shmid = shmget(100,0,IPC_CREAT);
       if(shmid==-1){
           perror("shmget");
           exit(-1);
       }
       printf("shmid:%d\n",shmid);
   
       //和当前进程关联
       void *ptr = shmat(shmid,NULL,0);
       if(ptr==(void*)-1){
           perror("shmat");
           exit(-1);
       }
   
       //读数据
       printf("%s\n",(char*)ptr);
   
       printf("按任意键继续...\n");
       getchar();
   
       //解除关联
       shmdt(ptr);
   
       //删除共享内存
       shmctl(shmid,IPC_RMID,NULL);
   
       return 0;
   }
   ```

   

   - ftok函数

     ```c
     key_t ftok(const char *pathname,int proj_id);
     ```

     - 功能：根据指定的路径名和int值，生成一个共享内存的key

     - 参数：

       ​		-pathname:指定一个存在且可访问的路径

       ​		-proj_id: int类型的值，但是系统调用只会使用其中的1个字节

     - 返回值：

       ​		成功：返回生成的key

       ​		失败：返回-1，并设置errno

4. 共享内存操作命令

   - ipcs命令
     - ```ipcs -a``` 	打印当前系统中所有进程间通信方式的信息
     - ```ipcs -m```     打印出使用共享内存进行进程间通信的信息
     - ```ipcs -q```     打印出使用消息队列进行进程间通信的信息
     - ```ipcs -s```     打印出使用信号进行进程间通信的信息
   - ipcrm命令
     - ```ipcrm -M shmkey```	移除用shmkey创建的共享内存段
     - ```ipcrm -m shmid```      移除用shmid标识的共享内存段
     - ```ipcrm -Q msqkey```    移除用msqkey创建的共享内存段
     - ```ipcrm -q msqid```      移除用msqid标识的共享内存段
     - ```ipcrm -S semkey ```    移除用semkey创建的共享内存段
     - ```ipcrm -s semid```      移除用semid标识的共享内存段

5. 共享内存和内存映射的区别：

   - 共享内存可以直接创建，内存映射需要磁盘文件（匿名映射除外）

   - 共享内存效率更高

   - 内存

     - 共享内存：所有进程操作的都是同一块共享内存
     - 内存映射：每个进程都在自己的虚拟地址空间中有一个独立的内存

   - 数据安全

     - 进程突然退出：

       ​		-共享内存还存在

       ​		-内存映射区消失

     - 运行进程的电脑死机、宕机：

       ​		-共享内存中的数据就没有了

       ​		-内存映射中的数据，由于磁盘文件中的数据还存在，所以内存映射区的数据还存在

   - 生命周期

     - 内存映射：进程退出，内存映射区销毁
     - 共享内存：进程退出，共享内存还存在，标记删除（所有的关联的进程数为0）或者关机，后共享内存才会释放。如果一个进程退出，会自动和共享内存进行取消关联

   ### 六、守护进程

   1. 进程组

      - 进程组是一组相关进程的集合，会话是一组相关进程组的集合。进程组会话是为支持shell作业控制而定义的抽象概念，用户通过shell能够交互地在前台或后台运行命令
      - 进程组由一个或多个进程共享同一进程标识符（PGID)的进程组成。一个进程组拥有一个进程组首进程，该进程是创建该组的进程，其进程ID为该进程组的ID，新进程会继承其父进程所属进程组ID。
      - 进程组拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个成员进程退出组的时刻。一个进程可能因为终止而退出进程组，也可能因为加入了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员。

   2. 会话

      - 会话是一组进程的集合。会话首进程是创建该新会话的进程，其ID会成为会话ID。新进程会继承其父进程的会话ID。
      - 一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。
      - 在任意时刻，会话中的其中一个进程组会成为终端前台进程组，其他进程组会成为后台进程组。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。
      - 当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程。

   3. 进程组、会话操作函数

      - ```pid_t getpgrp(void);```//获取当前的进程组id
      - ```pid_t getpgid(pid_T pid);```//获取指定的进程组的id
      - ```int setpgid(pid_t pid,pid_t pgid);```//设置进程组的id
      - ```pid_t getsid(pid_t pid);```//获取指定进程的会话id
      - ```pid_t setsid(void);```//设置进程的会话id

   4. 守护进程(Daemon Process)

      - 守护进程（精灵进程）时Linux中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周期性的执行某种任务或等待处理某些发生的事件。一般采用以d结尾的名字。

      - 守护进程的特点：

        （1）生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。

        （2）他在后台运行且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如SIGINT、SIGQUIT）。

      - Linux中大多数服务器就是用守护进程实现的。比如，Internet服务器inetd,Web服务器httpd等。

   5. 守护进程的创建步骤

      - 执行一个fork()，之后父进程退出，子进程继续执行；
      - 子进程调用setsid()开启一个新会话；
      - 清除进程的umask以确保当前守护进程创建文件和目录时拥有所需的权限；
      - 修改当前的工作目录，通常会改为根目录（/）；
      - 关闭守护进程从其父进程继承而来的所有打开着的文件描述符；
      - 在关闭了文件描述符0、1、2之后，守护进程通常会打开/dev/null并使用dup2()使所有这些文件描述符指向这个设备。
      - 核心业务逻辑

      示例：写一个守护进程，每个2 s获取系统时间，将这个时间写入到磁盘文件中

      ```c++
      #include<stdio.h>
      #include<sys/stat.h>
      #include<sys/types.h>
      #include<unistd.h>
      #include<fcntl.h>
      #include<sys/time.h>
      #include<signal.h>
      #include<time.h>
      #include<stdlib.h>
      #include<string.h>
      
      void work(int num){
          //捕捉到信号之后，获取系统时间
          time_t tm=time(NULL);
          struct tm *loc=localtime(&tm);
          
          char *str=asctime(loc);
          int fd=open("time.txt",O_RDWR | O_CREAT | O_APPEND,0664);
          write(fd,str,strlen(str));
          close(fd);
      }
      
      int main(){
          //1.创建子进程，退出父进程
          pid_t pid = fork();
          if(pid>0){
              exit(0);
          }
      
          //2.将子进程重新创建一个会话
          setsid();
      
          //3.设置掩码
          umask(022);
      
          //4.更改工作目录
          chdir("/home/nowcoder/");
      
          //5.关闭、重定向文件描述符
          int fd = open("/dev/null",O_RDWR);
          dup2(fd,STDIN_FILENO);
          dup2(fd,STDOUT_FILENO);
          dup2(fd,STDERR_FILENO);
      
          //6.业务逻辑
          //捕捉定时信号
          struct sigaction act;
          act.sa_flags=0;
          act.sa_handler=work;
          sigemptyset(&act.sa_mask);
          sigaction(SIGALRM,&act,NULL);
      
          struct itimerval val;
          val.it_value.tv_sec=2;
          val.it_value.tv_usec=0;
          val.it_interval.tv_sec=2;
          val.it_interval.tv_usec=0;
      
          setitimer(ITIMER_REAL,&val,NULL);
          while(1){
              sleep(10);
          }
          return 0;
      }
      ```

      

   

   
