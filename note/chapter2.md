# Docker的核心概念和安装

- 三大核心概念
  - 镜像（Image）
  - 容器（Container）
  - 仓库（Repository）

## 核心概念

### Docker镜像

- Docker镜像类似于虚拟机镜像，可以将它理解为一个面向Docker引擎的只读模板，包含了文件系统。

### Docker容器

- Docker容器类似于一个轻量级的沙箱，Docker利用容器来运行和隔离应用。
- 容器是从镜像创建的应用运行示例，可以将其启动、停止、删除，而这些容器都是互相隔离、互不可见的。

### Docker仓库

- Docker仓库类似于代码仓库，是Docker集中存放镜像文件的场所。
- 根据所存储的镜像公开分享与否，Docker仓库可以分为公开仓库和私有仓库。
- 最大的公开仓库是Docker Hub
- 国内公开的仓库是Doocker Pool

