---
title: Docker学习笔记
date: 2023-09-09 17:18:09
categories:
    - 学习笔记
    - Docker
tags:
    - Docker
abbrlink: 230909171809
---

## 1. 什么是Docker？

Docker是一个开源的容器化平台，用于构建、打包和部署应用程序。它允许开发人员将应用程序和它们的依赖项打包成一个轻量级、可移植的容器，然后在任何支持Docker的环境中运行。

### 1.1 Docker的核心概念

在深入学习Docker之前，了解以下几个核心概念是很重要的：

-   **容器（Container）**：一个**独立**运行的沙箱（sandboxed）应用程序实例，包含应用程序及其所有依赖项。容器是Docker的基本构建块，有以下特点：

	- 利用[kernel namespaces and cgroups](https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504)实现隔离/独立。

	- 是一个可运行的镜像的实例，可以使用DockerAPI或CLI创建、开始、停止、移动、删除一个容器。

	- 可以运行在本地机器上、虚拟机上或者部署在云上。

	- 可移植的（能运行在任何操作系统上）。

	- 与其它容器隔离并运行自己的软件、二进制文件和配置。

-   **镜像（Image）**：一个只读的模板，用于创建容器。镜像包含了运行应用程序所需的一切，例如代码、运行时环境、库和依赖项。

-   **仓库（Repository）**：用于存储和组织Docker镜像的地方。可以将仓库看作是镜像的集合。

-   **Dockerfile**：一个文本文件，包含了一系列命令和指令，用于从头开始构建Docker镜像。

<!-- more -->

### 1.2 Dcoker registry

Docker Registry是用于**存储、管理和分发Docker镜像的中央仓库**。它允许用户将自己创建的镜像上传到仓库，或者从仓库中拉取他人创建的镜像。在Docker中，默认使用Docker官方的公共注册服务器，称为Docker Hub。此外，还可以在本地或私有云环境中搭建自己的Docker Registry。

以下是Docker Registry的一些重要概念：

1.  镜像（Image）：Docker镜像是一个包含了应用程序、运行时环境、库和依赖的只读模板。镜像是构建和运行容器的基础。在Docker Registry中，镜像被存储和管理。

2.  仓库（Repository）：仓库是包含多个镜像的集合。仓库可以用于组织和管理镜像。每个镜像都有一个唯一的标签（tag），用于标识不同版本或变体的镜像。

3.  注册服务器（Registry Server）：注册服务器是存储和分发镜像的服务器。Docker Hub是Docker官方提供的公共注册服务器。除了Docker Hub，还可以搭建私有的注册服务器，例如Docker官方提供的开源项目Docker Registry或第三方的解决方案。

4.  仓库名称（Repository Name）：仓库名称用于唯一标识一个仓库。它由用户名（如果是私有仓库还包括域名）、斜杠和仓库名称组成，例如`username/repository`或`domain.com/username/repository`。

5.  标签（Tag）：标签用于标识仓库中的镜像的不同版本或变体。通过标签，可以区分不同的镜像，并拉取特定的版本。

常见的Docker Registry操作包括：

-   拉取镜像（Pull Image）：从注册服务器上拉取镜像到本地，以供使用或进一步构建容器。

-   推送镜像（Push Image）：将本地创建的镜像推送到注册服务器，以便其他人可以访问和使用。

-   搜索镜像（Search Image）：在注册服务器上搜索和浏览可用的镜像，以找到感兴趣的镜像。

-   删除镜像（Delete Image）：从注册服务器上删除不再需要的镜像，释放存储空间。

使用Docker Registry，你可以方便地共享和分发镜像，构建自己的镜像仓库，并进行版本控制和管理。无论是使用公共注册服务器还是搭建私有的注册服务器，Docker Registry都是Docker生态系统中重要的组成部分之一。


> 什么叫分发镜像？
> 分发镜像是指将 Docker 镜像从一个地方复制到另一个地方，以便其他用户可以访问和使用该镜像。
> 
> 分发镜像的过程一般包括以下步骤：1）Dockerfile构建镜像；2）打标签；3）推送到 Registry；4）拉取镜像。
> 
> 分发镜像的好处包括：1）共享性；2）可移植性；3）版本控制：使用标签来管理镜像的不同版本，可以轻松回退到先前的版本或升级到新的版本。


## 2. Docker的安装和配置

### 2.1 安装Docker

首先，需要安装Docker引擎。以下是在常见操作系统上安装Docker的步骤：

**Ubuntu**

```shell
$ sudo apt-get update
$ sudo apt-get install docker.io
```

