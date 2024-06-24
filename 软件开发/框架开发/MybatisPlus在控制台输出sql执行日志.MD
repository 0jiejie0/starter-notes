# SpringBoot-MybatisPlus项目中，在控制台查看sql执行日志的方法

    springboot、maven、mybatisplus、sql、日志、控制台、console、log

## 背景

在[baomidou.com](https://baomidou.com/getting-started/)
学习mybatisPlus入门的过程中，接触到[表名和关键词冲突](./MybatisPlus关键字冲突SyntaxErrorInSQLStatementSELECT_FROM_userExpectedIdentifier.MD)，
加注解加表名引号后问题解决。

不过我还想，在控制台打印一下执行的sql，看一下加了注解的执行sql与不加注解，有什么区别。

## 分析

问题没啥好描述的，也没啥好分析的，搜就完了，然而并没有一下就找到可行方案（我不得不说，文心一言是***汇编，它不止一次骗我了），故此记下。

搜到[这篇博客](https://blog.csdn.net/TM007_/article/details/137379988)。

## 解决方案

由于我惯常用`application.yml`文件，所以实验通过的方法特此记录下来，在其中添加如下：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 这里的 StdOutImpl 一串不用从头开始写，只写 StdOutImpl 开头几个字母即可补全 
```
