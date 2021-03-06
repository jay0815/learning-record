# docker 镜像

```
$ docker images u*

EPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               38c759202e30        5 days ago          196.6 MB
```

`docker images` 可以查看所有镜像。<br>
在 `images` 命令后可以添加通配符 `*` 找出符合条件的一系列镜像。

```
$ docker inspect ubuntu
```

查看镜像的详细信息可以通过 `inspect` 命令。

```
$ docker search ubuntu

NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
ubuntu                            Ubuntu is a Debian-based Linux operating s...   4192      [OK]       
ubuntu-upstart                    Upstart is an event-based replacement for ...   65        [OK]       
rastasheep/ubuntu-sshd            Dockerized SSH service, built on top of of...   29                   [OK]
torusware/speedus-ubuntu          Always updated official Ubuntu docker imag...   26                   [OK]
ubuntu-debootstrap                debootstrap --variant=minbase --components...   25        [OK]       
nickistre/ubuntu-lamp             LAMP server on Ubuntu                           8                    [OK]
```

通过 `search` 命令可以在 `Docker Hub` 上搜索符合要求的镜像。

- `DESCRIPTION` 镜像的别名。
- `STARS` 用户对镜像的评分。
- `OFFICIAL` 是否为官方镜像。
- `AUTOMATED` 是否使用了自动构建。

```
$ docker rmi ubuntu

Deleted: sha256:f05919001e0d37e8b66a2d057d674a7d9e2238216433fb9a2fbf1b2796c50875
Deleted: sha256:60156998057f59e3e757b34f60210b23a9996cf28f2dec0caaac883b88827111

$ docker rmi $(docker ps -a -q) // 删除所有镜像
```

可以通过 `rmi` 命令来删除镜像。

## 创建本地镜像

```
$ docker run -it ubuntu:14.04
root@3d71db0cbf42:/# apt-get update
... // 此处省略

root@3d71db0cbf42:/# apt-get install sqlite3
... // 此处省略

root@3d71db0cbf42:/# echo "test docker commit" >> hello docker
root@3d71db0cbf42:/# exit

$ docker ps -al

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
3d71db0cbf42        ubuntu:14.04        "/bin/bash"         5 minutes ago       Exited (0) 12 seconds ago                       distracted_bl

$ docker commit -m 'Message' --author="bert" 3d71db0cbf42 bert/sqlite3:v1
sha256:5df3e2b3031b42f26f645a1096fe32c12197dc155828a82f9b9bbc6aacceb9cd

$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
bert/sqlite3        v1                  5df3e2b3031b        5 seconds ago       201.4 MB

$ docker run -it bert/sqlite3:v1
root@13bac1f230d4:/# cat hello
test docker commit docker

root@13bac1f230d4:/# sqlite3 -version
3.8.2 2013-12-06 14:53:30 27392118af4c38c5203a04b8013e1afdb1cebd0d
```

1. 创建容器
2. 更新容器
3. 安装 `sqlite3`
4. 创建文件
5. 退出
6. 查询容器id
7. 通过id创建本地镜像
8. 进入镜像查看文件
9. 查看 `sqlit3` 版本

从以上信息可以看出，我们已经成功创建自定义镜像了。

## 使用 `Dockerfile` 创建镜像

推荐使用 `Dockerfile` 来构建镜像。想需要对镜像进行的操作全部写到一个文件中，将需要对镜像进行的操作全部写到一个文件中，然后使用 `docker build` 命令从这个文件中创建镜像。<br>
这种方法可以使镜像的创建变得透明和杜丽华，并且创建过程可以被重复执行。<br>
`Dockerfile` 文件以行为单位，行首为 `Dockerfile` 命令，命令都是大写形式，其后紧跟着的是命令的参数。

下面是一个书上抄来的示例，没有实际意义，但是覆盖的比较全面。

```
# Version: 1.0.1
FROM ubuntu:14.04

MAINTAINER bert "bert@qq.com"

# 设置rot用户为后续命令的执行者
USER root

# 执行操作
RUN apt-get update
RUN apt-get insatll -y nginx

# 使用&&拼接命令
RUN touch test.txt && echo "abc" >> abc.txt

# 对外暴露端口
EXPOSE 80 8080 1038

# 添加文件
ADD abc.txt /opt/

# 添加文件夹
ADD /webapp /opt/webapp

# 添加网络文件
ADD http://www.baidu.com/img/bd_logo1.png /opt/

# 设置环境变量
ENV WEBAPP_PORT=9090

# 设置工作目录
WORKDIR /opt/

# 设置启动命令
ENTRYPOINT ['ls']

# 设置启动参数
CMD ["-a", "-l"]

# 设置卷
VOLUME ["/data", "var/www"]

# 设置子镜像的触发操作
ONBUILD ADD ./app/src
ONBUILD RUN echo "on build excuted" >> onbuild.txt
```

构建命令

```
docker build -t xxx/test:v1 .
```

这个非常坑，我按照书上的配置执行了 `DockerFile` 只看见 `image` 在不断的变大中，直到90G,实在受不了了，就强制结束这次build。

之后就发现少了70G的内存，吓死宝宝了....

建议在第一次做测试的时候只编写简单的 `DockerFile`

```
# Version: 1.0.1
FROM ubuntu:14.04
```

亲测成功。^_^!!
