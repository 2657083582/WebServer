## Linux系统编程入门

### 一、GCC

1、工作流程

源代码->预处理->汇编代码->目标代码->可执行程序

![](D:\WebServer\NoteBook\第一章\GCC工作流程.png)

2、常用参数选项

| GCC编译选项                                | 说明                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| -E                                         | 预处理指定源文件，不编译                                     |
| -S                                         | 编译指定源文件，但不进行汇编                                 |
| -c                                         | 编译、汇编指定源文件，但是不进行链接                         |
| -o [file1]$[file2] / $[file2]  -o [file1]$ | 将文件file2编译成可执行文件file1                             |
| -I directory                               | 指定include包含文件的搜索目录                                |
| -g                                         | 在编译的时候生成调试信息，该程序可以被调试器调试             |
| -D                                         | 在编译的时候，指定一个宏                                     |
| -w                                         | 不生成任何警告信息                                           |
| -Wall                                      | 生成所有警告                                                 |
| -on                                        | n的取值范围:0~3。编译器的优化选项的4个级别,-o0表示没有优化,-o1为缺省值，-o3优化级别最高 |

### 二、静态库的制作与使用

#### 库文件

1、库文件是计算机上的一类文件，可以简单的把库文件看成―种代码仓库，它提供给使用者一些可以直接拿来用的变量、函数或类。库文件分为静态库和动态库。

2、库是特殊的一种程序，编写库的程序和编写一般的程序区别不大，只是库不能单独运行。

3、静态库与动态库的区别：

静态库：静态库在程序的链接阶段被复制到了程序中。

动态库：动态库在链接阶段没有被复制到程序中，而是程序在运行时由系统动态加载到内存中供程序调用。

4、库文件的好处：

（1）代码保密；

（2）方便部署和分发。

#### 静态库

1、命名规则：libxxx.a

lib—前缀（固定）

xxx—库名（自定义）

a—后缀（固定）

2、静态库的制作

（1）gcc获得.o文件

（2）将.o文件打包，使用ar工具：```ar -rcs libxxx.a xxx.o xxx..o```

-r:将文件插入备存文件中

-c:创建备存文件

-s:索引

####  静态库的使用

```shell
gcc main.c -o app -I include -l ./calc -L ./lib
```

-I:指定包含头文件的路径

-L:指定静态库的路径

-l:指定静态库的名称

### 三、动态库的制作和使用

#### 动态库

命名规则

(1)Linux: libxxx.so

lib：前缀（固定）

xxx:库的名字（自定义）

.so：后缀（固定）

(2)Windows: libxxx.dll

#### 动态库的制作

1、gcc得到.o文件，得到和位置无关的代码

```shell
gcc -c -fpic/-fPIC a.c b.c
```

2、得到动态库

```shell
gcc -shared a.o b.o -o libcalc.so
```

#### 动态库的使用

1、使用方式

与静态库相似```gcc main.c -o app -I include -l ./calc -L ./lib```

2、工作原理

静态库：GCC进行链接时，会把静态库中代码打包到可执行程序中

动态库：GCC进行链接时，不会把动态库中代码打包到可执行程序中

程序启动之后，动态库会被动态加载到内存中，通过ldd(list dynamic dependencies)命令检查动态库依赖关系

定位共享库文件：当系统加载可执行代码时，知道其所依赖的库的名字，但还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，由ld_linux.so来完成，它先后搜索elf文件的DT_RPATH段->环境变量LD_LIBRARY_PATH->/etc/ld.so.cache文件列表->/lib/,/usr/lib目录找到库文件后将其加载到内存中。

3、配置环境变量

（1）临时配置

终端输入脚本：

```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:动态库路径
```

$表示获取初始的LD_LIBRARY_PATH的内容

（2）永久配置

a、用户级

配置.bashrc

```shell
(1)vim .bashrc
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:动态库路径
(2)source .bashrc
```

b、系统级

配置/etc/profile

```shell
(1)sudo vim /etc/profile
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:动态库路径
(2)source /etc/profile
```



