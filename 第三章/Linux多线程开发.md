## Linux多线程开发

### 一、线程（thread)

1. 线程概述
   - 线程是允许应用程序并发执行多个任务的一种机制。一个进程可以包含多个线程。同一个程序中的所有线程均会独立执行相同程序，且共享同一份全局内存区域，其中包括初始化数据段、未初始化数据段以及堆内存段。
   - 进程是CPU分配资源的最小单位，线程是操作系统调度执行的最小单位
   - 线程是轻量级的进程，在Linux环境下的本质仍是进程
   - 查看指定进程的LWP号的命令：```ps -Lf pid```
2. 线程和进程的与区别
   - 进程间的信息难以共享。由于出去只读代码段外，父子进程并未共享内存，因此必须采用一些进程间通信方式，在进程间进行信息交换
   - 调用fork()来创建进程的代价相对较高，及时利用写时复制技术，仍需要复制诸如内存页表和文件描述符之类的多种进程属性，这意味着fork()调用在时间上的开销依然不菲。
   - 线程之间能够方便、快速地共享信息。只需要将数据复制到共享（全局或堆）变量中即可。
   - 创建线程比创建进程快得多。线程间是共享虚拟地址空间的，无需采用写时复制技术来复制内存，也无需复制页表。
3. 线程之间共享和非共享资源
   - 共享资源
     - 进程ID和父进程ID
     - 进程组ID和会话ID
     - 用户ID和用户组ID
     - 文件描述符
     - 信号处置
     - 文件系统相关信息
     - 虚拟地址空间（栈、.text除外）
   - 非共享资源
     - 线程ID
     - 信号掩码
     - 线程特有数据
     - error变量
     - 实时调度策略和优先级
     - 栈、本地变量和函数的调用链接信息

### 二、线程操作

1. 创建线程

   一般情况下，main函数所在的线程称为主线程，其余创建的线程称为子线程

   pthread_create函数

   ```c
   #include<pthread.h>
   int pthread_create(pthread_t *thread,const pthread_attr_t *attr,void *(*start_routine) (void *),void *arg);
   ```

   - 功能：创建一个子线程

   - 参数：

     - thread:传出参数，线程创建成功后，子线程的线程ID被写到改变量中。
     - attr:设置线程的属性，一般使用默认值——NULL
     - start_routine:函数指针，这个函数是子线程需要处理的逻辑代码
     - arg:给第三个参数使用，传参

   - 返回值

     - 成功，返回0

     - 失败，返回错误号

       获取错误号信息：```char *strerror(int errnum)```

   利用pthread_create函数创建子线程：

   ```c
   #include<stdio.h>
   #include<pthread.h>
   #include<string.h>
   #include<unistd.h>
   
   void *callback(void *arg){
       printf("child thread...\n");
       printf("arg value:%d\n",*(int*)arg);
       return NULL;
   }
   
   int main(){
       pthread_t tid;
   
       int num=10;
   
       //创建一个子线程
       int ret=pthread_create(&tid,NULL,callback,(void*)&num);
       if(ret!=0){
           char *errstr=strerror(ret);
           printf("error:%s\n",errstr);
       }
   
       for(int i=0;i<5;++i){
           printf("%d\n",i);
       }
   
       sleep(2);
   
       return 0;
   }
   ```

