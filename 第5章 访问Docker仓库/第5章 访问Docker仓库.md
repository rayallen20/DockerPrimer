# 第5章 访问Docker仓库

仓库(Repository)是集中存放镜像的地方,分为公有仓库和私有仓库.

注册服务器是存放仓库的具体服务器.1个注册服务器上可以有多个仓库,而每个仓库里可以有多个镜像.可以认为仓库是一个具体的项目或目录.

例如:`https://hub.docker.com/repository/docker/40486453/test_repo`.其中`https://hub.docker.com/repository/docker/40486453`是注册服务器的地址,`test_repo`是仓库名

## 5.1 Docker hub公共镜像市场

##### 1. 登录

`docker login`

##### 2. 基本操作

搜索镜像:`docker search`

```
root@docker-test:/home/roach# docker search centos
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
centos                            The official build of CentOS.                   6973      [OK]       
ansible/centos7-ansible           Ansible on Centos7                              135                  [OK]
consol/centos-xfce-vnc            Centos container with "headless" VNC session…   135                  [OK]
jdeathe/centos-ssh                OpenSSH / Supervisor / EPEL/IUS/SCL Repos - …   121                  [OK]
centos/systemd                    systemd enabled base container.                 105                  [OK]
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   92                   
imagine10255/centos6-lnmp-php56   centos6-lnmp-php56                              58                   [OK]
tutum/centos                      Simple CentOS docker image with SSH access      48                   
centos/postgresql-96-centos7      PostgreSQL is an advanced Object-Relational …   45                   
centos/httpd-24-centos7           Platform for running Apache httpd 2.4 or bui…   41                   
kinogmt/centos-ssh                CentOS with SSH                                 29                   [OK]
guyton/centos6                    From official centos6 container with full up…   10                   [OK]
nathonfowlie/centos-jre           Latest CentOS image with the JRE pre-install…   8                    [OK]
centos/tools                      Docker image that has systems administration…   7                    [OK]
drecom/centos-ruby                centos ruby                                     6                    [OK]
roboxes/centos8                   A generic CentOS 8 base image.                  4                    
mamohr/centos-java                Oracle Java 8 Docker image based on Centos 7    4                    [OK]
darksheer/centos                  Base Centos Image -- Updated hourly             3                    [OK]
amd64/centos                      The official build of CentOS.                   2                    
miko2u/centos6                    CentOS6 日本語環境                                   2                    [OK]
dokken/centos-7                   CentOS 7 image for kitchen-dokken               2                    
blacklabelops/centos              CentOS Base Image! Built and Updates Daily!     1                    [OK]
mcnaughton/centos-base            centos base image                               1                    [OK]
starlabio/centos-native-build     Our CentOS image for native builds              0                    [OK]
smartentry/centos                 centos with smartentry                          0                    [OK]
```

根据是否为官方提供,可将这些镜像资源分为2类:

- 一种是类似`centos`这样的基础镜像,也称为根镜像.这些镜像是由dOCKER公司创建、验证、支持、提供,这些镜像往往使用单个单词作为名字
- 另一种类似`ansible/centos7-ansible`的镜像,是由Docker用户ansible创建并维护的,带有用户名称为前缀,表明是某用户下的某仓库.可以通过`用户名/镜像名:TAG`来指定使用某个用户提供的镜像.

例:下载官方`centos`镜像到本地

