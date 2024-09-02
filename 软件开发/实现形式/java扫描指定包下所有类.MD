# Java如何扫描指定包下所有类？

    Java8、jdk8、idea、反射、class

## 背景

每次写算法题时，总觉得测试代码写起来又没营养又很麻烦，即便是借助junit测试框架也很麻烦，太重了。

正好在学习spring过程中接触到注解，研究其原理时了解到反射，借由注解和反射，应该可以自定义一个轻量级的测试框架。

预期分为两篇，一篇介绍 类文件的扫描和反射，一篇介绍 按照注解执行类中的方法

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

代码示例：

```java

// Main.java:

private static List<File> classFileList = new ArrayList<>();

private static String rootPackageName = "main";
// 控制是否扫描主类包下和主类同级的类文件
private static boolean scanMainPackageClass = false;

 /**
 * 将文件对象转换为 Class 对象
 */
 public static Class convertClass(File file) throws Exception {
     String path = file.getPath();
     String className = path.substring(path.indexOf(rootPackageName))
             .replaceAll("\\\\", ".")
             .replaceAll("\\.class", "");
     return Class.forName(className);
 }

 /**
  * 默认从主类所在包开始扫描class文件
  */
 public static void loadClassFiles() throws Exception {
     loadClassFiles(Main.class);
 }

 /**
  * 从指定类所在包开始，扫描class文件
  */
 public static void loadClassFiles(Class cls) throws Exception {
     Enumeration<URL> resources = Thread.currentThread().getContextClassLoader().getResources(cls.getPackage().getName());
     if (!resources.hasMoreElements()) {
         System.out.println("找不到包，请检查后重试");
         return;
     }
     loadClassFiles(resources.nextElement().getFile());
 }

 /**
  * 从指定资源路径扫描class文件
  * 控制是否扫描 参数路径下第一级类，不负责添加类文件到列表
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
  */
 public static void loadClassFiles(File file) {
     if (file.isFile()) {
         if (file.getName().endsWith(".class")) {
             classFileList.add(file);
         }
         return;
     }
     for (File listFile : Objects.requireNonNull(file.listFiles())) {
         loadClassFiles(listFile);
     }
 }
```