2. 终止线程

   pthread_exit函数

   ```c
   #include<pthread.h>
   void pthread_exit(void *retval);
   ```

   - 功能：终止一个线程，在哪个线程中调用，就表示终止哪个线程
   - 参数：retval:需要传递一个指针，作为一个返回值，可以在pthread_join()中获取

   利用pthread_exit终止线程：

   ```c
   #include<stdio.h>
   #include<pthread.h>
   #include<string.h>
   #include<unistd.h>
   
   void *callback(void *arg){
       printf("child thread,id:%ld\n",pthread_self());
       return NULL;
   }
   
   int main(){
       pthread_t tid;
   
       //创建一个子线程
       int ret=pthread_create(&tid,NULL,callback,NULL);
       if(ret!=0){
           char *errstr=strerror(ret);
           printf("error:%s\n",errstr);
       }
   
       for(int i=0;i<5;++i){
           printf("%d\n",i);
       }
   
       printf("tid:%ld,main thread id:%ld\n",tid,pthread_self());
       //让主线程退出，主线程退出时，不会影响其他正常运行的线程
       pthread_exit(NULL);
   
       return 0;
   }
   ```

   拓展：

   - pthread_self

     ```c
     pthread_t pthread_self(void);
     ```

     功能：获取当前线程的ID

   - pthread_equal函数

     ```c
     int pthread_equal(pthread_t t1,pthread_t t2);
     ```

     功能：比较两个线程ID是否相等

     不同操作系统中pthread_t类型是实现不一样，有的是无符号长整型，有的是使用结构体实现的，因此有些时候不能用“=="比较。

3. 连接已终止的线程

   pthread_join函数

   ```c
   #include<pthread.h>
   int pthread_join(pthread_t thread,void **retval);
   ```

   - 功能：和一个已终止的线程进行连接，回收子线程的资源

     ​			这个函数是阻塞的，调用一次只能回收一个子线程

     ​			一般在主线程中使用

   - 参数：

     - thread:需要回收的子线程的ID
     - retval:接收子线程退出时的返回值

   - 返回值：

     - 成功，返回0
     - 失败返回错误号

4. 线程分离

   pthread_detach分离线程

   ```c
   #include<pthread.h>
   int pthread_detach(pthread_t thread);
   ```

   - 功能：分离一个线程。被分离的线程在终止的时候，会自动释放资源返回给系统。

     ​			不能多次分离线程，否则会产生不可预料的行为。

     ​			不能去连接一个已经分离的线程，会报错。

   - 参数：thread:需要分离的线程的ID

   - 返回值：

     - 成功，返回0
     - 失败，返回错误号

5. 线程取消

   pthread_cancel函数

   ```c
   #include<pthread.h>
   int pthread_cancel(pthread_t thread);
   ```

   - 功能：取消线程（让线程终止）

     ​			取消某个线程，可以终止某个线程的运行。但是并不是立刻终止，而是当子线程执行到一个取消点，线程才会终止

### 三、线程属性

1. 相关函数

   - ```c
     int pthread_attr_init(pthread_attr_t *attr);
     ```

     初始化线程属性变量

   - ```c
     int pthread_attr_destroy(pthread_attr_t *attr);
     ```

     释放线程属性资源

   - ```c
     int pthread_attr_getdetachstate(const pthread_attr_t *attr,int *detachstate)
     ```

     获取线程分离的状态属性

   - ```c
     int phtread_attr_setdetachstate(phtread_attr_t *attr,int setachstate);
     ```

     设置线程分离的状态属性

2. 操作线程属性的例子

   ```c
   #include<stdio.h>
   #include<pthread.h>
   #include<string.h>
   #include<unistd.h>
   
   void *callback(void *arg){
       printf("child thread,id:%ld\n",pthread_self());
       sleep(3);
       return NULL;
   }
   
   int main(){
       //创建一个线程属性变量
       pthread_attr_t attr;
       //初始化属性变量
       pthread_attr_init(&attr);
   
       //设置属性
       pthread_attr_setdetachstate(&attr,PTHREAD_CREATE_DETACHED);
   
       pthread_t tid;
   
       //创建一个子线程
       int ret=pthread_create(&tid,&attr,callback,NULL);
       if(ret!=0){
           char *errstr=strerror(ret);
           printf("error:%s\n",errstr);
       }
   
       //获取线程栈的大小
       size_t size;
       pthread_attr_getstacksize(&attr,&size);
       printf("stack size:%ld\n",size);
       
       printf("tid:%ld,main thread id:%ld\n",tid,pthread_self());
   
       //释放线程属性资源
       pthread_attr_destroy(&attr);
       pthread_exit(NULL);
   
       return 0;
   }
   ```

   

