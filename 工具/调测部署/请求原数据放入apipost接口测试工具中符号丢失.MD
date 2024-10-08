# 将成功请求的数据 放入apipost接口测试工具，发送给后端后，部分符号丢失

    apipost、接口测试、符号、丢失、错乱、变成空格

## 背景

做CA对接，保存CA系统的校验数据，需要模仿前端请求调起接口，以便测试功能完整性。

## 问题描述

后端处理请求时向控制台输出请求内数据，发现该数据可以通过CA系统校验。遂将该数据原封不动，放入apipost中，发现校验失败。

## 原因分析

问询CA人员得知，这种现象不会出现，可以看一下字符集是否一致，或者看一下传值是否正确。

这么说的话，字符集或者编码是有可能出问题的，但是里面都是字母数字，没发现什么乱码，只能一点点看了。

将成功请求和失败请求的数据，都输出到IDEA控制台，粘贴到一个文本文件中手动对比，发现两次数据不一致，
成功请求中的加号“+”在失败的请求中都是空格。

搜到[一篇博客](https://www.cnblogs.com/jason-liu-blogs/archive/2012/09/12/2682324.html),感觉可以给符号加编码。

## 解决方案

将 + 替换为 %2B；
将 & 替换为 %26；
……
