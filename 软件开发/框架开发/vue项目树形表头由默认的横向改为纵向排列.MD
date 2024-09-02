# vue项目中，属性表头默认为横向排列，却没有纵向排列的组件或相关属性，如何将其纵向排列？

    vue2、树形表头、纵向、横向、排列、rotate

## 背景

客户要求在系统中加一个页面，是一个表格，表头是纵向排列的。

曾经的解决方案是一个个格子得写，或者自定义组件，也是一个个格生成，是固定死的，不能根据数据自动缩长。

## 问题描述

vue组件的表头，都是横向排列的，无法纵排，网络搜索不到相关解决方案，背景中提到的自定义组件一个个格子生成的方法应用了很久，都没有更好的解决方案。

这恰恰说明同事及项目前辈们的技术能力普遍不强。

今日在处理PDF生成业务时，意外发现了旋转表格的方法，特此记录（以前曾经试想过旋转，但未找到有效旋转方案，连这也是我自己尝试得到的）。

## 解决方案

1. 使用`document.getElementById("aPageId").style.rotate = '90deg'` 可以顺时针旋转90度（仅旋转表格）
2. 使用`document.getElementById("aPageId").style.font.rotate = '90deg'` 可以将文字顺时针旋转90度

有些时候这样直接将一整个元素旋转，并不能满足实际需要，所以在实际使用中可以结合for循环，进行更为精细的控制，实现在某一级结构下所有元素的旋转：

```javascript
document.getElementById("aTableId").childNodes.forEach(item => {
    item.style.rotate = '-90deg'
    if (item.style.font) {
        item.style.font.rotate = '-90deg'
    }
})
```
