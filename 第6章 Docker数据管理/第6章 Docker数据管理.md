# 第6章 Docker数据管理

在生产环境中使用Docker,往往要对数据进行持久化,或者要在多个容器之间进行数据共享.这必然涉及到容器的数据管理操作.

容器中的数据管理主要有2种方式:

- 数据卷(Data Volumes):容器内数据直接映射到本地主机环境
- 数据卷容器(Data Volumes Containers):使用特定容器维护数据卷

## 6.1 数据卷

Data Volumes:数据卷.是一个可供容器使用的特殊目录,它将宿主机OS中的目录直接映射进容器,类似于Linux中的`mount`.

数据卷可以提供很多有用的特性:

- 数据卷可以在容器之间共享和重用,容器间传递数据变的高效和方便
- 对数据卷内数据的修改会立刻生效,无论是容器内的操作还是宿主机上的操作
- 对数据卷的更新不会影响镜像,解耦了应用和数据
- 卷会一直存在,直到没有容器使用它时,可以安全的卸载

##### 1. 创建数据卷

`docker volume create`:创建数据卷

支持的选项:

- `-d`:指定数据卷的名字

例:

- step1. 创建数据卷

```
root@docker-test:/home/roach# docker volume create -d local test
test
```

- step2. 查看结果

```
root@docker-test:/home/roach# tree /var/lib/docker/volumes/
/var/lib/docker/volumes/
├── 3a40371d204fffa221a45bf1bae74ce215a8146944c336570362e38bd0b1d98a
│   └── _data
├── backingFsBlockDev
├── metadata.db
└── test
    └── _data

4 directories, 2 files
```

可以看到,docker默认将数据卷保存在`/var/lib/docker/volumes/`下(其中的`test/`目录就是我们例子中刚刚创建的数据卷).

##### 2. 查看数据卷

`docker volume inspect`:查看数据卷的详细信息

例:查看数据卷`test`的详细信息

```
root@docker-test:/home/roach# docker volume inspect test
[
    {
        "CreatedAt": "2022-01-15T05:13:20Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
```

##### 3. 列出已有数据卷

`docker volume ls`:列出已有数据卷

```
root@docker-test:/home/roach# docker volume ls
DRIVER    VOLUME NAME
local     test
```

##### 4. 清理无用数据卷

`docker volume prune`:清理无用数据卷

```
root@docker-test:/home/roach# docker volume prune
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
3a40371d204fffa221a45bf1bae74ce215a8146944c336570362e38bd0b1d98a
test

Total reclaimed space: 0B
root@docker-test:/home/roach# tree /var/lib/docker/volumes/
/var/lib/docker/volumes/
├── backingFsBlockDev
└── metadata.db

0 directories, 2 files
```

可以看到,由于刚刚创建的数据卷`test`没有被任何容器使用,因此被清理了.

##### 5. 删除数据卷

`docker volume rm`:删除数据卷

- step1. 创建数据卷

```
root@docker-test:/home/roach# docker volume create -d local test
test
root@docker-test:/home/roach# tree -L 1 /var/lib/docker/volumes/
/var/lib/docker/volumes/
├── backingFsBlockDev
├── metadata.db
└── test

1 directory, 2 files
```

- step2. 删除数据卷

```
root@docker-test:/home/roach# docker volume rm test
test
root@docker-test:/home/roach# tree -L 1 /var/lib/docker/volumes/
/var/lib/docker/volumes/
├── backingFsBlockDev
└── metadata.db

0 directories, 2 files
```

#####  6. 绑定数据卷

在使用`docker run`命令创建并运行一个镜像时,可使用`-mount`选项来使用数据卷.

`--mount`选项支持3种类型的数据卷:

- volume:普通数据卷,映射到宿主机`/var/lib/docker/volumes`下
- bind:绑定数据卷,映射到主机指定目录下
- tmpfs:临时数据卷,只存在于内存中

例:使用`training/webapp`镜像创建一个Web容器,并创建一个数据卷挂载到容器的`/opt/webapp`目录

- step1. 在宿主机上创建目录

```
root@docker-test:/home/roach# mkdir /webapp
```

- step2. 创建并运行容器的同时:
1. 指定挂载数据卷的方式为`bind`
2. 通过NAT机制将容器标记暴露的端口自动映射到本地主机的临时端口
3. 将宿主机的`/webapp`中的数据卷映射到容器的`/opt/webapp`上
4. 命名该容器为`web`
5. 后台运行该容器

```
root@docker-test:/home/roach# docker run -d -P --name web --mount type=bind,source=/webapp,destination=/home training/webapp python /opt/webapp/app.py
225fa7a93709362ed4af2bb592fd3ece76509eb1b6c45596bc382fa8b9f29021
```

参数说明:

- `-P`:通过NAT机制将容器标记暴露的端口自动映射到本地主机的临时端口
- `source`:指定宿主机上数据卷的路径,该路径必须是绝对路径,且该路径必须存在
- `destination`:指定数据卷映射到容器中的路径,该路径可以是相对路径(TODO:相对家目录的路径吗?)

注意:`--mount`绑定到容器的非空目录下,会隐藏容器目录下的现有内容

- step3. 查看结果

```
root@docker-test:/home/roach# docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS        PORTS                                         NAMES
225fa7a93709   training/webapp   "python /opt/webapp/…"   2 seconds ago   Up 1 second   0.0.0.0:49167->5000/tcp, :::49167->5000/tcp   web
```

此时访问`192.168.0.152:49617`是可以看到Hello world的

这个命令等同于:`docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py`

```
root@docker-test:/home/roach# docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py
27a68f758f322c574d805e85216512f725d8cff8dc668b9c421b9df318817794
```

