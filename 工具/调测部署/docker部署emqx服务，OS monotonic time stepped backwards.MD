# docker部署emqx服务容器，启动成功几分钟后，报错：OS monotonic time stepped backwards

    virtual box7.0、utuntu 20.04、docker、emqx 5.8.0

## 背景

想从移动安卓设备往服务器发点数据，想起来之前看过的这个mqtt，感觉比较合适，但是启动过程非常坎坷，本文记录其中一个问题。

## 问题描述

在服务器，使用官网命令启动容器：

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

```bash
EMQX_RPC__PORT_DISCOVERY [rpc.port_discovery]: manual
2024-09-18T15:02:41.480356524Z EMQX_NODE__COOKIE [node.cookie]: ******
2024-09-18T15:02:41.480367517Z EMQX_NODE__NAME [node.name]: emqx@172.17.0.2
```

服务器有点卡，在虚拟机尝试启动：

```bash
docker run -e EMQX_NODE__COOKIE=com.linux.utunbu.docker.emqx -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

然而就遇到了本文的问题：启动成功后，过十分钟左右就自动停了，报时钟回拨错误，日志如下：

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
Os monotonic time stepped backwards! Maybe add '+c false' in vm.args
Previous time: 334354311973031
Current time: 334354311968566
```

再次尝试使用官网命令，在虚拟机启动：

```bash
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```

可以看到启动日志中有和服务器一样的警告（说明这个警告不影响服务启动），但还是出现本文的问题：过十分钟左右就出错了。

```log
WARNING: Default (insecure) Erlang cookie is in use.
2024-09-18T15:05:12.013218504Z WARNING: Configure node.cookie in /opt/emqx/etc/emqx.conf or override from environment variable EMQX_NODE__COOKIE
2024-09-18T15:05:12.013220393Z WARNING: NOTE: Use the same cookie for all nodes in the cluster.
2024-09-18T15:05:12.945809242Z EMQX_RPC__PORT_DISCOVERY [rpc.port_discovery]: manual
2024-09-18T15:05:12.946053895Z EMQX_NODE__NAME [node.name]: emqx@172.17.0.2
2024-09-18T15:05:16.087475180Z Listener tcp:default on 0.0.0.0:1883 started.
2024-09-18T15:05:16.092158298Z Listener ssl:default on 0.0.0.0:8883 started.
2024-09-18T15:05:16.093685945Z Listener ws:default on 0.0.0.0:8083 started.
2024-09-18T15:05:16.099282516Z Listener wss:default on 0.0.0.0:8084 started.
2024-09-18T15:05:16.466725429Z Listener http:dashboard on :18083 started.
2024-09-18T15:05:16.579731083Z EMQX 5.8.0 is running now!
2024-09-18T15:15:23.499373563Z OS monotonic time stepped backwards! Maybe add '+c false' in vm.args
2024-09-18T15:15:23.500109220Z Previous time: 366377271755571
2024-09-18T15:15:23.500921994Z Current time:  366377271753471
```

## 原因分析

（本文要解决的问题是时间同步的问题，关于服务器没有启动成功回头另开一篇文章解决。）

### 虚拟机和容器的时间同步服务排查

搜索`emqx服务 如何处理系统时钟回拨`、`消息队列服务 如何处理系统时钟回拨`，找到的多是关于雪花算法的问题，其解决方案是写代码，不适用本问题；

