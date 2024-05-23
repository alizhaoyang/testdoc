# SOLA容器化示例

## 概述

Docker 保证了不同测试环境之间的一致性，简化了复杂依赖的管理，并且支持快速部署和扩展。此外，Docker 的资源隔离功能确保使用过程中的稳定性，同时容器的可重用性和共享性大大提高了团队合作的效率。最重要的是，Docker 可以无缝集成进 CI/CD 工作流程，加速整个软件开发周期，是一种高效、灵活且可靠的工具。

SOLA容器化（MACS Container Toolkit）使用容器运行时库和实用程序构建运行模型和应用，方便的利用墨芯计算卡的算力资源。

## 前提条件

- 已安装Docker，且安装的Docker版本不低于Docker 18.03。

- 启动Docker服务。启动命令如下：

  ```Bash
   systemctl status docker
   systemctl start docker 
  ```

- 在宿主机上已安装SOLA，安装的详细步骤请参见《SOLA Toolkit 安装指南》。

  > 说明：我们已经在镜像内预装SOLA稳定版本，您无需单独安装。

  需要注意的是，容器内安装的驱动版本需与宿主机安装的SOLA版本保持一致，您可通过以下命令查看容器内和 宿主机上安装的SOLA版本：

    ```Bash
  ls -al /usr/local/
  #查询 sola版本信息，如：/usr/local/sola-3.5.3
    ```

## 步骤一：获取镜像

### **基础镜像**

基础镜像基于主流OS系统构建，包括一些基础环境环境依赖，具备运行基本应用的能力。

> **注意**：镜像分为在线版和离线版两种。
>
> - 如果要使用离线版镜像，您需要在下载离线版本镜像后，导入离线版本镜像。导入命令例如`docker load -i moffettai-macs-<version>-<arch>-<os>.tar`。
> - 如果要使用在线版镜像，请确保测试服务器可访问互联网及Docker hub官方镜像仓库。



#### 离线版镜像

- macs-ubuntu

  ```Bash
   https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/Ubuntu/base/macs-ubuntu-sola3.5.3-v1.1.3.tar
  ```

- macs-debian

  ```Bash
   https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/Debian/base/macs-debian-sola3.5.3-v1.1.3.tar
  ```

- macs-rhel

  ```Bash
  https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/RHEL/base/macs-rhel-sola3.5.3-v1.1.3.tar       
  ```

#### 在线版镜像

- macs-ubuntu

  ```Bash
  docker pull moffett/macs-ubuntu:sola3.5.3-v1.1.3
  ```

- macs-debian

  ```Bash
  docker pull moffett/macs-debian:sola3.5.3-v1.1.3
  ```

- macs-rhel

  ```Bash
  docker pull moffett/macs-rhel:sola3.5.3-v1.1.3
  ```

### **业务镜像**

业务镜像主要为模型/项目/应用等提供运行环境。本文仅以sola-demo为例，说明项目镜像的获取方式。

- 离线版sola-demo-v3.5.1-data-offline

  ```Bash
  https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/Ubuntu/business/sola-demo-3.5.1.tar
  ```

- 在线版:sola-demo-v3.5.1

  ```Bash
  docker pull moffett/sola-demo:3.5.1
  docker images |grep sola-demo 
  ```

## 步骤二：获取镜像准备数据

执行以下命令，下载模型数据压缩包并解压：

```Bash
$ wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/sola-demo-3.5.1-data-offline.tar.gz
$ tar -zxvf sola-demo-3.5.0-offline.tar.gz
```

> 说明：模型数据文件相对较大，单独打包发布，在容器中使用，需在容器启动时单独挂载到容器内相应目录。数据上传至测试物理服务器指定目录，解压模型数据包。

## **步骤三：获取镜像创建容器**

基于导入或下载的镜像，为测试环境创建一个容器实例，创建容器通常只执行一次并设置开机容器自启动，一般情况无需重复执行。

