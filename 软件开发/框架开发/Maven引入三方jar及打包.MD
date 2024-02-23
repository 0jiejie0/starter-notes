# Maven打包运行，引入三方jar及打包，不导入本地库的方法

    maven、打包、springboot、jar、本地、引入

## 背景

业务系统要对接某硬件，需要用到其三方jar，maven官方仓库没有这个，我也没有maven，又不想mvn install污染本机环境（驻场开发用的自己电脑）。

## 原因分析

引入本地jar这个问题，借助ai助手似乎这件事不成问题，本地引入使用了一段时间后感觉没有什么异常；
但是在部署的时候发现，这个东西好像没有打进项目jar包里，同事给了我[这个网页](https://blog.csdn.net/u010321872/article/details/127366733)，
说scope为system的maven打包，默认是不打进jar包进去的，感觉写得还可以，下文两方法均摘录其中。

同时，我也发现了脱离这个事件之外隐藏的问题，我同事就能搜索出来这个问题的很多解法，而我却搜不出来，这是为什么呢？

我发现，我同事搜索的是某一现象，maven打不进本地jar（有没有本地两个字我忘了），
而我搜的是一个笼统的问题，maven引入本地jar打包，得到的，自然也是笼统泛泛的答案，几乎都是引入相关的。

看来，提问、搜索的方式，也是一门学问啊。

## 解决方案

1. 本地jar包引入：

```xml

<dependency>
    <groupId>org.crazy</groupId>
    <artifactId>crazy-core</artifactId>
    <version>${crazy.core.version}</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/src/main/resources/libs/crazy-core-1.0.0-SNAPSHOT.jar</systemPath>
</dependency>
```

2. 引用jar打入jar包配置：

```xml

<plugins>
    <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
            <includeSystemScope>true</includeSystemScope>
        </configuration>
    </plugin>
</plugins>
```