使用时需要配置环境变量

### 四、静态库与动态库的对比

|      | 静态库                                                       | 动态库                                                       |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 优点 | 静态库北大包到应用程序中**加载速度快**<br />发布程序无需提供静态库，移植方便 | 可以实现进程间资源共享（共享库）<br />更新、部署、发布简单<br />可以控制何时加载动态库 |
| 缺点 | 消耗系统资源，浪费内存<br />更新、部署、发布麻烦             | 加载速度比静态库慢<br />发布程序时需要提供依赖的动态库       |

### 五、MakeFile

1、基础

（1）MakeFile文件定义了一系列的规则来指定那些文件需要先编译，哪些文件后编译，哪些文件需要重新编译

（2）作用：自动化编译。MakeFile一旦写好，只需要make命令，整个工程就能完全自动编译。

2、文件命名和规则

（1）文件命名：makefile或Makefile

（2）Makefile规则：一个Makefile文件中可以由一个或多个规则

```
目标 ...: 依赖 ...
	命令(Shell命令)
	...
```

目标：最终要生成的文件

依赖：生成目标所需的文件或目标

命令：通过执行命令对依赖操作生成目标（命令前必须Tab缩进）

3、工作原理

（1）命令在执行之间，需要先检查规则中的依赖是否存在

如果存在，执行命令；

如果不存在，向下检查其他规则，检查是否有一个规则是用来生产这个依赖的，若找到则指令命令。

（2）检查更新，在执行规则中的命令时，会比较目标和依赖文件的时间

如果依赖的时间比目标的时间晚，需要重新生成目标

如果依赖的时间比目标的时间早，目标不需要更新，对应规则中的命令不需要被执行

4、变量

（1）自定义变量

变量名=变量值  var=hello

（2）预定义变量

| 变量名 | 作用                           |
| ------ | ------------------------------ |
| AR     | 归档维护程序的名称，默认值为ar |
| CC     | C编译器的名称，默认值为CC      |
| CXX    | C++编译器的名称，默认值为g++   |
| $@     | 目标的完整名称                 |
| $<     | 第一个依赖文件的名称           |
| $^     | 所有依赖文件                   |

（3）获取变量的值

$(变量名)

5、模式匹配

%：通配符

6、函数

### 六、GDB调试

#### 什么是GDB调试

1、定义：GDB是由GNU软件系统社区提供的调试工具，同GCC配套组成了一套完整的开发环境，GDB是Linux和许多类Unix系统中的标准开发环境

2、功能：

（1）启动程序，可以按自定义的要求运行程序；

（2）可以让程序在所指定的调置的断点处停住；

（3）当程序被停住时，可以检查此时程序中所发生的事；

（4）可以改变程序，将一个BUG产生的影响修正从而测试其他BUG。

#### 准备工作

通常，在为调试而编译时，我们会关掉编译器的优化选项（-o)，并打开调试选项（-g)。

另外，’-Wall'在尽量不影响程序行为的情况下选项打开 所有warning，也可以发现许多问题，避免一些不必哟啊的BUG。,

```shell
gcc -g -Wall program.c -o program 
```

-g的作用实在可执行程序文件中加入源代码的信息，比如可执行文件种第几条机器指令对应源代码的第几行，但并不是把整个源文件嵌入到可执行文件中，所以在调试时必须保证gdb能找到源文件。

#### 常用命令

1、基础命令

| 命令                   | 含义                         |
| ---------------------- | ---------------------------- |
| gdb 可执行程序         | 启动                         |
| quit/q                 | 退出                         |
| set args 10 20         | 给程序设置参数               |
| show args              | 获取设置参数                 |
| help                   | 使用帮助                     |
| list/l                 | 从默认位置显示当前文件代码   |
| list/l 行号            | 从指定行显示当前文件代码     |
| list/l 函数名          | 从指定函数显示当前文件代码   |
| list/l 文件名：行号    | 从指定行显示指定文件的代码   |
| list/l 文件名：函数名  | 从指定函数显示指定文件的代码 |
| show list/listsize     | 显示行数                     |
| set list/listsize 行数 | 设置行数                     |

