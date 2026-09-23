---
title: Linux 信号学习
updated: 2026-09-23
---
# Linux 信号学习

> 参考阅读路线

## 信号是什么？

以下是 [baeldung](https://www.baeldung.com/linux/) 的 [SIGINT And Other Termination Signals in Linux](https://www.baeldung.com/linux/sigint-and-other-termination-signals) 文章中关于信号的介绍：

> The [signals](https://man7.org/linux/man-pages/man7/signal.7.html) are a method of communication between processes. When a process receives a signal, the process interrupts its execution and a signal handler is executed.
>
> 以下是非官方翻译：
>
> 信号是进程间通信的一种方式。当进程接收到信号时，进程会中断其执行，并执行信号处理程序。

## 参考资料

### 入门

- [SIGINT And Other Termination Signals in Linux - Baeldung](https://www.baeldung.com/linux/sigint-and-other-termination-signals)
  - 适合第一次了解 Linux 信号。
  - 重点：`SIGINT`、`SIGTERM`、`SIGQUIT`、`SIGKILL` 的区别。

### Linux 官方 / 权威参考

- [signal(7) - Linux manual page](https://man7.org/linux/man-pages/man7/signal.7.html)
  - Linux 信号机制的核心参考资料。
  - 重点：signal disposition、default action、ignore、signal handler、blocking、pending。

- [kill(1) - Linux manual page](https://man7.org/linux/man-pages/man1/kill.1.html)
  - `kill` 命令的参考文档。
  - 重点理解：`kill` 本质上是“发送信号”，而不只是“杀死进程”。

- [Signal Handling - GNU C Library Manual](https://www.gnu.org/software/libc/manual/html_node/Signal-Handling.html)
  - 从程序和操作系统机制的角度理解信号。
  - 后续可以继续了解 `sigaction()`、signal mask 等内容。

### Shell、终端与 Job Control

- [GNU Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)
  - 用于理解 Bash 如何处理信号，以及 signal、终端、前台进程组和 Job Control 之间的关系。
  - 可以重点关注 `Ctrl+C`、`Ctrl+Z`、`SIGINT`、`SIGTSTP`。

### Docker / Kubernetes 实际应用

- [docker container stop - Docker Docs](https://docs.docker.com/reference/cli/docker/container/stop/)
  - 理解容器停止时的 `SIGTERM → 等待 → SIGKILL` 流程。

- [docker container kill - Docker Docs](https://docs.docker.com/reference/cli/docker/container/kill/)
  - 对比 `docker stop` 和 `docker kill`。

- [Pod Lifecycle - Kubernetes Documentation](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
  - 理解 Pod 终止、graceful shutdown、`terminationGracePeriodSeconds` 与 Linux 信号之间的关系。

- [Pod 的生命周期 - Kubernetes 中文文档](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/)
  - Kubernetes 官方中文版本。

## 建议阅读顺序

1. Baeldung：先建立 `SIGINT`、`SIGTERM`、`SIGKILL` 等信号的基本认识。
2. `signal(7)`：建立准确的 Linux signal 模型。
3. `kill(1)`：结合命令实际操作。
4. GNU Bash Manual：理解 `Ctrl+C`、终端和 Job Control。
5. Docker：观察 signal 在容器停止中的实际应用。
6. Kubernetes：进一步理解 Pod 的优雅终止机制。
7. GNU C Library：需要继续深入 signal 底层机制时再阅读。