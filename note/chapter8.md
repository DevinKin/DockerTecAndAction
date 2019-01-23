# 使用Dockerfile创建镜像

## 基本结构

- Dockerfile由一行命令语句组成，并 支持#开头的注释行。
- Dockerfile分为四部分
  - 基础镜像信息
  - 维护者信息
  - 镜像操作指令
  - 容器启动时执行指令

- 示例

  ```dockerfile
  # This dockerfile uses the ubuntu image
  # VERSION 2 - EDITION 1
  # Author: devinkin
  # Command format: Instruction [arguments / command] ..
  
  # 第一行必须制定基于的基础镜像
  FROM ubuntu
  
  # 维护者信息
  MAINTAINER devinkin devinkinwork@163.com
  
  # 镜像的操作指令
  RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
  
  RUN apt-get update && apt-get install -y nginx
  
  RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
  
  
  # 容器启动时执行指令
  CMD /usr/sbin/nginx
  ```

- 在ubuntu父镜像基础上安装`inotify-tools`，`nginx`，`apache2`，`openssh-server`等软件，从而创建一个新的`Nginx`镜像。

  ```dockerfile
  # Nginx
  # 
  # VERSION 0.0.1
  
  FROM ubuntu
  MAINTAINER devinkin devinkinwork@163.com
  
  RUN apt-get update && apt-get install -y inotify-tools nginx apache2 openssh-server
  ```

- 基于父镜像，安装`firefox`和`vnc`软件，启动后，用户可以通过5900端口通过vnc方式使用firefox。

  ```dockerfile
  # Firefox over VNC
  # 
  # VERSION 0.3
  
  FROM ubuntu
  
  # Install vnc, xvfb in order to create a 'fake' display and firefox
  RUN apt-get update && apt-get install -y x11vnc xvfb firefox
  RUN mkdir /.vnc
  # Setup a password
  RUN x11vnc -storepasswd 1234 ~/.vnc/passwd
  # Autostart firefox (might not be the best way, but it does the trick)
  RUN bash -c 'echo "firefox" >> /.bashrc'
  
  EXPOSE 5900
  CMD ["x11vnc", "-forever", "-usepw", "-create"]
  ```

## 指令

- 指令的一般格式为`INSTRUCTION arguments`

### FROM

- 指令格式`FROM <image>`或`FROM <image>:<tag>`
- 第一条指令必须为FROM指令。
- 如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令（每个镜像一次）。

### MAINTAINER

- 指令格式`MAINTAINER <name>`
- 指定维护者信息。

### RUN

- 指令格式`RUN <command>`或`RUN ["executable", "param1", "param2"]`
- 每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像。
- 当命令较长时，可以使用`\`来换行。

### CMD

- 支持三种指令格式
  - `CMD ["executable", "param1", "param2"]`使用`exec`执行，推荐方式。
  - `CMD command param1 param2`在`/bin/sh`中执行，提供给需要交互的应用。
  - `CMD ["param1", "param2"]`提供给`ENTRYPOINT`的默认参数。
- 指定启动容器时执行的命令，每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。
- 如果用户启动容器时指定了运行的命令，则会覆盖掉CMD指定的指令。

### EXPOSE

- 指令格式`EXPOSE <port> [<port>...]`
  - 示例`EXPOSE 22 80 8443`
- 告诉Docker服务端容器暴露的端口号，供互联系统使用。
- 在启动容器时需要通过`-P`，Docker主机会自动分配一个端口转发到指定的端口。
- 在启动容器时使用`-p`，则可以具体指定哪个本地端口映射过来。

### ENV

- 指令格式`ENV <key> <value>`

- 指定一个环境变量，会被后续RUN指令使用，并在容器运行时保持

- 示例

  ```dockerfile
  ENV FG_MAJOR 9.3
  ENV PG_VERSION 9.3.4
  RUN curl -SL http://example.com/postgress-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgres && ...
  ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
  ```

### ADD

- 指令格式`ADD <src> <dest>`
- 将复制指定的`<src>`到容器的`<dest>`。
- `<src>`可以是Dockerfile所在目录的一个相对路径（文件或目录），也可以是一个URL，还可以是一个tar文件（自动解压为目录）

### COPY

- 指令格式`COPY <src> <dest>`
- 复制本机的`<src>`（为Dockerfile所在目录的相对路径，文件或目录）到容器中的`<dest>`。目标路径不存在时，会自动创建。
- 当使用本地目录作为源目录时，推荐使用`COPY`

### ENTRYPOINT

- 指令格式有两种
  - `ENTRYPOINT["EXECUTABLE", "param1", "param2"]`
  - `ENTRYPOINT command param1 param2(shell中执行)`

- 配置容器启动后执行的命令，并且不可被`docker run`提供提供的参数覆盖。
- 每个Dockerfile中职能有一个`ENTRYPOINT`，当指定多个`ENTRYPOINT`时，只有最后一个生效。

### VOLUME

- 指令格式`VOLUME ["/data"]`
- 创建一个可以从本机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

### USER

- 指令格式`USER daemon`
- 指定运行容器时的用户名或UID，后续的RUN也会使用指定用户。

### WORKDIR

- 指令格式为`WORKDIR /path/to/workdir`

- 为后续的`RUN，CMD，ENTRYPOINT`指令配置工作目录。

- 可以使用多个`WORKDIR`指令，后续命令如果参数时相对路径，则会基于之前命令指定的路径。示例

  ```dockerfile
  WORKDIR /a
  WORKDIR b
  WORKDIR c
  RUN pwd
  # 最终路径是/a/b/c
  ```

### ONBUILD

- 指令格式为`ONBUILD [INSTRUCTION]`

- 配置当前创建的镜像作为其他创建镜像的基础镜像时，所执行的操作指令。

- Dockerfile使用如下内容创建了镜像`image-A`

  ```dockerfile
  [...]
  ONBUILD ADD . /app/src
  ONBUILD RUN /usr/local/bin/python-build --dir /app/src
  [...]
  ```

- 如果基于`image-A`创建新的镜像时，新的Dockerfile中使用`FROM image-A`指定基础镜像时，会自动执行ONBUILD指令的内容。

## 创建镜像

- 编写完Dockerfile之后，可以通过`docker build`命令来创建镜像。

- `docker build`命令格式`docker build [选项] 镜像tag Dockerfile路径`

  - `-t`选项指定镜像的标签信息

- 使用示例

  ```sh
  docker build -t newnginx:latest newNginx/
  ```

