# 两Docker异地间迁移容器，或同Docker不同时间备份和恢复容器

    docker、容器、迁移、备份、恢复

本文所有命令均使用root用户执行，使用`sudo -i`命令，然后输入当前用户的密码可以进入root。（使用等宽字体查看md源文件可以更好观察命令）

## 操作方法

搜索、查看镜像，查看容器
```Bash
$ docker search tomca
$ docker images
$ docker ps -a # -a参数查看所有包括停止运行的容器
```
拉取镜像、创建并运行容器
```Bash
$ docker pull tomcat
$ 
```

将某容器保存为镜像

```Bash
# docker commit -a='用户名uname' -m='描述信息描述信息描述信息描述' 镜像id  项目名项目名:版本号
$ docker commit -a='xkm-tomcat' -m='add index.html > hello'  cfc   xkm/tomcat:1.1
```

```Bash
$ 
$ 
```

```Bash
$ 
$ 
```
删除镜像、容器
```Bash
$ docker rmi 76
$ docker rmi -f redis:latest hello-world:latest
$ 
```

```Bash
$ 
$ 
```
