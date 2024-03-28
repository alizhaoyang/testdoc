# SOLA容器化示例

## 概述

Docker 保证了不同测试环境之间的一致性，简化了复杂依赖的管理，并且支持快速部署和扩展。此外，Docker 的资源隔离功能确保使用过程中的稳定性，同时容器的可重用性和共享性大大提高了团队合作的效率。最重要的是，Docker 可以无缝集成进 CI/CD 工作流程，加速整个软件开发周期，是一种高效、灵活且可靠的工具。

目前提供 3 种操作系统镜像，**请根据实际操作系统，使用对应的镜像。**

- debian
- rhel
- ubuntu

镜像下载链接：

docker 镜像 - debian

```bash 
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/Debian/macs-debian-sola350-v1.1.tar
```

docker 镜像 - rhel

```bash
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/RHEL/macs-rhel-sola350-v1.1.tar
```

docker 镜像 - ubuntu

```bash
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/images/Ubuntu/macs-ubuntu-sola350-v1.1.tar
```

## 环境要求

- SOLA：在宿主机安装SOLA，请参考  【 SOLA Toolkit 安装指南 】 章节
- Docker：建议版本高于 18.09

## 导入离线镜像

```bash 
docker load -i moffettai-macs-<version>-<arch>-<os>.tar
```

##  创建容器

```bash
docker run -itd  --restart always --privileged --cap-add=ALL  --net=host -v /dev:/dev  -v /usr/src:/usr/src -v /lib/modules:/lib/modules  --shm-size="900g" --name <container-name> <image-id> /bin/bash
```

--shm-size参数请根据实际情况进行设置，一般建议为宿主机总内存的90%，如总内存为1000G，则设置为 900g

## 注意事项

由于容器内使用的是挂载的方式，与宿主机存在着一定的差异，***以下可使用的功能\***

- mx-smi
- mx-qual
- sola demo

## 验证使用

- 进入容器
- 使用 mx-smi :  请参考  【 mx-smi 用户手册 】 章节
- 使用 mx-qual : 请参考  【 mx-qual 用户手册 】 章节
- 使用 sola demo :  容器内安装 sola demo ，请参考  【 SOLA 示例程序 】 章节

示例

```bash 
# 进入容器
docker exec -it <container-name> /bin/bash
# 容器内查看系统中的墨芯加速卡
mx-smi list
# 容器内使用 mx-qual
mx-qual list -i 0
# 容器内使用 sola demo
请参考【SOLA示例程序】章节进行环境安装、部署
```

## 清除环境

```bash
# 停止运行中的容器
docker stop <container-name>
# 删除对应容器
docker rm <container-name>
# 删除对应镜像
docker rmi <image-id>
```

