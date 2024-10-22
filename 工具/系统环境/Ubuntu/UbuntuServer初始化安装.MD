## 背景

为保持工作和学习中的环境一致性，减少因环境不同产生的麻烦，决定用兼容性高的虚拟机来完成这一要求，提高测试和迁移效率。

目前确定 Virtual Box 》 Ubuntu Server 20.04 LTS 》 Docker 技术架构；对于SQLServer等win应用，另外创建windows虚拟机来承载。

### 各技术版本选型及原因

* Virtual Box 最新版：可设定较大空间，不必再考虑迁移问题，方便复制、备份、迁移，对比 VMWare Workstation Pro，
    * VB 性能方面差一点。
    * vb虚拟磁盘的兼容性要好一些。
    * 易用性稍逊，配置复杂一点。
    * 选它的根本原因是，现在工位的电脑上已经安装了docker，无法同时运行vmware（虚拟化相关的系统设置有冲突）。
        * 最新好消息发现，在新版本vm上它们可以共存，这样虚拟机可以更平稳地向vb过度，升级到最新的 17 版本即可，
          [参照文章](https://blog.csdn.net/longfengshuanwu/article/details/133126116)。
    * 对于版本问题没有深究。
    * vb是多平台兼容且开源的，不用担心版权和费用问题。
    * vb可以无界面启动，配置成熟后，可以在后台无感运行
* Ubuntu Server 20.04.6 LTS (GNU/Linux 5.4.0-165-generic x86_64) ：主要承载开发及环境容器，便于携带开发工具迁移
    * linux开源，无版权和费用问题；在服务器市场linux应用最为广泛，选linux可以让开发生产环境更一致。
    * 之前用过ubuntu所以用这个顺手一些，再有一个就是ubuntu的生态支持好一些。
    * server版本无gui界面，性能更高（或说系统占用更少）；server和标准lts版本内核一致，问题方案好找。
    * ubuntu20开始，其公司开始加入snap，并开始逐步替代apt，在22版本中甚至去掉了apt，snap是好处挺多，但是国内用起来速度感人，关于snap的生态尚不完善。
    * 综上，最新的server版本选择20版更为合适。
* Docker：版本尚未深究，便于配置、升级、替换、备份、迁移部分软件环境。

### UbuntuServer安装及初始化过程

#### 用vbox从镜像手动创建虚拟机

本节与[下一节](#利用vagrant创建虚拟机)二选一

##### 1. openssh服务安装上（我感觉应该安装）；

##### 2. 设置固定IP（建议）或DHCP（默认）

**！这里没有成功设置固定IP！！** 开发环境需要固定IP的，移步[下一节利用vagrant安装](#利用vagrant创建虚拟机)

在虚拟机内安装不用考虑分区问题，网络会自动检测DHCP，但是为了以后连接方便还是建议用固定IP，已经启用DHCP的，进行如下操作可设置固定IP（未配置成功）：

        其实一般来讲，dhcp虚拟机结合vb的桥接模式，一般情况下IP是不会怎么变化的，我想用nat模式配合固定IP的主要原因就是在不变化IP的情况下，保证较高安全性（防止外部直接连接）
        但是，这并不是当下的主要矛盾，网络这一块能用就行，环境配好或者遇到问题以后再来研究也不算太迟

```shell
## 注意！本操作未成功配置

$ ip addr # 找到网卡enp0s3（后缀会有变化，enp打头的就是），本机IP 10.0.2.15/24
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:59:7f:0f brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 467sec preferred_lft 467sec
    inet6 fe80::a00:27ff:fe59:7f0f/64 scope link
       valid_lft forever preferred_lft forever
$ ip route show # 查看网关IP 10.0.2.1
default via 10.0.2.1 dev enp0s3 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15
10.0.2.1 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100
$ sudo cp /etc/netplan/00-installer-config.yaml /etc/netplan/00-installer-config.yaml.2023-11.bak # 备份网络配置文件
$ sudo vim /etc/netplan/00-installer-config.yaml # 编辑网络配置,文件内容如下：
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
# 可以看到，enp0s3的DHCP是开启状态，需要改成下面的内容：
network:  
  ethernets:    
    ens33:                        # 网卡
      dhcp4: false      
      addresses: [10.0.2.15/24]   # 设置本机IP及掩码
      routes:
          - to: default
            via: 10.0.2.1
      nameservers:        
        addresses: [114.114.114.114, 8.8.8.8]   #设置DNS  
  version: 2
  renderer: networkd
$ sudo netplan apply # 应用配置
$ sudo systemctl restart systemd-networkd # 重启网络服务
```

**注意！以上未配置成功**

搜素：[ubuntu dhcp 固定IP](https://www.baidu.com/s?wd=ubuntu+dhcp+%E5%9B%BA%E5%AE%9AIP&ie=UTF-8&tn=62095104_26_oem_dg)，
参考：[CSDN](https://blog.csdn.net/WwwALlll/article/details/133280034)、
[百度](https://baijiahao.baidu.com/s?id=1774340357193047656&wfr=spider&for=pc)

[//]: # (todo 暂时不知道上边这两条命令好不好使，也有只说明到倒二，没有最后这条命令的，都执行了，有问题再说)

##### 3. 选择程序组件

安装程序最后有选择初始化安装程序的步骤，按空格选择需要用的程序（但是似乎不太好用，我勾选docker安装完以后，docker命令没有，可以一个都不选），其余一般用默认选项即可

##### 4. 安装源手动输入阿里云的源：http://mirrors.aliyun.com/ubuntu

##### 5. 最后执行分区前记得断开网络连接，以免自动联网更新

#### 利用vagrant创建虚拟机

使用vagrant+ubuntu2004镜像，可实现nat，避免网络问题。

##### 1. virtual box安装好并启动，修改其默认虚拟机目录为`D:\simulation`。

##### 2. [下载vagrant](https://www.vagrantup.com/)，安装后需要重启电脑，

在系统环境变量中添加`VAGRANT_HOME`项，值为：`D:\simulation\vagrant\.vagrant.d`
（虚拟机路径+vagrant镜像路径，如果这里不设置会默认放到c盘用户目录下）

**注意注意！任何路径中都不能出现中文，否则后续过程会报错**

##### 3. 在虚拟机目录`D:\simulation\vagrant`下新建一个node01（虚拟机的名称可以写别的）文件夹，

然后命令行进入该目录`D:\simulation\vagrant\node01`，执行下文中的命令；

安装vagrant虚拟机，安装方式有两种：

###### 其一是使用命令在线下载并安装

```Bash
$ vagrant init generic/ubuntu2004 # 初始化配置
$ vagrant up # 第一次使用镜像会自动启动下载，之后会系统启动，在virtual box窗口中可以看到
```

###### 其二是手动下载[一个镜像](https://app.vagrantup.com/generic/boxes/ubuntu2004)，然后本地安装。

（个人推荐这种安装方法，因为1可以用工具加速下载，2下载以后可以永久使用，不再受网络及主机限制）

```Bash
# 下载的镜像应该是.box文件，如果不是把后缀名加上
# 将镜像文件放到虚拟机目录
$ vagrant box list # 查看是否存在
# 加载镜像，指定的名称要尽量简短，同时要体现镜像信息
$ vagrant box add ../vagrant-ubuntu2004-virtualbox-16cbdc9c-5d4a-4d68-bf17-f35bbc6946d2.box --name generic-ubuntu2004
$ vagrant box list # 查看是否存在
$ vagrant.exe init generic-ubuntu2004 # 初始化虚拟机配置（注意名称指定的是刚刚加载的镜像）
$ vagrant.exe up # 自动配置，自动启动
```

注意，这里init命令执行后会生成Vagrantfile文件，后续会用到。

##### 4. 配置虚拟机网络并连接

上一步启动后，可以直接在虚拟机目录使用`vagrant ssh`命令连接；
从启动窗口可以看到，其默认会将虚拟机22端口映射到主机的2222端口，所以使用`ssh vagrant@127.0.0.1 -p 2222`也能连接，密码默认为vagrant；
多一层映射总是不太方便，所以这里配置一下固定IP。

修改node01目录下的配置文件 Vagrantfile：

```Bash
# 下面这个vim是我用git bash打开用的命令，可以用记事本直接打开 Vagrantfile，修改对应行即可
$ vim Vagrantfile
# 输入 :set nu 回车
# 光标定位到35行  # config.vm.network "private_network", ip: "192.168.33.10"
# 按下yy p复制一下，按下i编辑其中一行，去掉注释，改为：
config.vm.network "private_network", ip: "192.168.56.10"
```

如果需要nat的同时还需要桥接，则可参照[这篇文章](https://blog.csdn.net/u013915286/article/details/117999505)，附加如下配置：

```bash
  config.vm.network "public_network", bridge: "Realtek USB GbE Family Controller", ip: "192.168.10.16"
```

其中，bridge和IP可省去，启动时根据提示选择要桥接的网卡，选择时记住网卡名称，启动后使用`ip addr`查看虚拟机分配到的IP,将网卡名称和ip在此处重新配置即可。

打开virtual box，关闭node01...虚拟机，打开刚刚的cmd窗口，再次执行`vagrant.exe up`命令；

使用`ssh vagrant@192.168.56.10`连接，密码为vagrant。
ps: 如果使用ssh连接没有反应，这可能是virtualbox的bug，重新打开一下vbox的管理页面就好了

#### 开启远程登录权限，以便使用远程工具

这里用ssh有可能会遇到如下错误，解决方案同附：

```bash
#在物理机尝试连接虚拟机
> ssh vagrant@192.168.56.70  # 出错
The authenticity of host '192.168.56.70 (192.168.56.70)' can't be established.
RSA key fingerprint is SHA256:Gk8aZS5IboYykGJkfOZnSXj4w+TgOq/6UOknVp7Xp4w.
RSA key fingerprint is MD5:83:dc:b3:1e:86:ac:09:a9:c5:a3:6f:82:fb:80:de:59.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.70' (RSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).

# 使用vagrant进入虚拟机，修改配置
> vagrant.exe ssh
> sudo -i
> vim ~/.ssh/config
UserKnownHostsFile ~/.ssh/known_hosts

> ssh vagrant@192.168.56.70  # 再次出错
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
> vim /etc/ssh/sshd_config
 62 # To disable tunneled clear text passwords, change to no here!
 63 PasswordAuthentication yes  ### 打开这行的注释,启用密码身份验证
 64 #PermitEmptyPasswords no
 65 #PasswordAuthentication no
> systemctl restart sshd.service  # 重启sshd服务，否则不生效，使用 service ssh reload 也可
> ssh vagrant@192.168.56.70
# 登录成功
```

超级管理员root使用密码连接：

```Bash
> sudo passwd # 修改密码
> su # 切换到root用户（或者重新登录也可），无特殊说明，下文均使用root权限操作，也可以使用`sudo -i`命令切换到root用户，使用当前用户密码
> vim /etc/ssh/sshd_config
# 输入 “:set nu ”打开行号显示
# 光标定位到 34 #PermitRootLogin prohibit-password，输入 yy p 复制一行，按下i编辑，修改为：
PermitRootLogin yes
# :wq保存并退出
> systemctl restart sshd
```

[参考链接](https://blog.csdn.net/m0_55905467/article/details/133744201)

自此，可以使用ssh工具记住密码连接了。推荐`mobaXterm`，不要用`Xshell`、`FinalShell`或者`git Bash`、`powershell`等，难用或有乱码。

[//]: # (todo 或者也可以配置免密登录，挖个坑。)

#### 启动系统输入密码（如果）发现系统时间不对

用如下命令修改时区

```Bash
$ timedatectl
               Local time: Fri 2024-02-23 09:04:09 UTC
           Universal time: Fri 2024-02-23 09:04:09 UTC
                 RTC time: Fri 2024-02-23 09:04:06
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
$ timedatectl set-timezone Asia/Shanghai
$ timedatectl
               Local time: Fri 2024-02-23 17:04:52 CST
           Universal time: Fri 2024-02-23 09:04:52 UTC
                 RTC time: Fri 2024-02-23 09:04:48
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
$ systemctl restart systemd-timesyncd # 这个命令好像没有十分必要，这步之前就已经改过来了
```

（参考链接：[ubuntu上修改时区的两种方法](https://blog.csdn.net/kaka_buka/article/details/131540461)）：

### docker 安装

#### 更新源（目前国内docker已无法下载镜像）

各种源(可以不换，docker用不到这个，各站点已关闭，完全不需要)；

```shell
#阿里云源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#测试版源
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
#源码
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#测试版源
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
#清华大学源
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
#测试版源
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
#源码
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
#测试版源
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

```shell
$ cp /etc/apt/sources.list /etc/apt/sources.list.bcakup # 备份
$ vim /etc/apt/sources.list
# 放入上面的清华和阿里源，保存退出
```

#### 不要更新，不要安装显卡驱动。

#### 卸载冲突包（卸载曾安装的文件）

```shell
# 卸载冲突包
$ for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; 
do sudo apt-get remove $pkg; 
done
# 清除Docker Engine, CLI, containerd, and Docker Compose包
$ apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras
# 清除主机上的Images, containers, volumes, or custom configuration files
$ rm -rf /var/lib/docker
$ rm -rf /var/lib/containerd
```

参考[CSDN](https://blog.csdn.net/m0_49336687/article/details/123953726)、
[docker官网](https://docs.docker.com/engine/install/ubuntu/)

#### 利用apt仓库安装

##### 1. 设置docker apt仓库

```Bash
# Add Docker's official GPG key:
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
```

##### 2. 安装 docker 包

安装最新ce版

```Bash
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
$ docker version # 看一下版本
Client: Docker Engine - Community
 Version:           25.0.4
 API version:       1.44
 Go version:        go1.21.8
 Git commit:        1a576c5
 Built:             Wed Mar  6 16:32:14 2024
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          25.0.4
  API version:      1.44 (minimum version 1.24)
  Go version:       go1.21.8
  Git commit:       061aa95
  Built:            Wed Mar  6 16:32:14 2024
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.28
  GitCommit:        ae07eda36dd25f8a1b98dfbf587313b99c0190bb
 runc:
  Version:          1.1.12
  GitCommit:        v1.1.12-0-g51d5e94
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
$ sudo docker run hello-world # 测试一下
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Digest: sha256:6352af1ab4ba4b138648f8ee88e63331aae519946d3b67dae50c313c6fc8200f
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
$ docker images # 刚刚下载的hello-world镜像
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    d2c94e258dcb   10 months ago   13.3kB
```

#### 配置阿里云镜像加速服务（目前国内docker已无法下载镜像）

1.

打开阿里云官网的控制台》左上角菜单》产品与服务》容器服务》容器镜像服务，找到[镜像加速器](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

2. 操作文档》Ubuntu
3. 执行如下命令：

```Bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://nb779h8d.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
