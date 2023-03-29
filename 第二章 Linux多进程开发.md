# 第二章 Linux多进程开发

## 1.进程

### 1.1 进程概述

#### **（1）程序和进程** 

进程是正在运行的程序的实例。是一个具有一定独立功能的程序关于某个数据集合的 一次运行活动。它是操作系统动态执行的基本单元，在传统的操作系统中，**进程既是基本的分配单元**，也是基本的执行单元。

**可以用一个程序来创建多个进程**



#### （2）单道、多道程序设计

- **单道程序**

​			即在计算机内存中只允许一个的程序运行

- **多道程序**

​			多道程序设计技术是在计算机内存中同时存放几道相互独立的程序，使它们在管理程序控制下，相互穿插运行，两个或两个以上程序在计算机系统中同处于开始到结束之间的状态, 这些程序共享计算机系统资源。引入多道程序设计技术的**根本目的是为了 提高 CPU 的利用率。**



#### （3）时间片

时间片（timeslice）又称为“量子（quantum）”或“处理器片（processor slice）” **是操作系统分配给每个正在运行的进程微观上的一段 CPU 时间**。事实上，**虽然一台计 算机通常可能有多个 CPU，但是同一个 CPU 永远不可能真正地同时运行多个任务**。在 只考虑一个 CPU 的情况下，这些进程“看起来像”同时运行的，实则是轮番穿插地运行， 由于时间片通常很短（在 Linux 上为 5ms－800ms），用户不会感觉到。

**时间片由操作系统内核的调度程序分配给每个进程**



#### （4）并行和并发

**并行(parallel)**：指在同一时刻，有多条指令在多个处理器上同时执行。

**并发(concurrency)：**指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的， 只是把时间分成若干段，使多个进程快速交替的执行。

![image-20230223203636964](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230223203636964.png)

#### （5）进程控制块（PCB）

为了管理进程，内核必须对每个进程所做的事情进行清楚的描述。内核为每个进程分 配一个 PCB(Processing Control Block)进程控制块，维护进程相关的信息， **Linux 内核的进程控制块是 task_struct 结构体**。

⚫ 进程id：系统中每个进程有唯一的 id，用 pid_t 类型表示，其实就是一个非负整数 

⚫ 进程的状态：有就绪、运行、挂起、停止等状态 

⚫ 进程切换时需要保存和恢复的一些CPU寄存器 

⚫ 描述虚拟地址空间的信息 

⚫ 描述控制终端的信息

### 1.2 进程状态转换⭐

#### （1）进程的状态

进程状态反映进程执行过程的变化。这些状态随着进程的执行和外界条件的变化而转换。

**在三态模型中，进程状态分为三个基本状态，即就绪态，运行态，阻塞态**。

![image-20230223205102899](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230223205102899.png)

◼ 运行态：进程占有处理器正在运行 

◼ 就绪态：进程具备运行条件，等待系统分配处理器以便运 行。当进程已分配到除CPU以外的所有必要资源后，只要再 获得CPU，便可立即执行。在一个系统中处于就绪状态的进程可能有多个，通常将它们排成一个队列，称为就绪队列 

◼ 阻塞态：又称为等待(wait)态或睡眠(sleep)态，指进程不具备运行条件，正在等待某个事件的完成

在五态模型中，进程分为**新建态、就绪态，运行态，阻塞态，终止态**。

![image-20230223205016087](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230223205016087.png)

◼ 新建态：进程刚被创建时的状态，尚未进入就绪队列 

◼ 终止态：进程完成任务到达正常结束点，或出现无法克服的错误而异常终止，或被操作系统及 有终止权的进程所终止时所处的状态。进入终止态的进程以后不再执行，但依然保留在操作系 统中等待善后。一旦其他进程完成了对终止态进程的信息抽取之后，操作系统将删除该进程。



#### （2）进程相关命令

◼ **查看进程**

```
ps aux / ajx
```

 a：显示终端上的所有进程，包括其他用户的进程 

u：显示进程的详细信息 

x：显示没有控制终端的进程

 j：列出与作业控制相关的信息



◼ **STAT参数意义：** 

D 不可中断 Uninterruptible（usually IO） 

R 正在运行，或在队列中的进程

S(大写) 处于休眠状态 

T 停止或被追踪 

Z 僵尸进程 

W 进入内存交换（从内核2.6开始无效） 

X 死掉的进程 < 高优先级 N 低优先级 

s 包含子进程 + 位于前台的进程组



◼ **实时显示进程动态** 

```
top 
```

可以在使用 top 命令时加上 -d 来指定显示信息更新的时间间隔，在 top 命令 执行后，可以按以下按键对显示的结果进行排序： 

⚫ M 根据内存使用量排序 

⚫ P 根据 CPU 占有率排序 

⚫ T 根据进程运行时间长短排序 

⚫ U 根据用户名来筛选进程 

⚫ K 输入指定的 PID 杀死进程



◼ 杀死进程 

​		kill [-signal] pid 

​		kill –l 列出所有信号 

​		kill –SIGKILL 进程ID 

​		kill -9 进程ID 

​		killall name 根据进程名杀死进程

#### （3）进程号和相关函数

◼ 每个进程都由进程号来标识，其类型为 pid_t（整型），进程号的范围：0～32767。 进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。 

◼ **任何进程（除 init 进程）都是由另一个进程创建**，该进程称为被创建进程的父进程， **对应的进程号称为父进程号（PPID）**。 

◼ 进程组是一个或多个进程的集合。他们之间相互关联，进程组可以接收同一终端的各种信号，**关联的进程有一个进程组号（PGID）**。默认情况下，当前的进程号会当做当前的进程组号。 

◼ 进程号和进程组相关函数： 

​	⚫ pid_t getpid(void); 

​	⚫ pid_t getppid(void); 

​	⚫ pid_t getpgid(pid_t pid);



### 1.3 进程创建

**系统允许一个进程创建新进程，新进程即为子进程**，子进程还可以创建新的子进程，形成进程树结构模型。

**fork进程创建的时候，子进程并不是完全的拷贝，采用的是copy on wirte,一开始会拷贝地址空间，共享一个地址空间，只有在写的时候才会重新拷贝，映射到新的内存空间**（读时共享（子进程被创建，两个进程没有做任何的写的操作），写时拷贝。）

 pid_t fork(void);
        函数的作用：用于创建子进程。返回值：