### 四、线程同步

1. 线程同步

   - 线程能通过全局变量共享信息。但是，必须确保多个线程不会同时操作同一变量（临界区资源），否则会引发线程安全的问题。
   - 临界区是指访问某一共享资源的代码片段，并且这段代码的执行应为原子操作，也就是同时访问同一资源的其他线程不应该终止该片段的执行。
   - 线程同步：即当有一个线程在对内存进行操作时，其他线程都不可以对这个内存地址进行操作，直到线程完成操作，其他线程才能对该内存地址进行操作，而其他线程处于等待状态。

2. 互斥量

   - 互斥量（mutex)可以确保同时只有一个线程可以访问某项共享资源，保证对任意共享资源的原子访问。

   - 互斥量有两种状态：已锁定（locked)和未锁定（unlocked)。任何时候，至多只能有一个线程可以锁定该互斥量。试图对已锁定的某一互斥量再次加锁，可能导致线程阻塞或报错。
   - 一旦线程锁定互斥量，随即成为改互斥量的所有者，只有所有者才能给互斥量解锁。一般情况下，对每一共享资源会使用不同的互斥量，每一线程在访问同一资源时将采用如下协议：
     - 针对共享资源锁定互斥量
     - 访问互斥资源
     - 对互斥量解锁
     - 同时只有一个线程能进入临界区

   ![](D:\WebServer\NoteBook\第三章\image\互斥量如何保证线程同步.jpg)

   - 互斥量相关的操作函数

     - 互斥量的类型：```pthread_mutex_t```

     - 初始化互斥量：

       ```c
       int pthread_mutex_init(pthread_mutex_t *restrict mutex,const pthread_mutexattr_t *restrict attr);
       ```

       参数：

       ​		-mutex:需要初始化的互斥量变量

       ​		-attr:互斥量相关的属性，一般设置为NULL

       -restrict: C语言的修饰符，不能由另外的指针进行操作

     - 销毁互斥量：

       ```c
       int pthread_mutex_destroy(pthread_mutex_t *mutex);
       ```

     - 加锁（阻塞的，如果有一个线程加锁了，那么其他线程只能阻塞等待）

       ```c
       int pthread_mutex_lock(pthread_mutex_t *mutex);
       ```

     - 尝试加锁（非阻塞的，如果加锁失败，会直接返回）：

       ```c
       int pthread_mutex_trylock(pthread_mutex_t *mutex);
       ```

     - 解锁：

       ```c
       int pthread_mutex_unlock(pthread_mutex_t *mutex);
       ```

     利用互斥量实现多线程买票程序

     ```c
     /*
         使用多线程实现买票的案例。
         3个窗口，100张票
     */
     
     #include<stdio.h>
     #include<pthread.h>
     #include<unistd.h>
     
     int tickets=10000;//全局变量，所有线程共享同一个资源
     
     //定义互斥量
     pthread_mutex_t mutex;
     
     void * sellticket(void *arg){
         //卖票
         while(1){
             //加锁
             pthread_mutex_lock(&mutex);
             if(tickets>0){
                 usleep(6000);
                 printf("%ld 正在卖第%d张门票\n",pthread_self(),tickets);
                 tickets--;
             }else{
                 printf("票已售罄...\n");
             }
             //解锁
             pthread_mutex_unlock(&mutex);
     
         }
         return NULL;
     }
     
     int main(){
     
         //初始化互斥量
         pthread_mutex_init(&mutex,NULL);
     
         //创建三个子线程
         pthread_t tid1,tid2,tid3;
         pthread_create(&tid1,NULL,sellticket,NULL);
         pthread_create(&tid2,NULL,sellticket,NULL);
         pthread_create(&tid3,NULL,sellticket,NULL);
     
         //回收子线程资源，阻塞
         pthread_join(tid1,NULL);
         pthread_join(tid2,NULL);
         pthread_join(tid3,NULL);
     
     	//销毁互斥量
         pthread_mutex_destroy(&mutex);
     
         pthread_exit(NULL);//退出主线程
         
         return 0;
     }
     ```

