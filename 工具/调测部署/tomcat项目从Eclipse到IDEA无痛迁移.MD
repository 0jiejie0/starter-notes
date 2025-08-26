# tomcat项目从Eclipse到IDEA无痛迁移

## 背景

## 解决方案

* 导入项目

1. 项目源码放到本地，记住路径，不要出现中文字符
2. 打开IDEA，`File`>`New`>`Project from Existing Sources...`
3. 选中项目源码的目录，点击`OK`
4. 在Import Project窗口中选择`import project from external model`>`Eclipse`
5. 点击next，检查项目和配置文件的位置，一般默认即可，再点击next
6. 可以看到IDEA自动扫描到了若干模块，将不必要的模块去勾选
7. 点击next，代码风格无需关注，再点击next
8. 检查jdk有无问题（先前配好以后一般没什么问题）
9. 点击finish
10. 可能弹窗提示sdk错误，忽略即可，稍后配置

* 项目初步配置

1. 点击IDEA `File`>`Project Structure` 打开项目配置
2. 在 Project Settings 栏`Project`菜单，在 Project SDK 一栏中选择合适JDK，点击`Apply`
3. 点击`Modules`菜单，点击模块列表第一个最顶级项目
4. 在`Sources`选项卡中的`Language level`选择`Project default`
5. 在`Dependencies`选项卡中的`Module SDK`选择`Project SDK`，点击`Apply`
6. 点击`Facets`菜单，点击列表顶部加号，选择`Web`
7. 点击 Deployment Descriptors 中的第一项，按下键盘上的回车
8. 点击路径末尾的`...`按钮，找到项目中的`WEB-INF\web.xml`，点击`OK`，再点击OK
9. 双击 Web Resource Directories 中的第一项
10. 点击第一行输入框末尾的文件夹图标，选择项目中的WebContent目录，点击`OK`
11. 点击`OK`，点击`Apply`
12. 点击`Artifacts`菜单，点击`+`，选择`Web Application: Exploded`>`From Modules`
13. 选择当前模块，点击`OK`
14. 点击`OK`，关闭项目配置

* 项目依赖导入

1. 在IDEA左侧project视图中找到`WEB-INF/lib`目录，右击
2. 点击`Add as Library...`（如果这项无法点击显示为灰色，等待项目加载完成后再次操作）
3. 点击`OK`
4. 点击IDEA `File`>`Project Structure` 打开项目配置
5. 点击`Libraries`菜单，点击顶部`+`，选择`Java`
6. 找到项目所需Tomcat所在位置的 lib 目录，点击`OK`
7. 选择项目模块，点击`OK`
8. 点击`Apply`
9. 点击`Artifacts`菜单，可以看到底部出现库依赖提示，点击`Fix...`按钮
10. 选择`Add all missing dependencies of *** to the artifact`
11. 点击`OK`

* 项目运行配置

1. 点击IDEA右上方`Add Configuration...`，点击左上角`+`
2. 选择`Tomcat Server`>`Local`
3. 点击`Deployment`选项卡，点击右侧`+`，选择`Artifact...`
4. 修改底部`Application context`（项目在浏览器的根路径），点击`Apply`
5. 点击`Server`选项卡，在Open browser一栏中
6. 在`After launch`前后选择 项目启动后要不要打开网页 以及启动哪个浏览器
7. 在`URL`中编辑启动后打开网页的URL路径（从项目根路径的后面编辑）
8. 在`On 'Update' action`选择，项目运行时 按下`Ctrl`+`F10`是否弹框
   以及默认执行的动作（更新页面静态html、js、css资源，更新class和静态资源，重新打包部署项目，重启服务器）
9. 在`On frame deactivation`选择IDEA窗口失焦时执行的动作
10. 在下方Tomcat Server Settings一栏中，在`HTTP port`编辑项目启动端口，各项目必须区分开
11. 在`JMX port`中选择Java服务器的管理端口，如果多个项目同时启动或关闭，这个端口也要区分开
12. 点击`OK`
13. 点击右上方的绿色三角符号，启动项目

* 其他异常情况
    * eclipse中项目可以正常运行，但是IDEA编译时提示缺少依赖
        1. 根据包名找到相关的jar包
        2. 确认导入的jar包中确实不包含这个依赖
        3. 找到项目源码目录
        4. 找到`.metedata`目录（不一定在哪里，可能在项目中，也可能在C盘的用户目录）
        5. 搜索可能包含包名（或相关）关键词的jar文件，复制粘贴到项目的`WEB-INF\lib`目录

