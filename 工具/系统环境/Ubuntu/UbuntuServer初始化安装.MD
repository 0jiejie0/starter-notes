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

1. openssh服务安装上（我感觉应该安装）；
2. 在虚拟机内安装不用考虑分区问题，网络会自动检测DHCP，但是为了以后连接方便还是建议用固定IP，已经启用DHCP的，进行如下操作可设置固定IP（未配置成功）：

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

3. 安装程序最后有选择初始化安装程序的步骤，按空格选择需要用的程序（但是似乎不太好用，我勾选docker安装完以后，docker命令没有，可以一个都不选），其余一般用默认选项即可
4. 安装源手动输入阿里云的源：http://mirrors.aliyun.com/ubuntu
5. 最后执行分区前记得断开网络连接，以免自动联网更新
6. 启动系统输入密码发现系统时间不对

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

7. 开启root远程登录权限，以便使用远程工具

```Bash
$ sudo passwd # 修改密码
$ su # 切换到root用户（或者重新登录也可），无特殊说明，下文均使用root权限操作
$ vim /etc/ssh/sshd_config
# 输入 “:set nu ”打开行号显示
# 光标定位到 34 #PermitRootLogin prohibit-password，输入 yy p 复制一行，按下i编辑，修改为：
PermitRootLogin yes
# :wq保存并退出
$ systemctl restart sshd
```
[参考链接](https://blog.csdn.net/m0_55905467/article/details/133744201)


```Bash
timedatectl  # 查看当前设置
sudo timedatectl set-timezone Asia/Shanghai
timedatectl  # 查看设置结果
```

### docker 安装

#### 更新源

各种源；

```shell
#阿里云源
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#測試版源
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
#源碼
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
#測試版源
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
#清华大学源
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
#測試版源
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
#源碼
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
#測試版源
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse
```

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bcakup # 备份
sudo vim /etc/apt/sources.list
# 放入上面的清华和阿里源，保存退出
```

不更新，不安装显卡驱动。


```shell
# 卸载冲突包
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; 
do sudo apt-get remove $pkg; 
done
# 清除数据
sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

```



参考[CSDN](https://blog.csdn.net/m0_49336687/article/details/123953726)、
docker官网