3. 死锁

   - 有时，一个线程需要同时访问多个不同的共享资源，而每个资源又由不同的互斥量管理。当超过一个线程加锁同一组互斥量时，就有可能发生死锁。
   - 两个或两个以上的进程都在执行过程中，因争夺资源而造成的一种互相等待，若无外力作用，它们都将无法继续执行，这种现象称为死锁。
   - 死锁的几种场景：
     - 忘记释放锁
     - 重复加锁
     - 多线程多锁，抢占锁资源

4. 读写锁

   - 在对数据的读写操作中，更多的是读操作，写操作较少。为了满足当前能够有多个读出，但只允许一个写入操作，线程提供了读写锁来实现。

   - 读写锁的特点：

     - 如果有其他线程读数据，则允许其他线程执行读数据，但不允许写操作；
     - 如果有其他线程写数据，则其他线程都不允许读、写操作；
     - 写是独占的，优先级更高。

   - 读写锁相关操作函数

     - 读写锁的类型：```pthread_relock_t```

     - 初始化读写锁：

       ```c
       int pthread_rwlock_init(pthread_rwlock_t *retstrict rwlock,const pthread_rwlockattr_t *restrict attr);
       ```

     - 销毁读写锁

       ```c
       int pthread_rwlock_destroy(pthread_rwlock_t *rwlock);
       ```

     - 加上读锁

       ```c
       int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
       ```

     - 尝试加上读锁

       ```c
       int pthread_rwlock_tryrdlock(pthread_rwlock_t *rwlock);
       ```

     - 加上写锁

       ```c
       int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
       ```

     - 尝试加上写锁

       ```c
       int pthread_rwlock_trywrlock(pthread_rwlock_t *rwlock);
       ```

     - 解锁

       ```c
       int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);
       ```

5. 生产者和消费者模型

   - 生产者—消费者模型中的对象：
     - 生产者
     - 消费者
     - 容器
   - 示例：生产者向链表中写入数据，消费者从链表中读取数据

   ```c
   #include<stdio.h>
   #include<pthread.h>
   #include<stdlib.h>
   #include<unistd.h>
   
   struct Node{
       int num;
       struct Node *next;
   };
   
   struct Node *head =NULL;
   
   
   
   void *producer(void *arg){
       //不断创建新的节点，添加到链表中
       while(1){
           struct Node * newNode = (struct Node*)malloc(sizeof(struct Node));
           newNode->next=head;
           head=newNode;
           newNode->num=rand()%1000;
           printf("add node,num : %d,tid : %ld\n",newNode->num,pthread_self());
           usleep(1000);
       }
   
       return NULL;
   }
   
   void *customer(void *arg){
       
       while(1){
           //保存头节点的指针
           struct Node *temp=head;
           head=head->next;
           printf("add node,num : %d,tid : %ld\n",temp->num,pthread_self());
           usleep(1000);
       }
   }
   
   int main(){
       //创建5个生产者线程，5个消费者线程
       pthread_t ptid[5],ctid[5];
   
       for(int i=0;i<5;i++){
           pthread_create(&ptid[i],NULL,producer,NULL);
           pthread_create(&ctid[i],NULL,customer,NULL);
       }
   
       for(int i=0;i<5;i++){
           pthread_detach(ptid[i]);
           pthread_detach(ctid[i]);
       }
   
       pthread_exit(NULL);
   
   }
   ```

   程序运行结果：
   add node,num : 383,tid : 139877010171648
   add node,num : 383,tid : 139877001778944
   add node,num : 886,tid : 139876993386240
   段错误 (核心已转储)

   可能的原因：

   （1）由于生产者与消费者线程交替执行，就有可能导致生产者还未写入数据，而消费者就开始读取，导致段错误；

   （2）可能是生产者线程写入了一个数据之后，执行力多次消费者线程，多次读取数据，从而导致段错误。