```
root@docker-test:/home/roach# docker pull centos
Using default tag: latest
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

##### 3. 自动创建

Automated Builds:自动创建是Docker Hub提供的自动化服务,这一功能可以自动跟随项目代码的变更而重新构建镜像.

例:

用户构建了某应用镜像,如果应用发布新版本,用户需要手动更新镜像.而自动创建则允许用户通过Docker Hub指定跟踪一个目标网站(目前支持GitHub或BitBucket)上的项目,一旦项目发生新的提交,则自动执行创建.

要配置自动创建,按照如下步骤操作即可:

1. 创建并登录Docker Hub和Github
2. 在目标网站中允许Docker Hub访问服务
3. 在Docker Hub中配置一个"自动创建"类型的项目
4. 选取一个目标网站中的项目(需要包含Dockerfile)和分支
5. 指定Dockerfile的位置,并提交创建

之后可以在Docker Hub的自动创建页面中跟踪每次的创建状态

## 5.2 第三方镜像市场

略

## 5.3 搭建本地私有仓库

##### 1. 使用registry镜像创建私有仓库

`docker run -d -p 宿主机端口:容器端口 registry:TAG`:运行官方提供的`registry`镜像.(注:官方建议`TAG`为2)

可能会用到的选项:

- `-v`,`--volume`:挂载宿主机上的文件卷到容器内.仓库默认被创建在容器的`/var/lib/registry`目录下.可使用该参数指定镜像文件存放的路径.


例:将宿主机的5000端口映射到仓库容器的5000端口,并将上传到该仓库的镜像存放在宿主机的`opt/data/registry`目录下

- step1. 宿主机上创建目录

```
root@docker-test:/opt# mkdir -p /opt/data/registry
```

- step2. 运行容器

```
root@docker-test:/opt# docker run -d -p 5000:5000 -v /opt/data/registry/:/var/lib/registry registry:2
```

- step3. 查看结果

```
root@docker-test:/opt# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                       NAMES
774c7deda006   registry:2     "/entrypoint.sh /etc…"   4 seconds ago   Up 3 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   vigilant_elbakyan
315e54cb5f8e   ubuntu:18.04   "bash"                   18 hours ago    Up 18 hours                                                test
```

##### 2. 管理私有仓库

我在写这份读书笔记时,搭建私有仓库的虚拟机地址是`192.168.0.152`,私有仓库的端口为`5000`.另一台用于上传镜像的虚拟机地址为`192.0168.0.153`我们称这台机器为客户机.

- step1. 在客户机上拉取镜像`ubuntu:18.04`

```
root@ubuntu-docker-test-client:/home/price# docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
284055322776: Pull complete 
Digest: sha256:0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

- step2. 在客户机上使用`docker tag`命令,将该镜像标记为`192.168.0.152:5000/test`

```
root@ubuntu-docker-test-client:/home/price# docker tag ubuntu:18.04 192.168.0.152:5000/test
root@ubuntu-docker-test-client:/home/price# docker images
REPOSITORY                TAG       IMAGE ID       CREATED        SIZE
192.168.0.152:5000/test   latest    5a214d77f5d7   3 months ago   63.1MB
ubuntu                    18.04     5a214d77f5d7   3 months ago   63.1MB
```

- step3. 在客户机上添加信任的私有仓库列表

```
root@ubuntu-docker-test-client:/home/price# cat /etc/docker/daemon.json 
{
    "registry-mirrors": ["https://sb6xpp51.mirror.aliyuncs.com"],
    "insecure-registries": [
    	"192.168.0.152:5000"
    ]
}
```

- step4. 在客户机上使用`docker push`上传标记的镜像

```
root@ubuntu-docker-test-client:/home/price# docker push 192.168.0.152:5000/test
Using default tag: latest
The push refers to repository [192.168.0.152:5000/test]
824bf068fd3d: Pushed 
latest: digest: sha256:fc0d6af5ab38dab33aa53643c4c4b312c6cd1f044c1a2229b2743b252b9689fc size: 529
```

- step5. 在客户机上使用`curl`查看仓库`192.168.0.152:5000`中的镜像

```
root@ubuntu-docker-test-client:/home/price# curl -XGET http://192.168.0.152:5000/v2/_catalog
{"repositories":["test"]}
```

- step6. 在客户机上删除打过TAG的镜像,以便后续测试拉取

```
root@ubuntu-docker-test-client:/home/price# docker images
REPOSITORY                TAG       IMAGE ID       CREATED        SIZE
192.168.0.152:5000/test   latest    5a214d77f5d7   3 months ago   63.1MB
ubuntu                    18.04     5a214d77f5d7   3 months ago   63.1MB
root@ubuntu-docker-test-client:/home/price# docker rmi 192.168.0.152:5000/test
Untagged: 192.168.0.152:5000/test:latest
Untagged: 192.168.0.152:5000/test@sha256:fc0d6af5ab38dab33aa53643c4c4b312c6cd1f044c1a2229b2743b252b9689fc
root@ubuntu-docker-test-client:/home/price# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

- step7. 从私有仓库拉取镜像

```
root@ubuntu-docker-test-client:/home/price# docker pull 192.168.0.152:5000/test
Using default tag: latest
latest: Pulling from test
Digest: sha256:fc0d6af5ab38dab33aa53643c4c4b312c6cd1f044c1a2229b2743b252b9689fc
Status: Downloaded newer image for 192.168.0.152:5000/test:latest
192.168.0.152:5000/test:latest
```

- step8. 查看结果

```
root@ubuntu-docker-test-client:/home/price# docker images
REPOSITORY                TAG       IMAGE ID       CREATED        SIZE
192.168.0.152:5000/test   latest    5a214d77f5d7   3 months ago   63.1MB
ubuntu                    18.04     5a214d77f5d7   3 months ago   63.1MB
```