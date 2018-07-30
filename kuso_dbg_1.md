---
title: 手撸调试器(1) —— 载入 inferior
date: 2018-07-30 16:18:50
tags:
    - 调试器
categories:
    - C
---

近几天放假无视可做，便开始翻 Linux 中国的微信公众号找乐子，在他们的推送里，我发现了这篇[]文章](https://linux.cn/article-9849-1.html)，看完后有了想要自己写一个玩具调试器玩玩的想法。于是就有了这个系列的<del>坑</del>文章。

在正式开始写调试器之前，我们先要了解下 `inferior` 的概念，在 `gdb` 里面，`inferior` 表示的是被调试的东西。它可能是一个进程，可能是运行在一个虚拟机上的内核，也可能是运行在通过各种方式与这台计算机连接起来的远程设备上的进程。我们要写的调试器的作用，就是帮助用户检测程序的错误。我们在本系列文章中实现的调试器大致支持的功能有如下几个：设置断点、暂挂已启动的程序、单步执行代码和检查变量的内容 —— 这几个功能也是我在使用 `gdb` 时候常用的几个初级功能。

### 载入 inferior

要实现一个调试器，我们首先要做的就是载入 `inferior`，这里我们主要是使用 `linux` 提供的 `execv()` 来实现这一功能，该函数的[声明](https://linux.die.net/man/3/execv)如下：

```c
int execv(const char *path, char *const argv[]);
```

其中，`path` 是我们要调试的程序的路径，`argv` 是附加进去的命令行选项。

#### ptrace()

载入 `inferior` 时，我们还需要使用[ `ptrace`](https://linux.die.net/man/2/ptrace) 函数来表明，我们创建的这个进程是可以被调试的，我们先来看一下 `ptrace` 函数的声明：

```c
long ptrace(enum __ptrace_request request, pid_t pid, void *addr, void *data);
```

通过这个函数，我们可以实现很多调试功能，甚至可以说我们整个编译器都是构建在其上的。我们在以后也会遇到很多次这个函数调用，但是现在我们需要的仅仅是把我们的 `inferior` 设置为可以被父进程调试的，这时我们只需要将 `__ptrace_request request` 设为 `PTRACE_TRACEME`，其余的参数我们暂时可以不管，只需要设为 `0` 即可。为了在最开始就可以调试我们的程序，我们需要在 `inferior` 运行之前就执行 `ptrace`，还需要让我们的调试器和 `inferior` 之间为父子进程关系，这时候我们就需要  `fork` 来解决这一问题。

#### fork()

`fork()`  以及 `exec()` 这两个函数家族是在符合 `POSIX` 标准的系统上创建新进程的最重要的函数。`fork` 的作用是克隆进程，也就是将原先的一个进程再克隆出一个来，克隆出的这个进程就是原进程的子进程，这个子进程和其他的进程没有什么区别，同样拥有自己的独立的地址空间。克隆之后父子进程如同分道扬镳。如下图：

![5b5eb4ee70165](https://i.loli.net/2018/07/30/5b5eb4ee70165.jpg)

`fork` 之后父子就开始走不同的路线。

`fork` 执行的返回值有三种可能：

- 该进程为父进程时，返回子进程的 `pid`  
- 该进程为子进程时，返回 `0`  
- `fork`执行失败，返回 `-1`

因此我们可以根据返回值来判断是父进程还是子进程。

#### 实现

根据前面的几个系统提供的函数，我们就能写出来装载 `inferior` 的函数：

```c
static void setup_inferior(const char *path, char **argv);
static void loop(void);

void kuso_exec_inferior(const char*  inferior_path,
                              char** argv) {
	pid_t result = fork();
	switch (result) {
    	case 0: // Now I am the son.
      		setup_inferior(inferior_path, argv);
      		break;
	    case -1: // Error
      		break;
    	default: // I am your father.
      		loop();
      		break;
	}
}

static void setup_inferior(const char *path, char **argv) {
	ptrace(PTRACE_TRACEME, 0, 0, 0);
	execv(path, argv);
}

static void loop(void) {
  for(;;); // Loop forever...
}
```

辅以相应的驱动程序：

```c
int main(void) {
	char *argv[] = {0};
	kuso_exec_inferior("./fuck",argv);
	return 0;
}
```

即可对 `./fuck` 这个程序进行调试。我们的程序正如预期的一样，陷入死循环，大致原理是这样的，`ptrace` 的手册里有如下说明：

>   While being traced, the tracee will stop each time a signal is
>        delivered, even if the signal is being ignored.  (An exception is
>        **SIGKILL**, which has its usual effect.)  The tracer will be notified at
>        its next call to [waitpid(2)](http://man7.org/linux/man-pages/man2/waitpid.2.html) (or one of the related "wait" system
>        calls); that call will return a *status* value containing information
>        that indicates the cause of the stop in the tracee.  While the tracee
>        is stopped, the tracer can use various ptrace requests to inspect and
>        modify the tracee.  The tracer then causes the tracee to continue,
>        optionally ignoring the delivered signal (or even delivering a
>        different signal instead).
> 
>        If the **PTRACE_O_TRACEEXEC** option is not in effect, all successful
>        calls to [execve(2)](http://man7.org/linux/man-pages/man2/execve.2.html) by the traced process will cause it to be sent a
>        **SIGTRAP** signal, giving the parent a chance to gain control before the new progra

根据这个说明，我们可以推测死循环时候发生了什么：

1. `inferior` 收到了 `SIGTRAP` 信号，并停下等待调试器做出回应

2. 调试器沉浸在死循环中毛都没做

现在我们了解了，父进程要被对调♂试的子进程负♂责。因此我们需要让父进程在子进程收到信号时做点什么。

### 如何对子进程负♂责

我们可以使用 `waitpid` 系统调用来获取子进程的状态并以此进行一些处理。该函数的声明如下：

```c
pid_t waitpid(pid_t pid, int *status, int options);
```

其中我们要用到的是前两个参数，第一个是我们在调试的进程的 `pid`，第二个是该函数要写入返回值的位置。第三个我们暂时不需要，填一个 `0` 即可。返回的数字人眼基本看不出来什么信息，但是好在我们有如下几个宏来帮助判断状态：

| 宏名         | 作用          |
| ---------- | ----------- |
| WIFEXITED  | 判断进程是否退出    |
| WIFSTOPPED | 判断进程是否停止    |
| WSTOPSIG   | 返回导致进程停止的信号 |

更多的宏请去手册内查看。

利用这几个宏，我们就可以对子进程做一些简单的操♂作了，如下：

```c
static void setup_inferior(const char *path, char **argv);
static void attach_inferior(pid_t inferior_pid);

void kuso_exec_inferior(const char*  inferior_path,
                              char** argv) {
	pid_t result = fork();
	switch (result) {
    	case 0:
      		setup_inferior(inferior_path, argv);
      		break;
	case -1:
      		break;
    	default:
-           loop();
+      		attach_inferior(result);
      		break;
	}
}

static void setup_inferior(const char *path, char **argv) {
	ptrace(PTRACE_TRACEME, 0, 0, 0);
	execv(path, argv);
}

static void attach_inferior(pid_t inferior_pid) {
	for(;;) {
		int status;
		waitpid(inferior_pid, &status, 0);

		if (WIFSTOPPED(status) && WSTOPSIG(status) == SIGTRAP) {
			puts("SIGTRAP Found...\n");
			ptrace(PTRACE_CONT, inferior_pid, 0, 0);
		} else if (WIFEXITED(status)) {
			puts("Inferior exited...\n");
			exit(0);
		}
	}
}
```

这样再次执行我们的程序，就可以看到对应的输出了。

现在这个调试器只是简单的实现了加载 `inferior` 的功能，仍有很多功能没有实现。我们会在以后的文章<del>如果有的话</del>里一步一步的实现完整。

本文的完整代码在[这里](https://github.com/kuso-kodo/kuso_dbg/tree/1773a6662345479dd081ed74edc4ef473e91e4f8)。
