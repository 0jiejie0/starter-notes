# Java 如何定义一个可重复在同一方法上使用的注解？

    java8、jdk8、注解、annotation、@interface、@Repeatable

## 背景

想实现一个轻量级测试框架（现有的都太重了），测试我写的算法代码，个人认为注解的形式最合适，无侵入性，不影响复制粘贴，代码量也少。

[轻量测试框架实现与使用的总篇可见此文](../框架开发/Java轻量级测试框架的实现与使用.MD)。

预期使用形式：

```java
public class T一道题目Solution {
    // 这里是三个测试用例
    @AssertExample(params = {"10101", "1"}, expectResult = "12")
    @AssertExample(params = {"1010101", "2"}, expectResult = "25")
    @AssertExample(params = {"11111", "1"}, expectResult = "15")
    public int solutionMethod(String s, int k) {  // 要提交的方法
        // ...
    }
}
```

网上其实有成型方案，但是我没抄明白，看花眼了，特此记录。

## 问题分析

首先，这注解的使用范围，应该限定在方法上，所以应该注明`@Target(ElementType.METHOD)`。

其次，在代码测试的整个运行过程中，是需要注解的，所以其保留期应该标记为`@Retention(RetentionPolicy.RUNTIME)`。

如果以后有缘整个文档的话，可以写上`@Documented`，以后在温习代码时能够看到我们的测试用例。

**重点**

如果要在方法上同时写多个用例，那就要用到jdk8的一个注解特性（实质是个语法糖）：`@Repeatable(AnAnnotation.class)`

但是，这个注解里的参数应该填什么呢？

网上的文章，写的是两个注解，一个是实际使用的注解（假设叫AssertExample），
另一个注解（假设叫AssertExamples，差一个s），仅包含一个AssertExample[]类型的value值，这里重复注解就标在了AssertExample上。

但是网上的 repeatable 参数怎么写的呢，写的是`AssertExamples.class`，但是却让我看成了“AssertExample.class”，调了半天也没整明白。

又看了好几篇文章，彼此相似，并无大差，有的提到，`AssertExamples.class`是一个容器，
然后联想到，repeatable注解需要的是包含AssertExample[]的value值的注解类，那我试一下填`AssertExamples.class`行不行呢？果然可以。

**`@Repeatable(...)`注解，实际接收的是盛放重复注解的容器类，不是要重复使用的注解。**

## 解决方案

**注意：** 代码编辑工具IDE可能会标红，我的idea就标红了，但是实际是可以正常运行无报错无警告的，可能是 idea 的bug

要重复使用的注解定义：

```Java
package main.customUtil;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)  // 注解保留到运行时
@Documented
@Repeatable(AssertExamples.class)  // 注明要使用哪个容器类
public @interface AssertExample {
    String[] params() default {};

    String expectResult() default "";
}
```

其注解容器定义：

```Java
package main.customUtil;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface AssertExamples {
    AssertExample[] value();
}
```
