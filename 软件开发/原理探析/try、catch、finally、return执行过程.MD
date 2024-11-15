# try、catch、finally、return执行过程

    try、finally、return、jdk1.8

本文提到的点均经测试，本文最后贴一下测试代码，如需再次验证，自行注释个别行即可。

## 基本认识

1. 在一个方法中，发生异常后，程序停止，剩余逻辑不会执行；
2. 使用try后，下方可用一个或若干catch捕获各类型的异常，若未捕获，程序仍会在异常处停止；
3. 使用try-catch捕获异常后，try内异常后逻辑不会执行，try-catch-finally后的方法逻辑会继续执行；
4. 一般情况下，不管是否发生任何异常，finally都一定会执行（在不一般的非异常情况下有个例外，下面会写）；
5. 一般情况下的执行流程，try前逻辑》try逻辑》（若发生异常，其适配的catch逻辑》）finally逻辑。

## 特别说明

1. catch捕获异常类型从上到下依次匹配，匹配到即执行该catch，后续异常类型不会继续匹配，所以需要将特别处理的子异常类型写在前面，最后可以用`Exception`兜底；
2. 不论何处使用`System.exit(0);`都会直接退出虚拟机，因而此后所有逻辑都不会再执行，包括 try、finally、try-catch-finally后逻辑、调用处等；
3. 若try和catch中执行了return语句，finally仍然会执行；
4. try或catch中有return时，finally在return语句计算之后，返回前执行，这里计算是已经计算出最终的返回结果，不管finally如何修改其变量，都不会改变之前计算的结果；
5. finally中若有return，那么程序将在此返回，且会用finally中的return表达式结果，覆盖先前try-catch中的结果值;
6. 如果在try-catch中均有return或在finally中有必然执行的return，那么该方法内try-catch-finally之后就不能再出现任何语句，否则编译不通过。

**重申第2点，不管在何时，只要出现该语句，程序都会直接停止。**

对于第四条，对于基本类型或个别返回值，finally不会对其产生影响，但例如`List`、`Map`等，是可以在finally中修改其属性或元素内容的（对象自身仍然没有改变）。
[这篇博文](https://blog.csdn.net/weixin_45759791/article/details/107687466)提到，date类型也不会被finally影响，由于其未提及是哪个包，故未验证。

最后贴一下测试代码，开头的注解参考[Java轻量级测试框架](../框架开发/Java轻量级测试框架的实现与使用.MD#使用说明)，其参数无实义，重点在代码。

```java
class Solution {
    @AssertExample(params = "1", expectResult = "0")
    public int hammingWeight(int n) {
        int ans = 0;
        System.out.println("开始喽开始喽,主调方法开始喽");
        System.out.println(test("诶嘿嘿"));
        System.out.println("方法还执行的吗");
        return ans;
    }

    public String test(String p) {
        System.out.println("这参数啥档次？" + p);
        String ans = "这可是test返回值哦~";
        try {
            System.out.println("try try try!");
//            System.exit(0);
//            if (!"".equals(p)) {
            if ("".equals(p)) {
                throw new RuntimeException("干啥呢你?");
            }
            System.out.println("哎？好像没发生什么异常啊");
            ans = "来看看，try返回了个啥东西？";
            return ans += "try整点东西到返回结果来";
        } catch (Exception e) {
            System.out.println("异常拿下啦");
            ans = "catch，也能返回？";
            return ans += "catch来给返回结果整点活";
        } finally {
            ans += "最终执行一下final";
            System.out.println(ans);
            return ans += "final 也给返回结果整活了";
        }
//        return "哦耶，被调方法总算执行完了"; // try、catch都返回了这里就不能有返回了
    }
}
```
