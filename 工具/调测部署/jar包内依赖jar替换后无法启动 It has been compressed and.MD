# jar包内替换依赖jar后无法启动，错误日志：It has been compressed and nested jar files must be stored without compression.

    ruoyi、springboot、java、jar、libs、压缩

## 背景

某服务jar包足足90MB有余，远程传输太慢，目前在改动的是其中的某子jar（项目内部依赖，另一个jar）。
之前有单传class文件的经验，这次想单独传输子jar，节约传输等待时间，不成想出问题了。

## 问题描述

压缩软件打开运行jar，找到libs目录，复制依赖jar粘贴到压缩包内，关闭压缩软件，发现启动失败，查看日志发现：

```log
Exception in thread "main" java.lang.IllegalStateException: Failed to get nested archive for entry BOOT-INF/lib/ruoyi-common-3.8.5.jar
	at org.springframework.boot.loader.archive.JarFileArchive.getNestedArchive(JarFileArchive.java:108)
	at org.springframework.boot.loader.archive.JarFileArchive.getNestedArchives(JarFileArchive.java:86)
	at org.springframework.boot.loader.ExecutableArchiveLauncher.getClassPathArchives(ExecutableArchiveLauncher.java:70)
	at org.springframework.boot.loader.Launcher.launch(Launcher.java:49)
	at org.springframework.boot.loader.JarLauncher.main(JarLauncher.java:51)
Caused by: java.io.IOException: Unable to open nested jar file 'BOOT-INF/lib/ruoyi-common-3.8.5.jar'
	at org.springframework.boot.loader.jar.JarFile.getNestedJarFile(JarFile.java:256)
	at org.springframework.boot.loader.jar.JarFile.getNestedJarFile(JarFile.java:241)
	at org.springframework.boot.loader.archive.JarFileArchive.getNestedArchive(JarFileArchive.java:103)
	... 4 more
Caused by: java.lang.IllegalStateException: Unable to open nested entry 'BOOT-INF/lib/ruoyi-common-3.8.5.jar'. It has been compressed and nested jar files must be stored without compression. Please check the mechanism used to create your executable jar file
	at org.springframework.boot.loader.jar.JarFile.createJarFileFromFileEntry(JarFile.java:284)
	at org.springframework.boot.loader.jar.JarFile.createJarFileFromEntry(JarFile.java:264)
	at org.springframework.boot.loader.jar.JarFile.getNestedJarFile(JarFile.java:252)
	... 6 more
```

## 原因分析

一看就是压缩软件导致的问题，本来以为就是不能改动这个jar包的，直到搜到[一篇博客](https://blog.csdn.net/weixin_50913916/article/details/120634229)。

其实是这个压缩软件不行，放入jar文件的时候会对该文件进行二次解压，只要不让它解压就好了。
但是先前服务器用的好压软件设置以后还是不能正常使用，换用winrar就可以了。

## 解决方案

1. 从远程机winrar安装目录压缩一份winrar，放到服务器中
2. 用其他工具解压压缩包，记住解压目录
3. 右击jar包》打开方式》从磁盘查找winrar
4. 关闭winrar窗口，双击jar重新打开
5. 复制依赖jar，在rar中打开目标路径
6. 粘贴，在弹出的确认框中，选择文件选项卡
7. 在第三行“不压缩直接存储的文件”中的最后输入一个空格，再输入`*.jar`
8. 点击确定，在目标目录中找到对应依赖jar，确认`大小`和`压缩后大小`两项一致
9. 运行jar包