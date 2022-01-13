# 第3章 使用Docker镜像

## 配置国内镜像源

- step1. 配置国内镜像源

```
root@docker-test:/home/roach# sudo vim /etc/docker/daemon.json
root@docker-test:/home/roach# cat /etc/docker/daemon.json 
{
    "registry-mirrors": ["https://sb6xpp51.mirror.aliyuncs.com"]
}
```

此处使用阿里云镜像源.阿里云镜像源需要登录万网申请(免费的).

- step2. 重启Docker

```
root@docker-test:/home/roach# sudo service docker restart
```

- step3. 查看Docker信息:

```
root@docker-test:/home/roach# docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.7.1-docker)
  scan: Docker Scan (Docker Inc., v0.12.0)

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 2
 Server Version: 20.10.12
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc version: v1.0.2-0-g52b36a2
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
 Kernel Version: 5.4.0-94-generic
 Operating System: Ubuntu 20.04.3 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 1
 Total Memory: 11.7GiB
 Name: docker-test
 ID: UOMX:TXX4:23S5:OMIM:HFYN:W542:XIQ5:XYM2:P6NF:CCTD:DVYT:7FAE
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Registry Mirrors:
  https://sb6xpp51.mirror.aliyuncs.com/
 Live Restore Enabled: false

WARNING: No swap limit support
```

确认修改镜像源成功.

镜像是Docker三大核心概念中最重要的,自Docker诞生之日起镜像就是相关社区最为热门的关键词.

**Docker运行容器前需要本地存在对应的镜像.如果镜像不存在,Docker会尝试先从默认镜像仓库下载**(默认使用Docker Hub公共注册服务器中的仓库),用户也可以通过配置,使用自定义的镜像仓库.

## 3.1 获取镜像

下载镜像:`docker [image] pull NAME[:TAG]`

`NAME`:镜像仓库名称(和git仓库的概念应该是一样的我猜测)

`TAG`:镜像的标签(通常用于表示版本信息,和git的tag一样)

**通常情况下,描述一个镜像需要包括"名称+标签"信息.**

例:拉取一个Ubuntu18.04的基础镜像

```
root@docker-test:/home/roach# docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
284055322776: Pull complete 
Digest: sha256:0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
```

对于Docker镜像而言,若不显式指定`TAG`,则会默认选择`latest`标签,这会下载仓库中最新版本的镜像.

从中国科技大的Ubuntu仓库下载一个最新版本的Ubuntu操作系统镜像:

```
root@docker-test:/home/roach# docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
docker.io/library/ubuntu:latest
```

**注意:一般来说,镜像的`latest`标签意味着该镜像的内容会跟踪最新版本的变更而变化,内容是不稳定的.因此从稳定性上考虑,不要在生产环境中忽略镜像的标签信息或使用默认的`latest`标记的镜像.**

从下载过程中可以看出,镜像文件一般由若干层(layer)组成,类似`0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6`这样的串是层的唯一id.使用`docker pull`命令下载时,会获取并输出镜像的各层信息.**当不同的镜像包括相同的层时,本地仅存储1份内容,减小了存储空间.**

Question:在不同的镜像仓库服务器中,出现镜像重名的情况怎么办?

Answer:严格来讲镜像的仓库名称中,还应该添加仓库地址(就是之前改过镜像源的地址),只是我们在`/etc/docker/daemon.json`中配置了,所以拉取镜像时可以忽略.

个人理解:就相当于github上的kubernetes项目和gitee上的kubernetes不是一个东西,二者各自独立存在,不冲突.道理是一样的.

`pull`子命令支持的选项:

`-a`,`--all-tags=true|false`:是否获取仓库中的所有镜像,默认为否

`--disable-content-trust`:取消镜像的内容校验,默认为真

`--registry-mirror=proxy_URL`:指定镜像代理服务地址

使用镜像:利用该镜像创建一个容器,并执行`bash`应用,打印"Fuck World"

```
root@docker-test:/home/roach# docker run -it ubuntu:18.04 bash
root@016329fae3cd:/# echo "Fuck World"
Fuck World
root@016329fae3cd:/# exit
exit
root@docker-test:/home/roach# 
```

注意命令提示符处,进入容器后有一个`/#`.

### 3.2 查看镜像信息

##### 1. 使用`images`命令列出镜像

列出镜像:`docker images`或`docker image ls`

```
root@docker-test:/home/roach# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

```
root@docker-test:/home/roach# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

字段含义:

- `REPOSITORY`:表示镜像来自哪个仓库.比如`ubuntu`表示ubuntu系列的基础镜像
- `TAG`:表示不同的版本信息.TAG只是标记,不能用于标识镜像内容
- `IMAGE ID`:镜像的唯一标识.若2个镜像的ID相同,说明它们实际上指向了同一个镜像,只是具有不同的标签名称.(TODO:是不是类似于一个iso文件,有2个不同的快捷方式指向了这个文件?)(我的理解:标签由`NAME`和`TAG`2部分构成)
- `CREATED`:镜像最后的更新时间
- `SIZE`:优秀的镜像往往体积较小

