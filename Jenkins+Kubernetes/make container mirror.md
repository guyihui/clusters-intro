# 将 web 应用打包成容器镜像
## 准备
* 镜像仓库账号（例如：[*Docker Hub*](https://hub.docker.com/)）
* 通过 Jenkins 构建的 web 应用 .war 包
* Docker (安装参照官网或其他教程)
## 目标
* 将项目部署到 tomcat 中，使用 MySQL 作为数据库。
* 将两者打包为 docker 镜像并上传到 Docker Hub 上以便远程部署到云服务器上。
--------
## 安装 Tomcat & MySQL
*（打包为镜像有两种方式：一种是通过 Dockerfile；一种是基于现有容器。出于易于学习的目的， 此次采用后者）*
### 下载官方 tomcat & mysql 镜像
```s
$ docker pull tomcat:latest
$ docker pull mysql:latest
```
其中`:tag_name`代表镜像的版本, `:latest`代表最新版本,其他可根据具体需求在 [*Docker Hub*](https://hub.docker.com/) 等镜像源网站上搜索. `docker pull`默认使用官方镜像网站,有时会出现连接失败的情况,此时可以采用更换镜像源的方式解决,具体解决方法因系统而异. Docker toolbox版本可以使用docker-machine进入default虚拟机,参考 Linux 修改 daemon.json , 添加指定源为 docker-cn 或其他.</br>
拉取成功后,可以使用`docker images`命令查看已有镜像.
## 部署项目
### MySQL
1. 首先,运行容器,跑起MySQL服务
```s
$ docker run  -d  -p 3306:3306  --name mysql  -e MYSQL_ROOT_PASSWORD=123456 mysql
```
-d 代表后台运行,并返回container-id</br>
-p 设置容器端口和服务端口的对应</br>
--name 设置容器name,对于之后连接两个容器有帮助</br>
-e 设置环境参数</br>
最后的`mysql`代表使用的镜像</br>

2. 数据可以从本地MySQL数据库中dump成.sql脚本,放入虚拟机中运行
```s
$ docker cp Dump.sql container-id:/
$ docker exec -it %container-id% /bin/bash
-----
$$ mysql -uroot -p123456
$$ source /Dump.sql
...
```
*如果是windows下使用toolbox,源文件根目录为toolbox安装目录

3. 退出准备部署tomcat (注意不要关闭docker, MySQL在后台运行)
### Tomcat
1. 运行
```s
$ docker run -d  -p 8080:8080  --name webapp  --link mysql:mysql tomcat
```
--link docker提供的连通容器间网络的参数,将mysql这个hostname写入/etc/hosts里,并对应ip

2. 部署项目
```s
$ docker cp app.war container-id:/usr/local/tomcat/webapps
```

3. 检验</br>
打开浏览器,输入(docker启动时分配的ip):8080/app应该可以看到页面.</br>

## 打包镜像
```s
$ docker login
Username: xxxx
Password: ******
$ docker commit container-id xxxx/repo:tag
$ docker push xxxx/repo:tag
```
登录Hub后可以看到自己的镜像仓库,如果是public的话,所有人都可以拉取你的镜像.

## 下一步
镜像部署到 Kubernetes