​	⚫ 成功：子进程中返回 0，父进程中返回子进程 ID 

​	⚫ 失败：返回 -1 

失败的两个主要原因： 

1. 当前系统的进程数已经达到了系统规定的上限，这时 errno 的值被设置 为 EAGAIN
2. 系统内存不足，这时 errno 的值被设置为 ENOMEM

```c
/*
    #include <sys/types.h>
    #include <unistd.h>

        父子进程之间的关系：
        区别：
            1.fork()函数的返回值不同
                父进程中: >0 返回的子进程的ID
                子进程中: =0
            2.pcb中的一些数据
                当前的进程的id pid
                当前的进程的父进程的id ppid
                信号集

        共同点：
            某些状态下：子进程刚被创建出来，还没有执行任何的写数据的操作
                - 用户区的数据
                - 文件描述符表
        
        父子进程对变量是不是共享的？
            - 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了。
            - 读时共享（子进程被创建，两个进程没有做任何的写的操作），写时拷贝。
        
*/

#include <sys/types.h>
#include <unistd.h>
#include <stdio.h>

int main() {

    int num = 10;

    // 创建子进程
    pid_t pid = fork();

    // 判断是父进程还是子进程
    if(pid > 0) {
        // printf("pid : %d\n", pid);
        // 如果大于0，返回的是创建的子进程的进程号，当前是父进程
        printf("i am parent process, pid : %d, ppid : %d\n", getpid(), getppid());

        printf("parent num : %d\n", num);
        num += 10;
        printf("parent num += 10 : %d\n", num);


    } else if(pid == 0) {
        // 当前是子进程
        printf("i am child process, pid : %d, ppid : %d\n", getpid(),getppid());
       
        printf("child num : %d\n", num);
        num += 100;
        printf("child num += 100 : %d\n", num);
    }

    // for循环
    for(int i = 0; i < 3; i++) {
        printf("i : %d , pid : %d\n", i , getpid());
        sleep(1);
    }

    return 0;
}

/*

*/
```

#### （1）父子进程虚拟地址空间情况

实际上，更准确来说，**Linux 的 fork() 使用是通过写时拷贝 (copy- on-write) 实**现。
写时拷贝是一种可以推迟甚至避免拷贝数据的技术。
内核此时并不复制整个进程的地址空间，而是让**父子进程共享同一个地址空间**。
**只有在需要写入的时候才会复制地址空间**，**从而使各个进行拥有各自的地址空间**。
也就是说，**资源的复制是在需要写入的时候才会进行，在此之前，只有以只读方式共享**。
注意：fork之后父子进程共享文件，
fork产生的子进程与父进程相同的文件文件描述符指向相同的文件表，引用计数增加，共享文件偏移指针。



#### （2）GDB多进程调试

使用 GDB 调试的时候，GDB 默认只能跟踪一个进程，可以在 fork 函数调用之前，通 过指令设置 GDB 调试工具跟踪父进程或者是跟踪子进程，默认跟踪父进程。 

**设置调试父进程或者子进程**：`set follow-fork-mode [parent（默认）| child]` 

设置调试模式：`set detach-on-fork [on | off]` 

​	默认为 on，表示调试当前进程的时候，其它的进程继续运行 

​	如果为 off，调试当前进程的时候，其它进程被 GDB 挂起。 

- 查看调试的进程：`info inferiors` 

- 切换当前调试的进程：`inferior id` 

- 使进程脱离 GDB 调试：`detach inferiors id`

#### （3）exec函数族

exec 函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的 内容，换句话说，就是**在调用进程内部执行一个可执行文件**。

```c
◼ int execl(const char *path, const char *arg, .../* (char *) NULL */); 

◼ int execlp(const char *file, const char *arg, ... /* (char *) NULL */);

 ◼ int execle(const char *path, const char *arg, .../*, (char *) NULL, char *  const envp[] */); 

◼ int execv(const char *path, char *const argv[]); 

◼ int execvp(const char *file, char *const argv[]); 

◼ int execvpe(const char *file, char *const argv[], char *const envp[]); 

◼ int execve(const char *filename, char *const argv[], char *const envp[]); 

l(list) 参数地址列表，以空指针结尾 

v(vector) 存有各参数地址的指针数组的地址 

p(path) 按 PATH 环境变量指定的目录搜索可执行文件 

e(environment) 存有环境变量字符串地址的指针数组的地址
```

### 1.4 进程控制

#### **（1）进程退出**

![image-20230224154458162](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224154458162.png)

```c
#include <stdlib.h>
void exit(int status);

#include <unistd.h>
void _exit(int status);
```



#### **（2）孤儿进程**

◼ 父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为孤儿进程 （Orphan Process）。 

◼ 每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init  进程会循环地 wait() 它的已经退出的子进程。这样，当一个孤儿进程凄凉地结束 了其生命周期的时候，init 进程就会代表党和政府出面处理它的一切善后工作。 

◼ 因此孤儿进程并不会有什么危害。



#### **（3）僵尸进程**

◼ 每个进程结束之后, 都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。 

◼ **进程终止时，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸 （Zombie）进程**。 

◼ 僵尸进程不能被 kill -9 杀死，这样就会导致一个问题，如果父进程不调用 wait()  或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用， 但是**系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进 程号而导致系统不能产生新的进程**，此即为僵尸进程的危害，应当避免。



#### **（4）进程回收**

```c
    #include <sys/types.h>
    #include <sys/wait.h>
  ◼  pid_t wait(int *wstatus);
        功能：等待任意一个子进程结束，如果任意一个子进程结束了，函数会回收子进程的资源。
        参数：int *wstatus
            进程退出时的状态信息，传入的是一个int类型的地址，传出参数。
        返回值：
            - 成功：返回被回收的子进程的id
            - 失败：-1 (所有的子进程都结束，调用函数失败)

    调用wait函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）
    如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1.
     
  ◼ pid_t waitpid(pid_t pid, int *wstatus, int options); (null,WNOHANG)
       #include <sys/types.h>
       #include <sys/wait.h>
       -1     所有子进程结束
       0      还有子进程活着
       > 0    等于Pid的子进程结束
```

