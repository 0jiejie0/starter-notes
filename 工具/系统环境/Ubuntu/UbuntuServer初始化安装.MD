## 背景

为保持工作和学习中的环境一致性，减少因环境不同产生的麻烦，决定用兼容性高的虚拟机来完成这一要求，提高测试和迁移效率。

目前确定 Virtual Box 》 Ubuntu Server 20.04 LTS 》 Docker 技术架构；对于SQLServer等win应用，另外创建windows虚拟机来承载。

### 各技术版本选型及原因

* Virtual Box 最新版：可设定较大空间，不必再考虑迁移问题，方便复制、备份、迁移，对比 VMWare Workstation Pro，
    * VB 性能方面差一点。
    * vb虚拟磁盘的兼容性要好一些。
    * 易用性稍逊，配置复杂一点。
    * 选它的根本原因是，现在工位的电脑上已经安装了docker，无法同时运行vmware（虚拟化相关的系统设置有冲突）。
    * 对于版本问题没有深究。
    * vb是多平台兼容且开源的，不用担心版权和费用问题。
* Ubuntu Server 20.04.6 LTS (GNU/Linux 5.4.0-165-generic x86_64) ：主要承载开发及环境容器，便于携带开发工具迁移
    * linux开源，无版权和费用问题；在服务器市场linux应用最为广泛，选linux可以让开发生产环境更一致。
    * 之前用过ubuntu所以用这个顺手一些，再有一个就是ubuntu的生态支持好一些。
    * server版本无gui界面，性能更高（或说系统占用更少）；server和标准lts版本内核一致，问题方案好找。
    * ubuntu20开始，其公司开始加入snap，并开始逐步替代apt，在22版本中甚至去掉了apt，snap是好处挺多，但是国内用起来速度感人，关于snap的生态尚不完善。
    * 综上，最新的server版本选择20版更为合适。
* Docker：版本尚未深究，便于配置、升级、替换、备份、迁移部分软件环境。

### UbuntuServer安装及初始化过程

1. 在虚拟机内安装不用考虑分区问题，网络会自动检测DHCP
2. 安装程序最后有选择初始化安装程序的步骤，按空格选择需要用的程序（但是似乎不太好用，我勾选docker安装完以后，docker命令没有），其余一般用默认选项即可
3. 启动系统输入密码发现系统时间不对，用如下命令修改时区（参考链接：[ubuntu上修改时区的两种方法](https://blog.csdn.net/kaka_buka/article/details/131540461)）：
   ```Bash
   timedatectl  # 查看当前设置
   sudo timedatectl set-timezone Asia/Shanghai
   timedatectl  # 查看设置结果
   ```
