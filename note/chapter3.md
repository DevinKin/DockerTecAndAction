# 第三章-镜像

## 获取镜像

- 镜像是Docker运行容器的前提。

- 命令格式

  - 不显式指定TAG，则默认会选择`latest`标签。

  ```sh
  docker pull NAME[:TAG]
  ```

- 示例

  ```sh
  docker pull ubuntu
  ```

- 从默认的注册服务器`registry.hub.docker.com`中的ubuntu仓库来下载标记为`latest`的镜像

  ```sh
  docker pull registry.hub.docker.com/ubuntu:latest
  ```

- 从DockerPool社区的镜像源`dl.dockerpool.com`下载最新的Ubuntu镜像

  ```java
  docker pull dl.dockerpool.com:5000/ubuntu
  ```

- 利用镜像创建一个容器，在其中运行bash应用

  ```sh
  docker run -t -i ubuntu /bin/bash
  ```

## 查看镜像信息

- 列出本地主机中已有的镜像

  ```sh
  docker images
  ```

- `docker images`列出的字段信息为

  - REPOSITORY：来自哪个仓库
  - TAG：镜像的标签信息
  - IMAGE ID：镜像的ID号
  - CREATED：创建时间
  - SIZE：镜像大小

- 为本地镜像添加新的标签，id一致，指向同一个镜像文件。

  ```sh
  docker tag ubuntu:latest ubuntu:lat
  ```

- 查看镜像的详细信息

  ```sh
  docker inspect ImageId
  ```

- 获取镜像详细信息的其中一项内容

  ```sh
  docker inspect -f {{".Architecture"}} ImageId
  ```

## 搜寻镜像

- 使用`docker search`可以搜索远端仓库中共享的镜像，默认搜索Docker Hub官方仓库中的镜像

- 命令格式

  ```sh
  docker search TEAM [argList]
  ```

- argList包括

  - `--automated=false`：仅显示自动创建的镜像。
  - `--no-trunc=false`：输出信息不截断显示。
  - `-s,--stars=0`：指定仅显示评价为指定星级以上的镜像

- 示例

  ```sh
  docker search mysql
  ```

- 返回信息

  - NAME：镜像明
  - STARS：星级数
  - OFFICIAL：是否为官方创建
  - AUTOMATED：是否自动创建

- 默认输出的结果是按照星级评价进行排序的。

- AUTOMATED资源允许用户验证镜像的来源和内容。

## 删除镜像

- 命令格式

  - IMAGE可以为标签或ID。

  ```sh
  docker rmi IMAGE[IMAGE...]
  ```

- 当同一个镜像拥有多个标签的，`docker rmi`命令指示删除了该镜像多个标签中指定的标签而已，不影响镜像文件。

- 当镜像创建的容器存在时，镜像文件默认是无法被删除的。

- 查看本机上存在的所有容器

  ```sh
  docker ps -a
  ```

- 强制删除镜像，可能会造成遗留问题

  ```sh
  docker rmi -f ubuntu
  ```

- 先删除依赖该镜像的所有容器，再来删除镜像。

## 创建镜像

- 创建镜像的三种方法
  - 基于已有的镜像的容器创建。
  - 基于本地模板导入。
  - 基于Dockerfile创建。

### 基于已有镜像的容器创建

- 命令格式

  ```sh
  docker commit [OPTIONS] CONTAINERID [REPOSITORY[:TAG]]
  ```

- 命令选项

  - `-a，--author=""`作者信息
  - `-m，--message=""`提交信息
  - `-p，--pause=true`提交时暂停容器运行

- 示例

  ```sh
  docker commit -m "Added a new file" -a "Docker devinkin" e2834821d165 test
  ```

- 创建成功，会返回新创建的镜像的ID信息。

### 基于本地模板导入

```sh
cat centos-6-x86-minimal.tar.gz | docker import - centos:6.0
```

## 存出和载入镜像

### 存出镜像

```sh
docker save -o ubuntu18.04.tar ubuntu:latest
```

### 载入镜像

```sh
docker load < ubuntu18.04.tar
```

## 上传镜像

- 默认上传到官方仓库，需要登录。

- 命令格式

  ```sh
  docker push NAME[:TAG]
  ```

- 上传镜像前使用`docker tag`改名字，名字规范为`username/xxx`

- 示例

  ```sh
  docker tag test:latest devinkin/test:latest
  docker push devinkin/test
  ```