◼ 在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息 （包括进程号、退出状态、运行时间等）。 

◼ 父进程可以通过调用wait或waitpid得到它的退出状态同时彻底清除掉这个进程。 

◼ **wait() 和 waitpid() 函**数的功能一样，区别在于，wait() 函数会阻塞， waitpid() 可以设置不阻塞，waitpid() 还可以指定等待哪个子进程结束。 

◼ 注意：一次wait或waitpid调用只能清理一个子进程，清理多个子进程应使用循环。



#### **（5）退出信息相关宏函数**

◼ WIFEXITED(status) 非0，进程正常退出 

◼ WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit的参数） 

◼ WIFSIGNALED(status) 非0，进程异常终止 

◼ WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号 

◼ WIFSTOPPED(status) 非0，进程处于暂停状态 

◼ WSTOPSIG(status) 如果上宏为真，获取使进程暂停的信号的编号 

◼ WIFCONTINUED(status) 非0，进程暂停后已经继续运行

### 1.5 进程间通信⭐

#### （1）概念

◼ 进程是一个独立的资源分配单元，不同进程（这里所说的进程通常指的是用户进程）之间 的资源是独立的，没有关联，不能在一个进程中直接访问另一个进程的资源。

◼ 但是，进程不是孤立的，不同的进程需要进行信息的交互和状态的传递等，因此需要进程 间通信( IPC：Inter Processes Communication )。 

◼ 进程间通信的目的：

​	 ◼ 数据传输：一个进程需要将它的数据发送给另一个进程。 

​	 ◼ 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种 事件（如进程终止时要通知父进程）。 

​	 ◼ 资源共享：多个进程之间共享同样的资源。为了做到这一点，需要内核提供互斥和同 步机制。 

​	 ◼ 进程控制：有些进程希望完全控制另一个进程的执行（如 Debug 进程），此时控制 进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

#### （2）通信方式

![image-20230224170346038](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224170346038.png)



#### （3）匿名管道

◼ 管道也叫无名（匿名）管道，它是是 UNIX 系统 IPC（进程间通信）的最古老形式， 所有的 UNIX 系统都支持这种通信机制。 

◼ 统计一个目录中文件的数目命令：`ls | wc –l`，为了执行该命令，shell 创建了两 个进程来分别执行 ls 和 wc。

![image-20230224170523616](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224170523616.png)

#### （4）管道特点

◼ 管道其实是一个**在内核内存中维护的缓冲器**，这个缓冲器的存储能力是有限的，不同的 操作系统大小不一定相同。 

◼ **管道拥有文件的特质**：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体， 但不存储数据。可以按照操作文件的方式对管道进行操作。 

◼ **一个管道是一个字节流**，使用管道时不存在消息或者消息边界的概念，从管道读取数据 的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。 

◼ **通过管道传递的数据是顺序的**，从管道中读取出来的字节的顺序和它们被写入管道的顺 序是完全一样的

◼ 管道中的数据传递方向是单向的，一端写入一端读取，**管道是半双工**

◼ **管道读数据是一次性操作**

◼ **匿名管道只能在具有公共祖先的进程中使用**（fork之后父子文件共享文件描述符）



```html
读管道：
    管道中有数据，read返回实际读到的字节数。
    管道中无数据：
        写端被全部关闭，read返回0（相当于读到文件的末尾）
        写端没有完全关闭，read阻塞等待

写管道：
    管道读端全部被关闭，进程异常终止（进程收到SIGPIPE信号）
    管道读端没有全部关闭：
        管道已满，write阻塞
        管道没有满，write将数据写入，并返回实际写入的字节数
```



#### （5）管道通信

![image-20230224170633605](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224170633605.png)

#### （6）匿名管道的使用

◼ 创建匿名管道 

```c
    #include <unistd.h>
    int pipe(int pipefd[2]);
        功能：创建一个匿名管道，用来进程间通信。
        参数：int pipefd[2] 这个数组是一个传出参数。
            pipefd[0] 对应的是管道的读端
            pipefd[1] 对应的是管道的写端
        返回值：
            成功 0
            失败 -1
```

◼ 查看管道缓冲大小命令 

​		`ulimit –a` 

◼ 查看管道缓冲大小函数 

```c
`#include  <unistd.h>`

`long fpathconf(int fd, int name)
    
 #include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main() {

    int pipefd[2];

    int ret = pipe(pipefd);

    // 获取管道的大小
    long size = fpathconf(pipefd[0], _PC_PIPE_BUF);

    printf("pipe size : %ld\n", size);

    return 0;
}
```

![image-20230224170746361](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224170746361.png)

#### （7）有名管道

◼ 匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提 出了有名管道（FIFO），也叫命名管道、FIFO文件。 

◼ 有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO  的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样 即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此 通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据。 

◼ 一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的 I/O系统调用了（如read()、write()和close()）。与管道一样，FIFO 也有一 个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO 的 名称也由此而来：先入先出。

◼ 有名管道（FIFO)和匿名管道（pipe）有一些特点是相同的，不一样的地方在于： 1. FIFO 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存放在内存中。 2. 当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用。 3. FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

#### （8）有名管道的使用

◼ 通过命令创建有名管道

 `mkfifo 名字` 

◼ 通过函数创建有名管道 

```c
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char *pathname, mode_t mode); 
```

◼ 一旦使用 mkfifo 创建了一个 FIFO，就可以使用 open 打开它，常见的文件 I/O 函数都可用于 fifo。

如：close、read、write、unlink 等。 

◼ FIFO 严格遵循先进先出（First in First out），对管道及 FIFO 的读总是 从开始处返回数据，对它们的写则把数据添加到末尾。它们不支持诸如 lseek()  等文件定位操作。

```
有名管道
读管道：
	管道中有数据，正常返回读取的字节数
	管道中无数据：
		写端关闭，read返回0
		写端没关闭，阻塞
写管道：
	管道读端关闭，异常终止，返回SIGPIPE信号
	管道读端没有全部关闭：
		管道已满：write阻塞
		管道未满：write写入多少字节，并返回写入的字节数
```



#### （9）内存映射

◼ 内存映射（Memory-mapped I/O）是将磁盘文件的数据映射到内存，用户通过修改内存就能修改磁盘文件。

![image-20230224171035888](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230224171035888.png)

