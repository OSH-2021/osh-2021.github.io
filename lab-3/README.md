# Lab 3

在本次实验中，我们将学习简单的 socket 编程，分别利用多线程技术、IO 复用技术、或是 async/await 协程编写一个简单的多人聊天室程序。

首先给出一个简单的双人聊天室示例程序：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pthread.h>

struct Pipe {
    int fd_send;
    int fd_recv;
};

void *handle_chat(void *data) {
    struct Pipe *pipe = (struct Pipe *)data;
    char buffer[1024] = "Message:";
    ssize_t len;
    while ((len = recv(pipe->fd_send, buffer + 8, 1000, 0)) > 0) {
        send(pipe->fd_recv, buffer, len + 8, 0);
    }
    return NULL;
}

int main(int argc, char **argv) {
    int port = atoi(argv[1]);
    int fd;
    if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket");
        return 1;
    }
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons(port);
    socklen_t addr_len = sizeof(addr);
    if (bind(fd, (struct sockaddr *)&addr, sizeof(addr))) {
        perror("bind");
        return 1;
    }
    if (listen(fd, 2)) {
        perror("listen");
        return 1;
    }
    int fd1 = accept(fd, NULL, NULL);
    int fd2 = accept(fd, NULL, NULL);
    if (fd1 == -1 || fd2 == -1) {
        perror("accept");
        return 1;
    }
    pthread_t thread1, thread2;
    struct Pipe pipe1;
    struct Pipe pipe2;
    pipe1.fd_send = fd1;
    pipe1.fd_recv = fd2;
    pipe2.fd_send = fd2;
    pipe2.fd_recv = fd1;
    pthread_create(&thread1, NULL, handle_chat, (void *)&pipe1);
    pthread_create(&thread2, NULL, handle_chat, (void *)&pipe2);
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    return 0;
}
```

使用如下命令编译并运行：

```shell
gcc 1.c -o 1 -lpthread
./1 6666
```

接着两个终端中分别使用如下命令连接：

```shell
nc 127.0.0.1 6666
```

其中 `nc` 是 `netcat` 包内的一个程序

这样从一个终端输入的消息，会转发到另一个终端，于是便实现了两个客户端间的聊天。这个聊天室程序十分简陋，忽略了 socket 编程中的很多细节，你的任务就是修改它（或者从头编写），使得它更健壮，并且支持多人聊天。

## 完善双人聊天室

### 实现以换行为分隔符的消息分割

TCP 是一个基于流的协议，本身没有“消息”的概念。在整个连接中，客户端发送的数据只是一连串的字节，服务端并不能知道这些数据是以怎样的方式分割的（一次 `send` 并不一定对应对方的一次 `recv`）。示例程序将一次 `recv` 得到的数据视为一个消息，是不合适的。请重新实现接收数据的部分，使得以换行符 `\n` 为分隔符接收和分割数据，并以一条消息为单位发送给另一个客户端。在助教的测试中消息的长度不超过 1 MiB（1,048,576 字节）。

注意，尽管每次只接受一个字符，并判断其字符是否为换行符是一种可行的方法，这种方法是非常非常低效的（很明显的是，每个字符都会有一次 syscall）。我们要求在此次实验中尽量避免这种方法（但不禁止），请考虑一下如何在一次接受大量字符的情况下完成上述的消息分割。如果你实在不知道如何实现，方才使用接受单个字符的方法，具体给分会按文末的评分标准给分。

### 处理可能的 `send` 阻塞

当发送的数据长度足够大时，可能会引起 `send` 的阻塞，这时在 `recv` 返回后只有部分数据被成功发送。请处理这种情况，使得所有数据都能保证发送到客户端。

### 一些小提示

`recv` 和 `send` 中指定的发送/接受字节数和实际的发送/接受字节数都是可能不同的，上述两个要求需要考虑如何利用这两个数值来完成处理。

## 利用多线程技术实现多人聊天室

接下来，我们将实现一个支持多人在线的聊天室，具体要求如下：

- 最多支持 32 个用户同时在线，用户可以随时加入与退出

- 程序将用户发出的数据以换行符分割为消息，并将消息完整地发送给其他用户

你需要使用阻塞 IO 与多线程技术来实现上述需求。可以使用的线程库有 pthread, Windows API 或 `std::thread`。除了标准库与上述库，请勿使用其他库。在测试中消息的长度不超过 1 MiB。

请注意，socket 是**全双工**的，即可以由两个线程同时分别读取与写入。但并发地读取或并发地写入可能会造成意想不到的后果。请利用操作系统课程中线程同步的相关知识实现对 socket 的安全操作。

### pthread

如果你在类 Unix 系统下编程，可以选择使用 pthread 线程库，这里将对其基本用法做出介绍。

```c
int pthread_create(pthread_t *thread, const pthread_attr_t *attr, void *(*start_routine) (void *), void *arg);
int pthread_join(pthread_t thread, void **retval);
int pthread_detach(pthread_t thread);
```

`pthread_create` 函数用于创建并启动一个新的进程。第一个参数为指向我们要创建的线程对象的指针；第二个参数为创建时的属性，目前可以忽略；第三个参数为线程启动后将要执行的函数指针；第四个参数为传入线程执行函数的数据。

`pthread_join` 用于等待线程的结束，并接收返回值。第一个参数为线程对象，第二个参数为接收返回值的变量的指针，如果为 `NULL` 表示忽略返回值。

`pthread_detach` 可以将线程与主线程分离，使该线程运行结束后得以终止自己并释放资源。分离后的线程将不能被 `pthread_join` 等待。

另外 pthread 线程库还包含处理互斥锁的函数。

```c
int pthread_mutex_init(pthread_mutex_t *restrict mutex, const pthread_mutexattr_t *restrict attr);
int pthread_mutex_destroy(pthread_mutex_t *mutex);
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```

正如函数名所示，这四个函数分别对应了互斥锁的初始化、销毁、加锁和解锁。你也可以直接使用 `PTHREAD_MUTEX_INITIALIZER` 宏来初始化互斥锁。

pthread 也包含处理条件变量的函数，可用于阻塞和同步线程。

```c
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr);
int pthread_cond_destroy(pthread_cond_t *cond);
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
```

下面的代码展示了条件变量和互斥锁简单的使用方法：

```c
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cv = PTHREAD_COND_INITIALIZER;
int ready = 0;
int data;

