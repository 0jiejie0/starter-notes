> 接手的上古项目需要重新打包部署，所以有了本文的问题，搜到了[一篇博客](https://www.xjx100.cn/news/452018.html?action=onClick)，
> 我的项目有原来的依赖要调用，故按照原来的方式打包，本人测试的部分概要如下：

1. file -> Project Structure打开项目配置
2. 点击Artifacts
3. 点击页面+号，选择jar>From modules with dependencies...
4. 1、点击Main Class选择主类
   2、选择copy to the output directory and link via manifest（把lib库里的jar文件放到输出目录下，上一个选项是打进jar包）
   3、选择MANIFEST.MF文件的保存路径（这步操作会创建文件），我的项目中本身有这个文件，所以我这里配了该文件的路径out/meta-inf。
   第二步那里勾选了以后，就会把依赖包放到统一目录中，不会再有外部依赖
5. 右击 < output root > ，点击Create Directory,创建一个libs，将下方所有的JAR放进libs目录下
6. 最下方的一个是我们要生成的jar包的名字，这个不需要移动，点击一下，选择下方classpath，把所有的jar包前加上“libs/”路径(
   正斜杠)
7. ~~原文还提到在配置页面，勾选build on make，我这同样位置有个 include in project build，
   用于指定文件或目录是否包含在项目构建中，不勾选。~~
8. 点击Build->Build Artifacts，选择build
9. 从项目里的classes目录中可以找到我们刚刚打好的jar包。

要说有的文章真是老太太裹脚布，到处复制，一点都不整理，最后都不验证，也没啥用，小心翼翼看了半天发现都是废话。

本文从[CSDN](https://blog.csdn.net/qq_41623592/article/details/132365005)迁移。