2、断点操作

| 命令                  | 含义                           |
| --------------------- | ------------------------------ |
| b/break 行号          | 在指定行设置断点               |
| b/break 函数名        | 在指定函数设置断点             |
| b/break 文件名：行号  | 在指定文件的指定行设置断点     |
| b/break 文件名：函数  | 在指定文件的指定函数设置断点   |
| i/info b/break        | 查看断点                       |
| d/del/delete 断点编号 | 删除断点                       |
| dis/disable 断点编号  | 设置断点无效                   |
| ena/enable 断点编号   | 设置断点生效                   |
| b/break 10 if i==5    | 设置断点条件（一般用于循环中） |

3、调试命令

| 命令                  | 含义                               |
| --------------------- | ---------------------------------- |
| start                 | 运行GDB程序（程序停在第一行）      |
| run                   | 运行GDB程序（遇到断电才停）        |
| c/continue            | 继续运行，到下一个断电才停         |
| n/next                | 向下执行一行代码（不会进入函数体） |
| p/print 变量名        | 打印变量值                         |
| ptype 变量名          | 打印变量类型                       |
| s/step                | 向下单步调试（遇到函数进入函数体） |
| finish                | 跳出函数体                         |
| display num           | 自动打印指定变量的值               |
| i/info display 编号   | 显示自动变量                       |
| i/info undisplay 编号 | 取消显示自动变量                   |
| set var 变量名=变量值 | 设置变量值                         |
| until                 | 跳出循环                           |

### 七、文件IO

![](D:\WebServer\NoteBook\第一章\标准C库IO函数.png)

#### 标准C库IO和Linux系统IO的关系

![](D:\WebServer\NoteBook\第一章\标准C库IO和Linux系统IO的关系.png)

#### 虚拟地址空间

![](D:\WebServer\NoteBook\第一章\虚拟地址空间.png)

进程是系统为程序分配资源的最小单位

程序是磁盘上的代码，进程（运行中的程序）会加载到内存里。

#### 文件描述符

![](D:\WebServer\NoteBook\第一章\文件描述符.png)

#### Linux系统IO函数

1、open函数

头文件：

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
```



(1) ```int open(const char *pathname,int flags);```

作用：打开一个已经存在的文件

参数：

​		-pathname:要打开的文件路径

​		-flags:对文件的操作权限设置还有其他的设置

​		O_RDONLY(只读)，O_WRONLY(只写),O_RDWR（可读写） 这三个参数互斥

返回值：返回一个新的文件描述符，如果调用失败，返回-1

errno： Linux系统函数库里面的一个全局变量，记录最近的错误号。

(2)```int open(const char *pathname,int flags,mode_t mode);```

参数：

​		-pathname:要打开的文件路径

​		-flags:对文件的操作权限设置还有其他的设置。

​				  flags参数是一个int类型的数据，占4字节，共32位。flags 32个位，每一位就是一个标志位。

​				-必选项：O_RDONLY(只读)，O_WRONLY(只写),O_RDWR（可读写） 这三个参数互斥

​				-可选项：O_CREATE 文件不存在，创建新文件

​		-mode:八进制的数，表示创建出的新的文件的操作权限，比如0775

​		最终的权限是mode & ~umask,umask的作用是抹去某些权限

2、close函数

3、read函数

```c
ssize_t read(int fd,void *buf,size_t count);
```

头文件：#include<unistd.h>

参数：

​			-fd:文件描述符，由open得到，通过这个文件描述符操作某个文件

​			-buf:需要读取数据存放的地方，数组的地址（传出参数）

​			-count:数组的大小

返回值：

​			-成功：

​					>0:返回时间的读取到的字节数

​					=0:文件已经读取完了

​			-失败：-1，并设置errno

4、write函数

```C
ssize_t write(int fd,cosnt void *buf,size_t count);
```

头文件:#include<unistd.h>

参数：

​			-fd:文件描述符，由open得到，通过这个文件描述符操作某个文件

​			-buf:要往磁盘中写入的数据

​			-count:要写的数据的实际大小

返回值：

​			-成功：实际写入的字节数

​			-失败：-1，并设置errno

利用open、read、write等函数实现文件复制的简易程序：

```c
#include<unistd.h>
#include<stdio.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<sys/fcntl.h>

