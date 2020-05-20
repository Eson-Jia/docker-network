# docker network

主要梳理`docker`网络相关的知识

## 基础结构

[容器]eth0 <--> vethxx [docker0 bridge]<--NAT-->主机A物理网卡 <-->  Internet

## 容器内部进程访问其他主机上的TCP服务

主机A: ip:192.168.1.200
主机B: ip:192.168.1.22

因为经过源地址映射（source NAT）,我们在其他主机上拿到的客户端 ip 是 主机A物理网卡的 ip

```bash
$ nc -l -k -v -n 192.168.1.200 8080
Listening on [192.168.1.200] (family 0, port 8080)
Connection from 192.168.1.22 43025 received!
hello world
```

```bash
nc 192.168.1.200 8080 -v
192.168.1.200 (192.168.1.200:8080) open
hello world
```

在主机A上使用`nc`在`8080`端口开启一个`TCP`服务,在主机B上运行`docker run --name client --rm -it busybosx`开启一个`busybox`容器的同时进入命令终端，在命令行输入`nc 192.168.1.200 8080 -v`连接到`TCP`服务，接着输入`hello world`然后回车，主机A上的服务就会收到并打印到终端上。

```bash
$ sudo conntrack -L --proto tcp | grep 43025
tcp      6 431595 ESTABLISHED src=172.17.0.2 dst=192.168.1.200 sport=43025 dport=8080 src=192.168.1.200 dst=192.168.1.22 sport=8080 dport=43025 [ASSURED] mark=0 use=1
```

## 容器内部进程访问其他容器的TCP服务