void *worker(void *p) {
    pthread_mutex_lock(&mutex);
    while (!ready) {
        pthread_cond_wait(&cv, &mutex);
    }
    printf("%d\n", data);
    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, worker, NULL);
    pthread_mutex_lock(&mutex);
    sleep(1);
    data = 1234;
    ready = 1;
    pthread_cond_signal(&cv);
    pthread_mutex_unlock(&mutex);
    pthread_join(thread, NULL);
    return 0;
}
```

以上就是 pthread 线程库最基本的用法，如需进一步学习更高级的特性，请自行在互联网上搜索教程与文档。

### C++ 线程库

如果你比较熟悉 C++ ，也可以使用 C++ 标准库中的线程库。这里给出一个与上面的示例程序功能相同的例子：

```c++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <chrono>

int main() {
    int n = 0;
    int data;
    bool ready = false;
    std::mutex mutex;
    std::condition_variable cv;
    auto f = [&] {
        std::unique_lock<std::mutex> lock(mutex);
        cv.wait(lock, [&]{return ready;});
        std::cout << data << std::endl;
    };
    std::thread thread(f);
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lock(mutex);
        data = 1234;
        ready = true;
        cv.notify_one();
    }
    thread.join();
    return 0;
}
```

请自行参阅 C++ 手册学习线程库的用法。

## 选做：利用 IO 复用 / 异步 IO 技术实现多人聊天室

除了多线程，IO 复用与异步 IO 技术也是实现并发服务器的常见方式。在示例程序中，我们可以发现 `recv` 大部分时间都在等待数据的到来，实际上处于空闲状态。因此在这种情况下，多线程实际上是没有必要的，我们可以使用非阻塞 IO 与 IO 复用技术实现相同的功能。这里使用 Unix 标准中用于 IO 多路复用的系统调用 `select` 实现了与第一个示例程序相同的功能：

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/socket.h>
#include <sys/select.h>
#include <sys/time.h>
#include <netinet/in.h>

struct Pipe {
    int fd_send;
    int fd_recv;
};

int main(int argc, char **argv) {
    int port = atoi(argv[1]);
    int fd;
    if ((fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket");
        return 1;
    }
    struct sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_addr.s_addr = INADDR_ANY;
    addr.sin_port = htons(port);
    socklen_t addr_len = sizeof(addr);
    if (bind(fd, (struct sockaddr *)&addr, sizeof(addr))) {
        perror("bind");
        return 1;
    }
    if (listen(fd, 2)) {
        perror("listen");
        return 1;
    }
    int fd1 = accept(fd, NULL, NULL);
    int fd2 = accept(fd, NULL, NULL);
    if (fd1 == -1 || fd2 == -1) {
        perror("accept");
        return 1;
    }
    fd_set clients;
    char buffer[1024] = "Message:";
    fcntl(fd1, F_SETFL, fcntl(fd1, F_GETFL, 0) | O_NONBLOCK); // 将客户端的套接字设置成非阻塞
    fcntl(fd2, F_SETFL, fcntl(fd2, F_GETFL, 0) | O_NONBLOCK);
    while (1) {
        FD_ZERO(&clients);
        FD_SET(fd1, &clients);
        FD_SET(fd2, &clients);
        if (select(fd2 + 1, &clients, NULL, NULL, NULL) > 0) { // 找出可以读的套接字
            if (FD_ISSET(fd1, &clients)) {
                ssize_t len = recv(fd1, buffer + 8, 1000, 0);
                if (len <= 0) {
                    break;
                }
                send(fd2, buffer, len + 8, 0);
            }
            if (FD_ISSET(fd2, &clients)) {
                ssize_t len = recv(fd2, buffer + 8, 1000, 0);
                if (len <= 0) {
                    break;
                }
                send(fd1, buffer, len + 8, 0);
            }
        } else {
            break;
        }
    }
    return 0;
}
```