其中`IMAGE ID`列非常重要,它唯一标识了镜像.在使用`IMAGE ID`时,一般可以使用该ID的前若干个字符组成的可区分串来替代完整的ID.

`TAG`信息用于标记来自同一个仓库的不同镜像.例如ubuntu仓库中有多个镜像,通过`TAG`信息来区分发行版本,如18.04、18.10等.

镜像大小信息只是表示了该镜像的逻辑体积大小,实际上由于相同的镜像层本地只会存储一份,所以物理上占用的存储空间会小于各镜像逻辑体积之和.

`images`子命令主要支持如下选项:

- `-a`,`--all=true|false`:列出所有(包括临时文件)镜像文件,默认为否
- `--digests=true|false`:列出镜像的数字摘要值,默认为否
- `-f`,`--filter=[]`:过滤列出的镜像
- `--format="TEMPLATE"`:控制输出格式
- `--no-trunc=true|false`:对输出结果中太长的部分是否进行截断,默认为是
- `-q`,`--quiet=true|false`:仅输出ID信息,默认为否

`man docker-images`查看更多子命令选项

##### 2. 使用`tag`命令添加镜像标签

`docker tag`:为本地镜像添加新的标签

例:添加一个新的`myubuntu:latest`标签

```
root@docker-test:/home/roach# docker tag ubuntu:latest myubuntu:latest
root@docker-test:/home/roach# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
myubuntu     latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

可以看到,`myubuntu:latest`和`ubuntu:latest`的`IMAGE ID`是相同的.实际上它们指向了同一个镜像文件,只是别名不同.`docker tag`命令添加的标签实际上起到了类似链接的作用.

##### 3. 使用`inspect`命令查看详细信息

`docker inspect`:获取指定镜像的详细信息.包括制作者、适应架构、各层的数字摘要等.(TODO:此处书上原文写的是`docker [image] inspect`,这个`[image]`是个啥?)

```
root@docker-test:/home/roach# docker inspect ubuntu:18.04
[
    {
        "Id": "sha256:5a214d77f5d747e6ed81632310baa6190301feeb875cf6bf9da560108fa09972",
        "RepoTags": [
            "ubuntu:18.04"
        ],
        "RepoDigests": [
            "ubuntu@sha256:0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2021-10-01T02:23:24.179667784Z",
        "Container": "20d614d2eca1b5a9ad6d5a56a80efce44096b87ca76a98256eb51f8dbaf7a8d2",
        "ContainerConfig": {
            "Hostname": "20d614d2eca1",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"bash\"]"
            ],
            "Image": "sha256:de5a48194cb6a383c018b7c57fa642457688605c5d6d4941db88fecabd225a55",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "20.10.7",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "bash"
            ],
            "Image": "sha256:de5a48194cb6a383c018b7c57fa642457688605c5d6d4941db88fecabd225a55",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 63136384,
        "VirtualSize": 63136384,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/5de9907d3d61c4e7928e9e3ea53f7d156d19fbe640dac79cb708fe17b92760eb/merged",
                "UpperDir": "/var/lib/docker/overlay2/5de9907d3d61c4e7928e9e3ea53f7d156d19fbe640dac79cb708fe17b92760eb/diff",
                "WorkDir": "/var/lib/docker/overlay2/5de9907d3d61c4e7928e9e3ea53f7d156d19fbe640dac79cb708fe17b92760eb/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:824bf068fd3dc3ad967022f187d85250eb052f61fe158486b2df4e002f6f984e"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

`docker inspect -f {{".项目名称"}}`:查看指定一项

```
root@docker-test:/home/roach# docker inspect -f {{".RootFS.Layers"}} ubuntu:18.04
[sha256:824bf068fd3dc3ad967022f187d85250eb052f61fe158486b2df4e002f6f984e]
```

##### 4. 使用`history`命令查看镜像历史

`docker history`:列出镜像中每个层的创建信息

```
root@docker-test:/home/roach# docker history ubuntu:18.04
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
5a214d77f5d7   3 months ago   /bin/sh -c #(nop)  CMD ["bash"]                 0B        
<missing>      3 months ago   /bin/sh -c #(nop) ADD file:0d82cd095966e8ee7…   63.1MB    
```

`docker history --no-trunc`:查看完整镜像历史信息

```
root@docker-test:/home/roach# docker history --no-trunc ubuntu:18.04
IMAGE                                                                     CREATED        CREATED BY                                                                                          SIZE      COMMENT
sha256:5a214d77f5d747e6ed81632310baa6190301feeb875cf6bf9da560108fa09972   3 months ago   /bin/sh -c #(nop)  CMD ["bash"]                                                                     0B        
<missing>                                                                 3 months ago   /bin/sh -c #(nop) ADD file:0d82cd095966e8ee78b593cb47a352eec842edb7bd9d9468e8a70154522447d1 in /    63.1MB    
```

### 3.3 搜寻镜像

`docker search`:搜索镜像源仓库中的镜像.语法为:`docker search [option] keyword`

`search`子命令的选项:

- `-f`,`--filter filter`:过滤结果
- `-format string`:格式化输出内容
- `--limit int`:限制输出结果个数,默认25个
- `--no-trunc`:不截断输出结果

例:搜索官方提供的,带有`nginx`关键字的镜像:

```
root@docker-test:/home/roach# docker search --filter=is-official=true nginx
NAME      DESCRIPTION                STARS     OFFICIAL   AUTOMATED
nginx     Official build of Nginx.   16124     [OK]       
```

(TODO:也就是说`--filter`中的选项不是随便写的?)

例:搜索收藏数超过4的,带有`tensorflow`的镜像:

```
root@docker-test:/home/roach# docker search --filter=stars=4 tensorflow
NAME                                       DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
tensorflow/tensorflow                      Official Docker images for the machine learn…   1983                 
jupyter/tensorflow-notebook                Jupyter Notebook Scientific Python Stack w/ …   287                  
tensorflow/serving                         Official images for TensorFlow Serving (http…   121                  
rocm/tensorflow                            Tensorflow with ROCm backend support            64                   
xblaster/tensorflow-jupyter                Dockerized Jupyter with tensorflow              56                   [OK]
floydhub/tensorflow                        tensorflow                                      29                   [OK]
opensciencegrid/tensorflow-gpu             TensorFlow GPU set up for OSG                   12                   
emacski/tensorflow-serving                 Project images from https://github.com/emacs…   11                   
tensorflow/tfx                                                                             6                    
tokunagaken/tensorflow-keras-jupyter-py3   TensorFlow-gpu 1.13.1 Keras 2.2.4 python 3.5…   5                    
tensorflow/tf_grpc_test_server             Testing server for GRPC-based distributed ru…   4                    
rocm/tensorflow-autobuilds                 The repo for building latest tensorflow dock…   4    
```

### 3.4 删除和清理镜像

##### 1. 使用标签删除镜像

`docker rmi`,`docker image rm`:删除镜像

- `-f`,`-force`:强制删除镜像,即使有容器依赖该镜像的状态下也会删除
- `--no-prune`:不清除不带标签的父镜像(TODO:父镜像是啥?)

例:删除`myubuntu:latest`

```
root@docker-test:/home/roach# docker rmi myubuntu:latest
Untagged: myubuntu:latest
root@docker-test:/home/roach# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

可以看到,`ubuntu: latest`没有受到影响.当同一个镜像有多个标签时,`docker rmi`命令只是删除了该镜像多个标签中指定的标签,并不影响镜像文件.

**当镜像只剩下1个标签时,`docker rmi`命令就会删除这个镜像的所有文件层.**

```
root@docker-test:/home/roach# docker pull busybox:latest
latest: Pulling from library/busybox
5cc84ad355aa: Pull complete 
Digest: sha256:5acba83a746c7608ed544dc1533b87c737a0b0fb730301639a0179f9344b1678
Status: Downloaded newer image for busybox:latest
docker.io/library/busybox:latest
root@docker-test:/home/roach# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
busybox      latest    beae173ccac6   13 days ago    1.24MB
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
root@docker-test:/home/roach# docker rmi busybox:latest
Untagged: busybox:latest
Untagged: busybox@sha256:5acba83a746c7608ed544dc1533b87c737a0b0fb730301639a0179f9344b1678
Deleted: sha256:beae173ccac6ad749f76713cf4440fe3d21d1043fe616dfbe30775815d1d0f6a
Deleted: sha256:01fd6df81c8ec7dd24bbbd72342671f41813f992999a3471b9d9cbc44ad88374
root@docker-test:/home/roach# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       latest    ba6acccedd29   2 months ago   72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago   63.1MB
```

##### 2. 使用镜像ID删除镜像

`docker rmi`命令后边也可以跟`IMAGE ID`.当使用镜像ID时,会先尝试删除所有指向该镜像的标签,再删除镜像文件本身.

**当有基于该镜像创建的容器存在时,镜像文件默认无法被删除.**

例:先使用`ubuntu:18.04`创建一个容器,再试图删除该镜像.

- 创建容器并输出内容

```
root@docker-test:/home/roach# docker run ubuntu:18.04 echo "fuck world"
fuck world
```

- `docker ps -a`:查看本机上存在的所有容器

```
root@docker-test:/home/roach# docker ps -a
CONTAINER ID   IMAGE          COMMAND               CREATED          STATUS                      PORTS     NAMES
8c4f9ddbb3ba   ubuntu:18.04   "echo 'fuck world'"   55 seconds ago   Exited (0) 55 seconds ago             elastic_turing
016329fae3cd   ubuntu:18.04   "bash"                3 hours ago      Exited (0) 3 hours ago                charming_ishizaka
```

可以看到,后台存在一个处于退出状态的容器,这个容器是基于`ubuntu:18.04`镜像创建的

- 试图删除该镜像

```
root@docker-test:/home/roach# docker rmi ubuntu:18.04
Error response from daemon: conflict: unable to remove repository reference "ubuntu:18.04" (must force) - container 8c4f9ddbb3ba is using its referenced image 5a214d77f5d7
```

可以使用`-f`参数来强行删除,但十分不建议.正确的做法是:**先删除依赖该镜像的所有容器,再删除镜像**.

- 删除容器`8c4f9ddbb3ba`和`016329fae3cd `(`docker rm`:删除容器)

```
root@docker-test:/home/roach# docker rm 8c4f9ddbb3ba
8c4f9ddbb3ba
root@docker-test:/home/roach# docker rm 016329fae3cd
016329fae3cd
```

- 使用`IMAGE ID`删除镜像

```
root@docker-test:/home/roach# docker rmi 5a214d77f5d7
Untagged: ubuntu:18.04
Untagged: ubuntu@sha256:0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6
Deleted: sha256:5a214d77f5d747e6ed81632310baa6190301feeb875cf6bf9da560108fa09972
Deleted: sha256:824bf068fd3dc3ad967022f187d85250eb052f61fe158486b2df4e002f6f984e
```

##### 3. 清理镜像

`docker image prune`:清理遗留的临时镜像文件和没有被使用过的镜像.

`prune`子命令支持的选项:

- `-a`,`-all`:删除所有无用镜像,不光是临时镜像
- `-filter filter`:只清理符合给定过滤器的镜像
- `-f`,`-force`:强制删除镜像,不进行确认

```
root@docker-test:/home/roach# docker image prune
WARNING! This will remove all dangling images.
Are you sure you want to continue? [y/N] y
Total reclaimed space: 0B
```

### 3.5 创建镜像

创建镜像的方法有3种:

1. 基于已有镜像的容器创建
2. 基于本地模板导入
3. 基于Dockerfile创建

##### 1. 基于已有的容器创建

`docker [container] commit`:提交容器.命令格式为:`docker [container] commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`

主要选项:

- `-a`,`--author=""`:作者信息
- `-c`,`--change=[]`:提交时执行的Dockerfile执行,包括`CMD|ENTRYPOINT|ENV|EXPOSE|LABEL|ONBUILD|USER|VOLUME|WORKDIR`等
- `-m`,`--message=""`:提交信息
- `-p`,`--pause=true`:提交时暂停容器运行

演示如何使用该命令创建一个新镜像:

- step1. 启动一个镜像,并在该镜像中创建一个文件

```
root@docker-test:/home/roach# docker run -it ubuntu:18.04 /bin/bash
root@4898d4f64978:/# touch test
root@4898d4f64978:/# exit
exit
```

记住这个容器的ID:`4898d4f64978`

- step2. 提交镜像

此时该容器与原来的`ubuntu:18.04`相比已经发生了变化(多了个我们自己创建的文件).可使用`docker [container] commit`命令提交为一个新的镜像

```
root@docker-test:/home/roach# docker commit -m "Add a test file" -a "Roach" 4898d4f64978 test:0.1
sha256:3eb3fadc7fa2b088f2eb2cc2048d8064e6fc2b26c00c2a48a13c867bf5c094c8
```

(TODO:这里`[container]`又是干啥的?)

- step3. 查看结果

```
root@docker-test:/home/roach# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
test         0.1       3eb3fadc7fa2   2 seconds ago   63.1MB
ubuntu       latest    ba6acccedd29   2 months ago    72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago    63.1MB
```

##### 2. 基于本地模板导入

`docker [container] import`:直接从一个OS模板文件中导入一个镜像.命令格式为:`docker [image] import [OPTIONS] file|URL|-[REPOSITORY[:TAG]]`

[模板下载地址](http://openvz.org/Download/templates/precreated)

此处我下载了一个ubuntu16.04

```
root@docker-test:/# cat /home/roach/ubuntu-16.04-x86_64.tar.gz |docker import - ubuntu:16.04
sha256:5662ae80aecded268eb5f38e01a1626aaad58a9b15d50a8b525a40e99dccedd3
```

查看结果:

```
root@docker-test:/# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
ubuntu       16.04     5662ae80aecd   5 seconds ago    505MB
test         0.1       3eb3fadc7fa2   52 minutes ago   63.1MB
ubuntu       latest    ba6acccedd29   2 months ago     72.8MB
ubuntu       18.04     5a214d77f5d7   3 months ago     63.1MB
```

##### 3. 基于Dockerfile创建

基于Dockerfile创建是最常见的方式.**Dockerfile是一个文本文件,利用给定的指令描述基于某个父镜像创建新镜像的过程.**

例:基于`debian:stretch-slim`镜像安装Python3环境,构成一个新的`python:3`镜像

```
FROM debian:stretch-slim

LABEL version="1.0"

RUN apt-get update && \
	apt-get install -y python3 && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*
```

Dockerfile内容:

```
root@docker-test:~# cat ./Dockerfile 
FROM debian:stretch-slim

LABEL version="1.0"

RUN apt-get update && \
	apt-get install -y python3 && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*
```

构建过程:

```
root@docker-test:~# docker build -t python:3 .
Sending build context to Docker daemon  27.65kB
Step 1/3 : FROM debian:stretch-slim
stretch-slim: Pulling from library/debian
35b2232c987e: Pull complete 
Digest: sha256:5913f0038562c1964c62fc1a9fcfff3c7bb340e2f5dbf461610ab4f802368eee
Status: Downloaded newer image for debian:stretch-slim
 ---> 4b673a9c386b
Step 2/3 : LABEL version="1.0"
 ---> Running in dfa13c648070
Removing intermediate container dfa13c648070
 ---> c71ca2555b43
Step 3/3 : RUN apt-get update && 	apt-get install -y python3 && 	apt-get clean && 	rm -rf /var/lib/apt/lists/*
 ---> Running in 2a7c66cee9da
Ign:1 http://deb.debian.org/debian stretch InRelease
Get:2 http://security.debian.org/debian-security stretch/updates InRelease [53.0 kB]
Get:3 http://deb.debian.org/debian stretch-updates InRelease [93.6 kB]
Get:4 http://deb.debian.org/debian stretch Release [118 kB]
Get:5 http://security.debian.org/debian-security stretch/updates/main amd64 Packages [763 kB]
Get:6 http://deb.debian.org/debian stretch Release.gpg [3177 B]
Get:7 http://deb.debian.org/debian stretch/main amd64 Packages [7080 kB]
Fetched 8110 kB in 3s (2585 kB/s)
Reading package lists...
Reading package lists...
Building dependency tree...
Reading state information...
The following additional packages will be installed:
  bzip2 dh-python file libexpat1 libmagic-mgc libmagic1 libmpdec2
  libpython3-stdlib libpython3.5-minimal libpython3.5-stdlib libreadline7
  libsqlite3-0 libssl1.1 mime-support python3-minimal python3.5
  python3.5-minimal readline-common xz-utils
Suggested packages:
  bzip2-doc libdpkg-perl python3-doc python3-tk python3-venv python3.5-venv
  python3.5-doc binutils binfmt-support readline-doc
The following NEW packages will be installed:
  bzip2 dh-python file libexpat1 libmagic-mgc libmagic1 libmpdec2
  libpython3-stdlib libpython3.5-minimal libpython3.5-stdlib libreadline7
  libsqlite3-0 libssl1.1 mime-support python3 python3-minimal python3.5
  python3.5-minimal readline-common xz-utils
0 upgraded, 20 newly installed, 0 to remove and 0 not upgraded.
Need to get 7897 kB of archives.
After this operation, 36.7 MB of additional disk space will be used.
Get:1 http://deb.debian.org/debian stretch/main amd64 libexpat1 amd64 2.2.0-2+deb9u3 [83.7 kB]
Get:2 http://security.debian.org/debian-security stretch/updates/main amd64 libssl1.1 amd64 1.1.0l-1~deb9u4 [1359 kB]
Get:3 http://deb.debian.org/debian stretch/main amd64 python3-minimal amd64 3.5.3-1 [35.3 kB]
Get:4 http://deb.debian.org/debian stretch/main amd64 mime-support all 3.60 [36.7 kB]
Get:5 http://deb.debian.org/debian stretch/main amd64 libmpdec2 amd64 2.4.2-1 [85.2 kB]
Get:6 http://deb.debian.org/debian stretch/main amd64 readline-common all 7.0-3 [70.4 kB]
Get:7 http://deb.debian.org/debian stretch/main amd64 libreadline7 amd64 7.0-3 [151 kB]
Get:8 http://deb.debian.org/debian stretch/main amd64 libpython3-stdlib amd64 3.5.3-1 [18.6 kB]
Get:9 http://deb.debian.org/debian stretch/main amd64 dh-python all 2.20170125 [86.8 kB]
Get:10 http://deb.debian.org/debian stretch/main amd64 python3 amd64 3.5.3-1 [21.6 kB]
Get:11 http://deb.debian.org/debian stretch/main amd64 bzip2 amd64 1.0.6-8.1 [47.5 kB]
Get:12 http://deb.debian.org/debian stretch/main amd64 libmagic-mgc amd64 1:5.30-1+deb9u3 [222 kB]
Get:13 http://deb.debian.org/debian stretch/main amd64 libmagic1 amd64 1:5.30-1+deb9u3 [111 kB]
Get:14 http://deb.debian.org/debian stretch/main amd64 file amd64 1:5.30-1+deb9u3 [64.2 kB]
Get:15 http://deb.debian.org/debian stretch/main amd64 xz-utils amd64 5.2.2-1.2+b1 [266 kB]
Get:16 http://security.debian.org/debian-security stretch/updates/main amd64 libpython3.5-minimal amd64 3.5.3-1+deb9u5 [574 kB]
Get:17 http://security.debian.org/debian-security stretch/updates/main amd64 python3.5-minimal amd64 3.5.3-1+deb9u5 [1692 kB]
Get:18 http://security.debian.org/debian-security stretch/updates/main amd64 libsqlite3-0 amd64 3.16.2-5+deb9u3 [574 kB]
Get:19 http://security.debian.org/debian-security stretch/updates/main amd64 libpython3.5-stdlib amd64 3.5.3-1+deb9u5 [2167 kB]
Get:20 http://security.debian.org/debian-security stretch/updates/main amd64 python3.5 amd64 3.5.3-1+deb9u5 [231 kB]
debconf: delaying package configuration, since apt-utils is not installed
Fetched 7897 kB in 3s (2479 kB/s)
Selecting previously unselected package libssl1.1:amd64.
(Reading database ... 6319 files and directories currently installed.)
Preparing to unpack .../00-libssl1.1_1.1.0l-1~deb9u4_amd64.deb ...
Unpacking libssl1.1:amd64 (1.1.0l-1~deb9u4) ...
Selecting previously unselected package libpython3.5-minimal:amd64.
Preparing to unpack .../01-libpython3.5-minimal_3.5.3-1+deb9u5_amd64.deb ...
Unpacking libpython3.5-minimal:amd64 (3.5.3-1+deb9u5) ...
Selecting previously unselected package libexpat1:amd64.
Preparing to unpack .../02-libexpat1_2.2.0-2+deb9u3_amd64.deb ...
Unpacking libexpat1:amd64 (2.2.0-2+deb9u3) ...
Selecting previously unselected package python3.5-minimal.
Preparing to unpack .../03-python3.5-minimal_3.5.3-1+deb9u5_amd64.deb ...
Unpacking python3.5-minimal (3.5.3-1+deb9u5) ...
Selecting previously unselected package python3-minimal.
Preparing to unpack .../04-python3-minimal_3.5.3-1_amd64.deb ...
Unpacking python3-minimal (3.5.3-1) ...
Selecting previously unselected package mime-support.
Preparing to unpack .../05-mime-support_3.60_all.deb ...
Unpacking mime-support (3.60) ...
Selecting previously unselected package libmpdec2:amd64.
Preparing to unpack .../06-libmpdec2_2.4.2-1_amd64.deb ...
Unpacking libmpdec2:amd64 (2.4.2-1) ...
Selecting previously unselected package readline-common.
Preparing to unpack .../07-readline-common_7.0-3_all.deb ...
Unpacking readline-common (7.0-3) ...
Selecting previously unselected package libreadline7:amd64.
Preparing to unpack .../08-libreadline7_7.0-3_amd64.deb ...
Unpacking libreadline7:amd64 (7.0-3) ...
Selecting previously unselected package libsqlite3-0:amd64.
Preparing to unpack .../09-libsqlite3-0_3.16.2-5+deb9u3_amd64.deb ...
Unpacking libsqlite3-0:amd64 (3.16.2-5+deb9u3) ...
Selecting previously unselected package libpython3.5-stdlib:amd64.
Preparing to unpack .../10-libpython3.5-stdlib_3.5.3-1+deb9u5_amd64.deb ...
Unpacking libpython3.5-stdlib:amd64 (3.5.3-1+deb9u5) ...
Selecting previously unselected package python3.5.
Preparing to unpack .../11-python3.5_3.5.3-1+deb9u5_amd64.deb ...
Unpacking python3.5 (3.5.3-1+deb9u5) ...
Selecting previously unselected package libpython3-stdlib:amd64.
Preparing to unpack .../12-libpython3-stdlib_3.5.3-1_amd64.deb ...
Unpacking libpython3-stdlib:amd64 (3.5.3-1) ...
Selecting previously unselected package dh-python.
Preparing to unpack .../13-dh-python_2.20170125_all.deb ...
Unpacking dh-python (2.20170125) ...
Setting up libssl1.1:amd64 (1.1.0l-1~deb9u4) ...
debconf: unable to initialize frontend: Dialog
debconf: (TERM is not set, so the dialog frontend is not usable.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up libpython3.5-minimal:amd64 (3.5.3-1+deb9u5) ...
Setting up libexpat1:amd64 (2.2.0-2+deb9u3) ...
Setting up python3.5-minimal (3.5.3-1+deb9u5) ...
Setting up python3-minimal (3.5.3-1) ...
Selecting previously unselected package python3.
(Reading database ... 7312 files and directories currently installed.)
Preparing to unpack .../0-python3_3.5.3-1_amd64.deb ...
Unpacking python3 (3.5.3-1) ...
Selecting previously unselected package bzip2.
Preparing to unpack .../1-bzip2_1.0.6-8.1_amd64.deb ...
Unpacking bzip2 (1.0.6-8.1) ...
Selecting previously unselected package libmagic-mgc.
Preparing to unpack .../2-libmagic-mgc_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking libmagic-mgc (1:5.30-1+deb9u3) ...
Selecting previously unselected package libmagic1:amd64.
Preparing to unpack .../3-libmagic1_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking libmagic1:amd64 (1:5.30-1+deb9u3) ...
Selecting previously unselected package file.
Preparing to unpack .../4-file_1%3a5.30-1+deb9u3_amd64.deb ...
Unpacking file (1:5.30-1+deb9u3) ...
Selecting previously unselected package xz-utils.
Preparing to unpack .../5-xz-utils_5.2.2-1.2+b1_amd64.deb ...
Unpacking xz-utils (5.2.2-1.2+b1) ...
Setting up readline-common (7.0-3) ...
Setting up mime-support (3.60) ...
Setting up libreadline7:amd64 (7.0-3) ...
Setting up libmagic-mgc (1:5.30-1+deb9u3) ...
Setting up bzip2 (1.0.6-8.1) ...
Setting up libmagic1:amd64 (1:5.30-1+deb9u3) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Setting up xz-utils (5.2.2-1.2+b1) ...
update-alternatives: using /usr/bin/xz to provide /usr/bin/lzma (lzma) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/lzma.1.gz because associated file /usr/share/man/man1/xz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/unlzma.1.gz because associated file /usr/share/man/man1/unxz.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcat.1.gz because associated file /usr/share/man/man1/xzcat.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzmore.1.gz because associated file /usr/share/man/man1/xzmore.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzless.1.gz because associated file /usr/share/man/man1/xzless.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzdiff.1.gz because associated file /usr/share/man/man1/xzdiff.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzcmp.1.gz because associated file /usr/share/man/man1/xzcmp.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzgrep.1.gz because associated file /usr/share/man/man1/xzgrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzegrep.1.gz because associated file /usr/share/man/man1/xzegrep.1.gz (of link group lzma) doesn't exist
update-alternatives: warning: skip creation of /usr/share/man/man1/lzfgrep.1.gz because associated file /usr/share/man/man1/xzfgrep.1.gz (of link group lzma) doesn't exist
Setting up libsqlite3-0:amd64 (3.16.2-5+deb9u3) ...
Setting up libmpdec2:amd64 (2.4.2-1) ...
Setting up libpython3.5-stdlib:amd64 (3.5.3-1+deb9u5) ...
Setting up file (1:5.30-1+deb9u3) ...
Setting up python3.5 (3.5.3-1+deb9u5) ...
Setting up libpython3-stdlib:amd64 (3.5.3-1) ...
Setting up dh-python (2.20170125) ...
Setting up python3 (3.5.3-1) ...
running python rtupdate hooks for python3.5...
running python post-rtupdate hooks for python3.5...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Removing intermediate container 2a7c66cee9da
 ---> 94e291cebd8b
Successfully built 94e291cebd8b
Successfully tagged python:3
```

查看结果:

```
root@docker-test:~# docker images
REPOSITORY   TAG            IMAGE ID       CREATED         SIZE
python       3              94e291cebd8b   5 seconds ago   95.2MB
ubuntu       16.04          5662ae80aecd   4 hours ago     505MB
test         0.1            3eb3fadc7fa2   5 hours ago     63.1MB
debian       stretch-slim   4b673a9c386b   3 weeks ago     55.3MB
ubuntu       latest         ba6acccedd29   2 months ago    72.8MB
ubuntu       18.04          5a214d77f5d7   3 months ago    63.1MB
```

### 3.6 存出和载入镜像

##### 1. 存出镜像

`docker [image] save`:导出镜像到本地文件.

例:将镜像`ubuntu:18.04`导出为`ubuntu_18.04.tar`

```
root@docker-test:~# docker save -o ubuntu_18.04.tar ubuntu:18.04
root@docker-test:~# ls
Dockerfile  snap  ubuntu_18.04.tar
```

之后,就可以通过复制该文件,将镜像分享给他人

##### 2. 载入镜像

`docker [image] load`:将导出的`tar`文件再导入到本地镜像库.

例:将刚才的`ubuntu_18.04.tar`导入为镜像`ubuntu:18.04`

- step1. 查看目前镜像情况

```
root@docker-test:~# docker images
REPOSITORY   TAG            IMAGE ID       CREATED         SIZE
python       3              94e291cebd8b   4 minutes ago   95.2MB
ubuntu       16.04          5662ae80aecd   4 hours ago     505MB
test         0.1            3eb3fadc7fa2   5 hours ago     63.1MB
debian       stretch-slim   4b673a9c386b   3 weeks ago     55.3MB
ubuntu       latest         ba6acccedd29   2 months ago    72.8MB
ubuntu       18.04          5a214d77f5d7   3 months ago    63.1MB
```

记住`ubuntu:18.04`的元信息

- step2. 删除`ubuntu:18.04`

```
root@docker-test:~# docker rmi ubuntu:18.04
Untagged: ubuntu:18.04
Untagged: ubuntu@sha256:0fedbd5bd9fb72089c7bbca476949e10593cebed9b1fb9edf5b79dbbacddd7d6
root@docker-test:~# docker images
REPOSITORY   TAG            IMAGE ID       CREATED         SIZE
python       3              94e291cebd8b   6 minutes ago   95.2MB
ubuntu       16.04          5662ae80aecd   4 hours ago     505MB
test         0.1            3eb3fadc7fa2   5 hours ago     63.1MB
debian       stretch-slim   4b673a9c386b   3 weeks ago     55.3MB
ubuntu       latest         ba6acccedd29   2 months ago    72.8MB
```

- step3. 通过`ubuntu_18.04.tar`导入镜像

```
root@docker-test:~# docker load -i ubuntu_18.04.tar 
Loaded image: ubuntu:18.04
```

- step4. 查看结果

```
root@docker-test:~# docker images
REPOSITORY   TAG            IMAGE ID       CREATED          SIZE
python       3              94e291cebd8b   10 minutes ago   95.2MB
ubuntu       16.04          5662ae80aecd   4 hours ago      505MB
test         0.1            3eb3fadc7fa2   5 hours ago      63.1MB
debian       stretch-slim   4b673a9c386b   3 weeks ago      55.3MB
ubuntu       latest         ba6acccedd29   2 months ago     72.8MB
ubuntu       18.04          5a214d77f5d7   3 months ago     63.1MB
```

可以看到,元信息是完全相同的.

### 3.7 上传镜像

`docker push`:上传镜像.默认上传到Docker Hub官方仓库

例:创建一个镜像并上传至仓库

- step1. 创建镜像

```
root@docker-test:~# docker run -it ubuntu:18.04 /bin/bash
root@2e67bf264388:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@2e67bf264388:/# touch test
root@2e67bf264388:/# exit
exit
root@docker-test:~# docker commit -m "Add a test file" -a "Roach" 2e67bf264388 test:0.1
sha256:42ed017c869e6a274dd08d1aaac50ff8a83561882c03e0eba37d9df075bbe344
root@docker-test:~# docker images
REPOSITORY   TAG            IMAGE ID       CREATED          SIZE
test         0.1            42ed017c869e   53 seconds ago   63.1MB
python       3              94e291cebd8b   20 minutes ago   95.2MB
ubuntu       16.04          5662ae80aecd   4 hours ago      505MB
debian       stretch-slim   4b673a9c386b   3 weeks ago      55.3MB
ubuntu       latest         ba6acccedd29   2 months ago     72.8MB
ubuntu       18.04          5a214d77f5d7   3 months ago     63.1MB
```

- step2. 添加标签(`docker tag 已有镜像 新标签信息`)

注意:此处的仓库名要和远端仓库名一致(我第一次干这个,貌似是必须一致?)

```
root@docker-test:~# docker tag test:0.1 40486453/test_repo
root@docker-test:~# docker images
REPOSITORY           TAG            IMAGE ID       CREATED          SIZE
40486453/test_repo   latest         42ed017c869e   19 minutes ago   63.1MB
test                 0.1            42ed017c869e   19 minutes ago   63.1MB
python               3              94e291cebd8b   38 minutes ago   95.2MB
ubuntu               16.04          5662ae80aecd   4 hours ago      505MB
debian               stretch-slim   4b673a9c386b   3 weeks ago      55.3MB
ubuntu               latest         ba6acccedd29   2 months ago     72.8MB
ubuntu               18.04          5a214d77f5d7   3 months ago     63.1MB
```

- step3. 登录docker hub(`docker login`)

```
root@docker-test:~# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: 40486453
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

- step4. 推送镜像

```
root@docker-test:~# docker push 40486453/test_repo
Using default tag: latest
The push refers to repository [docker.io/40486453/test_repo]
6960796724cc: Pushed 
824bf068fd3d: Mounted from library/ubuntu 
latest: digest: sha256:2a31b20db0f11a5bee54278e1f6cefc9239007d7deea7fa2f1b9397a17925954 size: 736
```

[仓库](https://hub.docker.com/r/40486453/test_repo/tags)中此时就有这个镜像了.

TODO:此处必须先登录,再推送.可能是我的Docker和书中的版本差异问题,也有可能是我哪里没配置对,导致推送时不提示登录信息,必须手动登录.