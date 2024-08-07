# 在vue项目中，将某元素铺满整个屏幕

    vue、js、screenfull、插件、全屏、大屏、若依

## 背景

甲方需要一个大屏效果的页面，但是框架自带的菜单、标题、页脚栏似乎不太好隐去，搜到这么[一篇博客](https://blog.csdn.net/caseywei/article/details/109065351)
，实测可行，特此记录。

## 解决方案

1. 安装依赖 `npm install --save screenfull`
2. 要全屏的位置引用依赖 `import screenfull from "screenfull"`
3. 将要全屏显示的元素设定好元素id
4. 事件触发或者页面加载时，执行：

```javascript
const element = document.getElementById('yourNeededFullScreenElementId');
if (screenfull.isEnabled) {
    screenfull.request(element); // 元素全屏
}
```