对于一般的阻塞的 socket ，进行 `recv` 操作时如果缓冲区中无数据可读，将会阻塞直到有数据为止，这样使得大量时间用于等待数据上。而对于非阻塞的 socket ，若无数据可读 `recv` 将直接返回，而不会阻塞住。再配合 `select` ，每次找出可读的文件描述符，就可以实现在一个线程中同时处理多个连接。`select` 系统调用还可以找出可写及出错的文件描述符，并设定等待检查完成的最长时间。你可以在互联网上查找更多关于 `select` 的资料。

除了所有 Unix 系统都有的 `select` 系统调用，Linux 的 `epoll`，FreeBSD 和 macOS 的 `kqueue` ，以及 Windows 的 `IOCP` 也能实现类似的功能，并且相比 `select` 都有很高的效率。这几种 IO 复用 / 异步 IO 接口有各自的语义与用法，你可以选择任意一种，并以此为基础实现一个多人聊天室。你可能需要学习一些非阻塞 IO 网络编程的基础知识，比如 Reactor 设计模式。

聊天室的要求与多线程版本的相同，但你**不得**使用多线程技术。除了标准库与所使用系统的库外，请勿使用任何第三方的网络库或事件处理库。

## 选做：async/await 协程

由于 C 是一种比较古老的语言，而 C++ 的 coroutine 支持在 C++20 才加入且不够完全，我们建议在实现此部分内容时使用 Rust 完成，但如果你愿意挑战 C/C++ 下的协程编程也是可行的。

