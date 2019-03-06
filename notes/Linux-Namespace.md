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

## PID Namespace

再修改以下代码，添加一个 `syscall.CLONE NEWPID`，并且使用 `pstree -pl` 查看进程树关系。

```shell
$ pstree -pl
...
        ...go(6550)-+-main(6569)-+-sh(6572)
...
```

然后在 go 运行的 sh 查看进程 id：

```shell
$ echo $$
1
```

## Mount Namespace

Mount Namespace 是 Linux 第 一个实现 的 Namespace 类型，因此，它的系统调用参数 是 NEWNS ( New Namespace 的缩写)。

首先运行一下代码，然后查看一下 `/proc` 的文件内容。

```shell
$ go run main.go
$ ls /proc
1     16   360   6649  999          ipmi           sched_debug
10    17   390   6650  acpi         irq            schedstat
1026  18   407   6664  asound       kallsyms       scsi
11    19   409   6683  buddyinfo    kcore          self
119   2    415   6686  bus          key-users      slabinfo
12    20   46    6690  cgroups      keys           softirqs
122   21   47    7     cmdline      kmsg           stat
127   22   48    70    consoles     kpagecount     swaps
129   23   49    71    cpuinfo      kpageflags     sys
13    24   5     713   crypto       latency_stats  sysrq-trigger
136   25   6     8     devices      loadavg        sysvipc
137   27   6171  881   diskstats    locks          timer_list
1380  28   6262  884   dma          mdstat         timer_stats
1383  29   6263  889   driver       meminfo        tty
1386  3    6264  890   execdomains  misc           uptime
139   30   6314  892   fb           modules        version
14    301  6315  9     filesystems  mounts         version_signature
142   31   6375  913   fs           mtrr           vmallocinfo
15    32   6376  919   interrupts   net            vmstat
157   33   6377  920   iomem        pagetypeinfo   zoneinfo
158   34   6599  922   ioports      partitions
```

这里看到的 `/proc` 还是宿主机的，内容比较多，接下来把 `/proc` mount 到我们自己的 Namespace 下面来。

```shell
$ mount -t proc proc /proc
1          dma          key-users      mtrr          sysrq-trigger
4          driver       keys           net           sysvipc
acpi       execdomains  kmsg           pagetypeinfo  timer_list
asound     fb           kpagecount     partitions    timer_stats
buddyinfo  filesystems  kpageflags     sched_debug   tty
bus        fs           latency_stats  schedstat     uptime
cgroups    interrupts   loadavg        scsi          version
cmdline    iomem        locks          self          version_signature
consoles   ioports      mdstat         slabinfo      vmallocinfo
cpuinfo    ipmi         meminfo        softirqs      vmstat
crypto     irq          misc           stat          zoneinfo
devices    kallsyms     modules        swaps
diskstats  kcore        mounts         sys
```

可以看到，少了很多文件。下面就可以使用 `ps` 来查看系统的进程了。  

```shell
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 00:41 pts/0    00:00:00 sh
root         5     1  0 00:45 pts/0    00:00:00 ps -ef
```

可以看到，在当前的 Namespace 中， sh 进程是 PID 为 1 的进程。这就说明，当前的 Mount Namespace 中的 mount 和外部空间是隔离的， mount 操作并没有影响到外部。`Docker volume` 也 是利用了这个特性。

## User Namespace

现在宿主机上查看 root 用户 id：

```shell
$ id
uid=0(root) gid=0(root) groups=0(root)
```

然后查看容器内的 root 用户 id：
```shell
$ go run main.go
$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
```

## Network Namespace

首先，在宿主机上查看一下自己的网络设备，结果如下：

```shell
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:da:fb:b7  
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          inet6 addr: fe80::a00:27ff:feda:fbb7/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:141879 errors:0 dropped:0 overruns:0 frame:0
          TX packets:50171 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:139369760 (139.3 MB)  TX bytes:4052092 (4.0 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

可以看到，宿主机上有 lo、 ethO、 ethl 等网络设备。下面，运行一下程序去 Network Namespace 里面看看。

```shell
$ go run main.go
$ ifconfig
$ 
```

我们发现，在 Namespace 里面什么网络设备都没有。这样就能断定 Network Namespace 与宿主机之间的网络是处于隔离状态了。