6. 条件变量

   - 条件变量不是锁，它只是用于辅助互斥锁来提高程序运行效率的变量
   - 条件变量可以引起阻塞或者解除阻塞

   - 条件变量相关操作函数

     - 条件变量的类型：```pthread_cond_t```

     - 初始化条件变量

       ```c
       int pthread_cond_init(pthread_cond_t *restrict cond,const pthread_condattr_t *restrict attr);
       ```

     - 销毁条件变量

       ```c
       int pthread_cond_destroy(pthread_cond_t *cond);
       ```

     - 等待（等待生产者生产数据,该函数会阻塞线程）

       ```c
       int pthread_cond_wait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex);
       ```

     - 等待（等待一定时间，该函数会阻塞线程）

       ```c
       int pthread_cond_timedwait(pthread_cond_t *restrict cond,pthread_mutex_t *restrict mutex,const struct timespec *restrict abstime);
       ```

     - 发送信号，唤醒一个或多个等待的线程

       ```c
       int pthread_cond_signal(pthread_cond_t *cond);
       ```

     - 广播，唤醒所有等待的线程

       ```c
       int pthread_cond_broadcast(pthread_cond_t *cond);
       ```

   - 利用互斥量和条件变量解决上述段错误的问题：

     ```c
     #include<stdio.h>
     #include<pthread.h>
     #include<stdlib.h>
     #include<unistd.h>
     
     //创建一个互斥量
     pthread_mutex_t mutex;
     //创建一个条件变量
     pthread_cond_t cond;
     
     struct Node{
         int num;
         struct Node *next;
     };
     
     struct Node *head =NULL;
     
     
     
     void *producer(void *arg){
         //不断创建新的节点，添加到链表中
         while(1){
             pthread_mutex_lock(&mutex);
             struct Node * newNode = (struct Node*)malloc(sizeof(struct Node));
             newNode->next=head;
             head=newNode;
             newNode->num=rand()%1000;
             printf("add node,num : %d,tid : %ld\n",newNode->num,pthread_self());
     
             //只要生产了一个就通知消费者消费
             pthread_cond_signal(&cond);
     
             pthread_mutex_unlock(&mutex);
             usleep(1000);
         }
     
         return NULL;
     }
     
     void *customer(void *arg){
         
         while(1){
             pthread_mutex_lock(&mutex);
             //保存头节点的指针
             struct Node *temp=head;
             if(head!=NULL){
                 //有数据
                 head=head->next;
                 printf("delete node,num : %d,tid : %ld\n",temp->num,pthread_self());
                 free(temp);
                 usleep(1000);
             }else{
                 //没有数据，需要等待
                 pthread_cond_wait(&cond,&mutex); /*
                                                     调用这个函数：
                                                     阻塞时，会对互斥锁进行解锁；
                                                     不阻塞时，继续向下执行会重新加锁。 
                                                 */
             }
             pthread_mutex_unlock(&mutex);
         }
     }
     
     int main(){
     
         pthread_mutex_init(&mutex,NULL);
         pthread_cond_init(&cond,NULL);
     
         //创建5个生产者线程，5个消费者线程
         pthread_t ptid[5],ctid[5];
     
         for(int i=0;i<5;i++){
             pthread_create(&ptid[i],NULL,producer,NULL);
             pthread_create(&ctid[i],NULL,customer,NULL);
         }
     
         for(int i=0;i<5;i++){
             pthread_detach(ptid[i]);
             pthread_detach(ctid[i]);
         }
     
         while(1){
             sleep(10);
         }
     
         pthread_mutex_destroy(&mutex);
         pthread_cond_destroy(&cond);
     
         pthread_exit(NULL);
     
     }
     ```

     