内存映射相关系统调用

```c
   #include <sys/mman.h>
    void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset);
        - 功能：将一个文件或者设备的数据映射到内存中
        - 参数：
            - void *addr: NULL, 由内核指定
            - length : 要映射的数据的长度，这个值不能为0。建议使用文件的长度。
                    获取文件的长度：stat lseek
            - prot : 对申请的内存映射区的操作权限
                -PROT_EXEC ：可执行的权限
                -PROT_READ ：读权限
                -PROT_WRITE ：写权限
                -PROT_NONE ：没有权限
                要操作映射内存，必须要有读的权限。
                PROT_READ、PROT_READ|PROT_WRITE
            - flags :
                - MAP_SHARED : 映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项
                - MAP_PRIVATE ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy on write）
            - fd: 需要映射的那个文件的文件描述符
                - 通过open得到，open的是一个磁盘文件
                - 注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。
                    prot: PROT_READ                open:只读/读写 
                    prot: PROT_READ | PROT_WRITE   open:读写
            - offset：偏移量，一般不用。必须指定的是4k的整数倍，0表示不便宜。
        - 返回值：返回创建的内存的首地址
            失败返回MAP_FAILED，(void *) -1

    int munmap(void *addr, size_t length);
        - 功能：释放内存映射
        - 参数：
            - addr : 要释放的内存的首地址
            - length : 要释放的内存的大小，要和mmap函数中的length参数的值一样。
```

​    使用内存映射实现进程间通信：
​    1.有关系的进程（父子进程）
​        - 还没有子进程的时候
​            - 通过唯一的父进程，先创建内存映射区
​                - 有了内存映射区以后，创建子进程
​                    - 父子进程共享创建的内存映射区

​    2.没有关系的进程间通信
​        - 准备一个大小不是0的磁盘文件
​                - 进程1 通过磁盘文件创建内存映射区
​            - 得到一个操作这块内存的指针
​                - 进程2 通过磁盘文件创建内存映射区
​            - 得到一个操作这块内存的指针
​                - 使用内存映射区通信

​    注意：内存映射区通信，是非阻塞。



```
1.如果对mmap的返回值(ptr)做++操作(ptr++), munmap是否能够成功?
void * ptr = mmap(...);
ptr++;  可以对其进行++操作
munmap(ptr, len);   // 错误,要保存地址

2.如果open时O_RDONLY, mmap时prot参数指定PROT_READ | PROT_WRITE会怎样?
错误，返回MAP_FAILED
open()函数中的权限建议和prot参数的权限保持一致。

3.如果文件偏移量为1000会怎样?
偏移量必须是4K的整数倍，返回MAP_FAILED

4.mmap什么情况下会调用失败?

   - 第二个参数：length = 0
     - 第三个参数：prot
     - 只指定了写权限
     - prot PROT_READ | PROT_WRITE
       第5个参数fd 通过open函数时指定的 O_RDONLY / O_WRONLY

5.可以open的时候O_CREAT一个新文件来创建映射区吗?
    - 可以的，但是创建的文件的大小如果为0的话，肯定不行
    - 可以对新的文件进行扩展
        - lseek()
        - truncate()

6.mmap后关闭文件描述符，对mmap映射有没有影响？
    int fd = open("XXX");
    mmap(,,,,fd,0);
    close(fd); 
    映射区还存在，创建映射区的fd被关闭，没有任何影响。

7.对ptr越界操作会怎样？
void * ptr = mmap(NULL, 100,,,,,);
4K
越界操作操作的是非法的内存 -> 段错误
```



## 2.信号

### 2.1 信号的概念

信号是 Linux 进程间通信的最古老的方式之一，是**事件发生时对进程的通知机制**，有时也称之为**软件中断**，它是在软件层次上对中断机制的一种模拟，是一种**异步通信**的方式。信号可以导致一个正在运行的进程被另一个正在运行的异步进程中断，转而处理某一个突发事件。

◼ 发往进程的诸多信号，通常都是源于内核。引发内核为进程产生信号的各类事件如下： 

- 对于**前台进程**，用户可以通过输入特殊的终端字符来给它发送信号。比如输入Ctrl+C  通常会给进程发送一个中断信号。 

- **硬件发生异常**，即硬件检测到一个错误条件并通知内核，随即再由内核发送相应信号给相关进程。比如执行一条异常的机器语言指令，诸如被 0 除，或者引用了无法访问的内存区域。 

- **系统状态变化**，比如 alarm 定时器到期将引起 SIGALRM 信号，进程执行的 CPU  时间超限，或者该进程的某个子进程退出。 

- 运行 kill 命令或调用 kill 函数。

  

◼ 使用信号的两个主要目的是：

- 让进程知道已经发生了一个特定的事情。 

- 强迫进程执行它自己代码中的信号处理程序。



◼ 信号的特点： 

- 简单 
- 不能携带大量信息 
- **满足某个特定条件才发送** 
- 优先级比较高 



◼ 查看系统定义的信号列表：`kill –l`  

◼ 前 31 个信号为常规信号，其余为实时信号。



### 2.2 Linux信号一览表

| 编号 |  信号名称   |                           对应事件                           |        默认动作        |
| :--: | :---------: | :----------------------------------------------------------: | :--------------------: |
|  1   |   SIGHUP    |   用户退出shell时，由该shell启动的所有进程将 收到这个信号    |        终止进程        |
|  2   | **SIGINT**  | 当用户按下了组合键时，用户终端向正 在运行中的由该终端启动的程序发出此信号 |        终止进程        |
|  3   | **SIGQUIT** | 用户按下组合键时产生该信号，用户终 端向正在运行中的由该终端启动的程序发出些信号 |        终止进程        |
|  4   |   SIGILL    |                CPU检测到某进程执行了非法指令                 | 终止进程并产生core文件 |
|  5   |   SIGTRAP   |             该信号由断点指令或其他 trap指令产生              | 终止进程并产生core文件 |
|  6   |   SIGABRT   |                  调用abort函数时产生该信号                   | 终止进程并产生core文件 |
|  7   |   SIGABRT   |              非法访问内存地址，包括内存对齐出错              | 终止进程并产生core文件 |
|  8   |   SIGFPE    | 在发生致命的运算错误时发出。不仅包括浮点运算 错误，还包括溢出及除数为0等所有的算法错误 | 终止进程并产生core文件 |