int main(){
    
    //1.通过open打开文件
    int srcfd=open("english.txt",O_RDONLY);
    if(srcfd==-1){
        perror("open");
        return -1;
    }

    //2.创建一个新的文件——拷贝文件
    int destfd=open("cpy.txt",O_WRONLY | O_CREAT, 0064);
    if(destfd==-1){
        perror("open");
        return -1;
    }

    //3.频繁的读写操作
    char buf[1024]={0};
    int len=0;
    while((len=read(srcfd,buf,sizeof(buf)))>0){
        write(destfd,buf,len);
    }

    //4.关闭文件
    close(srcfd);
    close(destfd);

    return 0;
}
```

5、lseek函数

```c
off_t lseek(int fd,off_t offset,int whence);
```

头文件：#include<sys/types.h>

​			   #include<unistd.h>

参数：

​		-fd:文件描述符，通过open得到的，通过这个fd操作某个文件

​		-offset:偏移量

​		-whence:

​				SEEK_SET:直接设置文件指针的偏移量

​				SEEK_CUR:设置偏移量：当前位置+第二个参数offset的值

​				SEEK_END:设置偏移量：文件大小+第二份参数offset的值

返回值：返回文件指针的位置

作用：

​		（1）移动文件指针到头文件```lseek(fd,0,SEEK_SET);```

​		（2）获取当前文件指针的位置```lseek(fd,0,SEEK_CUR);```

​		（3）获取文件长度```lseek(fd,0,SEEK_END);```

​		（4）拓展文件长度，当前文件10b,拓展到110b,增加100个字节```lseek(fd,100,SEEK_END)l```

​			注意：需要写入一次数据

利用lseek拓展文件长度的简易程序：

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>
#include<unistd.h>
#include<stdio.h>
int main(){
    //打开文件
    int fd=open("hello.txt",O_RDWR);
    if (fd==-1)
    {
        perror("open");
        return -1;
    }

    //扩展文件的长度
    int ret=lseek(fd,100,SEEK_END);
    if(ret==-1){
        perror("lseek");
        return -1;
    }

    //写入空数据
    write(fd," ",1);

    //关闭文件
    close(fd);
    return 0;
}
```

6、stat函数与lstat函数

```c
int stat(const char *pathname,struct stat *statbuf);
```

作用：获取一个文件相关的信息

```c
int lstat(const char *pathname,struct stat *statbuf);
```

作用：lstat函数与 stat 函数类似，用于获取文件或目录的元数据信息。但是，lstat 函数对于符号链接文件会返回链接本身而不是链接所指向的文件的元数据信息。

头文件：\#include<sys/stat.h>

 			  #include<unistd.h>

参数：

​		-pathname：操作文件的路径

​		-statbuf:结构体变量，传出参数，用于保存获取到的文件的信息

补充：

stat结构体：

```c
struct stat{
	dev_t st_dev;						//文件的设备编号
	ino_t st_ino;						//节点
	mode_t st_mode;						//文件的类型和存取的权限
	nlink_t st_nlink;					//连到该文件的硬连接数目
	uid_t st_uid;						//用户ID
	gid_t st_gid;						//组ID
	dev_t st_rdev;						//设备文件的设备编号
	off_t st_size;						//文件字节数（文件大小）
	blksize_t st_blkszie;				//块大小
	blkcnt_t st_blocks;					//块数
	time_t st_attime;					//最后一次访问时间
	time_t st_mtime;					//最后一次修改时间
	time_t st_ctime;					//最后一次改变时间（指属性）
};
```

st_mode:

