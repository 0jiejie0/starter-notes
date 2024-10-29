# 若依框架定时任务无法执行，NoSuchMethodException。。。

    若依、springboot、ruoyi-vue

## 背景

定时任务开发，写的小demo无法使用，报错。发现若依的定时任务不太好使，参数写着有点别扭。

看[官方文档](http://doc.ruoyi.vip/ruoyi/document/htsc.html#%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)的说法，似乎还不支持数组

## 问题描述

1. 定时任务方法代码写法：`public void demo(int n, String s) {...}`
2. 后台管理界面，配置任务的调用方法写法：`task.demo(2, 'hello')`
3. 点击管理界面执行一次，发现控制台报错：

```log
2024-10-29 09:59:20.017 DEBUG 22772 --- [eduler_Worker-7] c.r.q.m.SysJobLogMapper.insertJobLog     : <==    Updates: 1
2024-10-29 09:59:30.000 ERROR 22772 --- [eduler_Worker-5] com.ruoyi.quartz.util.AbstractQuartzJob  : 任务执行异常  - ：

java.lang.NoSuchMethodException: com.ruoyi.quartz.task.Task.demo(java.lang.Integer, java.lang.String)
	at java.lang.Class.getMethod(Class.java:1786)
	at com.ruoyi.quartz.util.JobInvokeUtil.invokeMethod(JobInvokeUtil.java:55)
	at com.ruoyi.quartz.util.JobInvokeUtil.invokeMethod(JobInvokeUtil.java:33)
	at com.ruoyi.quartz.util.QuartzJobExecution.doExecute(QuartzJobExecution.java:17)
	at com.ruoyi.quartz.util.AbstractQuartzJob.execute(AbstractQuartzJob.java:53)
	at org.quartz.core.JobRunShell.run(JobRunShell.java:202)
	at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:573)
```

## 原因分析

其实可以看到控制台显示框架需要的类型为`Integer`的参数，而我的方法定义为了基本类型`int`，不过当时已经叫来同事看，
好在找到了[若依官方demo的代码](https://gitee.com/y_project/RuoYi-Vue/blob/master/ruoyi-quartz/src/main/java/com/ruoyi/quartz/task/RyTask.java)

看来确实是应该用包装类型，可是这个若依为什么不做兼容呢？

## 解决方案

将定时任务方法代码写法：`public void demo(int n, String s) {...}`

改为：`public void demo(Integer n, String s) {...}`