![image-20230227162226499](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230227162226499.png)

![image-20230227162233461](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230227162233461.png)

![image-20230227162244329](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230227162244329.png)

### 2.3 信号的五种默认处理动作

◼ 查看信号的详细信息：`man 7 signal`

 ◼ 信号的 5 中默认处理动作 

- Term		 终止进程 
- Ign             当前进程忽略掉这个信号 
- Core          终止进程，并生成一个Core文件 
- Stop          暂停当前进程  
- Cont          继续执行当前被暂停的进程 

◼ 信号的几种状态：产生、未决、递达 

◼ SIGKILL 和 SIGSTOP 信号不能被捕捉、阻塞或者忽略，只能执行默认动作。 



### 2.4 信号相关的函数

```c
◼ int kill(pid_t pid, int sig);
#include <sys/types.h>
    #include <signal.h>

    int kill(pid_t pid, int sig);
        - 功能：给任何的进程或者进程组pid, 发送任何的信号 sig
        - 参数：
            - pid ：
                > 0 : 将信号发送给指定的进程
                = 0 : 将信号发送给当前的进程组
                = -1 : 将信号发送给每一个有权限接收这个信号的进程
                < -1 : 这个pid=某个进程组的ID取反 （-12345）
            - sig : 需要发送的信号的编号或者是宏值，0表示不发送任何信号

        kill(getppid(), 9);
        kill(getpid(), 9);
        
    int raise(int sig);
        - 功能：给当前进程发送信号
        - 参数：
            - sig : 要发送的信号
        - 返回值：
            - 成功 0
            - 失败 非0
        kill(getpid(), sig);   

    void abort(void);
        - 功能： 发送SIGABRT信号给当前的进程，杀死当前进程
        kill(getpid(), SIGABRT);

◼ unsigned int alarm(unsigned int seconds);
   #include <unistd.h>
    unsigned int alarm(unsigned int seconds);
        - 功能：设置定时器（闹钟）。函数调用，开始倒计时，当倒计时为0的时候，
                函数会给当前的进程发送一个信号：SIGALARM
        - 参数：
            seconds: 倒计时的时长，单位：秒。如果参数为0，定时器无效（不进行倒计时，不发信号）。
                    取消一个定时器，通过alarm(0)。
        - 返回值：
            - 之前没有定时器，返回0
            - 之前有定时器，返回之前的定时器剩余的时间

    - SIGALARM ：默认终止当前的进程，每一个进程都有且只有唯一的一个定时器。
        alarm(10);  -> 返回0
        过了1秒
        alarm(5);   -> 返回9

    alarm(100) -> 该函数是不阻塞的
                
◼ int setitimer(int which, const struct itimerval *new_val, 
	struct itimerval *old_value);

    #include <sys/time.h>
    int setitimer(int which, const struct itimerval *new_value,
                        struct itimerval *old_value);
    
        - 功能：设置定时器（闹钟）。可以替代alarm函数。精度微妙us，可以实现周期性定时
        - 参数：
            - which : 定时器以什么时间计时
              ITIMER_REAL: 真实时间，时间到达，发送 SIGALRM   常用
              ITIMER_VIRTUAL: 用户时间，时间到达，发送 SIGVTALRM
              ITIMER_PROF: 以该进程在用户态和内核态下所消耗的时间来计算，时间到达，发送 SIGPROF

            - new_value: 设置定时器的属性
            
                struct itimerval {      // 定时器的结构体
                struct timeval it_interval;  // 每个阶段的时间，间隔时间
                struct timeval it_value;     // 延迟多长时间执行定时器
                };

                struct timeval {        // 时间的结构体
                    time_t      tv_sec;     //  秒数     
                    suseconds_t tv_usec;    //  微秒    
                };

            过10秒后，每个2秒定时一次
           
            - old_value ：记录上一次的定时的时间参数，一般不使用，指定NULL
        
        - 返回值：
            成功 0
            失败 -1 并设置错误号
```



### 2.5 信号捕捉函数⭐

```c
◼ sighandler_t signal(int signum, sighandler_t handler);
    #include <signal.h>
    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);
        - 功能：设置某个信号的捕捉行为
        - 参数：
            - signum: 要捕捉的信号
            - handler: 捕捉到信号要如何处理
                - SIG_IGN ： 忽略信号
                - SIG_DFL ： 使用信号默认的行为
                - 回调函数 :  这个函数是内核调用，程序员只负责写，捕捉到信号后如何去处理信号。
                回调函数：
                    - 需要程序员实现，提前准备好的，函数的类型根据实际需求，看函数指针的定义
                    - 不是程序员调用，而是当信号产生，由内核调用
                    - 函数指针是实现回调的手段，函数实现之后，将函数名放到函数指针的位置就可以了。

        - 返回值：
            成功，返回上一次注册的信号处理函数的地址。第一次调用返回NULL
            失败，返回SIG_ERR，设置错误号
            
    SIGKILL SIGSTOP不能被捕捉，不能被忽略。

◼ int sigaction(int signum, const struct sigaction *act, 
							struct sigaction *oldact)
   #include <signal.h>
    int sigaction(int signum, const struct sigaction *act,
                            struct sigaction *oldact);

        - 功能：检查或者改变信号的处理。信号捕捉
        - 参数：
            - signum : 需要捕捉的信号的编号或者宏值（信号的名称）
            - act ：捕捉到信号之后的处理动作
            - oldact : 上一次对信号捕捉相关的设置，一般不使用，传递NULL
        - 返回值：
            成功 0
            失败 -1

     struct sigaction {
        // 函数指针，指向的函数就是信号捕捉到之后的处理函数
        void     (*sa_handler)(int);
        // 不常用
        void     (*sa_sigaction)(int, siginfo_t *, void *);
        // 临时阻塞信号集，在信号捕捉函数执行过程中，临时阻塞某些信号。
        sigset_t   sa_mask;
        // 使用哪一个信号处理对捕捉到的信号进行处理
        // 这个值可以是0，表示使用sa_handler,也可以是SA_SIGINFO表示使用sa_sigaction
        int        sa_flags;
        // 被废弃掉了
        void     (*sa_restorer)(void);
    };
```



