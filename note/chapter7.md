# 第7章-网络基础配置

## 端口映射实现访问容器

### 从外部访问容器应用

- 容器中运行一些网络应用，要让外部访问这些应用时，可以通过`-P`或`-p`参数来指定端口映射。

- 当使用`-P`标记时，Docker会随机映射一个49000~49900的端口至容器内部开放的网络端口。

  ```sh
  docker run -d -P training/webapp python app.py
  docker ps -l
  docker logs Container_Name
  ```

- `-p`（小写p）则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有

  - `ip:hostPort:containerPort`
  - `ip::containerPort`
  - `hostPort:containerPort`

### 映射所有接口地址

- 使用`hostPort:containerPort`格式将本地的5001端口映射到容器的5000端口。

  ```sh
  docker run -d -p 5001:5000 training/webapp python app.py
  ```

### 映射到指定地址的指定端口

- 使用`ip:hostPort:containerPort`格式指定映射使用一个特定地址。

  ```sh
  docker run -d -p 127.0.0.1:5002:5000 training/webapp python app.py
  ```

### 映射到指定地址的任意端口

- 使用`ip::containerPort`绑定localhost的任意端口到容器的5000端口，本地主机会自动分配一个端口。

  ```sh
  docker run -d -p 127.0.0.1::5000 training/webapp python app.py
  ```

- 可以使用udp标记来指定udp端口

  ```sh
  docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
  ```

### 查看映射端口配置

- 使用`docker port`查看当前映射的端口配置，也可以查看到绑定的地址。

  ```sh
  docker port Container_Name
  ```

- 容器有自己的内部网络和IP地址，可以使用`docker inspect ContainerID`获取所有的变量值。

## 容器互联实现容器间通信

- 容器的连接系统是除了端口映射外另一种可以与容器中应用进行交互的方式。它会在源和接收容器之间创建一个隧道，接收容器可以看到源容器指定的信息。

### 自定义容器命名

- 使用`--name`标记可以为容器自定义命名

  ```sh
  docker run -d -P --name web training/webapp python app.py
  ```

- 执行`docker run`时候添加`--rm`选项，则容器在终止后会立即删除。`--rm`和`-d`不能同时使用。

### 容器互联

- 使用`--link`选项可以让容器之间安全的进行交互。

- `--link`参数格式是`--link name:alias`

  - `name`是要链接的容器的名称。
  - `alias`是这个连接的别名。

  ```sh
  # 创建一个新的数据库容器
  docker run -d --name db training/postgres
  # 删除之前创建的web容器
  docker rm -f web
  # 创建一个新的web容器，并将它连接到db容器
  docker run -d -P --name web --link db:db training/webapp python app.py
  # 使用docker ps来查看容器的连接
  docker ps
  ```

- Docker通过两种方式公开容器的连接信息

  - 环境变量

    ```sh
    # 使用env命令来查看web容器的环境变量
    docker run --rm --name web2 --link db:db training/webapp env
    #PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    #HOSTNAME=e8f7e4e78f1d
    #DB_PORT=tcp://172.17.0.9:5432
    #DB_PORT_5432_TCP=tcp://172.17.0.9:5432
    #DB_PORT_5432_TCP_ADDR=172.17.0.9
    #DB_PORT_5432_TCP_PORT=5432
    #DB_PORT_5432_TCP_PROTO=tcp
    #DB_NAME=/web2/db
    #DB_ENV_PG_VERSION=9.3
    #HOME=/root
    ```

  - 更新`/etc/hosts`文件

    ```sh
    docker run -t -i --rm --link db:db training/webapp /bin/bash
    cat /etc/hosts
    ```