# class not defined error 和 class not found exception

## 背景

如题，在古老项目的部署与调配过程中，不止一次地遇到了这两个问题。

## 问题描述

在打包或者转移jar包（服务器间）并启动的过程中，会反复遇到这两个问题（具体错误信息下次遇到了再来粘贴）。

## 原因分析

这个exception其实还好处理，替换一下jar包里的class文件就可以；
但是这个error感觉处理起来就要棘手一些（而且目前好像没有解决这个error的成功实践），
看了[这篇博文](https://blog.csdn.net/anzhuangguai/article/details/50854008)后，感觉好像确实有哪里不对劲，
但是在处理的过程中，服务又正常运行了，这个错误也诡异地消失了。

## 解决方案

针对 class not found exception ，其中的错误信息中一定会包含某个类，从原jar包中复制对应类的class文件，复制到报错jar包中替换。
对于 class not defined error ，下次遇到再来记录。