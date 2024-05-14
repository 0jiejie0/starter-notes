# idea远程连接docker

    docker、ubuntu、linux、远程连接、IntelliJ idea

**注意！本文中开启docker远程连接的方法只能在确定环境安全的内网中使用，不可在公网服务器设置，有极大安全风险！**

**注意！本文中开启docker远程连接的方法只能在确定环境安全的内网中使用，不可在公网服务器设置，有极大安全风险！**

**注意！本文中开启docker远程连接的方法只能在确定环境安全的内网中使用，不可在公网服务器设置，有极大安全风险！**

## 步骤

1.

```Bash
> vim /usr/lib/systemd/system/docker.service # set nu 打开行号
```

2. 找到第13行ExecStart，在末尾添加`-H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375`，如下：

```Bash
  1 [Unit]
  2 Description=Docker Application Container Engine
  3 Documentation=https://docs.docker.com
  4 After=network-online.target docker.socket firewalld.service containerd.service time-set.target
  5 Wants=network-online.target containerd.service
  6 Requires=docker.socket
  7
  8 [Service]
  9 Type=notify
 10 # the default is not to use systemd for cgroups because the delegate issues still
 11 # exists and systemd currently does not support the cgroup feature set required
 12 # for containers run by docker
 13 ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
 14 ExecReload=/bin/kill -s HUP $MAINPID
 15 TimeoutStartSec=0
 16 RestartSec=2
 17 Restart=always
 18
 19 # Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
 20 # Both the old, and new location are accepted by systemd 229 and up, so using the old location
 21 # to make them work for either version of systemd.
 22 StartLimitBurst=3
 23
 24 # Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
 25 # Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
 26 # this option work for either version of systemd.
 27 StartLimitInterval=60s
 28
 29 # Having non-zero Limit*s causes performance problems due to accounting overhead
 30 # in the kernel. We recommend using cgroups to do container-local accounting.
 31 LimitNPROC=infinity
 32 LimitCORE=infinity
 33
 34 # Comment TasksMax if your systemd version does not support it.
 35 # Only systemd 226 and above support this option.
 36 TasksMax=infinity
 37
 38 # set delegate yes so that systemd does not reset the cgroups of docker containers
 39 Delegate=yes
 40
 41 # kill only the docker process, not all processes in the cgroup
 42 KillMode=process
 43 OOMScoreAdjust=-500
 44
 45 [Install]
 46 WantedBy=multi-user.target
```

3. 重启

```Bash
systemctl daemon-reload
systemctl restart docker 
```

重启后不要忘记把开发环境使用的MySQL和Redis容器启动起来

