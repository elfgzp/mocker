# Linux Namespace

> 学习笔记  

## UTS Namespace
使用 `pstree -pl` 查看系统进程之间的关系：

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