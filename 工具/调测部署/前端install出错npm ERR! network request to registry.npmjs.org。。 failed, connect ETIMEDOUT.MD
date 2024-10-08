# 前端install出错npm ERR! network request to registry.npmjs.org。。 failed, connect ETIMEDOUT

    npm install、vue、idea

## 背景

小长假复工，处理不同项目切换前后端代码，npm install卡半天，最后失败

## 问题描述

切换代码后，idea提示重新install，点击其按钮后，依赖安装进度安装了一阵卡住不动了，控制台最后爆出错误如下：

```log
npm WARN Conflicting peer dependency: eslint-plugin-promise@7.1.0
npm WARN node_modules/eslint-plugin-promise
npm WARN   peer eslint-plugin-promise@">=4.0.0" from eslint-config-standard@12.0.0
npm WARN   node_modules/@vue/eslint-config-standard/node_modules/eslint-config-standard
npm WARN     eslint-config-standard@"^12.0.0-alpha.0" from @vue/eslint-config-standard@3.0.5
npm WARN     node_modules/@vue/eslint-config-standard
npm WARN ERESOLVE overriding peer dependency
npm WARN While resolving: eslint-config-standard@12.0.0
npm WARN Found: eslint-plugin-standard@3.1.0
npm WARN node_modules/eslint-plugin-standard
npm WARN   eslint-plugin-standard@"^3.1.0" from @vue/eslint-config-standard@3.0.5
npm WARN   node_modules/@vue/eslint-config-standard
npm WARN     dev @vue/eslint-config-standard@"^3.0.0-beta.10" from the root project
npm WARN
npm WARN Could not resolve dependency:
npm WARN peer eslint-plugin-standard@">=4.0.0" from eslint-config-standard@12.0.0
npm WARN node_modules/@vue/eslint-config-standard/node_modules/eslint-config-standard
npm WARN   eslint-config-standard@"^12.0.0-alpha.0" from @vue/eslint-config-standard@3.0.5
npm WARN   node_modules/@vue/eslint-config-standard
npm WARN
npm WARN Conflicting peer dependency: eslint-plugin-standard@5.0.0
npm WARN node_modules/eslint-plugin-standard
npm WARN   peer eslint-plugin-standard@">=4.0.0" from eslint-config-standard@12.0.0
npm WARN   node_modules/@vue/eslint-config-standard/node_modules/eslint-config-standard
npm WARN     eslint-config-standard@"^12.0.0-alpha.0" from @vue/eslint-config-standard@3.0.5
npm WARN     node_modules/@vue/eslint-config-standard
npm WARN EBADENGINE Unsupported engine {
npm WARN EBADENGINE   package: 'v-click-outside-x@4.0.19',
npm WARN EBADENGINE   required: { node: '>=8.11.4', npm: '6.10.1' },
npm WARN EBADENGINE   current: { node: 'v16.19.1', npm: '8.19.3' }
npm WARN EBADENGINE }
npm WARN EBADENGINE Unsupported engine {
npm WARN EBADENGINE   package: 'v-click-outside-x@3.7.1',
npm WARN EBADENGINE   required: { node: '>=8.11.4', npm: '6.4.1' },
npm WARN EBADENGINE   current: { node: 'v16.19.1', npm: '8.19.3' }
npm WARN EBADENGINE }
npm ERR! code ETIMEDOUT
npm ERR! syscall connect
npm ERR! errno ETIMEDOUT
npm ERR! network request to registry.npmjs.org/jspdf/-/jspdf-2.5.2.tgz failed, reason: connect ETIMEDOUT 104.16.27.34:443
npm ERR! network This is a problem related to network connectivity.
npm ERR! network In most cases you are behind a proxy or have bad network settings.
npm ERR! network
npm ERR! network If you are behind a proxy, please make sure that the
npm ERR! network 'proxy' config is set properly.  See: 'npm help config'
```

## 原因分析

控制台的日志还提示看某日志文件，然而其内没有找到有价值信息。

咨询ai助手，却告诉我检查网络，配置代理，这可是公司电脑，胡说啥呢？

bing搜索：`npm ERR! network request to https://registry.npmjs.org/jspdf/-/jspdf-2.5.2.tgz failed, reason: connect ETIMEDOUT 104.16.27.34:443`
找到[这篇博客](https://blog.csdn.net/weixin_43405300/article/details/131717884)，其中提到切换镜像地址，
但其地址已失效，好在评论区有人给出了[可用地址](https://registry.npmmirror.com/)

## 解决方案

1. 命令行进入前端项目目录，或在项目目录打开命令行
2. 查看镜像地址`npm config get registry`，https://registry.npmjs.org/
3. 切换地址`npm config set registry https://registry.npmmirror.com`
4. 重新查看镜像地址`npm config get registry`，https://registry.npmmirror.com/
5. 重试install，成功！
