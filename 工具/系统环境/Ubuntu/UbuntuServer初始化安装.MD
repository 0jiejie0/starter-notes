## 背景

为保持工作和学习中的环境一致性，减少因环境不同产生的麻烦，决定用兼容性高的虚拟机来完成这一要求，提高测试和迁移效率。

目前确定 Virtual Box 》 Ubuntu Server 20.04 LTS 》 Docker 技术架构；对于SQLServer等win应用，另外创建windows虚拟机来承载。

### 各技术版本选型及原因

* Virtual Box 最新版：对比 VMWare Workstation Pro，VB 性能方面差一点，虚拟磁盘的兼容性要好一些，易用性稍逊，配置复杂一点。
  选它的根本原因是，现在工位的电脑上已经安装了docker，无法同时运行vmware（虚拟化相关的系统设置有冲突）。
  对于版本问题没有深究，另外vb是多平台兼容且开源的，不用担心版权和费用问题。
* Ubuntu Server 20.04.6LTS live-server-amd64，