另外还搜索到[部分文章](https://blog.csdn.net/java_wxid/article/details/131384369)中提到这个（泛泛而谈的废话，无用）：

    三、中间件处理时钟回拨的方法
    1.使用NTP保持时钟同步
    使用NTP是保持时钟同步的最常见方法之一。NTP会定期向时间服务器请求时间同步，并根据响应结果调整系统时钟。
    不过，由于NTP依赖于网络传输，因此会存在延迟、丢包等问题。在网络不稳定的环境中，NTP同步可能会失败。
    此时，中间件需要有相应的容错机制来处理时钟回拨。

然而同样的时钟回拨的方法说明，[这篇文章](https://blog.csdn.net/u012534547/article/details/142324064)说的就详细一点，有可取之处：

    如何应对时钟回拨问题？
    分布式系统通常会采取一些策略来避免或缓解时钟回拨带来的问题：
    NTP渐进调整（Slew Mode）：
    NTP有两种调整时间的方式：步进调整（Step Mode）和渐进调整（Slew Mode）。
    渐进调整不会直接回拨时间，而是通过调整时钟速度，让系统时间逐渐与标准时间对齐，这样可以避免时间突然回退带来的问题。

这提示我可能是虚拟机的时间同步带来的问题，然而搜索`linux   时间渐进调整`
，找到[这篇文章](https://www.cnblogs.com/xujiecnblogs/p/16787567.html)，
其中提到的ntp相关命令以及所谓配置文件`/etc/sysconfig/ntpd`（还有其它文章里提到的配置文件）我都没有；

我想到linux不同版本会有区别，调整关键词搜索`utuntu  时间渐进调整`，[这篇文章](https://blog.csdn.net/sorcererr/article/details/128675919)
以及搜索`ubuntu查看是否开始时钟同步`
，和[这篇文章](https://blog.csdn.net/h4241778/article/details/108921585#:~:text=%E6%88%91%E4%BB%AC%E5%8F%AF%E4%BB%A5%E9%80%9A%E8%BF%87%E8%BF%90%E8%A1%8C%E6%B2%A1%E6%9C%89%E5%8F%82%E6%95%B0%E7%9A%84timedatectl%E6%9D%A5%E6%9F%A5%E8%AF%A2timesyncd%E7%9A%84%E7%8A%B6%E6%80%81%E3%80%82%20%E8%BF%99%E5%B0%86%E6%89%93%E5%8D%B0%E5%87%BA%E6%9C%AC%E5%9C%B0%E6%97%B6%E9%97%B4%EF%BC%8C%E9%80%9A%E7%94%A8%E6%97%B6%E9%97%B4%EF%BC%88%E5%A6%82%E6%9E%9C%E6%82%A8%E6%B2%A1%E6%9C%89%E4%BB%8EUTC%E6%97%B6%E5%8C%BA%E5%88%87%E6%8D%A2%EF%BC%8C%E5%8F%AF%E8%83%BD%E4%B8%8E%E6%9C%AC%E5%9C%B0%E6%97%B6%E9%97%B4%E7%9B%B8%E5%90%8C%EF%BC%89%EF%BC%8C%E4%BB%A5%E5%8F%8A%E4%B8%80%E4%BA%9B%E7%BD%91%E7%BB%9C%E6%97%B6%E9%97%B4%E7%8A%B6%E6%80%81%E4%BF%A1%E6%81%AF%E3%80%82%20System%20clock%20synchronized%3A%20yes%E8%A1%A8%E7%A4%BA%E6%97%B6%E9%97%B4%E5%B7%B2%E6%88%90%E5%8A%9F%E5%90%8C%E6%AD%A5%EF%BC%8C%20systemd-timesyncd.service%20active%3A,-ntp%20on%20%E5%86%8D%E6%AC%A1%E8%BF%90%E8%A1%8Ctimedatectl%E4%BB%A5%E7%A1%AE%E8%AE%A4%E7%BD%91%E7%BB%9C%E6%97%B6%E9%97%B4%E7%8A%B6%E6%80%81%E3%80%82%20%E5%AE%9E%E9%99%85%E5%90%8C%E6%AD%A5%E5%8F%AF%E8%83%BD%E9%9C%80%E8%A6%81%E4%B8%80%E5%88%86%E9%92%9F%EF%BC%8C%E4%BD%86%E6%9C%80%E7%BB%88Network%20time%20on%3A%E5%92%8CNTP%20synchronized%3A%E5%BA%94%E8%AF%A5%E8%AF%BB%E5%8F%96yes%20%E3%80%82)
提到查看时间同步状态的命令：

```bash
timedatectl status
```

然而我的虚拟机中并没有开启时间同步：

```log
               Local time: Wed 2024-09-18 15:26:39 UTC
           Universal time: Wed 2024-09-18 15:26:39 UTC
                 RTC time: Wed 2024-09-18 12:57:02
                Time zone: UTC (UTC, +0000)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

没开启就无所谓怎么同步了，我启动emqx容器，想看看容器的同步状态，排查容器问题，然而容器中连查看状态的命令`timedatectl`
都没有，更没有所谓同步服务了。

### 硬件时钟同步，vbox虚拟机宿主机间同步排查

这之中还想到一个virtual box虚拟机与宿主机的硬件时钟同步问题，使用如下方法：

首先打开vbox安装目录，打开命令行工具

```bash
./VBoxManage.exe list vms  # 查看虚拟机列表 
```

禁用时间同步：

```bash
VBoxManage setextradata "<虚拟机名/虚拟机UUID>" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" "1"
```

启用时间同步：

```bash
VBoxManage setextradata "<虚拟机名/虚拟机UUID>" "VBoxInternal/Devices/VMMDev/0/Config/GetHostTimeDisabled" "0"
```

然而如此禁用以后还是有时钟回拨的错误，重复开启关闭也是如此。

### 咨询ai

咨询文心一言：`你好，我在virtualbox的ubuntu虚拟机的dodker中启动emqx服务容器，但是运行一段时间后爆出了这个错误 OS monotonic time stepped backwards! Maybe add '+c false' in vm.args 请问这是怎么回事`
回答如下：

    当你在使用 Docker 容器运行 EMQX（或其他任何基于 Erlang 的应用，如 RabbitMQ）并遇到 "OS monotonic time stepped backwards!" 这样的错误时，
    这通常与 Erlang 运行时环境对系统时钟的敏感性有关。Erlang 运行时使用单调时间（monotonic time）来确保时间的连续性和递增性，
    这对于处理超时、调度等任务至关重要。
    
    这个错误表明 Erlang 运行时检测到了系统时间的不一致或倒退，这可能是由于多种原因造成的，包括但不限于：
    
    虚拟机或宿主机的时钟问题：如果宿主机的时钟设置不正确，或者虚拟机与宿主机的时钟同步出现问题，就可能导致虚拟机内的系统时间出现倒退。
    NTP 服务：如果虚拟机或宿主机正在使用 NTP 服务来同步时间，但在某些情况下 NTP 服务突然调整时间（特别是向后调整），就可能触发这个错误。
    Docker 容器的时间同步：Docker 容器本身可能不会默认与宿主机同步时间，除非宿主机的 NTP 服务非常稳定且同步准确。
    解决方案
    确保宿主机时间同步：
    确保宿主机（即运行 VirtualBox 的机器）上安装了 NTP 客户端，并且已经配置为与可靠的 NTP 服务器同步。
    检查并修复宿主机的时钟设置，确保它准确无误。
    虚拟机设置：
    在 VirtualBox 的设置中，确保启用了与宿主机的时间同步（如果 VirtualBox 支持并默认启用此功能）。
    如果可能，尝试将虚拟机的时间设置与宿主机解耦，并让虚拟机内部运行 NTP 客户端来同步时间。
    Docker 容器设置：
    确保 Docker 容器内的时间与宿主机保持一致，或者让容器内部的 NTP 客户端与宿主机或外部 NTP 服务器同步。
    如果容器内部不需要与宿主机时间完全同步，可以考虑在 EMQX 的启动参数中添加 +c false 来禁用 Erlang 的单调时间检查。
    这通常不推荐作为长期解决方案，因为它可能会隐藏其他潜在的时间同步问题。
    EMQX 配置：
    如果决定在 EMQX 的启动参数中添加 +c false，你需要在 EMQX 的启动脚本或 Dockerfile 中修改启动命令。
    例如，在 Docker 容器中，你可以在启动 EMQX 的命令前添加 -e ERL_FLAGS="+c false" 环境变量
    （注意：具体的环境变量名称可能因 EMQX 的版本和部署方式而异，这里只是一个示例）。
    监控和日志：
    监控宿主机和虚拟机的时间同步状态。
    检查 EMQX 和 Docker 容器的日志文件，以获取更多关于时间错误的上下文信息。
    最后，请注意，在生产环境中禁用 Erlang 的单调时间检查可能会引入其他难以预测的问题，因此最好先尝试解决时间同步的根本问题。

照这样说，我可以加个参数（原来这个参数是这么加的，网上死活搜不到啊，还是ai好，不容易人为控制资料权重），禁用单调时间检查，但治标不治本，用集群的话还是要解决这个问题。

### 手动触发回拨，缩小排查范围

如果启动容器，手动回拨虚拟机的时间，没立即报错，说明容器和虚拟机时钟是隔离的，是容器内部的问题，要设法启动时间同步或同步虚拟机时钟；
如果容器立即报错，说明是虚拟机时钟回拨引起容器回拨，虚拟机和容器没有隔离，可以排查虚拟机回拨原因，应该不能隔离容器时钟，否则各容器时钟漂移影响服务；
将虚拟机调正后，手动调整物理机时间，如果没立即报错，说明虚拟机和物理机时钟是隔离的，是虚拟机内部的时间同步问题；
如果立即报错，说明是物理机时钟回拨引起了虚拟机回拨，需要解除同步时钟。

在虚拟机手动执行`date -s 06:12:00`或`date -s 2024-09-19`修改虚拟机时间，

1. 开启容器，`date`查看容器时间和虚拟机时间一致
2. 将虚拟机时间往大调，查看容器时间同步变大了
3. 将虚拟机时间往小调，查看容器时间同步变小了
4. 使用`hwclock --systohc`将虚拟机硬件时间更新为系统时间，没什么变化
5. 然而，emqx容器却没有报错！真tm神奇了！！容器时间都改了，竟然没报时间回拨错误！竟然可以坚持20多分钟，这，超乎我的预料了

通过先修改虚拟机时间，再启动容器的顺序，发现emqx的服务的时间记录是跟随虚拟机时钟变化的，但是容器的退出的时间戳差值量级却没有变大

后来也不报错了，自己直接就停了，我想是不是由于我手动修改的时间跨度太大，导致其内部时间同步失败了，失败卡退了？

然而，不管我多久调一次时间，容器并没有立即报错，依然可以运行，过段时间又会自己关闭。

### 排查其他因素（通过上一步，本步不再有必要细究）

突然想到，我是用的vagrant启动的虚拟机，虽然依赖vbox，但是部分配置是靠vg的配置文件配置启动项的，但网上没有找到相关说明；

还想到要不要另外创建一个专门用来同步时钟的容器，现在看没必要了；

有人告诉我之前他们生产环境一直用的centos7，让我换一下试试，我换了试了（不得不说yum是真难用），然而还是有这个问题。

### 柳暗花明

偶然看到之前搜索`OS monotonic time stepped backwards! Maybe add '+c false' in vm.args`bing的结果，
第一条有个[github上rabbitmq出现时钟回拨错误的讨论](https://github.com/rabbitmq/rabbitmq-server/discussions/8940)，
说这是erlang25之后才出现的特性，搜了一下erlang，25那是22年才出来，挺新的，禁掉无妨，就算是集群应该也问题不大，禁了得了，里边我也改不了，最多调参至此。

答案中还提到[一个链接](https://learnyousomeerlang.com/time)，
详细讲述了erlang时间的前世今生，然而太长我草草看了看开头和结尾，没看懂，说实话，只知道较真的话这个时间很难搞。

#### 我还没看其文件日志

我先看一眼再试用ai的方案吧，挂载文件重开容器(注意这个命令要使用root账户，在root的home目录以外执行，否则容器无权操作)：

```Bash
docker run -d --name emqx   -p 1883:1883 -p 8083:8083   -p 8084:8084 -p 8883:8883   -p 18083:18083   -v $PWD/data:/opt/emqx/data   -v $PWD/log:/opt/emqx/log   emqx/emqx:5.8.0
```

然而半个小时容器时钟回拨报错后，并无任何日志文件产生，白瞎！

## 解决方案

采用[ai助手回答](#咨询ai)末尾的解决方案，直接加`-e ERL_FLAGS="+c false"`参数禁用单调时间检查，再挂载一下日志和数据看看有没有。

到`/tmp/emqx`目录执行：

```Bash
docker run -d --name emqx -e ERL_FLAGS="+c false" -p 1883:1883 -p 8083:8083   -p 8084:8084 -p 8883:8883   -p 18083:18083   -v $PWD/data:/opt/emqx/data   -v $PWD/log:/opt/emqx/log   emqx/emqx:5.8.0
```

现在再等半个小时，看看还会不会再报错。成功超过一个小时，应该没问题了。

```Bash
docker run -d --name emqx -e ERL_FLAGS="+c false" -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:5.8.0
```
