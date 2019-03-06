# Linux Namespace

> 学习笔记  

## UTS Namespace
运行 `go run main.go` 后，使用 `pstree -pl` 查看系统进程之间的关系：

```shell
$ pstree -pl
...
|           ...go(6019)-+-main(6038)-+-sh(6041)---pstree(6042)
|                                                                                                  |            |-{main}(6039)
|                                                                                                  |            `-{main}(6040)
...
```
然后，输出一下当前的 `PID`：

```shell
$ echo $$
6041
```

验证父进程和子进程是否不在同一 `UTS Namespace` 中：

```shell
$ readlink /proc/5845/ns/uts
uts:[4026531838]
$ readlink /proc/6038/ns/uts
uts:[4026531838]
$ readlink /proc/6041/ns/uts
uts:[4026532191]
```

由于 `UTS Namespace` 对 `hostname` 做了隔离，所以在这个环境内修改 `hostname` 应该不影响外部主机，做一下实验：

```shell
$ hostname -b bird
$ hostname
bird 
```

另开一个 shell，在宿主机查看 `hostname`：

```shell
$ hostname
ubuntu14
```

## IPC Namespace

在 `Cloneflags` 增加 `syscall.CLONE_NEWPIC`，来创建 IPC Namespace，打开两个 shell 来测试一下效果。  

创建一个 queue, 并查看现有的 IPC Message Queues：

```shell
$ ipcmk -Q
Message queue id: 0
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    
0x66d8524b 0          gzp        644        0            0           

```

然后运行一下 `go run main.go`，查看一下 queues：

```shell
$ ipcs -q

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages    

```



