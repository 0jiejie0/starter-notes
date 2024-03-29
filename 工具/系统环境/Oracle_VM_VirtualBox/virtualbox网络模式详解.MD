## 未指定

此模式可用于测试。例如，您可以短时间启用此网络模式以模拟拔下电缆。

## NAT

默认情况下，为虚拟网络适配器启用此网络模式。
VM 上的来宾操作系统可以使用虚拟 NAT（网络地址转换）设备访问物理局域网 （LAN） 中的主机。
当NAT模式用于VirtualBox网络时，无法从主机或网络中的其他计算机访问来宾计算机。

例如，对于希望仅将 VM 用于 Internet 访问的用户，此默认网络模式已足够。

可以通过单击端口转发按钮直接从VirtualBox VM网络设置窗口配置端口转发

## NAT网络

此模式类似于用于配置路由器的 NAT 模式。
如果对多个虚拟机使用 NAT网络 模式，则它们可以通过网络相互通信。

VM 可以访问物理网络中的其他主机，也可以访问外部网络（包括 Internet）。
来自外部网络的任何计算机都无法访问配置为使用 NAT 网络模式的 VM（类似于将路由器配置为从家庭网络访问 Internet 时）。
使用 NAT 网络模式时，您无法从主机访问来宾计算机（除非您在全局 VirtualBox 网络设置中配置端口转发）。

内置的VirtualBox NAT路由器使用VirtualBox主机的物理网络接口控制器作为外部网络接口（就像NAT模式一样）。

[//]: # (todo virtual box 网络模式 后续内容再次配置时再行整理)
## 桥接网卡

## 内部网络

## 仅主机网络 host-only

## 通用驱动

本文仅介绍各功能作用，详细用法见[参考出处](https://segmentfault.com/a/1190000043810778)。