![](D:\WebServer\NoteBook\第一章\st_mode.png)

返回值：

​		成功：返回0

​		失败：返回-1，设置errno



利用stat等函数简单模拟实现ls -l

```C
//模拟实现li-l指令
//-rw-rw-r-- 1 nowcoder nowcoder 6 6月  25 11:03 a.txt
#include<stdio.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<pwd.h>
#include<grp.h>
#include<time.h>
#include<string.h>

int main(int argc,char * argv[]){

    //判断输入参数是否正确
    if(argc < 2){
        printf("%s filename\n",argv[0]);
        return -1;
    }

    //通过stat函数获取文件的信息
    struct stat st;
    int ret = stat(argv[1],&st);
    if(ret == -1){
        perror("stat");
        return -1;
    }

    //获取文件类型和文件权限
    char perms[11]={};

    switch(st.st_mode & S_IFMT){
        case S_IFLNK:
            perms[1]='l';
            break;
        case S_IFDIR:
            perms[0]='d';
            break;
        case S_IFREG:
            perms[0]='-';
            break;
        case S_IFBLK:
            perms[0]='b';
            break;
        case S_IFCHR:
            perms[0]='c';
            break;
        case S_IFSOCK:
            perms[0]='s';
            break;
        case S_IFIFO:
            perms[0]='p';
            break;
        default:
            perms[0]='?';
            break;
    }

    //判断文件的访问权限
    //文件所有者
    perms[1]=(st.st_mode & S_IRUSR)?'r':'-';
    perms[2]=(st.st_mode & S_IWUSR)?'w':'-';
    perms[3]=(st.st_mode & S_IXUSR)?'x':'-';

    //文件所在组
    perms[4]=(st.st_mode & S_IRGRP)?'r':'-';
    perms[5]=(st.st_mode & S_IWGRP)?'w':'-';
    perms[6]=(st.st_mode & S_IXGRP)?'w':'-';

    perms[7]=(st.st_mode & S_IROTH)?'r':'-';
    perms[8]=(st.st_mode & S_IWOTH)?'w':'-';
    perms[9]=(st.st_mode & S_IXOTH)?'x':'-';

    //硬连接数
    int linknum=st.st_nlink;

    //文件所有者
    char * fileUser = getpwuid(st.st_uid)->pw_name;

    //文件所在组
    char * fileGrp = getgrgid(st.st_gid)->gr_name;

    //文件大小
    long int fileSzie = st.st_size;

    //获取修改时间
    char * time = ctime(&st.st_mtime);
    char mtime[512] = {0};
    strncpy(mtime, time, strlen(time) - 1);

    char buf[1024];
    sprintf(buf,"%s %d %s %s %ld %s %s",perms,linknum,fileUser,fileGrp,fileSzie,mtime,argv[1]);
    printf("%s\n",buf);
    return 0;
}
```

### 八、文件属性操作函数

1、access函数

```c
int access(const char *pathname,int mode);
```

作用：判断某个文件是否有某个权限，或者判断文件是否存在

头文件：#include<unistd.h>
参数：

​		-pahtname:判断的文件路径

​		-mode:

​				R_OK：判断是否有读权限

​				W_OK：判断是否有写权限

​				X_OK：判断是否有执行权限

​				F_OK：判断文件是否存在

返回值：成功返回0，失败返回-1

查看文件权限的简易程序实例：

```c
#include<unistd.h>
#include<stdio.h>

int main(){
    int ret=access("a.txt",F_OK);
    if(ret==-1){
        perror("access");
        printf("文件不存在");
        return -1;
    }
    printf("文件存在！");
    return 0;
}
```



2、chmod函数

```c
int chmod(const char *pathname,mode_t mode);
```

作用：修改文件的权限

头文件：#include<unistd.h>

参数：	

​		-pathname:需要修改的文件的路径

​		-mode:需要修改的权限值，八进制的数

返回值：成功返回0，失败返回-1

修改文件权限的简易程序实例：

