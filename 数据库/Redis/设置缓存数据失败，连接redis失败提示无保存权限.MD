# 程序设置Redis数据出错，UI工具连接Redis失败，提示：not able to persist on disk. Commands that may modify the data set are disabled

    Redis-x64-2.8.2402、springboot、windows、Redis desktop manager 0.8.0.3844

## 背景

正常运行的业务系统，客户说某功能不行了，提示失败。

## 问题描述

远程到线上操作，确实复现了这个问题，梳理代码，发现错误根源是在设置Redis缓存值的时候出现的。

查看服务，发现Redis是在启动状态。

使用Redis desktop manager连接Redis，没连上，不能显示数据库，右下方窗口提示：

```log
2024-08-15 09:24:34 : Connection: AMS > [runCommand] PING -> response received : 
    -MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. 
    Commands that may modify the data set are disabled. Please check Redis logs for details about the error.
```

## 原因分析

这是说，Redis保存数据出了问题？搜索错误信息，借助[这篇文章](https://cloud.tencent.com/developer/ask/sof/101981812/answer/107917540)
的提示，查看Redis日志文件的写入权限，发现确实有用户没有写权限，我尝试将权限打开，只读关闭，还是不行。

错误信息让我看日志，那我就看日志把，发现日志太大打不开，那我就把日志文件重命名，新建一份日志吧。

这个时候发现，有程序占用无法重命名，关闭Redis服务依然如此，使用任务管理器查看，发现竟然同时运行了好几个Redis-server程序。

## 解决方案

1. 检查log文件权限，查看是否有写权限
2. 在任务管理器中，将异常的进程全部停止
3. 使用连接工具连接成功，业务系统恢复正常
