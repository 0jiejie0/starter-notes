# 启动VM报错：VMware Workstation 与 Device/Credential Guard 不兼容。在禁用 Device/Credenti...

    关键词：VMware Workstation、Device/Credential Guard、WSL、Docker、Hyper-V

## 背景

* 环境版本：win10、Ubuntu 22.04.2LTS、Docker、VM Workstation Pro 15
* 业务场景：想在工位电脑用虚拟机新开个系统，减少冲突，方便测试破坏性操作

## 问题描述

* 本来是可以启动vm虚拟机的，但是在安装完WSL及Docker并启动（期间设置过BIOS）后，无法启动VM，提示标题的错误。

## 原因分析

* 经验之谈：看到提示信息，推测和Docker有关
* 网络搜索：错误信息贴到百度，找到[这么一篇文章](https://www.cnblogs.com/sunny3158/p/16656965.html)，文中提到原因有二：

      1. Device Guard或Credential Guard与Workstation不兼容。
      2. Windows系统的Hyper-V不兼容导致。

  操作完成后可以打开VM，但是Docker打不开了，只能二选一

## 解决方案

步骤：

1. “win+ R”打开运行，输入gpedit.msc，确定 打开本地组策略编辑器
    * 转到 本地计算机策略 > 计算机配置 > 管理模板 > 系统 > Device Guard
    * 双击打开 基于虚拟化的安全设置为“已禁用”
2. “win+ R”打开运行，输入services.msc，确定
    * 打开 本地服务 > HV主机服务
    * 启动类型设置为“禁用”
3. “ 控制面板” > “卸载程序” > “打开或关闭Windows功能”
    * 取消勾选Hyper-V，选择不重启
4. 通过命令关闭Hyper-V（控制面板关闭Hyper-V起不到决定性作用）
    * “win+ x”,然后运行Windows Powershell (管理员) 也用cmd
    ```Bash
    bcdedit /set hypervisorlaunchtype off
    # bcdedit /set hypervisorlaunchtype auto  # 如需重新开启hyper，用本条命令
    ```
5. 重启电脑
