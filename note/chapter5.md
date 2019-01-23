# 第五章-仓库

## Docker  Hub

- 可以通过`docker login`输入用户名，密码和邮箱来完成注册和登录。注册成功后，本地用户目录的`.dockercfg`中将保存用户的认证信息。
- docker官方公共仓库https://hub.docker.com

## 创建和使用私有仓库

### 使用registry镜像创建私有仓库

- 可以通过官方提供的registry镜像来简单搭建一套本地私有仓库环境

  ```sh
  docker run -d -p 5000:5000 registry
  ```

- 默认情况下，会将仓库创建在容器的`/tmp/registry`目录下，可以通过`-v`参数将镜像文件存放在本地的指定路径上。

  ```sh
  docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
  ```

