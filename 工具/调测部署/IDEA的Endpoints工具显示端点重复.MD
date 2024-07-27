# IDEA的Endpoints工具显示端点重复

    IDEA、springboot

## 背景

熟悉新项目过程中，为方便浏览接口，使用该工具

## 问题描述

打开endpoints，发现其中每个端点都并排显示了两次，双击打开对应的代码位置，发现两个端点的代码定义都是一个地方。

## 原因分析

自己分析不出来了，从bing上也没搜索到想要的答案，询问chatGPT “idea endpoint 端点重复 每个端点都显示两次是怎么回事”，得：“

* 项目结构问题：检查你的项目结构和配置文件（如application.properties或application.yml）。
  有时候，如果在配置文件中重复定义了端点，比如两次配置了相同的management.endpoints.web.base-path，会导致端点重复显示。
* 依赖冲突：如果项目中引入了多个依赖或框架，它们可能会注册相同的端点，导致端点在IDEA中显示两次。这通常需要检查并解决依赖版本或排除重复注册的端点。
* IDEA本身问题：有时IDEA的缓存可能会导致显示不正确的问题。
  可以尝试清理IDEA的缓存（File -> Invalidate Caches / Restart）或者重新导入项目来解决。

”

上文对于`application.yml`文件的描述我觉得很有道理，然而，我检查了所有有关“application.yml”的文件，并没有发现关于“management”的路径定义。

不过，却发现yml配置文件中有别的几个路径，于是留意到`server.servlet.context-path: /`这项在两个配置文件中都出现了，总共出现了两次。
但其实在`application.yml`文件中只启用了其中一个，这应该是IDEA的Endpoints工具的bug。

## 解决方案

1. 将包含`server.servlet.context-path: /`的不使用的yml配置文件内容全部注释掉；
2. 关闭Endpoints工具；
3. 重新打开Endpoints工具，发现显示正常了。
