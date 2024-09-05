# java实例化class对象，利用注解入参并执行其方法

    Java8、jdk8、idea、反射、class、注解、Annotation、invoke

## 背景

每次写算法题时，总觉得测试代码写起来又没营养又很麻烦，即便是借助junit测试框架也很麻烦，太重了。

正好在学习spring过程中接触到注解，研究其原理时了解到反射，借由注解和反射，应该可以自定义一个轻量级的测试框架。

大致分为两篇，上篇介绍 [类文件的扫描和反射](./java扫描指定包下所有类.MD)
，本篇介绍 [按照注解执行类中的方法](./java实例化class对象，利用注解入参并执行其方法.MD)

## 解决方案

1. 获取各Class（是Class对象，代表各‘类’，不是各具体对象）的各方法对象：`aClass.getDeclaredMethods()`
2. 检查各方法是否标有注解容器：`method.isAnnotationPresent(AssertExamples.class)`（AssertExamples.class是注解容器类）
3. 获取方法的注解容器中的注解：`method.getAnnotation(AssertExamples.class).value()`
4. 检查各方法是否标有注解：`method.isAnnotationPresent(AssertExample.class)`（AssertExample.class是注解类）
5. 获取方法上的注解：`method.getAnnotation(AssertExample.class)`
6. 获取各注解中的入参及返回值数据：`annotation.params()`、`annotation.expectResult()`
7. 获取各方法上的参数类型列表和返回值类型：`method.getParameterTypes()`、`method.getReturnType()`
8. [根据方法各参数类型，将注解数据转换成相应对象](./数据类型自动转换的解决方案.MD)
9. 各方法对象执行调用方法（invoke）：`method.invoke(object, paramObjectList)`，
   其中第一个参数需要其类的实例化对象：`method.getDeclaringClass().getDeclaredConstructor().newInstance()`
10. 执行前后记录时间戳：`System.currentTimeMillis()`或`System.nanoTime()`

代码示例：

```java 
public class Task {
    // 获取时间戳统一入口
    public static long getTimeStamp() {
        return System.currentTimeMillis();  // 当前时间与1970年1月1日0点之间的毫秒数
//        return System.nanoTime();  // 最准确的可用系统计时器的当前值，以纳秒为单位
    }

    private List<Class<?>> classList;

    public Task(List<Class<?>> classList) {
        this.classList = classList;
    }

    /**
     * 测试列表中的类的个方法（实际可能只有一个类及方法）
     *
     * @throws Exception 底层传递，各方法参数及结果转化成对象可能产生的异常，方法对应类实例化可能产生的异常
     */
    public void testClasses() throws Exception {
        for (Class aClass : classList) {
            System.out.print("待测试类 : \t");
            System.out.println(aClass.getName());
            for (Method method : aClass.getDeclaredMethods()) {
                runMethod(method);
            }
        }
    }

    /**
     * 解析方法注解中的参数列表及返回值数据
     * <p>
     * 为减少对具体注解类型的耦合，特将使用到注解的部分集中到本方法，本模块其他部分不再隐含或使用注解。
     * （尝试过使用泛型，但泛型信息在编译期间会消除，且仅凭泛型类型无法获取其类型及相关参数，又无法实例化泛型类型，作罢）
     *
     * @param method 要执行的方法对象
     * @throws Exception 底层传递，参数及结果转化成对象可能产生的异常，方法对应类实例化可能产生的异常
     */
    public void runMethod(Method method) throws Exception {
        System.out.print("执行方法 : \t" + method.getName());
        System.out.println(Convert.toString(method.getParameterTypes(), new String[]{"(", ")"}));
        Class<AssertExample> assertExampleClass = AssertExample.class;
        Class<AssertExamples> assertExamplesClass = AssertExamples.class;
        if (method.isAnnotationPresent(assertExampleClass)) {
            AssertExample annotation = method.getAnnotation(assertExampleClass);
            runMethod(method, annotation.params(), annotation.expectResult());
        } else if (method.isAnnotationPresent(assertExamplesClass)) {
            for (AssertExample annotation : method.getAnnotation(assertExamplesClass).value()) {
                runMethod(method, annotation.params(), annotation.expectResult());
            }
        }
    }

    /**
     * 转换入参列表和期望结果，利用方法的参数列表及返回值类型，将参数数据转化成对应对象
     *
     * @param method       方法对象
     * @param params       测试实参的字符串形式
     * @param expectResult 期望返回值的字符串形式
     * @throws Exception 参数及结果转化成对象可能产生的异常，方法对应类实例化可能产生的异常
     */
    public void runMethod(Method method, String[] params, String expectResult) throws Exception {
        Object[] paramObjects = Convert.toInstances(method.getParameterTypes(), params);
        Object expectResultObject = Convert.toInstance(method.getReturnType(), expectResult);
        runMethod(method, paramObjects, expectResultObject);
    }

    /**
     * 查看入参、期望结果，运行完成后，显示执行时间
     *
     * @param method             方法对象
     * @param paramObjects       入参对象列表
     * @param expectResultObject 期望结果对象
     * @throws Exception 调用方法 内部抛出的异常
     */
    public void runMethod(Method method, Object[] paramObjects, Object expectResultObject) throws Exception {
        System.out.print("\t测试样例 : \t");
        System.out.println(Convert.toString(paramObjects) + " => " + Convert.toString(expectResultObject));
        System.out.print("\t样例用时(ms)：");
        System.out.println(invoke(method, paramObjects, expectResultObject));
    }

    /**
     * 调用方法并判定结果，记录执行时间
     *
     * @param method             要执行的方法对象
     * @param paramObjects       参数对象列表
     * @param expectResultObject 期望返回值
     * @return
     * @throws Exception 方法对应类 实例化产生的异常
     */
    public long invoke(Method method, Object[] paramObjects, Object expectResultObject) throws Exception {
        Object instance = method.getDeclaringClass().getDeclaredConstructor().newInstance();
        long start = getTimeStamp();
        Object executeResult = method.invoke(instance, paramObjects);  // 为更准确反映真实运行时间，紧邻执行动作前后记录时间戳，不穿插其他动作
        long end = getTimeStamp();
        assert executeResult.equals(expectResultObject) : "\n" +
                "期望值：\t" + Convert.toString(expectResultObject) + "\n" +
                "实际值：\t" + Convert.toString(executeResult);
        return end - start;
    }
}
```
