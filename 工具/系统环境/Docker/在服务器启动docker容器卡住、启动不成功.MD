# 在服务器启动docker容器卡住、无启动成功信息，docker ps 一a状态码137

    docker、ubuntu 20.04、emqx 5.8.0

## 背景

想从移动安卓设备往服务器发点数据，因为服务器有固定IP，想起来之前看过的这个mqtt，感觉比较合适，但是启动过程有点坎坷，本文记录其中一个问题。

## 问题描述

在服务器，使用emqx官网命令启动容器：

```bash
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

然而中间卡住了，没有启动成功，服务也无法使用，启动日志如下：

```log
WARNING: Default (insecure) Erlang cookie is in use.
2024-09-18T14:58:11.325377408Z WARNING: Configure node.cookie in /opt/emqx/etc/emqx.conf or override from environment variable EMQX_NODE__COOKIE
2024-09-18T14:58:11.325389391Z WARNING: NOTE: Use the same cookie for all nodes in the cluster.
2024-09-18T14:58:12.660521365Z EMQX_RPC__PORT_DISCOVERY [rpc.port_discovery]: manual
2024-09-18T14:58:12.660970597Z EMQX_NODE__NAME [node.name]: emqx@172.17.0.2
```

看警告提示，感觉会不会是cookie参数的问题，于是搜索后使用下面的命令启动容器：

```bash
docker run -e EMQX_NODE__COOKIE=com.linux.utunbu.docker.emqx -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

然而没有什么作用：

```log
EMQX_RPC__PORT_DISCOVERY [rpc.port_discovery]: manual
2024-09-18T15:02:41.480356524Z EMQX_NODE__COOKIE [node.cookie]: ******
2024-09-18T15:02:41.480367517Z EMQX_NODE__NAME [node.name]: emqx@172.17.0.2
```

服务器有点卡，在虚拟机尝试启动：

```bash
docker run -e EMQX_NODE__COOKIE=com.linux.utunbu.docker.emqx -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

启动成功了，服务也可用，日志如下：

```log
EMQX_RPC__PORT_DISCOVERY [rpc.port_discovery]: manual
EMQX_NODE__CoOKIE [node.cookie]: *****
EMQX_NODE__NAME [node.name]: emqx@172.17.0.2
Listener tcp:default on 0.0.0.0:1883 started.
Listener ssl:default on 0.0.0.0:8883 started.
Listener ws:default on 0.0.0.0:8083 started.
Listener wss:default on 0.0.0.0:8084 started.
Listener http:dashboard on :18083 started.
EMQx 5.8.0 is running now!
```

再次尝试使用官网命令，在虚拟机启动：

```bash
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

启动成功，服务可用，日志同上。

## 原因分析

同样的docker版本，同样的系统版本，为什么虚拟机能行，服务器就不行了呢？

求助ai助手：
> 你好，我的emqx服务容器启动到这里就不动了，也没有报错退出也没有成功启动服务，但是容器也没有停止运行，请问这是怎么回事

(实际上容器是停止运行了，只是因为idea的服务插件没有刷新容器状态，所以我这里没有发现)

回答提到：
> 服务启动卡住：
你提到 EMQX 容器启动后没有报错退出，也没有成功启动服务，但容器仍在运行。这种情况通常有几个可能的原因：
配置问题：虽然直接的错误没有显示在日志中，但某些配置错误可能导致 EMQX 无法完全启动。检查 emqx.conf 和其他相关配置文件，确保所有必要的配置都已正确设置。
资源限制：容器可能由于资源限制（如 CPU、内存或磁盘空间）而无法完全启动。检查 Docker 容器的资源限制设置。
网络问题：如果 EMQX 需要访问外部服务（如数据库、认证服务等），网络问题可能导致启动过程卡住。
依赖服务未就绪：如果 EMQX 依赖于其他服务（如数据库服务），而这些服务在 EMQX 启动时尚未就绪，也可能导致启动卡住。
> 
> 解决步骤
查看更详细的日志：
尝试增加日志级别，以获取更多关于启动过程的信息。
查看 Docker 容器的日志（使用 docker logs [容器ID或名称] 命令）。
> 
> 检查资源限制：
查看 Docker 容器的资源使用情况，确保有足够的资源供 EMQX 使用。

提到两点：1. 日志；2. 资源限制

1.1. 使用`docker logs ****`没有发现有效信息
1.2. 使用外挂日志和数据，也没有有效信息：
```bash
docker run -d --name emqx -e ERL_FLAGS="+c false" -p 1883:1883 -p 8083:8083   -p 8084:8084 -p 8883:8883   -p 18083:18083   -v $PWD/data:/opt/emqx/data   -v $PWD/log:/opt/emqx/log   emqx/emqx:5.8.0
```