7. 信号量操作函数

   - 头文件：```#include<semaphore.h>```

   - 信号量的类型：```sem_t```

   - 初始化信号量

     ```c
     int sem_init(sem_t *sem,int pshared,unsigned int value);
     ```

     - 参数：

       ​	-sem:信号量变量的地址

       ​	-pshared:指定信号量用于进程间还是线程间

       ​			-0：用在线程间

       ​			-非0：用在进程间

       ​	-value:设置信号量的值

   - 销毁信号量

     ```c
     int sem_destroy(sem_t *sem);
     ```

   - 对信号量加锁，调用一次信号量的值减1，如果值为0就阻塞

     ```
     int sem_wait(sem_t *sem);
     ```

   - 尝试对信号量加锁，调用一次信号量的值减1，如果信号量大于0，则减1

     ```c
     int sem_trywait(sem_t *sem);
     ```

   - 对信号量加锁，调用一次，信号量的值减1，如果值为0，则阻塞一定时间

     ```c
     int sem_timedwait(sem_t *sem,const struct timespec *abs_timeout);
     ```

   - 对信号量解锁，调用一次，信号量的值加1

     ```c
     int sem_post(sem_t *sem);
     ```

   - 获取信号量的值

     ```c
     int sem_getvalue(sem_t *sem,int *sval);
     ```

   - 利用互斥量和信号量解决上述段错误问题:

     ```c
     #include<stdio.h>
     #include<pthread.h>
     #include<stdlib.h>
     #include<unistd.h>
     #include<semaphore.h>
     
     //创建一个互斥量
     pthread_mutex_t mutex;
     
     //创建信号量
     sem_t psem;
     sem_t csem;
     
     
     struct Node{
         int num;
         struct Node *next;
     };
     
     struct Node *head =NULL;
     
     
     
     void *producer(void *arg){
         int value;
         //不断创建新的节点，添加到链表中
         while(1){
             sem_wait(&psem);
             pthread_mutex_lock(&mutex);
             struct Node * newNode = (struct Node*)malloc(sizeof(struct Node));
             newNode->next=head;
             head=newNode;
             newNode->num=rand()%1000;
             sem_getvalue(&csem,&value);
             printf("add node,num : %d,psem:%d,tid : %ld\n",newNode->num,value,pthread_self());
             pthread_mutex_unlock(&mutex);
             sem_post(&csem);
         }
         return NULL;
     }
     
     void *customer(void *arg){
         int value;
         while(1){
             sem_wait(&csem);
             pthread_mutex_lock(&mutex);
             //保存头节点的指针
             struct Node *temp=head;
             head=head->next;
             sem_getvalue(&csem,&value);
             printf("delete node,num : %d,csem:%d,tid : %ld\n",temp->num,value,pthread_self());
             free(temp);
             pthread_mutex_unlock(&mutex);
             sem_post(&psem);
         }
         return NULL;
     }
     
     int main(){
     
         pthread_mutex_init(&mutex,NULL);
     
         sem_init(&psem,0,8);
         sem_init(&csem,0,0);
     
         //创建5个生产者线程，5个消费者线程
         pthread_t ptid[5],ctid[5];
     
         for(int i=0;i<5;i++){
             pthread_create(&ptid[i],NULL,producer,NULL);
             pthread_create(&ctid[i],NULL,customer,NULL);
         }
     
         for(int i=0;i<5;i++){
             pthread_detach(ptid[i]);
             pthread_detach(ctid[i]);
         }
     
         while(1){
             sleep(10);
         }
     
         sem_destroy(&psem);
         sem_destroy(&csem);
         pthread_mutex_destroy(&mutex);
     
         pthread_exit(NULL);
     
     }
     ```

     

     

   