4. 检查2375[端口是否开放](../Ubuntu/查看端口开放状态.MD#ss)

```Bash
> ss -tuln
```

使用curl连接一下

```Bash
> curl http://localhost:2375/version
{"Platform":{"Name":"Docker Engine - Community"},"Components":[{"Name":"Engine","Version":"26.0.0","Details":{"ApiVersion":"1.45","Arch":"amd64","BuildTime":"2024-03-20T15:17:51.000000000+00:00","Experimental":"false","GitCommit":"8b79278","GoVersion":"go1.21.8","KernelVersion":"5.4.0-169-generic","MinAPIVersion":"1.24","Os":"linux"}},{"Name":"containerd","Version":"1.6.28","Details":{"GitCommit":"ae07eda36dd25f8a1b98dfbf587313b99c0190bb"}},{"Name":"runc","Version":"1.1.12","Details":{"GitCommit":"v1.1.12-0-g51d5e94"}},{"Name":"docker-init","Version":"0.19.0","Details":{"GitCommit":"de40ad0"}}],"Version":"26.0.0","ApiVersion":"1.45","MinAPIVersion":"1.24","GitCommit":"8b79278","GoVersion":"go1.21.8","Os":"linux","Arch":"amd64","KernelVersion":"5.4.0-169-generic","BuildTime":"2024-03-20T15:17:51.000000000+00:00"}
```

5. 生成安全证书

执行下面这个脚本：

```Bash
#!/bin/bash

#相关配置信息
SERVER="192.168.56.10"
PASSWORD="vagrant"
COUNTRY="CN"
STATE="Shan Dong"
CITY="Ji Nan"
ORGANIZATION="Huai Yin"
ORGANIZATIONAL_UNIT="Dev"
EMAIL="xkm.0jiejie0@qq.com"

###开始生成文件###
echo "开始生成文件"

#切换到生产密钥的目录
if [ ! -d "/opt/docker_ca" ]; then
mkdir -p /opt/docker_ca
fi
cd /opt/docker_ca
#生成ca私钥(使用aes256加密)
openssl genrsa -aes256 -passout pass:$PASSWORD  -out ca-key.pem 4096
#生成ca证书，填写配置信息
openssl req -new -x509 -passin "pass:$PASSWORD" -days 365 -key ca-key.pem -sha256 -out ca.pem -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$SERVER/emailAddress=$EMAIL"

#生成server证书私钥文件
openssl genrsa -out server-key.pem 4096
#生成server证书请求文件
openssl req -subj "/CN=$SERVER" -sha256 -new -key server-key.pem -out server.csr
#配置白名单，多个用逗号隔开
sh -c 'echo subjectAltName = IP:'$SERVER',IP:0.0.0.0 >> extfile.cnf'
#把 extendedKeyUsage = serverAuth 键值设置到extfile.cnf文件里，限制扩展只能用在服务器认证
sh -c 'echo extendedKeyUsage = serverAuth >> extfile.cnf'
#使用CA证书及CA密钥以及上面的server证书请求文件进行签发，生成server自签证书
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -\CAcreateserial -out server-cert.pem -extfile extfile.cnf

#生成client证书RSA私钥文件
openssl genrsa -out key.pem 4096
#生成client证书请求文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
#继续设置证书扩展属性
sh -c 'echo extendedKeyUsage = clientAuth >> extfile.cnf'
#生成client自签证书（根据上面的client私钥文件、client证书请求文件生成）
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -\CAcreateserial -out cert.pem -extfile extfile.cnf

#更改密钥权限
chmod 0400 ca-key.pem key.pem server-key.pem
#更改密钥权限
chmod 0444 ca.pem server-cert.pem cert.pem
#删除无用文件
rm client.csr server.csr

echo "生成文件完成"
###生成结束### 
```

6. 修改Docker配置

```Bash
# 打开docker配置文件 (这个和上面的是一个文件，其使用了软链接，路径不同而已，不用纠结)
> vim /lib/systemd/system/docker.service
# 在ExecStart的/usr/bin/dockerd后面加\，然后添加如下的配置信息
ExecStart=/usr/bin/dockerd \
        --tlsverify \
        --tlscacert=/opt/docker_ca/ca.pem \
        --tlscert=/opt/docker_ca/server-cert.pem \
        --tlskey=/opt/docker_ca/server-key.pem \
        -H tcp://0.0.0.0:2376 \
        -H unix:///var/run/docker.sock \
        -H fd:// --containerd=/run/containerd/containerd.sock
# 重新启动docker
> systemctl daemon-reload && systemctl restart docker
```

7. `ctrl`+`alt`+`s`打开设置

Tools > SSH Configurations，点击+，输入IP、用户名、密码，点击apply

8. 配置idea客户端

将/opt/docker-ca目录下的ca.pem,cert.pem,key.pem（我这复制了所有.pem文件，懒得挑了）复制到自己的电脑中某固定目录（和虚拟机放在一起，建个文件夹：dockerFile）

在idea设置中，找到 Build,Execution,Deployment > Docker，点击+，连接方式选择`TCP socket`

url: https://192.168.56.10:2376
证书路径：D:\dev\iso\dockerFile\dockerFile

可以看到下方提示 Connection successful

9. 点击确定，可以看到下方service窗口中出现了docker服务，双击连接，可以看到容器、镜像、网络、卷（英文）

点击具体容器实例的看板，可以看到其版本配置信息；
点击镜像，可以在右侧框中输入镜像名称拉取镜像（不知道为什么没有搜索功能，可能是我没找到）；
这样一来用起docker要方便得多。

[参考文章](https://blog.csdn.net/code_yun/article/details/103021771)
