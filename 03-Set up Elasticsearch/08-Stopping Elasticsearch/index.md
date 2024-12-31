[TOC]


# [Stopping Elasticsearch | 停止 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/stopping-elasticsearch.html)

---

An orderly shutdown of Elasticsearch ensures that Elasticsearch has a chance to cleanup and close outstanding resources. For example, a node that is shutdown in an orderly fashion will remove itself from the cluster, sync translogs to disk, and perform other related cleanup activities. You can help ensure an orderly shutdown by properly stopping Elasticsearch.
有序关闭 Elasticsearch 可确保 Elasticsearch 有机会清理和关闭未完成的资源。例如，有序关闭的节点将自身从集群中移除、将 translog 同步到磁盘并执行其他相关清理活动。您可以通过正确停止 Elasticsearch 来帮助确保有序关闭。

If you’re running Elasticsearch as a service, you can stop Elasticsearch via the service management functionality provided by your installation.
如果您将 Elasticsearch 作为服务运行，则可以通过安装提供的服务管理功能停止 Elasticsearch。

If you’re running Elasticsearch directly, you can stop Elasticsearch by sending control-C if you’re running Elasticsearch in the console, or by sending `SIGTERM` to the Elasticsearch process on a POSIX system. You can obtain the PID to send the signal to via various tools (e.g., `ps` or `jps`):
如果您直接运行 Elasticsearch，您可以通过发送 control-C（如果您在控制台中运行 Elasticsearch）来停止 Elasticsearch，或者通过向 POSIX 系统上的 Elasticsearch 进程发送`SIGTERM`来停止 Elasticsearch。您可以通过各种工具（例如`ps`或`jps`）获取要发送信号的 PID：

``` shell
$ jps | grep Elasticsearch
14542 Elasticsearch
```

From the Elasticsearch startup logs:
从 Elasticsearch 启动日志中：

``` log
[2016-07-07 12:26:18,908][INFO ][node                     ] [I8hydUG] version[5.0.0-alpha4], pid[15399], build[3f5b994/2016-06-27T16:23:46.861Z], OS[Mac OS X/10.11.5/x86_64], JVM[Oracle Corporation/Java HotSpot(TM) 64-Bit Server VM/1.8.0_92/25.92-b14]
```

Or by specifying a location to write a PID file to on startup (`-p <path>`):
或者通过指定启动时写入 PID 文件的位置（`-p <path>`）：

``` shell
$ ./bin/elasticsearch -p /tmp/elasticsearch-pid -d
$ cat /tmp/elasticsearch-pid && echo
15516
$ kill -SIGTERM 15516
```

---

## [Stopping on Fatal Errors | 发生严重错误时停止](https://www.elastic.co/guide/en/elasticsearch/reference/current/stopping-elasticsearch.html#fatal-errors)

During the life of the Elasticsearch virtual machine, certain fatal errors could arise that put the virtual machine in a questionable state. Such fatal errors include out of memory errors, internal errors in virtual machine, and serious I/O errors.
在 Elasticsearch 虚拟机的生命周期中，可能会发生某些致命错误，导致虚拟机处于可疑状态。此类致命错误包括内存不足错误、虚拟机内部错误和严重 I/O 错误。

When Elasticsearch detects that the virtual machine has encountered such a fatal error Elasticsearch will attempt to log the error and then will halt the virtual machine. When Elasticsearch initiates such a shutdown, it does not go through an orderly shutdown as described above. The Elasticsearch process will also return with a special status code indicating the nature of the error.
当 Elasticsearch 检测到虚拟机遇到此类致命错误时，Elasticsearch 将尝试记录错误，然后停止虚拟机。当 Elasticsearch 启动此类关闭时，它不会像上面描述的那样进行有序关闭。Elasticsearch 进程还将返回一个特殊的状态代码，指示错误的性质。

|                               |     |
| ----------------------------- | --- |
| Killed by jvmkiller agent     | 158 |
| User or kernel SIGTERM        | 143 |
| Slain by kernel oom-killer    | 137 |
| Segmentation fault            | 134 |
| JVM internal error            | 128 |
| Out of memory error           | 127 |
| Stack overflow error          | 126 |
| Unknown virtual machine error | 125 |
| Serious I/O error             | 124 |
| Bootstrap check failure       | 78  |
| Unknown fatal error           | 1   |