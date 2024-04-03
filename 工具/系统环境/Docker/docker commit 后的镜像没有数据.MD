# docker commit 后的镜像没有数据

    docker commit load save mysql

## 背景

工位机环境迁移MySQL时，使用commit》save》MySQL转移》load》run -p，使用数据库连接工具连接成功后，发现没有MySQL中没有库表等数据。

## 原因分析

直接搜，找到[这篇博客](https://www.cnblogs.com/scfssq/p/17516803.html)

## 解决方案

1. 修改mysql容器的数据目录 在根目录下 新建目录 mdkir /data
2. 将 /var/lib/mysql 复制 到新建目录中
3. 修改 /etc/mysql/mysql.conf.d/mysqld.cnf 》 ‘datadir = /data/mysql ’
4. 退出容器
5. 重新将该容器 commit 为新 的镜像， 再打开新容器

### 注

这个博客原文还提到命令行改文件的方法（但是我没有尝试，因为我是在win desktop界面里改的，如果无界面可以尝试这个方法）：

```Bash
docker exec -it mysql  bash
mkdir /data
mv var/lib/mysql   /data/
#查看一下 datadir       = /var/lib/mysql   这行的行号
cat -n /etc/mysql/mysql.conf.d/mysqld.cnf
#由于没有 vi 命令  所以用 sed 命令把这行删除
sed -i  32d  /etc/mysql/mysql.conf.d/mysqld.cnf
#在最后一行加上修改后的数据目录
echo  "datadir = /data/mysql/"  >> /etc/mysql/mysql.conf.d/mysqld.cnf
```
