# JavaScript使用console.log()输出日志，发现控制台中的结果不是调用console.log()时对象的内容

## 问题描述

前端VUE2项目中，我想查看某对象在处理过程中的属性变化，写了类似这样的一段代码：

```javascript
console.log('params', params)
params['content'] = data
console.log('paramss', params)
```

两个console.log()语句输出的内容相同：

```log
params {content: {name: ''}, id: 1}
paramss {content: {name: ''}, id: 1}
```

看起来中间的赋值语句好像没有什么作用，然而如果注掉这行，控制台又会变成这样：

```log
params {content: {}, id: 1}
paramss {content: {}, id: 1}
```

这样看，中间的赋值语句又好像是有作用的。难道，发生灵异事件了？

## 分析

计算机里的一切都有迹可循，灵异事件当然是不可能的。

网上搜索及ai助手给我的答案都是说，console.log()调用时保存了一份快照，对于复杂对象类型，实际上就是保存对对象的引用。
所以如果后续代码修改了该对象的部分内容，就会影响最终的日志输出。

这一点我当然是猜到了（我也猜可能是异步，但应该不是，异步应该会存在 不受后续代码影响 的日志），
可是js为什么要这么做呢？如果js在执行console.log()时仅保存引用，那js日志框架是什么时候
把要输出的对象转换成一个个字符的呢？是空闲时？还是当前方法执行完成时？还是当前页面渲染完成时？还是所有动作都完成时？

## 解决方案

解决起来很简单，使用复制对象的方法`{...params}`进行输出：

```javascript
console.log('params', {...params})
params['content'] = data
```

实际上这个方法不光是在日志中，在别的场景也会用到。

或者也可以用前端调试（待整理）的方法来查看实时运行值。
