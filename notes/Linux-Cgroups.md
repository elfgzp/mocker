# Linux-Cgroups

> Linux Cgroups (Control Groups)提供了对 一 组进程及将来子进程的资源限制、控制和统 计的能力，这些资源包括 CPU、内存、存储、网络等 。 通过 Cgroups，可以方便地限制某个进 程的资源占用，并且可以实时地监控进程的监控和统计信息 。

## Cgroups 中的 3 个组件

* cgroup 
* subsystem
* hierarchy

> 如何看到当前的内核支持哪 些 subsystem 昵?可以安装 cgroup 的命令行工具( apt-get install cgroup-bin)，然后通过
lssubsys 看到 Kernel 支持的 subsystem。
```shell
$ lssubsys -a cpuset
cpu , cpuacct blkio
memory
devices
freezer
net_cls, net_prio
perf_event
hugetlb
pids
```

> 三个组件相互的关系
> 通过上面组件的描述，不难看出， Cgroups 是凭借这三个组件的相互协> 作实现的 。 那么， 这三个组件是什么关系呢?
> * 系统在创建了新的 hierarchy 之后，系统中所有的进程都会加入这个 hierarchy 的 cgroup 根节点，这个 cgroup 根节点是 hierarchy 默认创建的， 2.2.2 小 节在这个 hierarchy 中创建 的 cgroup 都是这个 cgroup 根节点的子节点。
> * 一个 subsystem 只能附加到一个 hierarchy上面。
> * 一个 hierarchy 可以附加多个 subsystem。
> * 一个进程可以作为多个 cgroup 的成员，但是这些 cgroup 必须在不同的 hierarchy 中 。
> * 一 个进程 fork 出子进程时，子进程是和父进程在同一个 cgroup 中的，也可以根据需要
将其移动到其他 cgroup 中 。 


