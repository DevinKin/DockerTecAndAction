# 第六章-数据管理

- 容器中管理数据主要有两种方式
  - 数据卷（Data Volumes）
  - 数据卷容器（Data Volume Dontainers）

## 数据卷

- 数据卷是一个可供容器使用的特殊目录，它可以绕过文件系统，特性如下
  - 数据卷可以在容器之间共享和重用
  - 对数据卷的修改会立马生效
  - 对数据卷的更新，不会影响镜像
  - 卷会一直存在，知道没有容器使用

### 在容器内创建一个数据卷

- 使用`docker run`命令时，使用`-v`标记可以在容器内创建一个数据卷。多次使用`-v`可以创建多个数据卷。

  - `-P`允许外部访问容器需要暴露的端口

  ```sh
  docker run -d -P --name web -v /webapp training/webapp python app.py
  ```

### 挂载一个主机目录作为数据卷

- 使用`-v`标记也可以指定挂载一个本地的已有目录到容器中作为数据卷

  - 加载主机的`/src/webapp`目录到容器的`/opt/webapp`目录

  ```sh
  docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp py
  thon app.py
  ```

- Docker挂载数据卷的默认权限是读写（rw），用户可以指定权限，下面修改为只读权限。

  ```sh
  docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp py
  thon app.py
  ```

## 数据卷容器

- 用户在容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。

- 创建一个数据卷容器dbdata

  ```sh
  docker run -it -v /dbdata --name dbdata ubuntu
  ```

- 在其他容器中使用`--volumes-from`来挂载dbdata容器中的数据卷。

  ```sh
  docker run -it --volumes-from dbdata --name db1 ubuntu
  docker run -it --volumes-from dbdata --name db2 ubuntu
  ```

- 验证dbdata目录内容是不是同步更新

- 使用`--volumes-from`参数所挂载的数据卷容器自身并不需要保持在运行状态。

- 多次使用`-volumes-from`参数来从多个容器挂载多个数据卷。还可以从其他已经挂载了容器卷的容器来挂载数据卷。

  ```sh
  docker run -d --name db3 --volumes-from db1 training/postgres
  ```

- 如果删除了挂载的容器（包括dbdata，db1和db2），数据卷并不会自动被删除。如果烟删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用`docker rm -v`来指定同时删除关联的容器。

## 利用数据卷容器迁移数据

- 可以利用数据卷容器对其中数据进行备份，恢复以实现数据的迁移。

### 备份

- 利用ubuntu镜像创建一个容器worker。
- 使用`--volumes-from`参数来让worker容器挂载dbdata容器的数据卷。
- 使用`-v ${pwd}:/backup`参数来挂载本地的当前目录到worker容器的/backup目录。
- worker容器启动后，使用`tar cvf /backup/backup.tar /dbdata`命令来将`/dbdata`下内容备份到容器的`/backup/backup.tar`，即宿主机当前目录下的`backup.tar`

```
docker run --volumes-from dbdata -v `pwd`:/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
```

### 恢复

- 首先创建一个带有数据卷的容器dbdata2

  ```sh
  docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
  ```

- 然后创建另一个新的容器，挂载dbdata2的容器，并使用untar解压备份文件到所挂载的容器卷中即可。

  ```sh
  docker run --volumes-from dbdata2 -v `pwd`:/backup --name busybox ubuntu tar xvf /backup/backup.tar
  ```