### 2.6 信号集

◼ 许多信号相关的系统调用都需要能表示一组不同的信号，多个信号可使用一个称之为 信号集的数据结构来表示，其系统数据类型为 sigset_t。 

◼ 在 PCB 中有两个非常重要的信号集。一个称之为 “**阻塞信号集**” ，另一个称之为 “**未决信号集**” 。这两个信号集都是内核使用位图机制来实现的。但操作系统不允许我 们直接对这两个信号集进行位操作。而需自定义另外一个集合，借助信号集操作函数 来对 PCB 中的这两个信号集进行修改。 

◼ 信号的 “未决” 是一种状态，指的是从信号的产生到信号被处理前的这一段时间。 

◼ 信号的 “阻塞” 是一个开关动作，指的是阻止信号被处理，但不是阻止信号产生。 

◼ 信号的阻塞就是让系统暂时保留信号留待以后发送。由于另外有办法让系统忽略信号， 所以一般情况下信号的阻塞只是暂时的，只是为了防止信号打断敏感的操作。



### 2.7 阻塞信号和未决信号集

![image-20230227162624406](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230227162624406.png)

```
1.用户通过键盘  Ctrl + C, 产生2号信号SIGINT (信号被创建)

2.信号产生但是没有被处理 （未决）
    - 在内核中将所有的没有被处理的信号存储在一个集合中 （未决信号集）
    - SIGINT信号状态被存储在第二个标志位上
        - 这个标志位的值为0， 说明信号不是未决状态
        - 这个标志位的值为1， 说明信号处于未决状态
    
3.这个未决状态的信号，需要被处理，处理之前需要和另一个信号集（阻塞信号集），进行比较
    - 阻塞信号集默认不阻塞任何的信号
    - 如果想要阻塞某些信号需要用户调用系统的API

4.在处理的时候和阻塞信号集中的标志位进行查询，看是不是对该信号设置阻塞了
    - 如果没有阻塞，这个信号就被处理
    - 如果阻塞了，这个信号就继续处于未决状态，直到阻塞解除，这个信号就被处理
```



### 2.8 信号集相关函数

```c
◼ int sigemptyset(sigset_t *set);
    int sigemptyset(sigset_t *set);
        - 功能：清空信号集中的数据,将信号集中的所有的标志位置为0
        - 参数：set,传出参数，需要操作的信号集
        - 返回值：成功返回0， 失败返回-1
◼ int sigfillset(sigset_t *set);
    int sigfillset(sigset_t *set);
        - 功能：将信号集中的所有的标志位置为1
        - 参数：set,传出参数，需要操作的信号集
        - 返回值：成功返回0， 失败返回-1
◼ int sigaddset(sigset_t *set, int signum);
    int sigaddset(sigset_t *set, int signum);
        - 功能：设置信号集中的某一个信号对应的标志位为1，表示阻塞这个信号
        - 参数：
            - set：传出参数，需要操作的信号集
            - signum：需要设置阻塞的那个信号
        - 返回值：成功返回0， 失败返回-1
◼ int sigdelset(sigset_t *set, int signum);
    int sigdelset(sigset_t *set, int signum);
        - 功能：设置信号集中的某一个信号对应的标志位为0，表示不阻塞这个信号
        - 参数：
            - set：传出参数，需要操作的信号集
            - signum：需要设置不阻塞的那个信号
        - 返回值：成功返回0， 失败返回-1
◼ int sigismember(const sigset_t *set, int signum);
    int sigismember(const sigset_t *set, int signum);
        - 功能：判断某个信号是否阻塞
        - 参数：
            - set：需要操作的信号集
            - signum：需要判断的那个信号
        - 返回值：
            1 ： signum被阻塞
            0 ： signum不阻塞
            -1 ： 失败

◼ int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
    int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
        - 功能：将自定义信号集中的数据设置到内核中（设置阻塞，解除阻塞，替换）
        - 参数：
            - how : 如何对内核阻塞信号集进行处理
                SIG_BLOCK: 将用户设置的阻塞信号集添加到内核中，内核中原来的数据不变
                    假设内核中默认的阻塞信号集是mask， mask | set
                SIG_UNBLOCK: 根据用户设置的数据，对内核中的数据进行解除阻塞
                    mask &= ~set
                SIG_SETMASK:覆盖内核中原来的值
            
            - set ：已经初始化好的用户自定义的信号集
            - oldset : 保存设置之前的内核中的阻塞信号集的状态，可以是 NULL
        - 返回值：
            成功：0
            失败：-1
                设置错误号：EFAULT、EINVAL
                
◼ int sigpending(sigset_t *set);
    int sigpending(sigset_t *set);
        - 功能：获取内核中的未决信号集
        - 参数：set,传出参数，保存的是内核中的未决信号集中的信息。
```



### 2.9 内核实现信号捕捉的过程

![image-20230227162714180](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230227162714180.png)

### 2.10 SIGCHLD信号

◼ SIGCHLD信号产生的条件 

- 子进程终止时 
- 子进程接收到 SIGSTOP 信号停止时 
- 子进程处在停止态，接受到SIGCONT后唤醒时 

◼ 以上三种条件都会给父进程发送 SIGCHLD 信号，父进程默认会忽略该信号



## 3.共享内存

### 3.1 共享内存概念

◼  共享内存**允许两个或者多个进程共享物理内存的同一块区域**（通常被称为段）。由于 一个共享内存段会称为一个进程用户空间的一部分，因此这种 IPC 机制**无需内核介入**。所有需要做的就是让一个进程将数据复制进共享内存中，并且这部分数据会对其 他所有共享同一个段的进程可用。

◼ 与管道等要求发送进程将数据从用户空间的缓冲区复制进内核内存和接收进程将数据 从内核内存复制进用户空间的缓冲区的做法相比，这种 IPC 技术的速度更快。



### 3.2 共享内存使用步骤

◼ 调用 shmget() 创建一个新共享内存段或取得一个既有共享内存段的标识符（即由其 他进程创建的共享内存段）。这个调用将返回后续调用中需要用到的共享内存标识符。 **(1.创建共享内存)**

◼ 使用 shmat() 来附上共享内存段，即使该段成为调用进程的虚拟内存的一部分。 **（2.与当前进程关联起来）**