- **创建基础环境标准容器**：

  ```Bash
  docker run -itd --privileged --name $(CONTAINER_NAME) $(IMAGE_REPO)/$(IMAGE_NAME):$(IMAGE_TAG) /bin/bash
   docker run -itd  --restart always --privileged --cap-add=ALL  --net=host -v /dev:/dev  -v /usr/src:/usr/src -v /lib/modules:/lib/modules  --shm-size="900g" --name $(CONTAINER_NAME) $(IMAGE_NAME/IMAGE_ID) /bin/bash
  ```

  命令启动参数可根据需要灵活调整，简要说明，详情请参见Docker文档:

  | **参数**                                  | **是否必选** | **说明**                                                     |
  | ----------------------------------------- | ------------ | ------------------------------------------------------------ |
  | $(CONTAINER_NAME)                         | 否           | 容器名称。建议您定义一个容器名。                             |
  | $(IMAGE_REPO）/$(IMAGE_NAME):$(IMAGE_TAG) | 是           | 镜像仓库地址/镜像名:镜像tag或镜像ID                          |
  | -itd:                                     | 是           | 交互方式后台运行容器。                                       |
  | /bin/bash                                 | 是           | 指定进入容器shell环境为bash。                                |
  | –privileged                               | 是           | 使用特权模式                                                 |
  | –net=host                                 | 否           | 默认容器网络为bridge模式，容器网络与宿主机网络隔离； 设置该参数为host后，容器网络为主机模式，可方便共享宿主机网络环境。 |
  | –restart always                           | 否           | 容器退出时应用的重新启动策略为always，默认为no               |
  | -v                                        | 否           | --volume，声明将宿主机存储目录挂载到容器对应的存储目录。方便宿主机和容器间文件共享。 |
  | –shm-size                                 | 否           | 可用的物理内存。建议为宿主机总内存的90%，如物理总内存为1000G，该参数可设置为 900g。 |

示例如下：

```Bash
docker run -itd --privileged --name macs-1.1.3 moffett/macs-ubuntu:sola3.5.3-v1.1.3 /bin/bash
```

- **创建sola-demo容器**

  > **注意**：如果您想在容器中直接执行SOLA示例，可执行以下命令创建sola-demo容器。创建容器前请注意所在位置，`$PWD `为当前目录。

  ```Bash
  docker run -itd --privileged -v $PWD/sola-demo-3.5.1-data-offline:/home/moffett/workspace/sola-demo-3.5.1 --name sola-demo-3.5.1 moffett/sola-demo:3.5.1 /bin/bash
  ```

## **步骤四：获取镜像验证使用**

可针对以下工具进行验证：

- 使用 mx-smi ，使用方式请参见《mx-smi 使用说明》。
- 使用 mx-qual，使用方式请参见《mx-qual 使用说明》。
- 使用 sola demo，使用方式请参见《SOLA示例验证手册》。

基本示例如下：

```Bash
# 进入容器, $(CONTAINER_NAME) 替换为容器名字/ID
docker exec -it $(CONTAINER_NAME) /bin/bash
# 容器内查看系统中的墨芯加速卡
mx-smi list
# 容器内使用 mx-qual
mx-qual list -i 0
```

## **SOLA示例**

### ResNet50

- 进入容器内运行模型 

  ```Bash
  docker exec -it sola-demo-3.5.1 /bin/bash
  sola-demo-3.5.1/models/resnet50/run.sh && sola-demo-3.5.1/models/resnet50/verify.sh
  ```

- 创建临时容器并运行模型

  ```Bash
  docker run -it --rm --privileged -v $PWD/sola-demo-3.5.1-data-offline:/home/moffett/workspace/sola-demo-3.5.1 moffett/sola-demo:3.5.1 /bin/bash -c "sola-demo-3.5.1/models/resnet50/run.sh && sola-demo-3.5.1/models/resnet50/verify.sh"
  ```

### Bert

- 进入容器内运行模型 

  ```Bash
  docker exec -it sola-demo-3.5.1 /bin/bash
  sudo sola-demo-3.5.1/models/bert/run.sh && sola-demo-3.5.1/models/bert/verify.sh
  ```

- 创建临时容器并运行模型

  ```Bash
  docker run -it --rm --privileged -v $PWD/sola-demo-3.5.1-data-offline:/home/moffett/workspace/sola-demo-3.5.1 moffett/sola-demo:3.5.1 /bin/bash -c "sudo sola-demo-3.5.1/models/bert/run.sh && sola-demo-3.5.1/models/bert/verify.sh"
  ```

### BLOOM 7B

- 进入容器内运行模型 

  ```Bash
  docker exec -it sola-demo-3.5.1 /bin/bash
  sola-demo-3.5.1/models/bloom-7b/run.sh && sola-demo-3.5.1/models/bloom-7b/verify.sh
  ```

- 创建临时容器并运行模型

  ```Bash
  docker run -it --rm --privileged -v $PWD/sola-demo-3.5.1-data-offline:/home/moffett/workspace/sola-demo-3.5.1 moffett/sola-demo:3.5.1 /bin/bash -c "sola-demo-3.5.1/models/bloom-7b/run.sh && sola-demo-3.5.1/models/bloom-7b/verify.sh"
  ```

### BLOOM 176B

- 进入容器内运行模型 

  ```Bash
  docker exec -it sola-demo-3.5.1 /bin/bash
  sola-demo-3.5.1/models/bloom-176b/run.sh && sola-demo-3.5.1/models/bloom-7b/verify.sh
  ```

- 创建临时容器并运行模型

  ```Bash
  docker run -it --rm --privileged -v $PWD/sola-demo-3.5.1-data-offline:/home/moffett/workspace/sola-demo-3.5.1 moffett/sola-demo:3.5.1 /bin/bash -c "sola-demo-3.5.1/models/bloom-176b/run.sh && sola-demo-3.5.1/models/bloom-7b/verify.sh"
  ```

## **清除环境**

> **注意**：<CONTAINER_NAME>请替换成实际的容器名。

1. 在宿主机上执行以下命令，停止运行中的容器。

   ```Bash
   docker stop $(CONTAINER_NAME/ID)
   ```

2. 在宿主机上执行以下命令，删除对应容器。

   ```Bash
   docker rm $(CONTAINER_NAME/ID)
   ```

3. 在宿主机上执行以下命令，删除对应镜像。

   ```Bash
   docker rmi $(IMAGE_NAME:TAG/IMAGE_ID)
   ```

4. 在宿主机上执行以下命令，删除指定容器/镜像。

   ```Bash
   export container_name="your container name"
   export image_name="your images name:tag or images id"
   docker rm -f $(docker ps -a -q | grep -E "^($container_name)")
   docker rmi -f $(docker images -a -q | grep -E "^($image_name:tag)")
   ```
