# 两Docker异地间迁移容器，或同Docker不同时间备份和恢复容器等docker维护命令

    docker、容器、迁移、备份、恢复、维护

本文所有命令均使用root用户执行，使用`sudo -i`命令，然后输入当前用户的密码可以进入root。

## 操作方法

### 搜索、查看镜像，查看容器及状态

```Bash
$ docker search tomca
$ docker images
$ docker ps -aq # -a查看所有包括停止运行的容器 -q quiet 只查看容器id
$ docker logs --tail 3 8c -t -f # 查看从末尾3行起开始的日志，-t包含时间戳，-f不会退出一直查看，直到按下ctrl+c
$ docker top 8c # 相当于linux的top命令，查看容器内运行的程序
$ docker inspect 8c # 查看容器或镜像的json格式元数据
```

### 拉取镜像、创建并运行容器

```Bash
$ docker pull tomcat # 最新版
$ docker pull centos:centos7 # 指定版本
$ docker run -it -p 8888:8080 tomcat # 创建容器并指定映射端口 主机端口：容器端口
$ docker run -it centos:centos7 /bin/bash # 创建容器并启动一个终端，容器内必须有进程运行，否则会立即停止容器
$ docker run -d centos:centos7 /bin/bash -c 'while true; do echo hello;sleep 2;done' # 创建容器并执行命令，两秒输出一个条hello
```

### 命令执行及开关容器

#### 执行命令、连接运行中的容器

```Bash
$ docker exec -it a9 ls /root/ # 连接容器运行命令，查看指定目录
$ docker exec -it 50c /bin/bash # 连接容器，重新启动一个bash终端
$ docker attach a # 连接容器，连接其默认启动的程序（该程序停止后会关闭容器，若进入的是bash终端，不可使用ctrl+d退出）
```

#### 退出容器

在容器内退出容器：

退出容器bash终端界面但保留容器后台运行：`ctrl`+`p`，`ctrl`+`q`；

关闭终端并退出容器（对于创建时开启bash的容器）：`ctrl`+`d`；

关闭（除bash终端外的）程序并退出容器：`ctrl`+`c`；

#### 容器启动与停止

在容器外启动或停止容器：

启动、重启、停止容器，杀死正在运行的容器

```Bash
$ docker start 50
$ docker restart 50
$ docker stop 50
$ docker kill 6
```

### 镜像、容器及其相关文件的备份、转移及删除

#### 将某容器保存为镜像

```Bash
# docker commit -a='用户名uname' -m='描述信息描述信息描述信息描述' 镜像id  项目名项目名:版本号
$ docker commit -a='xkm-tomcat' -m='add index.html > hello'  cfc   xkm/tomcat:1.1
```

#### 文件转移

```Bash
$ docker cp 50a:/root/hello.txt /root/ # 容器 》 docker所在的主机位置（虚拟机）
```

#### 删除镜像或容器

**慎用`-f`强制删除参数！** 正在运行中的容器默认不能删除容器，镜像之上有容器默认不能删除镜像。
可以加`-f`强制删除，但有不可恢复的风险。

删除及批量删除容器：

```Bash
$ docker rm 87
$ docker rm -f $(docker ps -aq)
$ docker ps -aq | xargs docker rm
```

删除镜像：

```Bash
$ docker rmi 76
$ docker rmi -f redis:latest hello-world:latest
```

```Bash
$ 
$ 
```
