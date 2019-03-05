# mocker
🤔《自己动手写 docker》学习代码

## 环境准备

* Ubuntu 14.04
* 内核版本 3.13.0-83-generic
* Go 版本 1.7.1

由于需要更换特定内核，docker 共享内核无法更换，需要安装虚拟机。

1. Mac 安装 virtualbox，[下载地址](https://www.virtualbox.org/wiki/Downloads)

2. 下载 Ubuntu 14.04 镜像，[下载地址](https://mirrors.163.com/ubuntu-releases/14.04/)

3. 修改 Ubuntu 源：

```shell
$ wget git.io/superupdate.sh
$ chmod +x superupdate.sh
$ ./superupdate.sh
```

4. 更换 Ubuntu 内核

```shell
$ sudo apt-get install linux-image-3.13.0-83-generic linux-image-extra-3.13.0-83-generic
$ sudo apt-get purge linux-image-4.4.0-31-generic linux-image-extra-4.4.0-31-generic
$ sudo update-grub
$ sudo reboot
```

5. 下载 Go 1.7.1

```shell
$ wget https://dl.google.com/go/go1.7.1.linux-amd64.tar.gz
$ tar -C /usr/local -xzf go1.7.1.linux-amd64.tar.gz
$ mkdir -p /go/src
```

在 `~/.bashrc` 中增加：
```plain
export GOPATH="/go"
export PATH="$PATH:/usr/local/go/bin"
```
然后执行：
```shell
$ source ~/.bashrc
```