◼ 此刻在程序中可以像对待其他可用内存那样对待这个共享内存段。为引用这块共享内存， 程序需要使用由 shmat() 调用返回的 addr 值，它是一个指向进程的虚拟地址空间 中该共享内存段的起点的指针。 

◼ 调用 shmdt() 来分离共享内存段。在这个调用之后，进程就无法再引用这块共享内存 了。这一步是可选的，并且在进程终止时会自动完成这一步。 （**3.共享内存分离**）

◼ 调用 shmctl() 来删除共享内存段。只有当当前所有附加内存段的进程都与之分离之 后内存段才会销毁。只有一个进程需要执行这一步。**（4.删除共享内存）**



### 3.3 共享内存操作函数

```c
共享内存相关的函数
#include <sys/ipc.h>
#include <sys/shm.h>
◼ int shmget(key_t key, size_t size, int shmflg); 
    int shmget(key_t key, size_t size, int shmflg);
        - 功能：创建一个新的共享内存段，或者获取一个既有的共享内存段的标识。
            新创建的内存段中的数据都会被初始化为0
        - 参数：
            - key : key_t类型是一个整形，通过这个找到或者创建一个共享内存。
                    一般使用16进制表示，非0值
            - size: 共享内存的大小
            - shmflg: 属性
                - 访问权限
                - 附加属性：创建/判断共享内存是不是存在
                    - 创建：IPC_CREAT
                    - 判断共享内存是否存在： IPC_EXCL , 需要和IPC_CREAT一起使用
                        IPC_CREAT | IPC_EXCL | 0664
            - 返回值：
                失败：-1 并设置错误号
                成功：>0 返回共享内存的引用的ID，后面操作共享内存都是通过这个值。
                
◼ void *shmat(int shmid, const void *shmaddr, int shmflg); 
    void *shmat(int shmid, const void *shmaddr, int shmflg);
        - 功能：和当前的进程进行关联
        - 参数：
            - shmid : 共享内存的标识（ID）,由shmget返回值获取
            - shmaddr: 申请的共享内存的起始地址，指定NULL，内核指定
            - shmflg : 对共享内存的操作
                - 读 ： SHM_RDONLY, 必须要有读权限
                - 读写： 0
        - 返回值：
            成功：返回共享内存的首（起始）地址。  失败(void *) -1
◼ int shmdt(const void *shmaddr); 
	int shmdt(const void *shmaddr);
    - 功能：解除当前进程和共享内存的关联
    - 参数：
        shmaddr：共享内存的首地址
    - 返回值：成功 0， 失败 -1
        
◼ int shmctl(int shmid, int cmd, struct shmid_ds *buf); 
	int shmctl(int shmid, int cmd, struct shmid_ds *buf);
    - 功能：对共享内存进行操作。删除共享内存，共享内存要删除才会消失，创建共享内存的进行被销毁了对共享内存是没有任何影响。
    - 参数：
        - shmid: 共享内存的ID
        - cmd : 要做的操作
            - IPC_STAT : 获取共享内存的当前的状态
            - IPC_SET : 设置共享内存的状态
            - IPC_RMID: 标记共享内存被销毁
        - buf：需要设置或者获取的共享内存的属性信息
            - IPC_STAT : buf存储数据
            - IPC_SET : buf中需要初始化数据，设置到内核中
            - IPC_RMID : 没有用，NULL
                
◼ key_t ftok(const char *pathname, int proj_id);
	key_t ftok(const char *pathname, int proj_id);
    - 功能：根据指定的路径名，和int值，生成一个共享内存的key
    - 参数：
        - pathname:指定一个存在的路径
            /home/nowcoder/Linux/a.txt
            / 
        - proj_id: int类型的值，但是这系统调用只会使用其中的1个字节
                   范围 ： 0-255  一般指定一个字符 'a'


```



### 3.4 共享内存操作命令

◼ ipcs 用法 

-  ipcs -a // 打印当前系统中所有的进程间通信方式的信息 
-  **ipcs -m // 打印出使用共享内存进行进程间通信的信息** 
-  ipcs -q // 打印出使用消息队列进行进程间通信的信息 
-  ipcs -s // 打印出使用信号进行进程间通信的信息 

◼ ipcrm 用法 

-  ipcrm -M shmkey // 移除用shmkey创建的共享内存段 
-  ipcrm -m shmid // 移除用shmid标识的共享内存段 
-  ipcrm -Q msgkey // 移除用msqkey创建的消息队列 
-  ipcrm -q msqid // 移除用msqid标识的消息队列 
-  ipcrm -S semkey // 移除用semkey创建的信号 
-  ipcrm -s semid // 移除用semid标识的信号



```
问题1：操作系统如何知道一块共享内存被多少个进程关联？

   - 共享内存维护了一个结构体struct shmid_ds 这个结构体中有一个成员 shm_nattch
     - shm_nattach 记录了关联的进程个数

问题2：可不可以对共享内存进行多次删除 shmctl

   - 可以的
     - 因为shmctl 标记删除共享内存，不是直接删除
       - 什么时候真正删除呢?
         当和共享内存关联的进程数为0的时候，就真正被删除
       - 当共享内存的key为0的时候，表示共享内存被标记删除了
         如果一个进程和共享内存取消关联，那么这个进程就不能继续操作这个共享内存。也不能进行关联。

    共享内存和内存映射的区别
    1.共享内存可以直接创建，内存映射需要磁盘文件（匿名映射除外）
    2.共享内存效果更高
    3.内存
        所有的进程操作的是同一块共享内存。
        内存映射，每个进程在自己的虚拟地址空间中有一个独立的内存。
    4.数据安全
   - 进程突然退出
     共享内存还存在
     内存映射区消失
        - 运行进程的电脑死机，宕机了
          数据存在在共享内存中，没有了
          内存映射区的数据 ，由于磁盘文件中的数据还在，所以内存映射区的数据还存在。
    5.生命周期
   - 内存映射区：进程退出，内存映射区销毁
     内存：进程退出，共享内存还在，标记删除（所有的关联的进程数为0），或者关机
     如果一个进程退出，会自动和共享内存进行取消关联。


```

## 4.守护进程⭐

### 4.1 终端