```c
#include<unistd.h>
#include<sys/stat.h>
#include<stdio.h>

int main(){
    int ret=chmod("a.txt",0777);
    if(ret == -1){
        perror("chmod");
        return -1;
    }
    return 0;
}
```



3、chown函数

```c
int chown(const char *pathname,uid_t owner,gid_t group);
```

作用：修改文件的所有者或者所在组

头文件：#include<unistd.h>

参数：

​		-pathname：要更改的文件名

​		-owner:拥有者uid

​		-group:所属组gid

返回值：成功返回0，失败返回-1

4、truncate函数

```c
int truncate(const char *path,off_t length);
```

作用：缩减或者扩展文件的尺寸至指定大小

头文件：#include<unistd.h>

​			   #include<sys/types.h>

参数：

​		-path:需要修改的文件的路径

​		-length:修改后的目标大小

返回值：成功返回0，失败返回-1

扩展/缩减文件大小的简易程序实例：

```c
#include<unistd.h>
#include<sys/types.h>
#include<stdio.h>
/*
 *b.txt的大小：13	
 *利用truncate函数扩展/缩减文件大小
 */
 
int main(){
	//将文件大小扩展为20
    int ret=truncate("b.txt",20);
    //将文件大小缩减为5
    //int ret=truncate("b.txt",5);
    if(ret==-1){
        perror("truncate");
        return -1;
    }
    return 0;
}
```



### 九、目录操作函数

1、mkdir函数

```c
int mkdir(const *pathname,mode_t mode);
```

作用：创建一个目录
头文件：#include<sys/stat.h>

​			   #include<sys/types.h>

参数：

​		-pathname:创建的目录的路径

​		-mode:权限，八进制的数

返回值：成功返回0，失败返回-1

2、rmdir函数

```c
int rmdir(const char *pathname);
```

作用：删除空目录

头文件：#include<sys/stat.h>

​			   #include<sys/types.h>

参数：-pathname:创建的目录的路径

返回值：成功返回0，失败返回-1

3、rename函数

```c
int rename(const char *oldpath,const char *newpath);
```

作用：修改文件名称

头文件：#include<sys/stat.h>

​			   #include<sys/types.h>

参数：

​		-oldpath:要修改名称的文件的原路径

​		-newpath:修改后的的新路径（文件名）

返回值：成功返回0，失败返回-1

4、chdir函数

```c
int chdir(const char *path);
```

作用：修改进程的工作目录

头文件：#include<unistd.h>

参数：

​		-path:需要修改的工作路径

5、getcwd函数

```c
char *getcwd(char *buf,size_t size);
```

作用：获取当前工作目录

头文件：#include<unistd.h>

参数：

​		-buf:存储的路径，指向的是一个数组（传出参数）

​		-size:数组的大小

返回值：返回值指向一块内存，这个数据就是第一个参数



获取并修改当前工作目录的简易程序：			

```c
#include<unistd.h>
#include<stdio.h>
#include<sys/stat.h>
#include<sys/types.h>
#include<fcntl.h>

int main(){

    //获取当前的工作目录
    char buf[1024];
    getcwd(buf,sizeof(buf));
    printf("当前的工作目录是：%s\n",buf);

    //修改工作目录
    int ret=chdir("/home/nowcoder/Linux/lesson1-14/test/");
    if(ret==-1){
        perror("chdir");
        printf("sb xiongwei\n");
        return -1;
    }

    //创建一个新文件
    int fd=open("b.txt",O_CREAT | O_RDWR,0664);
    if(fd==-1){
        perror("open");
        return -1;
    }

    close(fd);

    //获取当前的工作目录
    char buf1[1024];
    getcwd(buf1,sizeof(buf1));
    printf("当前的工作目录是：%s\n",buf1);

    return 0;
}
```

### 十、目录遍历函数

1、opendir函数

```c
DIR *open(const *char name);
```

作用：打开一个目录

头文件：#include<sys/types.h>

​			   #include<dirent.h>

参数：-name:需要打开的目录名称

返回值：

