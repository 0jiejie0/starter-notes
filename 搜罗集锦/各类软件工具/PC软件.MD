# PC平台的软件工具

## 易用软件

utool，ai资源内容使用[芒果商城](https://www.mgidshop.com/)

## 配置过程

新机装机时记录，特此留存备用

包含装机初期各软件配置步骤及要点

```
联想驱动 无配置
安全软件360离线版，安装病毒库 360卫士 360杀毒
office 无配置
360zip
关闭防火墙/自动更新
	安全中心：计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\wscsvc start值改为4
	更新：计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UsoSvc start值改为4
激活 w10 - 带红点的激活工具（小马激活工具大按钮，好像不行）
任务栏合并按钮/开始菜单设置
鼠标指针速度
修改文件管理器隐藏项、默认打开资源管理、添加快速访问项目
更改默认安装位置
	设置中调整默认盘区
	计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion 修改ProgramFilesDir及86项路径（别的不要修改）
显示秒数 计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Advanced，添加一项32位值为1的ShowSecondsInSystemClock
Windows Search服务改为手动
联网，登录microsoft账户
更改更新服务为手动，安装todo
罗技驱动 登录账号同步
vmware 无配置
virtual box 无配置
360极速浏览器X 暴力猴插件
chrome 暴力猴插件
mobaxterm 复制配置文件.ini
wps 登录
向日葵 登录、记录验证码、修改开机启动、截图等路径
todesk 登录、记录验证码、修改开机启动、截图等路径
手心输入法 C:\Users\O\AppData\LocalLow\PalmInput配置替换，重启电脑
迅雷
quicker 登录即可
随手写 登录即可，网页版可用
potplayer 运行注册表文件即可
Git bash 配置放到c>users>用户账号 目录下
intellij idea 需激活、jdk,git环境配置，连接MySQL、redis，tomcat解压版配置
花生壳
another redis desktop
navicat premium 断网后启动注册机
soupui
ps

vpn

-=apipost
-=词典
-=百度网盘
visio
Android studio
spss
cajview
```

软件开发基础环境配置说明

```
jdk8 未在系统中配置path, 有从win虚拟机复制下来的jdk8，仅从idea项目中引用，jdk安装包无法安装，原因未知，可能是注册表有问题
idea 和 git 软件已安装
mysql、运行jdk、redis现在运行在win虚拟机内，通过nat转发访问
```
