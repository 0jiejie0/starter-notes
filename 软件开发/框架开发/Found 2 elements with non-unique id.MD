# [DOM] Found 2 elements with non-unique id VUE子页面调用父页面以及父页面调用子页面的方法

    vue、springboot、idea

## 背景

业务系统中接入三方电子签章，签名形式是在pc插入uk，在电脑（自行封装的）的弹窗中选择改uk对应用户名，输入密码，验证成功后即签名。
其前端API中使用getElementById获取dom元素，签名弹窗内的元素必须要设置元素id。

十分不巧的是，在某功能的父子两级页面中都要用电子签章。

父页面中包含若干子页面，使用Tabs-TabPane进行子页面切换及数据输入。

子页面中对单条信息进行签章，父页面对所有子页面进行签章。

## 问题描述

起先，在各子页面中引入了签名组件（已自行封装好），签名及查看各页面均无问题。

后来发现，父页面有个总签名还没有设置好，于是采用一样的套路，引入、调用、改dom元素。

然而父页面写完后，发现该功能页面下所有的签名弹窗都加载不出信息了（有弹窗，但无uk有效信息），输入密码也无法启动校验。

## 原因分析

排查半天没有发现任何问题和错误，直到最后终于注意到控制台中，一行行黄底黑字的警告（竟然只是警告而已）：

```log
[DOM] Found 2 elements with non-unique id #CertValidDate: (More info: https://goo.gl/9p2vKq) <input id=... value>
```

仔细查看代码，发现组件里确实用到了id，是父子页面同时引入这个组件，导致了id冲突。

那么，假设我在父页面中引入组件，如何才能在子页面中使用这个组件呢？

搜到了[这么一篇博客](https://blog.csdn.net/weixin_42401141/article/details/123985842)，其中提到（下文用词不严谨，我是后端，见谅）：

父页面定义好方法后，使用`provide`将子页面要使用的方法导出：

```vue

<script>
export default {
    name: 'App',
    provide() {
        return {
            say: this.say
        }
    },
    methods: {
        say() {
            alert("这是父页面的方法");
        }
    }
}
</script>
```

子页面使用`inject`声明父页面的方法，使用`this.`方式调用：

```vue

<script>
export default {
    inject: ['say'],
    methods: {
        recv() {
            this.say();
        }
    }
}
</script>
```

到此，仅仅是解决了子页面调起父页面组件的方法，但是父页面的组件的返回值怎么处理呢？

自然想到，要使用方法的参数，将子页面的数据结构或方法传入父页面去。

试过
传输子页面变量、
传输子页面方法（子页面中返回值处理方法已经写好，然而传进来的只是方法，其数据结构完全没有进父页面，）、
传输子页面数据结构（写了一半感觉太麻烦了，作罢），
最终，将整个子页面都传进参数了（this指针）。

正好想起先前看过一中指针叫做that，子页面this传进父页面方法后，父页面使用that接收子页面的this指针，在接收到ca组件返回值后，使用that指针调起子页面对应方法。

为了不影响父页面的正常调用，使用字段标记来区分父子页面，以便调起父子页面不同处理逻辑。

## 解决方案

上文博客的方法仅用一种，特此记录

父页面定义好方法后，使用`provide`将子页面要使用的方法导出：

```vue

<script>
export default {
    name: 'App',
    provide() {
        return {
            say2: this.say
        }
    },
    methods: {
        say(tag, code) { // 调起组件
            this.tag = tag
            if ('father' == this.tag) {
                this.code = code
            } else if ('child' == this.tag) {
                this.that = code
            }
            alert('open CA')
        }

        hear(data) { // 接收数据
            if ('father' == this.tag) {
                // 父页面原数据处理逻辑
            } else if ('child' == this.tag) {
                this.that.hear(data) // 子页面处理
            }
        }
    }
}
</script>
```

子页面使用`inject`声明父页面的方法，使用`this.`方式调用：

```vue

<script>
export default {
    inject: ['say2'],
    methods: {
        say(code) {
            this.code = code  // 预期要接收数据的结构
            this.say2('child', this);
        }

        hear(data) {
            // 子页面原处理数据逻辑
            this.data[this.code] = data
        ...
        }
    }
}
</script>
```
