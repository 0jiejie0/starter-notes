# Java 如何获取指定包下所有类的 Class 对象？

    Java8、jdk8、idea、反射、class、注解、Annotation

## 背景

每次写算法题时，总觉得测试代码写起来又没营养又很麻烦，即便是借助junit测试框架也很麻烦，太重了。

正好在学习spring过程中接触到注解，研究其原理时了解到反射，借由注解和反射，应该可以自定义一个轻量级的测试框架。

大概分为两篇，本篇介绍 [类文件的扫描和反射](./java扫描指定包下所有类.MD)，下篇介绍 [按照注解执行类中的方法](./java实例化class对象，利用注解入参并执行其方法.MD)

[轻量测试框架实现与使用的总篇可见此文](../框架开发/Java轻量级测试框架的实现与使用.MD)。

本文方法部分借鉴于[这篇博客](https://zhuanlan.zhihu.com/p/355050724)，有所改动。

## 解决方案

1. 获取包名 `Main.class.getPackage().getName()` （我的启动类是“Main”）
2. 利用包名获取资源路径列表 `Thread.currentThread().getContextClassLoader().getResources(pkgName)`
   返回值类型 `Enumeration<URL>`
3. 遍历资源路径（上一步返回值） `resources.nextElement().getFile()`
4. 利用资源路径新建文件对象 `new File(pkgResourcePathName)`
5. 利用 `file.isFile()` 检查文件对象是文件还是目录，文件直接添加进文件列表（注意查看文件名后缀，筛选“.class”文件）
6. 若文件对象是目录，使用 `file.listFiles()` 依次列举其内对象，回到第5步，递归判断（目录结构层次不深的可以采用此法，目录深的可以利用队列或栈手动操作）
7. 文件类型的对象，可以使用`Class.forName(fileName)`获取其对应 Class 对象，文件名需要处理成类似 `com.example.main.Main` 形式
8. （筛选，利用`cls.getDeclaredMethods()`和`method.isAnnotationPresent(annotation)`检验class对象是否符合注解条件）

代码示例：

```java
// Loader.java

import java.io.File;
import java.lang.annotation.Annotation;
import java.lang.reflect.Method;
import java.net.URL;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.Objects;

public class Loader {
   // todo: 为方便层层调用时少些点参数和返回值，将类文件列表定义为全局变量，若多次加载文件，需要重新调整其结构
   private static List<File> classFileList = new ArrayList<>();

   // 项目根目录包名
   private static String rootPackageName = "main";

   // 扫描标记，控制是否扫描主类包下和主类同级的类文件的
   private static boolean scanMainPackageClass = false;

   /**
    * 将文件列表中的文件对象转化为 class 对象，筛选后添加到 class 列表
    *
    * @param annotations 要满足的注解条件（标明此注解的才会添加到 class 对象列表）
    * @return
    * @throws Exception 底层传递，获取 Class 对象异常
    */
   public static List<Class<?>> getClasses(Class<? extends Annotation>[] annotations) throws Exception {
      List<Class<?>> classList = new ArrayList<>();
      for (File file : classFileList) {
         Class<?> convertedClass = getClass(file.getPath());
         for (Method declaredMethod : convertedClass.getDeclaredMethods()) {
            boolean needAdd = false;  // 辅助跳出两层循环，防止同一个类两次加入列表
            for (Class<? extends Annotation> annotation : annotations) { // 多注解筛选，只要方法标明列表其一注解，就需要其类添加到类列表（筛选需要运行的类）
               if (declaredMethod.isAnnotationPresent(annotation)) {
                  needAdd = true;
                  break;
               }
            }
            if (needAdd) {
               classList.add(convertedClass);
               break;
//                } else {
//                    System.out.println("pass " + convertedClass.getName() + declaredMethod.getName());  // 不满足注解条件的类
            }
         }
      }
      return classList;
   }

   /**
    * 获取指定文件路径对应的 Class 对象
    *
    * @param filePath 文件路径
    * @return Class 对象
    * @throws Exception 获取 Class 对象异常
    */
   public static Class<?> getClass(String filePath) throws Exception {
      String className = filePath.substring(filePath.indexOf(rootPackageName))
              .replaceAll("\\\\", ".")
              .replaceAll("\\.class", "");
      return Class.forName(className);
   }

   /**
    * 默认从主类所在包开始扫描class文件
    *
    * @throws Exception 获取资源路径时IO错误，或找不到对应包资源
    */
   public static void loadClassFiles() throws Exception {
      loadClassFiles(Main.class);
   }

   /**
    * 从指定类所在包开始，扫描class文件
    * <p>
    * 获取包资源路径
    *
    * @param cls 要扫描的包中类的Class对象
    * @throws Exception 获取资源路径时IO错误，或找不到对应包资源
    */
   public static void loadClassFiles(Class<?> cls) throws Exception {
      Enumeration<URL> resources = Thread.currentThread().getContextClassLoader().getResources(cls.getPackage().getName());
      if (!resources.hasMoreElements()) {
         throw new Exception("找不到包，请检查后重试");
      }
      loadClassFiles(resources.nextElement().getFile());  // 正常来说对应包只有一个，所有只需要获取一次资源路径就够了
   }

   /**
    * 从指定资源路径扫描class文件，借扫描标记筛选 参数路径下第一级类文件，调用实现添加类文件到列表
    *
    * @param pkgResourcePathName 扫描文件的起始包资源路径
    */
   public static void loadClassFiles(String pkgResourcePathName) {
      File pkg = new File(pkgResourcePathName);
      // 利用静态标记控制扫描参数路径包下第一级类文件
      for (File listFile : Objects.requireNonNull(pkg.listFiles(pathname -> scanMainPackageClass || !pathname.isFile()))) {
         loadClassFiles(listFile);
      }
   }

   /**
    * 递归扫描给定目录及目录下所有类文件，添加到静态列表
    *
    * @param file 要扫描的对象，实质为目录或文件
    */
   public static void loadClassFiles(File file) {
      if (file.isFile()) {  // 文件直接添加到列表
         if (file.getName().endsWith(".class")) {
            classFileList.add(file);
         }
         return;
      }
      for (File listFile : Objects.requireNonNull(file.listFiles())) {
         loadClassFiles(listFile);
      }
   }
}
```
