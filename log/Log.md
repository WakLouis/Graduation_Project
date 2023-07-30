<!--
 * @Descripttion: 
 * @version: 
 * @Author: WakLouis
 * @Date: 2023-07-05 10:15:43
 * @LastEditors: WakLouis
 * @LastEditTime: 2023-07-30 16:35:10
-->
## <center>支持作业控制的unix shell程序的设计与实现</center>

## 介绍
通过编写简单的Unix shell以熟悉作业控制和信号量。

## 安装步骤

1. 解压缩安装包 tar xvf shlab-handout.tar
1. 使用 make 进行编译
1. 编辑tsh.c

## 内容

需要实现以下函数：

1. eval： Main routine that parses and interprets the command line.解析和解释命令行的主例程。

1. builtin_cmd: Recognizes and interprets the built-in commands: quit, fg, bg, and jobs. 识别和解释内置命令：quit, fg, bg 和 jobs.

1. do_bgfg: Implements the bg and fg built-in commands. 实现内置命令bg, fg.

1. waitfg: Waits for a foreground job to complete. 等待前台任务完成。

1. sigchld_handler: Catches SIGCHILD signals.捕获SIGCHILD信号。

1. sigint_handler: Catches SIGINT(ctrl-c) signals.捕获SIGINT信号。

1. sigtstp_handler: Catches SIGTSTP(ztrl-z) signals. 捕获SIGTSTP信号。

## shell 特性
我的 tsh 应该具有如下特性：
1. 具有提示符："tsh> "。
1. 用户输入的命令应该由 name 和零或多个参数所组成，分隔符为数个空格，如果 name 是内置命令，tsh 应该立即处理并等待下一个命令，否则，tsh 应该认为 name 是某个可执行文件的路径，其被装载并运行在一个初始化的子进程中。
1. tsh 可以不支持管道和 IO 重定向。
1. 键入ctrl-c（ctrl-z）应该产生一个SIGINT（SIGTSTP）信号被发送至当前前台进程以及其任何子进程。如果不存在前台作业，则该信号应该没有任何影响。
1. 如果一个命令结束于&，则该命令运行于后台，否则前台。
1. 每一个作业都该被标识为一个PID或JID（正整数），JID具有前缀%。例如 %5 表示为 JID 5，而 5 则表示为 PID 5。
1. 支持以下内置命令：
    1. quit 命令用于结束该tsh
    1. jobs 命令用于列出所有后台作业。
    1. bg "job" 命令用于发送一个SIGCONT命令给该job，使得该job运行于后台。
    1. fg "job" 亦是如此，但是使其运行于前台。
1. tsh应该回收所有僵尸进程，如果一个作业因为收到一个未捕获的信号而终止（不理解什么意思），tsh应该注意到这个事件并打印出该作业的PID和这个“冒犯”信号的描述。

## 提示
1. 通过 trace.txt 文件逐步完成项目。
1. waitpid，kill，fork，execve，setpgid，sigprocmask 函数都非常有用，特别是waitpid的 WUNTRACED， WNOHANG 选项。
1. 当你部署你的信号处理函数后，确保 SIGINT 和 SIGTSTP 信号发送至整个前台进程组，在kill函数中使用 "-pid" 代替 "pid" （在sdriver中将会报错）。
1. 项目中最棘手的部分之一是决定在 waitfg 和 sigchld_handler 函数之间工作的分配，我们推荐如下方法：
    1. 在 waitfg 中，在 sleep 函数周围使用繁忙的循环(busy loop)。
    1. 在 sigchld_handler 中， 仅对 waitpid 进行一次调用。
1. 在 eval 中，必须在子进程创建前使用 sigprocmask 阻塞 SIGCHLD，在将子进程放入作业池后 unblock SIGCHLD，由于子进程继承了父进程的阻塞集合，因此需要确保子进程也 unblock SIGCHLD 在执行新程序之前。（书上详细说明了该进行该措施的原因）
1. 使用基于文本的程序，如ls，echo等。（无法使用more，vi等）
1. 当您从标准 Unix shell 运行 shell 时，您的 shell 正在前台进程组中运行。 如果您的 shell 随后创建了一个子进程，则默认情况下该子进程也将成为前台进程组的成员。 由于输入 ctrl-c 会将 SIGINT 发送到前台组中的每个进程，因此输入 ctrl-c 会将 SIGINT 发送到您的 shell 以及您的 shell 创建的每个进程，这显然是不正确的。
解决方法如下：在 fork 之后，在 execve 之前，子进程应该调用 setpgid(0, 0)，这会将子进程放入一个新的进程组中，其组 ID 与子进程的 PID 相同。 这可以确保前台进程组中只有一个进程，即您的 shell。 当您键入 ctrl-c 时，shell 应该捕获生成的 SIGINT，然后将其转发到适当的前台作业（或更准确地说，包含前台作业的进程组）。

## 日志

### 2023.7.5

Shell 接收一个 command line 时，如果第一个单词是一个内置命令，则立刻在当前进程中执行该命令，否则认为是一个可执行程序的路径。![Alt text](image-1.png)


### 2023.7.30

看完了CSAPP第八章