仅在此章节中，我们允许使用热门的 Rust 异步编程库，例如 `tokio` 和 `async-std`。
如果你想使用其他的异步库亦可提前询问我们。

关于协程是什么可以参见这篇文章：https://www.baeldung.com/java-threading-models 。
async/await 的具体使用方法可以参见 `tokio` 的文档 https://docs.rs/tokio/1.6.1/tokio/ 。

async/await 可以如下理解：

`async fn f()` 将一个函数声明为了一个特殊的函数，进行 `f()` 调用时不会执行函数内容，而是返回一个 coroutine 对象，例如可以 `let a = f()`；对这个对象执行 `a.await` 会在运行到此处时实际运行函数内容，但同时会定义一个 breakpoint，线程到达这里时，可以继续执行 `a.await` 的函数内容，也可以去执行其他的 `.await` 点位。比喻性的说，协程是代码控制（而不是 OS 调度控制）的线程，代码可以实际规定一部分协程的运行顺序。因此，协程可以与线程**不**一一对应，这些协程应该由多少线程执行、应该先执行哪些协程是由最外层的 runner，或者说 runtime 决定的。

一个简单的 Rust async/await 协程例子可以参考 https://github.com/myl7/dnsl ，这是一个基于 async/await 实现的 DNS 中继器。

在足够现代的编程语言几乎都有 async/await 的实现，包括：

- Python，从 3.5 开始正式叫 async/await
- Javascript，Promise，从 ECMAScript 2017 开始标准化
- Rust，Future trait，从 1.39.0 开始
- Kotlin，structured coroutine，主打点之一/对比 Java 的优势之一

在本部分中，你需要将自己实现的客户端改写为用 async/await 实现。
一般来说，只要你的几个工作函数是 `async fn` 即可。
如果你不确定自己的实现是否“足够 async/await”，请积极询问助教们。

## 实验要求

请按照以下目录结构组织你的 GitHub 仓库：

```
(Git)                     // Git 仓库目录
├── lab3                  // 实验三根目录
│   ├── 1.c               // 完善后的双人聊天室的源代码
│   ├── 2.c               // 基于多线程的多人聊天室的源代码
│   ├── 3.c               // （可选）基于 IO 复用 / 异步 IO 的多人聊天室的源代码
│   ├── 4.c               // （可选）基于协程的多人聊天室的源代码
│   ├── Makefile          // （可选）你提供的 Makefile
│   └── README.md         // 实验报告
├── .gitignore
└── README.md
```

如果你的文件名与上述不同，在 README 中说明即可。

本次实验满分为 10 分。你需要完成：

- 你的经过完善的双人聊天室能够以换行符分割消息，处理长达 1 MiB 的消息，并完整地发送到另一客户端（2 分）
  - 回避了每次处理单个字符（2 分）
- 你的多线程聊天室能够满足前述要求，正确地实现群聊，并正确地处理对数据和 socket 对象的并发访问，不会造成数据损坏与崩溃（5 分）
- （可选）你的 IO 复用 / 异步 IO 聊天室能够满足前述要求，正确地实现群聊（5 分）
- （可选）你的协程聊天室能够满足前述要求，正确地实现群聊（2 分）

性能不是本实验关注的重点，只需要正确实现功能即可。与实验二相似，本实验不要求实验报告，但是推荐简单描述你的实现，以使助教能更好地理解你的代码，以免造成误解。

本次实验满分为 14 分，多出的分数不会被计入，但是可以兑换助教们的称赞。

## 其他说明

本实验可以使用 C/C++/Rust 语言完成，如果需要使用其他语言请先询问助教。

本实验可以使用 C/C++ 的标准库，也可以使用操作系统的库（如 POSIX 标准库或 Windows API），但除了 await/async 部分外其他部分不得使用任何第三方的网络库或事件处理库。使用此处没有列出的库前请询问助教。