2.1. ssh工具显示内存使用了一半多，服务器运营平台显示内存512MB，是有点少，但是也是有200MB，理论上应该够用了才对。
但是，多次询问ai都提示我资源问题，使用`docker ps -a` 命令，显示容器是`Exited (137) 14 minutes ago` 的状态。

再次问询爱助手得：
> 当你遇到 Docker 容器启动后很快退出（状态为 Exited (137)），这通常意味着容器内的进程被系统强制终止了。状态码 137 通常与容器内的进程收到了 SIGKILL 信号（即被系统强制杀死）有关。这种情况可能由多种原因引起，但以下是一些可能的解释和排查步骤：
>
> 内存不足：
尽管你提到服务器资源空间还足够（可能是指磁盘空间），但内存不足也可能导致容器被杀死。你可以检查服务器的内存使用情况，特别是当 EMQX 启动时是否有大量的内存被其他进程占用。你可以使用 free -m 或 top 命令来查看内存使用情况。
OOM Killer（Out of Memory Killer）：
如果系统内存不足，Linux 的 OOM Killer 会选择并杀死一些进程以释放内存。这通常发生在内存使用接近或超过物理内存和交换空间的总和时。你可以检查 /var/log/syslog 或 /var/log/messages（取决于你的系统配置）来查看是否有关于 OOM Killer 的日志条目。
>
> Docker 配置：
检查 Docker 的内存限制设置。如果你为 Docker 容器设置了内存限制，并且这个限制太低，那么容器可能会因为超出限制而被杀死。你可以使用 docker inspect [容器ID或名称] 命令来查看容器的内存限制设置。
> 
> EMQX 配置问题：
尽管 EMQX 本身的配置问题不太可能导致容器被杀死（除非它导致了内存泄漏或类似的严重问题），但检查 emqx.conf 文件中的配置仍然是一个好主意。特别是与内存使用相关的配置，如缓存大小、消息队列大小等。
> 
> 系统日志：
查看系统的日志文件，如 /var/log/syslog 或 /var/log/messages，以获取有关容器退出时系统状态的更多信息。
> 
> Docker 日志：
尽管你提到外挂的日志目录没有信息，但 Docker 本身的日志可能包含有关容器退出的有用信息。你可以使用 docker logs [容器ID或名称] 命令来查看 Docker 容器的日志。
尝试不同的 Docker 镜像或版本：
如果可能，尝试使用不同版本的 EMQX Docker 镜像来查看问题是否仍然存在。
检查 Docker 守护进程和内核日志：
有时 Docker 守护进程或内核日志中会包含有关容器退出的更多详细信息。
逐步调试：
如果可能，尝试以非守护进程模式（即不带 -d 选项）运行容器，以便你可以看到容器的实时输出。这可能会提供有关为什么容器退出的更多线索。
联系支持：
如果上述步骤都无法解决问题，你可能需要联系 EMQX 的支持团队或 Docker 社区以获取帮助。
最后，请注意，即使你的本地 Ubuntu 20.04 虚拟机可以正常运行相同的命令，服务器环境也可能存在不同的配置或限制，这可能导致不同的行为。因此，在排查问题时，请务必考虑服务器环境的特定因素。

一个是提醒我检查配置文件（其实没有配置文件），再一个就是说，可以使用`-m 32M --memory-swap 64M`（搜索来的命令）限制容器空间。

使用这个命令限制最大空间为32MB，丢包严重，频率稍高就会丢失，显然不可用：
```Bash
docker run -d -m 32M --memory-swap 64M --name emqx-min -e ERL_FLAGS="+c false" -p 1893:1883 -p 8093:8083 -p 8094:8084 -p 8893:8883 -p 18093:18083 emqx/emqx:5.8.0
```

使用这个命令限制最大空间为64MB，频率特别高的时候也会丢失，管理控制台的刷新率同样降频了，但丢包率比32MB大大降低了：
```Bash
docker run -d -m 64M --memory-swap 128M --name emqx-min -e ERL_FLAGS="+c false" -p 1893:1883 -p 8093:8083 -p 8094:8084 -p 8893:8883 -p 18093:18083 emqx/emqx:5.8.0
```

然而，然而64MB的容器启动了半天，卡了半天，终究还是死了，这说明服务器不可能运行这个mqtt的服务了。

## 解决方案

既然如此，我觉得就只能尝试，在服务器转发所有网络请求，到我的电脑上；在我的电脑上运行服务，处理请求了