​		DIR* 类型，理解为目录流

​		错误返回NULL



2、readdir函数

```c
struct dirent *readdir(DIR *dirp);
```

作用：读取目录

头文件：#include<dirent.h>

参数：-dirp:opendir的返回结果

返回值：

​		struct dirent，代表读取到的文件的信息

​		读取到末尾或失败了，返回NULL

```c
struct dirent{
	ino_t d_ino;				//此目录进入点的inode
	off_t d_off;				//目录文件开头至此目录进入点的位移
	unsigned short d_reclen;	//d_name的长度，不包含NULL字符
	unsignde char d_type;			//d_name所指的文件类型
	char d_name[256];			//文件名
}

d_type:
	DT_BLK		块设备
	DT_CHR		字符设备
    DT_DIR 		目录
    DT_LNK 		软连接
    DT_FIFO 	管道
    DT_REG 		普通文件
    DT_SOCK 	套接字
    DT_UNKNOWN	未知
```



3、closedir函数

```c
int closedir(DIR *dirp);
```

作用：关闭目录

头文件：#include<sys/types.h>

​			   #include<dirent.h>

参数：-dirp:opendir的返回结果

返回值：成功返回0，失败返回-1



统计文件夹下的普通文件数量的程序：

```c
#include<sys/types.h>
#include<dirent.h>
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

int getFileNum(const char *path);

int main(int argc,char *argv[]){
    if(argc<2){
        printf("%s path\n",argv[0]);
        return -1;
    }

    int num=getFileNum(argv[1]);

    printf("普通文件的个数为：%d\n",num);

    return 0;
}

//用于获取目录下所有普通文件的个数
int getFileNum(const char *path){

    //打开目录
    DIR * dir=opendir(path);
    if(dir == NULL){
        perror("opendir");
        exit(0);
    }

    struct dirent *ptr;

    //记录普通文件的个数
    int total=0;

    while((ptr=readdir(dir))!=NULL){
        //获取名称
        char *dname=ptr->d_name;
        //忽略.和..
        if(strcmp(dname,".")==0 || strcmp(dname,"..")==0){
            continue;
        }
        if(ptr->d_type==DT_DIR){
            //目录，需要继续读取这个目录
            char newpath[256];
            sprintf(newpath,"%s/%s",path,dname);
            total+=getFileNum(newpath);
        }
        if(ptr->d_type=DT_REG){
            //普通文件
            total++;
        }
    }

    //关闭目录
    closedir(dir);

    return total;
}
```

### 十一、dup、dup2函数

1、dup函数

```c
int dup(int oldfd);
```

作用：复制一个新的文件描述符,从空闲的文件描述符表中找一个最小的，作为新的拷贝的文件描述符

例如：

```
fd=3,int fd1=dup(fd);
fd指向的是a.txt，那么fd1也是指向a.txt
```

头文件：#include<unistd.h>

参数：-oldfd:文件描述符



利用dup函数复制文件描述符，并利用新的文件描述符操作文件的程序：

```c
#include<unistd.h>
#include<stdio.h>
#include<fcntl.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<string.h>

int main(){

    int fd=open("a.txt",O_RDWR | O_CREAT,0664);
    int fd1=dup(fd);
    if(fd1==-1){
        perror("dup");
        return -1;
    }

    printf("fd:%d,fd1:%d\n",fd,fd1);
    close(fd);
    
    char *str="hello,world!";
    int ret=write(fd1,str,strlen(str));
    if(ret==-1){
        perror("write");
        return -1;
    }

    close(fd1);
    return 0;
}
```



2、dup2函数

```c
int dup2(int oldfd,int newfd);
```

作用：重定向文件描述符

例如：

```
oldfd指向a.txt,newfd指向b.txt
调用函数成功后：newfd和b.txt做close,newfd指向了a.txt
oldfd必须是一个有效的文件描述符
如果Oldfd和newfd值相同，相当于什么都没有做
```



利用dup2函数复制文件描述符，并利用新的文件描述符操作文件的程序：

