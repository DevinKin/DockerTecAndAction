# Docker-操作系统

## Busybox

- Busybox是一个集成了一百多个最常用Linux命令和工具的软件工具箱，它在单一的可执行文件中提供了精简的Unix工具集。

- 获取BusyBox

  ```shell
  docker search busybox
  docker pull busybox
  docker images
  ```

### 运行Busybox

- 启动Busybox容器

  ```shell
  docker run -it busybox
  ```

## Debian/Ubuntu

### 搜索Debian

```shell
docker search debian
```

### 搜索Ubuntu

- 搜索被收藏10次以上的镜像

  ```shell
  docker search -s 10 ubuntu
  ```

- 使用`-ti`参数启动的容器，更适合作为测试、学习使用。实际场景比较少使用该选项。

## CentOS/Fedora

### 搜索Centos

```shell
docker search -s 2 centos
```

### 搜索Fedora

```shell
docker search -s 2 fedora
```



