## 问题描述
spring、执行任务包jdk8、定时框架包jdk7
windows server 2008
多jar包、不同编译版本、多服务远程调用
定时项目部署时端口冲突，需要在代码中修改端口，接手项目时别人告诉我，所有项目都是jdk8，然后把项目jdk配成8打包。
但是在部署到服务器上时，在一个很正常的类上报错了，出现了类似这个的错误：
```log
java.lang.UnsupportedClassVersionError: test_hello_world :
Unsupported major.minor version 51.0
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClassCond(Unknown Source)
```
## 原因分析
网络搜索可知是jdk版本问题，对于开发、编译和部署的jdk版本，有这么一个基本原则：
> 高版本jre可运行低版本jdk编译的jar包（以下简称低版本jar包），但是不绝对，因为部分类或功能模块在迭代时会弃用；
反之不可，低版本jre不能运行高版本jar包，因为高版本一般包含新特性，所以虚拟机会直接终止高版本程序的运行

一定不能啰嗦，啰嗦会同时降低编写和阅读效率。

第一步搜索“怎么看jar编译的jdk”,找到[一篇文章](https://blog.csdn.net/m0_47406832/article/details/129960699)，文章说：
1. 查看META-INF/MANIFEST.MF文件的Created-By属性看jdk版本，我没这个属性，
2. 还是这个文件，看Build-Jdk-Spec属性看java**规范**版本（**不是实现**）,我也没有， 
3. 利用命令javap
```bash
java -version
javap -verbose MyClass.class | grep "major version"  # linux or Git Bash - MINGW64
javap -verbose MyClass.class | findstr "major version" # win
```
题外话：建议使用gitbash弥合win和linux的差异，统一使用linux风格

[拷贝合并目录命令](https://www.coder.work/article/2209910)：
```bash
$ cp -rf --backup --suffix='.copy' */* temp
```
利用for+javap批量获取同一目录下所有字节码文件编译版本的脚本：
```bash
#! bash
for n in $@
do
         javap -verbose $n | grep "major version"
done
```
其中，[$@表示所有参数](https://www.lsjlt.com/news/315339.html)，脚本使用方法如下：

```bash
$ chmod.exe +x aaa.bash # 或chmod 777 aaa.bash 打开运行权限
$ ll # 查看文件是否有运行权限x，上条命令无反应需用管理员身份打开gitbash重新赋权
-rwxr-xr-x 1 Administrator 197121    71 Sep  1 10:30 aaa.bash*
$ ./aaa.bash * >> aaa.txt # 查看版本结果追加到txt文件
$ rm -f ./*.class
```
上述批量合并目录后同名文件会产生.copy后缀导致这个javap报错找不到类文件，这需要我们去掉.copy后缀，脚本：
```bash
#! bash
for n in $@
do
        mv $n ${n%*.copy}
done
```
其中，n%*.copy表示保留字符串n的.copy前的字符，是一种[字符串截取方法](https://blog.csdn.net/weixin_44730298/article/details/128799791)，脚本用法：
```bash
$ chmod.exe +x aacp.bash
$ ./aacp.bash *
$ ./aaa.bash * >> aaa.txt
$ rm -f ./*.class
$ ./aacp.bash * # 二级后缀
$ ./aaa.bash * >> aaa.txt
```
查看aaa.txt，可以看出jar包中所有class文件的编译版本了

注意：
1. 这里用的是查看class文件的命令，（好像不能看jar包，时间关系不复现了），要看jar的，需要用压缩软件打开jar包，从里边拽出来一个class文件，
2. 这个命令查看的是Java编译器内部的版本号，不是规范版本也不是各发行版
   jdk内部版本号和**规范**版本之间的对应关系：
```log
Java SE 20 = 64,
Java SE 19 = 63,
Java SE 18 = 62,
Java SE 17 = 61,
Java SE 16 = 60,
Java SE 15 = 59,
Java SE 14 = 58,
Java SE 13 = 57,
Java SE 12 = 56,
Java SE 11 = 55,
Java SE 10 = 54,
Java SE 9 = 53,
Java SE 8 = 52,
Java SE 7 = 51,
Java SE 6.0 = 50,
Java SE 5.0 = 49,
JDK 1.4 = 48,
JDK 1.3 = 47,
JDK 1.2 = 46,
JDK 1.1 = 45
```
数据来源：[一篇博客](https://www.cnblogs.com/edwinchen/p/3979586.html)，
还有[一篇没看完的stackoverflow](https://stackoverflow.com/questions/10382929/how-to-fix-java-lang-unsupportedclassversionerror-unsupported-major-minor-versi)

## 解决方案
从[Oracle官网中找到jdk7，登录下载安装](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html)，不用改开发环境变量。
回到idea的项目中，ctrl+alt+shift+s打开项目环境配置，
1. 点击platforms settings>sdks,添加刚刚安装的jdk，
2. 点击project更换项目sdk，
3. 检查项目语法级别，要和jdk同步，
4. 点击modules》右方dependencies选项卡，调整模块sdk
   
项目打包完成后，部署到服务器上还是有个类报错，从原包中找出对应的class，替换到新包中，不再报错。我下载的是7u51，猜测版本还是没有完全对应。

[关于涉及jdk等版本的其他问题，可看这篇博客](https://blog.csdn.net/qq_41623592/article/details/132036580)

本文从[CSDN](https://blog.csdn.net/qq_41623592/article/details/132362205)迁移。