也可以指定容器对数据卷的读写权限.

例:创建并运行一个`ubuntu:18.04`的容器,同时:

1. 指定挂载数据卷的方式为`bind`
2. 将宿主机的`/ubuntu_container_volume`中的数据卷映射到容器的`/opt/`上
3. 指定容器对该数据卷只读

- step1. 创建数据卷目录

```
root@docker-test:/home/roach# mkdir /ubuntu_container_volume
```

- step2. 创建容器并绑定数据卷

```
root@docker-test:/home/roach# docker run -itd --mount type=bind,source=/ubuntu_container_volume,destination=/opt,readonly ubuntu:18.04 bash
a8a860c4c24182f3476581f77b5f9316ca768d3645504637886564238f3c40f0
```

- step3. 查看结果

```
root@docker-test:/home/roach# docker ps
CONTAINER ID   IMAGE          COMMAND   CREATED          STATUS          PORTS     NAMES
a8a860c4c241   ubuntu:18.04   "bash"    13 seconds ago   Up 12 seconds             vibrant_jones
```

这个命令等同于:`docker run -d -v /ubuntu_container_volume:/opt:ro ubuntu:18:04 bash`

如果直接挂载一个文件到容器,那么在使用文件编辑工具(包括`vi`、`sed`)时,可能会造成文件`inode`的改变.从Docker 1.1.0开始,这回导致报错信息.所以推荐的方式是直接挂载文件所在的目录到容器内.

## 6.2 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据,最简单的方式是使用数据卷容器.数据卷容器也是一个容器,但它存在的目的是专门提供数据卷给其他容器挂载.

- step1. 创建一个数据卷容器`dbdata`,并在其中创建一个数据卷挂载到`/dbdata`

```
root@docker-test:/home/roach# mkdir /dbdata
root@docker-test:/home/roach# docker run -it -v /dbdata --name dbdata ubuntu
```

- step2. 在容器`dbdata`内查看目录

```
root@7f30c2890a3b:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

- step3. 退出容器`dbdata`

```
root@7f30c2890a3b:/# exit
exit
```

注:使用`--volumes-from`参数挂载数据卷时,容器自身不需要保持在运行状态

- step4. 再创建2个容器,使用`--volumes-from`参数挂载容器`dbdata`中的数据卷

```
root@docker-test:/home/roach# docker run -it --volumes-from dbdata --name db1 ubuntu
root@f13e15739e8e:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@docker-test:/home/roach# docker run -it --volumes-from dbdata --name db2 ubuntu
root@01da30b752f2:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

`--volumes-from`:从其他容器挂载数据卷时,指定其他容器

- step5. 创建容器`db3`,从`db1`上挂载数据卷

```
root@docker-test:/home/roach# docker run -it --volumes-from db1 --name db3 ubuntu
root@6ec45b0f003d:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

`--volumes-from`可以从多个容器中挂载数据卷.

例:创建容器`db4`,从容器`db1`和`db2`中挂载数据卷

```
root@docker-test:/home/roach# docker run -it --volumes-from db1 --volumes-from db2 --name db4 ubuntu
root@38823d2b729e:/# ls
bin  boot  dbdata  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

如果删除了挂载的容器(本例中就是容器`dbdata`),数据卷并不会被自动删除.如果要删除一个数据卷,必须在删除最后一个挂载着该数据卷的容器时,显示使用`docker rm -v`命令来指定同时删除关联的容器.

## 6.3 利用数据卷容器来迁移数据

##### 1. 备份

- step1. 创建一个容器并备份数据卷内的数据

```
root@docker-test:/home/roach# docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
/dbdata/
tar: Removing leading `/' from member names
```

- step2. 查看结果

```
root@docker-test:/home/roach# ls
backup.tar  crazy_chaplygin_echo_fuck.tar  Dockerfile  test_for_bash.tar  ubuntu-16.04-x86_64.tar.gz
```

解释一下step1.的命令.

- 使用`ubuntu`镜像创建了一个容器,命名该容器为`worker`
- 挂载容器`dbdata`的数据卷
- 挂载宿主机当前路径到容器的`/backup`下
- 在宿主机内,将`/dbdata`备份为`/backup/backup.tar`.而实际上容器`worker`的`/backup`目录是宿主机执行step1那条命令时所在的路径.就完成了备份

##### 2. 恢复

- step1. 创建一个容器`dbdata2`,挂载宿主机的`/dbdata`数据卷

```
root@docker-test:/home/roach# docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
root@docker-test:/home/roach# docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS                      PORTS                                         NAMES
b1f1557ce24f   ubuntu            "/bin/bash"              10 seconds ago   Exited (0) 9 seconds ago                                                  dbdata2
225fa7a93709   training/webapp   "python /opt/webapp/…"   10 minutes ago   Up 10 minutes               0.0.0.0:49167->5000/tcp, :::49167->5000/tcp   web
4daaf29f7b65   ubuntu            "tar cvf /backup/bac…"   37 minutes ago   Exited (0) 37 minutes ago                                                 worker
35ba1d37743e   ubuntu            "bash"                   41 minutes ago   Exited (0) 41 minutes ago                                                 dbdata
774c7deda006   registry:2        "/entrypoint.sh /etc…"   6 hours ago      Exited (2) 4 hours ago                                                    vigilant_elbakyan
d237a4444f40   a70035d2ec41      "/bin/bash"              24 hours ago     Created                                                                   reverent_dewdney
```

- step2. 创建一个新的容器,挂载容器`dbdata2`的数据卷,并解压备份文件该容器挂载的数据卷中

```
root@docker-test:/home/roach# docker run --volumes-from dbdata2 -v $(pwd):/backup busybox tar xvf /backup/backup.tar
dbdata/
```