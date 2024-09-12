# Java轻量级测试框架的实现与使用 总篇

    java8，jdk8，测试，assert

## 背景

每次写算法题，用例不过总要到本地调试一下，总觉得测试代码写起来又没营养又很麻烦，即便是借助junit测试框架也很麻烦，太重了，写完又觉得测试代码不美观需要删掉。

正好在学习spring过程中接触到注解，研究其原理时了解到反射，借由注解和反射，应该可以自定义一个轻量级的测试框架。

耗时十多天（业余时间），总算大概完成了，可喜可贺。内容大致分为两篇，同时还有整个过程中遇到的各种问题。

首先来看一下实际使用效果：

## 运行效果

待测方法（计算逻辑和返回值简化处理）：

```java
class Solution {
    @AssertExample(params = "[2,451,1]", expectResult = "0")
    public int singleNumber(int[] nums) {
        return 0;
    }
}
```

```java
public class TQ1判断是否可以赢得数字游戏 {
    @AssertExample(params = {"[1,2,3,4,10]"}, expectResult = "false")
    @AssertExample(params = {"[1,2,3,4,5,14]"}, expectResult = "true")
    @AssertExample(params = {"[5,5,5,25]"}, expectResult = "true")
    public boolean canAliceWin(int[] nums) {
        // calculate...
        return 0;
    }
}
```

```java
public class TQ1统计满足K约束的子字符串数量I {

    @AssertExample(params = {"10101", "1"}, expectResult = "12")
    @AssertExample(params = {"1010101", "2"}, expectResult = "25")
    @AssertExample(params = {"11111", "1"}, expectResult = "15")
    public int countKConstraintSubstrings(String s, int k) {
        int ans = 0;
        // calculate...
        return ans;
    }
}
```

正常测试通过的情况：

```log
测试程序已启动，时间戳：1726036055742
待测试类 : 	main.leetcode.editor.cn.Solution
执行方法 : 	singleNumber(int[])
	测试样例 : 	[2, 451, 1] => 0
	样例用时(ms)：0
待测试类 : 	main.problemAndSolving.leetcode_20240728WeekRankList.TQ1判断是否可以赢得数字游戏
执行方法 : 	canAliceWin(int[])
	测试样例 : 	[1, 2, 3, 4, 10] => false
	样例用时(ms)：0
	测试样例 : 	[1, 2, 3, 4, 5, 14] => true
	样例用时(ms)：0
	测试样例 : 	[5, 5, 5, 25] => true
	样例用时(ms)：0
待测试类 : 	main.problemAndSolving.leetcode_20240818WeekRankList.TQ1统计满足K约束的子字符串数量I
执行方法 : 	countKConstraintSubstrings(java.lang.String, int)
	测试样例 : 	10101, 1 => 12
	样例用时(ms)：0
	测试样例 : 	1010101, 2 => 25
	样例用时(ms)：0
	测试样例 : 	11111, 1 => 15
	样例用时(ms)：0
测试程序已启动，时间戳：1726036055918，总计用时（ms）：176
```

用例不通过的情况（下半错误信息是红色的，第一行标明运行类文件位置，点击可跳转）：

```log
测试程序已启动，时间戳：1726123986010
待测试类 : 	main.leetcode.editor.cn.Solution
执行方法 : 	singleNumber(int[])
	测试样例 : 	[2, 451, 1] => 1
	样例用时(ms)：Exception in thread "main" java.lang.AssertionError: 
期望值：	1
实际值：	0
main.leetcode.editor.cn.Solution.singleNumber(Solution.java:100)
	at main.customUtil.Task.invoke(Task.java:120)
	at main.customUtil.Task.runMethod(Task.java:90)
	at main.customUtil.Task.runMethod(Task.java:75)
	at main.customUtil.Task.runMethod(Task.java:56)
	at main.customUtil.Task.testClasses(Task.java:35)
	at main.Main.main(Main.java:23)
```

## 解决方案

1. [扫描class文件，获取Class对象](../实现形式/java扫描指定包下所有类.MD#解决方案)
2. [开发注解及注解容器](../实现形式/Java%20如何定义一个可重复在同一方法上使用的注解.MD#解决方案)
3. [开发数据类型转换工具类](../实现形式/数据类型自动转换的解决方案.MD#问题分析)
4. [扫描注解入参并执行其方法，类和文件不同名的需要在包内安插一个 Sentry 类辅助实例化](../实现形式/java实例化class对象，利用注解入参并执行其方法.MD#解决方案)，
   其中断言定位错误代码的行数并不准确，因为反射方法已经执行完毕，无法取得其线程栈信息，也无法通过返回值返回行数（原本方法就有返回值，而且那样会破坏了原方法）
5. [添加IDE的Java虚拟机启动参数，开启断言](../../工具/调测部署/IDEA2023.1添加java虚拟机启动参数，打开断言.MD#解决方案)
6. 最后把启动类写上：

```Java
public class Main {
    public static void main(String[] args) throws Exception {
        long start = Task.getTimeStamp();
        System.out.println("测试程序已启动，时间戳：" + start);
        loadClassFiles();  // 加载类文件，内部文件列表为全局变量，若多次加载文件，需要重新调整其内部结构
        List<Class<?>> classList = getClasses(new Class[]{AssertExample.class, AssertExamples.class});  // 文件转化为class对象，筛选可运行类
        new Task(classList).testClasses();
        long end = Task.getTimeStamp();
        System.out.println("测试程序已启动，时间戳：" + end + "，总计用时（ms）：" + (end - start));
    }
}
```

主要重点在class扫描和方法的入参和执行上，此二者逻辑链较长，其他问题相较起来就简单零碎些，好处理。

## 使用说明

假设现有一个待测试的类文件 Solution.java 如下：

```Java
public class Solution {
    public int singleNumber(int[] nums, String whiteString) {
        return nums.length;
    }
}
```

样例注解`AssertExample`
使用格式：`@AssertExample(params = {"第一参数数据", "第二参数数据", ...}, expectResult = "返回值")`；

参数数据和返回值均为原始直观的字符串形式，一般来说，将OJ网站的用例直接粘贴即可，除数组需要前后缀`[]`以及分隔符`,`
（无空格），其他类型无需其他符号，测试框架可自动完成类型转换。
（字符串不需要额外再加引号，其他的List类型以及链表、树、Map等，遇到时再在类型转换工具类中另作扩展）

如果加注解的方法只有一个参数，那注解的params属性可以省去大括号，如[运行效果](#运行效果)中的第一个样例所示。

这个待测试的方法加注解后的代码如下：

```Java
public class Solution {  // 这个public关键字没有也可以，但是那就需要一个辅助类，辅助类见上文解决方案的第五步
    @AssertExample(params = {"[2,451,1]", "aStr"}, expectResult = "3")
    public int singleNumber(int[] nums, String whiteString) {
        return nums.length;
    }
}
```

启动之后，如果要观察不同参数下方法的执行过程，或者调试错误断言，在被测试的代码里直接打断点，然后debug模式运行即可，不再需要转移注意力，这极大方便了代码调试工作。

本来我想要不把参数和返回值合到一个参数里或者分成两个注解来添加，但是那样的话框架封装对注解类的耦合又提高了，而且在使用效率上的提升似乎并不明显，所以还是用了这个初版方案。