◼ 在 UNIX 系统中，用户通过终端登录系统后得到一个 shell 进程，这个终端成 为 shell 进程的控制终端（Controlling Terminal），进程中，控制终端是 保存在 PCB 中的信息，而 fork() 会复制 PCB 中的信息，因此由 shell 进 程启动的其它进程的控制终端也是这个终端。 

◼ 默认情况下（没有重定向），每个进程的标准输入、标准输出和标准错误输出都指 向控制终端，进程从标准输入读也就是读用户的键盘输入，进程往标准输出或标准 错误输出写也就是输出到显示器上。 

◼ 在控制终端输入一些特殊的控制键可以给前台进程发信号，例如 Ctrl + C 会产 生 SIGINT 信号，Ctrl + \ 会产生 SIGQUIT 信号。



### 4.2 进程组

◼ 进程组和会话在进程之间形成了一种两级层次关系：**进程组是一组相关进程的集合**， **会话是一组相关进程组的集合**。进程组和会话是为支持 shell 作业控制而定义的抽 象概念，用户通过 shell 能够交互式地在前台或后台运行命令。 

◼ 进行组由一个或多个共享同一**进程组标识符（PGID）**的进程组成。一个进程组拥有一 个进程组首进程，该进程是创建该组的进程，其进程 ID 为该进程组的 ID，新进程 会继承其父进程所属的进程组 ID。 

◼ 进程组拥有一个生命周期，其开始时间为首进程创建组的时刻，结束时间为最后一个 成员进程退出组的时刻。一个进程可能会因为终止而退出进程组，也可能会因为加入 了另外一个进程组而退出进程组。进程组首进程无需是最后一个离开进程组的成员。



### 4.3 会话

◼ 会话是一组进程组的集合。**会话首进程是创建该新会话的进程**，其进程 ID 会成为会 话 ID。新进程会继承其父进程的会话 ID。 

◼ 一个会话中的所有进程共享单个控制终端。控制终端会在会话首进程首次打开一个终端设备时被建立。一个终端最多可能会成为一个会话的控制终端。

◼ 在任一时刻，会话中的其中一个进程组会成为终端的前台进程组，其他进程组会成为 后台进程组。只有前台进程组中的进程才能从控制终端中读取输入。当用户在控制终 端中输入终端字符生成信号后，该信号会被发送到前台进程组中的所有成员。 

◼ 当控制终端的连接建立起来之后，会话首进程会成为该终端的控制进程。



### 4.4 进程组、会话、控制终端之间的关系

`◼ find / 2 > /dev/null | wc -l &` 

`◼ sort < longlist | uniq -c`

![image-20230301092919069](D:\WorkData\MarkdownData\Linux高并发\assets\image-20230301092919069.png)

### 4.5 进程组、会话操作函数

```c
◼ pid_t getpgrp(void); 

◼ pid_t getpgid(pid_t pid); 

◼ int setpgid(pid_t pid, pid_t pgid); 

◼ pid_t getsid(pid_t pid); 

◼ pid_t setsid(void);
```

### 4.6 守护进程

◼ 守护进程（Daemon Process），也就是通常说的 Daemon 进程（精灵进程），是 Linux 中的后台服务进程。它是一个生存期较长的进程，通常独立于控制终端并且周 期性地执行某种任务或等待处理某些发生的事件。一般采用以 d 结尾的名字。 

◼ 守护进程具备下列特征： 

- 生命周期很长，守护进程会在系统启动的时候被创建并一直运行直至系统被关闭。 
- 它在后台运行并且不拥有控制终端。没有控制终端确保了内核永远不会为守护进程自动生成任何控制信号以及终端相关的信号（如 SIGINT、SIGQUIT）。 

◼ Linux 的大多数服务器就是用守护进程实现的。比如，Internet 服务器 inetd， Web 服务器 httpd 等。



### 4.7 守护进程的创建步骤⭐

◼ 执行一个 fork()，之后父进程退出，子进程继续执行。 

◼ 子进程调用 setsid() 开启一个新会话。 setsid()

◼ 清除进程的 umask 以确保当守护进程创建文件和目录时拥有所需的权限。 	umask(022)

◼ 修改进程的当前工作目录，通常会改为根目录（/）。 chdir("/")

◼ 关闭守护进程从其父进程继承而来的所有打开着的文件描述符。 dup2(fd,STDIN_FILENO)

◼ 在关闭了文件描述符0、1、2之后，守护进程通常会打开/dev/null 并使用dup2()  使所有这些描述符指向这个设备。 

◼ 核心业务逻辑

```c
void work(int num) {
    // 捕捉到信号之后，获取系统时间，写入磁盘文件
    time_t tm = time(NULL);
    struct tm * loc = localtime(&tm);
    // char buf[1024];

    // sprintf(buf, "%d-%d-%d %d:%d:%d\n",loc->tm_year,loc->tm_mon
    // ,loc->tm_mday, loc->tm_hour, loc->tm_min, loc->tm_sec);

    // printf("%s\n", buf);

    char * str = asctime(loc);
    int fd = open("time.txt", O_RDWR | O_CREAT | O_APPEND, 0664);
    write(fd ,str, strlen(str));
    close(fd);
}

int main() {

    // 1.创建子进程，退出父进程
    pid_t pid = fork();

    if(pid > 0) {
        exit(0);
    }

    // 2.将子进程重新创建一个会话
    setsid();

    // 3.设置掩码
    umask(022);

    // 4.更改工作目录
    chdir("/home/nowcoder/");

    // 5. 关闭、重定向文件描述符
    int fd = open("/dev/null", O_RDWR);
    dup2(fd, STDIN_FILENO);
    dup2(fd, STDOUT_FILENO);
    dup2(fd, STDERR_FILENO);

    // 6.业务逻辑

    // 捕捉定时信号
    struct sigaction act;
    act.sa_flags = 0;
    act.sa_handler = work;
    sigemptyset(&act.sa_mask);
    sigaction(SIGALRM, &act, NULL);

    struct itimerval val;
    val.it_value.tv_sec = 2;
    val.it_value.tv_usec = 0;
    val.it_interval.tv_sec = 2;
    val.it_interval.tv_usec = 0;

    // 创建定时器
    setitimer(ITIMER_REAL, &val, NULL);

    // 不让进程结束
    while(1) {
        sleep(10);
    }

    return 0;
}
```

