# SpringBoot-MybatisPlus项目中，在控制台查看sql执行日志的方法

    springboot、maven、mybatisplus、sql、日志、控制台、console、log

## 背景

在[baomidou.com](https://baomidou.com/getting-started/)学习mybatisPlus入门的过程中，接触到表名和关键词的冲突，加注解加表名引号后问题解决。

不过我还想，在控制台打印一下执行的sql，看一下加sql与不加，有什么区别。

## 分析

问题没啥好描述的，也没啥好分析的，搜就完了，然而并没有一次性就搜到可行答案，故此记下。

搜到[这篇博客](https://blog.csdn.net/TM007_/article/details/137379988)。

## 解决方案

由于我惯常用`application.yml`文件，所以实验通过的方法特此记录下来，在其中添加如下：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```