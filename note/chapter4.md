# 第四章-容器

## 创建容器

### 新建容器

- 命令格式

  ```sh
  docker create -it ubuntu:latest
  ```

- 使用`docker create`命令新建的容器处于停止状态，可以使用`docker start ImageId`命令启动它。

### 新建并启动容器

- 启动容器有两种方式
  - 基于镜像新建一个容器并启动
  - 将在终止状态的容器重新启动
- 命令`docker run`等价于先执行`docker create`再执行`docker start`
- 当利用`docker run`来创建并启动容器时，Docker在后台运行的标准操作包括
  - 检查本地是否存在指定镜像，不存在就从公有仓库下载。
  - 利用镜像创建并启动一个容器。
  - 分配一个文件系统，并在只读的镜像层外面挂在一层可读写层。
  - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去。
  - 从地址池配置一个IP地址给容器。
  - 执行用户指定的应用程序。
  - 执行完毕后容器被终止。
- 选项说明
  - `-t`选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上。
  - `-i`则让容器的标准输入保持打开。
  - `-d`让容器在后台以守护态形式运行。

### 守护态运行

- 用户可以通过添加`-d`选项让Docker容器以守护态（Daemonized）形式运行。

  ```sh
  docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
  docker ps
  docker logs ImageId
  ```

- `docker logs`命令获取容器的输出信息。

## 终止容器

- 命令格式

  ```sh
  docker stop [-t|--time[=10]] imageID
  ```

- `docker stop`首先向容器发送SIGTERM信号，等待一段时间后（默认为10秒），再发送SIGKILL信号终止容器

- 示例

  ```sh
  docker stop f7e684565c41
  ```

- `docker restart`命令会将一个运行态的容器终止，然后再启动它。

  ```sh
  docker restart f7e684565c41
  ```

## 进入容器

- `docker attach`命令
- `docker exec`命令
- `nsenter`工具

### attach命令

- 多个窗口同事attach到同一个容器的时候，所有窗口都会同步显示。当某个窗口命令阻塞时，其他窗口也无法执行操作了。

- 使用示例

  ```sh
  docker run -idt ubuntu
  docker attach 18696bf762ce
  ```

### exec命令

- 可以直接在容器内运行命令

- 进入到刚创建的容器，并启动一个bash

  ```sh
  docker exec -it 475130689c3a /bin/bash
  ```

## 删除容器

- 使用`docker rm`命令删除处于终止状态的容器，命令格式为

  ```sh
  docker rm [OPTIONS] CONTAINER [CONTAINER...]
  ```

- 支持的选项

  - `-f，--force=false`强行终止并删除一个运行中的容器。
  - `-l，--link=false`删除容器的连接，但保留容器。
  - `-v，--volumes=false`容器挂在的数据卷

- 使用示例

  ```sh
  docker run -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
  docker rm -f f593907d2c96
  ```

## 导入和导出容器

### 导出容器

- 导出容器是指导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态。

- 命令格式

  ```sh
  docker export CONTAINER
  ```

- 使用示例

  ```sh
  docker ps -a
  docker export 7375a4b5dcce > test_for_run.tar
  docker export f7e684565c41 > test_for_stop.tar
  ```

### 导入容器

- 可以使用`docker load`命令导入镜像存储文件到本地的镜像库。

- 可以使用`docker import`命令导入容器快照文件到本地的镜像库。

- 两者的区别

  - 容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态）
  - 镜像存储文件将保存完整记录，体积也要大，此外从容器快照文件导入时可以重新制定标签等元数据信息。

- 使用示例

  ```sh
  cat test_for_run.tar | docker import - test/ubuntu:v1.0
  ```

  