```c
#include<sys/types.h>
#include<sys/stat.h>
#include<unistd.h>
#include<fcntl.h>
#include<stdio.h>
#include<string.h>
int main(){
    int fd=open("1.txt",O_RDWR | O_CREAT,0664);
    if(fd==-1){
        perror("open");
        return -1;
    }

    int fd1=open("2.txt",O_RDWR | O_CREAT,0664);
    if(fd1==-1){
        perror("open");
        return -1;
    }

    printf("fd:%d,fd1:%d\n",fd,fd1);
    int fd2=dup2(fd,fd1);
    if(fd2==-1){
        perror("dup2");
        return -1;
    }

    //通过fd1去写数据，实际操作的是1.txt,而不是2.txt
    char *str="hello,dup2";
    int len=write(fd2,str,strlen(str));

    if(len==-1){
        perror("write");
        return -1;
    }

    printf("fd:%d,fd1:%d,fd2:%d\n",fd,fd1,fd2);

    close(fd);
    close(fd2);

    return 0;
}
```



### 十二、fcntl函数

```c
int fcntl(int fd,int cmd,.../*arg*/);
```

作用：（1）复制文件描述符

​			（2）设置/获取文件的状态标志

头文件：#include<unistd.h>

​			   #include<fcntl.h>

参数：

​		-fd:需要操作的文件描述符

​		-cmd:表示对文件描述符进行如何操作

​					-F_DUPFD:复制文件描述符，复制的是第一个参数fd,得到一个新的文件描述符（返回值）

​					```int ret=fcntl(fd,F_DUPFD);```

​					-F_GETFL:获取指定的文件描述符文件状态flag

​					获取的flag和open函数传递的flag是一样的。

​					-F_SETFL:设置文件描述符文件状态flag

​							必选项：O_RDONLY,O_WRONLY,O_RDWR

​							可选项：O_APPEND			表示追加数据

​										   O_NONBLOCK	  设置成非阻塞

补充：

阻塞和非阻塞：描述的是函数调用的行为。

**阻塞（Blocking）**：当一个线程或进程在执行某个操作时，如果无法立即完成该操作，则会进入阻塞状态。在阻塞状态下，该线程或进程会暂停执行，直到满足执行条件或时间过去后才会继续执行。在阻塞状态下，该线程或进程通常会主动放弃CPU的使用权，让其他可运行的线程或进程有机会执行。典型的阻塞操作包括等待用户输入、读取磁盘文件等。

**非阻塞（Non-blocking）**：当一个线程或进程在执行某个操作时，如果无法立即完成该操作，它会立即返回并告知调用者当前无法完成请求。换句话说，非阻塞操作不会导致线程或进程暂停执行。在非阻塞模式下，线程或进程可以继续执行其他任务，而不会等待被操作的事件完成。非阻塞操作通常会通过轮询或回调等方式来检查操作是否已经完成。



利用fcntl函数获取并设置文件状态的标志，修改文件的程序:

```c
#include<unistd.h>
#include<fcntl.h>
#include<stdio.h>
#include<string.h>
int main(){

    //复制文件描述符
    //int fd=open("1.txt",O_RDONLY);
    //int ret=fcntl(fd,F_DUPFD);

    //修改或获取文件状态flag
    int fd=open("1.txt",O_RDWR);
    if(fd==-1){
        perror("open");
        return -1;
    }

    //获取文件描述符状态flag
    int flag=fcntl(fd,F_GETFL);
    if(flag == -1) {
        perror("fcntl");
        return -1;
    }
    flag |= O_TRUNC;//flag=flag | O_APPEND


    //修改文件描述符状态的flag,给flag加入O_APPEND这个标记
    int ret=fcntl(fd,F_SETFL,flag);
    if(ret==-1){
        perror("fcntl");
        return -1;
    }

    char *str="hello";
    int w=write(fd,str,strlen(str));
    if(w==-1){
        perror("write");
        return -1;
    }

    close(fd);

    return 0;
}
```