**Windows**

1.  前往[Docker官方网站](https://www.docker.com/products/docker-desktop)下载Docker Desktop。
2.  双击安装包并按照指示完成安装。可能需要更新WSL，**需要把Windows的更新计划打开**，否则可能无法更新WSL。

### 2.2 配置Docker

安装完成后，你可能需要进行一些配置。例如，你可以设置Docker镜像加速器以加快下载速度。以下是一个示例：

1.  打开Docker配置文件：
-   **Ubuntu**：`/etc/docker/daemon.json`
-   **MacOS** / **Windows**：Docker Desktop 应用程序 > Preferences > Docker Engine

2.  如果文件不存在，创建它。然后，将以下内容添加到配置文件中：

```json
{
  "registry-mirrors": ["https://your-mirror.example.com"],
  "insecure-registries":[]
}
```

3.  保存配置文件并重启Docker服务：

-   **Ubuntu**：
	- `sudo systemctl daemon-reload`
	- `sudo systemctl restart docker`
-   **MacOS** / **Windows**：在Docker Desktop应用程序中点击重启按钮。

## 3. 使用Docker

### 3.1 运行容器

要运行一个容器，首先需要一个Docker镜像。可以从Docker仓库中获取现有的镜像，或者自己创建一个。以下是一个运行基于Ubuntu的容器的示例：

1.  拉取Ubuntu镜像：
	
```shell
$ docker pull ubuntu
```

2.  运行容器：

```shell
$ docker run -it ubuntu /bin/bash
```

这将以交互式终端的方式在Ubuntu容器中运行一个bash shell。你可以在容器中执行任意命令。

3.  退出容器：

```shell
$ exit
```

### 3.2 构建自定义镜像

使用Dockerfile可以构建自定义的Docker镜像。以下是一个简单的示例：

1.  创建一个新目录，例如`myapp`，并在其中创建一个名为`Dockerfile`的文件。

2.  在`Dockerfile`中添加以下内容：

```Dockerfile
# 使用较小的基础镜像
FROM python:3.9-slim

# 维护者信息
LABEL maintainer="Your Name <your.email@example.com>"

# 安装 Vim 编辑器
RUN apt-get update && \
    apt-get install -y vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 安装所需的软件包, 将多个RUN指令合并为一个（减小image的体积）
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \
    pip install --no-cache-dir \
    langchain==0.0.162 \
    openai \
    fastapi \

# 设置工作目录
WORKDIR /app

# 将本地项目文件复制到容器中
COPY ./ /app

# 设置环境变量
ENV ENV_VARIABLE_NAME=value

# 运行命令
CMD ["python", "/app/src/main.py", "--extra-arg", "value"]
```

以上示例使用了python作为基础镜像，并在容器中安装了一些软件包，设置了工作目录，复制了本地文件，并设置了环境变量和默认运行命令。

3.  在`Dockerfile`所在的目录中打开终端，并构建镜像：

```shell
$ docker build -t aigc/plugins:v1.0.0 .
```

这将使用`Dockerfile`创建一个名为`aigc/plugins`的镜像，版本号为`v1.0.0`。

4.  运行容器：

```shell
$ docker run --restart=always -d --name aigc_plugins -p 8385:8385 \
    -v /home/aigc/mount/plugins/data/store:/app/store \
    -v /home/aigc/mount/plugins/data/cfg:/app/cfg aigc/plugins:v1.0.0 python /app/src/main.py \
    --ip 0.0.0.0 --port 8385 --url http://0.0.0.0:8383/v1/completions
```

现在你可以在自定义镜像的容器中运行你的应用程序。

5.  进入容器：

```shell
$ docker exec -it aigc_plugins bash
```

6.  删除容器：

```shell
$ docker rm -f aigc_plugins
$ docker rm -f <container-id>
```

7.  查看容器运行的日志，并保存到文件中：

```shell
$ docker logs -f aigc_plugins | tee -a aigc_plugins_logs.txt
```

8.  打包保存镜像：

```shell
$ docker save -o aigc_plugins.tar aigc/plugins:v1.0.0
```

9.  加载本地镜像包：

```shell
$ docker load -i aigc_plugins.tar
```

## 4. Docker Volumes与Mount

Docker Volumes（卷）是 Docker 中用于**持久化数据**的一种机制，它允许将主机上的目录或文件与容器内的路径进行映射，从而实现数据在容器和主机之间的共享和持久化存储。

### 4.1 创建 Volume

使用 `docker volume create` 命令可以创建一个新的 Volume。

```shell
$ docker volume create my-volume
```

上述命令将创建一个名为 `my-volume` 的新 Volume。Docker 会在宿主机的默认 Volume 存储位置上创建一个目录，用于保存 Volume 的数据。

> Where is Docker storing my data when I use a volume?

使用 `docker volume inspect` 命令查看 Volume 存储位置。

```shell
$ docker volume inspect my-volume
```

接着，你将看到如下信息：

```json
[
    {
        "CreatedAt": "2021-06-01T16:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-volume/_data",
        "Name": "my-volume",
        "Options": {},
        "Scope": "local"
    }
]
```


> Mountpoint 是数据在磁盘上的实际位置。请注意，在大多数机器上，您需要具有根访问权限才能从主机访问此目录。但是，这就是它所在的位置。


### 4.2 容器中使用 Volume

在运行容器时，可以使用 `-v` 或 `--mount` 参数将 Volume 挂载到容器内的路径上。

#### 4.2.1 -v参数卷挂载

```shell
$ docker run -d -v my-volume:/path/in/container my-image
```

上述命令将创建一个基于 `my-image` 镜像的容器，并将名为 `my-volume` 的 Volume 挂载到容器内的 `/path/in/container` 路径上。

#### 4.2.2 --mount参数卷挂载

使用 `--mount` 参数可以更细粒度地配置容器中的 Volume 挂载。

```shell
$ docker run -d --mount type=volume,source=my-volume,target=/path/in/container my-image
```

在上述命令中，我们使用了 `--mount` 参数来指定挂载的类型和配置。`type=volume` 表示挂载类型为 Volume，`source=my-volume` 指定了要挂载的 Volume 的名称为 `my-volume`，`target=/path/in/container` 指定了挂载到容器内的路径为 `/path/in/container`。

除了指定挂载类型、Volume 名称和路径外，`--mount` 参数还可以接受其他选项，例如读写权限、文件系统类型等。下面是一个更详细的示例：

```shell
$ docker run -d --mount type=volume,source=my-volume,target=/path/in/container,ro,volume-opt=option=value my-image
```

在这个示例中，我们添加了一些额外的选项。`ro` 表示将 Volume 挂载为只读（read-only），`volume-opt=option=value` 表示指定了额外的 Volume 选项。你可以根据需要使用适当的选项来满足挂载的要求。

使用 `--mount` 参数进行 Volume 挂载可以提供更多的灵活性和配置选项，例如指定挂载类型、读写权限、Volume 选项等。这使得我们能够更好地控制容器与 Volume 之间的数据交互。

### 4.3 挂载主机目录作为 Volume

除了使用命名 Volume，还可以直接将主机上的目录或文件挂载为 Volume，即**绑定挂载（Bind mounts）**。

```shell
$ docker run -d -v /path/on/host:/path/in/container my-image
```

上述命令将主机上的 `/path/on/host` 目录挂载到容器内的 `/path/in/container` 路径上。

> 绑定挂载（Bind mounts）允许将主机文件系统上的特定路径直接挂载到容器内的路径上。
> 卷挂载（Volume mounts）通过使用 Docker 卷 (Volume) 将容器内的路径与主机上的路径进行关联。

### 4.4 查看 Volume

使用 `docker volume ls` 命令可以列出所有已创建的 Volumes。

```shell
$ docker volume ls
```

上述命令将显示所有已创建的 Volumes 的列表，包括它们的名称和所在的路径。

### 4.5 删除 Volume

要删除一个不再需要的 Volume，可以使用 `docker volume rm` 命令。

```shell
$ docker volume rm my-volume
```

上述命令将删除名为 `my-volume` 的 Volume。请注意，只有在没有任何容器使用该 Volume 时，才能成功删除。

### 4.6 Volume Mounts Vs Bind Mounts

#### 4.6.1 Volume Mounts的特性

卷挂载通过使用 Docker 卷 (Volume) 将容器内的路径与主机上的路径进行关联。Docker 卷是一种特殊的目录，用于存储容器中的数据。卷挂载具有以下特点：

-  **主机位置**：Docker 选择卷的主机位置，通常在宿主机上的特定目录中。

-  **数据持久化**：Volume 提供了一种持久化存储数据的方式。即使容器被删除或重新创建，Volume 中的数据仍然保持不变。

-  **容器间数据共享**：多个容器可以共享同一个 Volume，使得它们可以轻松地共享数据。

-  **与主机文件系统交互**：通过将主机上的目录挂载为 Volume，容器可以访问主机的文件系统，方便与容器之外的环境交互。

-  **备份和迁移**：使用 Volumes 可以方便地备份和迁移容器的数据，通过备份或迁移 Volume，可以快速复制或迁移整个容器的数据。

-  **卷驱动程序支持**：卷挂载支持使用不同的卷驱动程序，例如本地主机文件系统、网络存储或其他存储后端。

#### 4.6.2 Bind Mounts的特性

绑定挂载允许将主机文件系统上的特定路径直接挂载到容器内的路径上。绑定挂载具有以下特点：

-  **主机位置**：您可以自行选择要挂载的主机路径，它可以是任何主机上的目录或文件。

-  **与主机文件系统实时同步**：容器中的更改会实时反映在绑定挂载的主机路径上，反之亦然。

-  **灵活性**：绑定挂载允许容器与宿主机之间进行更紧密的交互，方便开发人员对文件进行编辑、调试或测试。

-  **支持权限和访问控制**：绑定挂载可以继承主机文件系统的权限设置，以控制容器对文件的读写权限。

#### 4.6.3 选择使用卷挂载还是绑定挂载

|                         | 命名卷 (Named Volumes)                             | 绑定挂载 (Bind Mounts)                               |
| ----------------------- | -------------------------------------------------- | ---------------------------------------------------- |
| 主机位置                | 由Docker选择                                       | 由您决定                                             |
| 挂载示例 (使用 `--mount`) | `type=volume,src=my-volume,target=/usr/local/data` | `type=bind,src=/path/to/data,target=/usr/local/data` |
| 容器内容填充新卷        | 是                                                 | 否                                                   |
| 支持卷驱动程序          | 是                                                 | 否                                                   |

在选择使用卷挂载还是绑定挂载时，可以根据具体需求和场景来进行决策：

-   如果需要数据持久性、跨容器共享和卷驱动程序支持，则应选择卷挂载。

-   如果需要与主机文件系统实时同步、灵活的文件访问和交互，则应选择绑定挂载。

在实际应用中，根据不同的需求，卷挂载和绑定挂载可以结合使用，以满足容器与主机之间数据共享和交互的需求。


> 容器内容填充新卷是指在使用卷挂载（Volume Mounts）时，如果目标卷是新创建的并且为空（即没有初始数据），**容器启动时会将容器内的内容填充到新创建的卷中**。
> 
> 这意味着，如果使用一个新的、空的卷挂载到容器内的路径上，并且容器内的路径中有预先存在的数据，那么在容器启动时，这些数据会被复制到新的卷中，以便卷中具有与容器内部相同的初始数据。这样可以确保在容器重新创建或迁移时，新创建的卷具有与原始容器相同的初始状态。


## 5. Container networking


> 请记住，默认情况下，容器是独立运行的，对同一台机器上的其他进程或容器一无所知。那么，如何让一个容器与另一个容器通信呢？答案是网络（network）。如果将两个容器放在同一个网络上，它们就可以相互通信。


### 5.1 容器网络特性

容器网络是 Docker 中一个重要的概念，它允许**在容器之间建立网络连接和通信**。通过容器网络，可以实现容器间的数据传输、服务发现和跨主机的容器通信等功能。以下是容器网络的一些关键点和特性：

-   **网络命名空间 (Network Namespace)**：每个容器都有自己独立的网络命名空间，使得容器可以拥有自己的网络栈，包括网络接口、IP 地址和路由表等。这样可以实现容器之间的网络隔离，避免网络冲突和干扰。
    
-   **虚拟以太网桥 (Virtual Ethernet Bridge)**：Docker 使用虚拟以太网桥来连接容器和宿主机的网络。每个宿主机上的 Docker 守护进程会创建一个名为 docker0 的桥接接口，**通过该接口与容器的网络命名空间相连**。
    
-   **网络驱动程序 (Network Drivers)**：Docker 提供了多种网络驱动程序，用于实现不同的网络功能和配置选项。默认情况下，Docker 使用桥接网络驱动程序 (Bridge Network Driver)，但还支持其他驱动程序，如 Overlay、Host、Macvlan 等，以满足不同的网络需求。
    
-   **容器间通信**：容器可以通过使用网络连接和端口来实现相互通信。**可以使用容器的 IP 地址、端口号或容器名称来访问其他容器上的服务**。容器网络还支持多个容器共享同一个网络端口的负载均衡和服务发现功能。
    
-   **跨主机容器通信**：通过使用网络驱动程序中的 Overlay 网络模式，可以在多个主机上创建一个虚拟网络，使得不同主机上的容器可以直接通信。这为构建分布式应用和容器编排平台（如 Docker Swarm 和 Kubernetes）提供了基础。
    
-   **外部网络访问**：容器可以通过端口映射 (Port Mapping) 的方式将容器内部的服务暴露给外部网络。通过将容器的特定端口映射到宿主机上的端口，可以使得外部网络能够访问容器中的应用程序或服务。
    

容器网络是 Docker 中实现容器间通信和与外部网络交互的重要机制。通过灵活配置和管理容器网络，可以构建具有高可用性、可伸缩性和弹性的容器化应用程序。

### 5.2 容器网络实践命令

#### 5.2.1 创建自定义容器网络

使用 `docker network create` 命令可以创建自定义的 Docker 网络。例如，创建一个名为 `my-network` 的自定义网络：

```shell
$ docker network create my-network
```

这将在 Docker 中创建一个新的网络，供容器使用。

#### 5.2.2 运行容器并连接到网络

在创建容器时，可以使用 `--network` 参数将容器连接到指定的网络。例如，将一个容器连接到 `my-network` 网络：

```shell
$ docker run --network my-network my-image
```

这将在启动容器时将其连接到 `my-network` 网络，使得容器可以与该网络中的其他容器进行通信。

#### 5.2.3 查看容器网络信息

使用 `docker network inspect` 命令可以查看特定网络的详细信息。例如，查看名为 `my-network` 的网络信息：

```shell
$ docker network inspect my-network
```

 这将显示与 `my-network` 相关的网络配置和容器列表等信息。

```json
[
    {
        "Name": "my-network",
        "Id": "000f5e28bfcafa18622593e04dba1bc70c1bc9480a055c0a0bb2b8ec13039138",
        "Created": "2023-06-01T22:51:27.366486209-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

#### 5.2.4 容器间通信

容器可以通过使用容器名称或 IP 地址进行互相通信。例如，通过容器名称进行通信：

```shell
$ docker run --name container1 --network my-network my-image
$ docker run --name container2 --network my-network my-image
```

在上述示例中，`container1` 和 `container2` 这两个容器可以通过彼此的名称进行通信。例如，通过 `ping` 命令测试容器之间的连通性：

```shell
$ docker exec container1 ping container2
```

在上述示例中，`container1` 容器使用 `ping container2` 的命令来 ping `container2` 容器。由于容器名称在容器网络中被解析为相应容器的 IP 地址，因此可以直接使用容器名称来实现容器间的通信。


> 通过使用容器名称进行通信，可以减少对容器 IP 地址的依赖，提高容器间通信的灵活性。注意，容器名称必须在同一网络中才能相互解析，因此需要将容器都连接到同一个网络中。


#### 5.2.5 端口映射

使用 `-p` 参数可以将容器内的端口映射到宿主机上，以使得外部网络可以访问容器中的服务。例如，将容器的 8080 端口映射到宿主机的 80 端口：

```shell
$ docker run -p 80:8080 my-image
```

这将使得宿主机上的 80 端口转发到容器内的 8080 端口，从而允许外部网络通过宿主机的 80 端口访问容器中的服务。

以上是一些与容器网络相关的实践命令和示例。通过灵活运用这些命令，可以有效管理和配置容器网络，实现容器间的通信和与外部网络的交互。

## 6. Docker Compose

Docker Compose是一个用于定义和运行**多个容器**的工具。它使用一个`docker-compose.yml`文件来配置应用程序的服务、网络和卷。以下是一个示例：

1.  创建一个名为`docker-compose.yml`的文件，并在其中定义服务：

```yaml
version: "3.9"

services:
  plugins:
    image: aigc/plugins:v1.0.0
    container_name: aigc_plugins
    restart: always
    networks:
      - my_network
    command: python /app/src/main.py 
        --ip 0.0.0.0 
        --port 8385 
        --url http://0.0.0.0:8383/v1/completions
    ports:
      - 8385:8385
    volumes:
      - /home/aigc/mount/plugins/data/store:/app/store
      - /home/aigc/mount/plugins/data/cfg:/app/cfg
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "0.5"
          memory: 1024M

networks:
  my_network:
    name: plugins_net
    driver: bridge
```

以上示例定义了一个名为`plugins`的服务，使用当前目录中的`Dockerfile`构建镜像。它将主机的`8385`端口映射到容器的`8385`端口，并将本地的`./`目录挂载到容器的`/app`目录，还设置了一个环境变量。

2.  在包含`docker-compose.yml`文件的目录中打开终端，并运行服务：

```shell
$ docker-compose up
```

Docker Compose将自动构建镜像并运行服务。


