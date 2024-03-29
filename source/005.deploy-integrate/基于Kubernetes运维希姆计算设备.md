# 基于Kubernetes运维希姆计算设备

## 版本历史

| **文档版本** | **对应产品版本**             | **作者** | **日期**   | **描述**                                                     |
| ------------ | ---------------------------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.3.0       | stc-k8s-device-plugin V1.0.1 | 希姆计算 | 2023-02-08 | 增加Pod镜像的OS和VM的OS要一致，否则可能因glibc、gcc等基础软件不匹配出现异常说明。<br/>增加非映射方式的说明与示例。 |
| V1.2.0       | stc-k8s-device-plugin V1.0.1 | 希姆计算 | 2023-01-16 | 更新pod目录内的example，以适配HPE V1.5.0。                   |
| V1.1.3       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2022-12-01 | 验证文档内容。<br/>编辑优化。                                |
| V1.1.2       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2022-08-30 | 验证文档内容。<br/>                                          |
| V1.1.1       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2022-07-07 | 验证文档内容。                                               |
| V1.1.0       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2022-06-02 | 整篇编辑优化。<br/>添加示例的回显。<br/>添加文件示例章节。   |
| V1.0.1       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2022-04-11 | 处理格式问题。                                               |
| V1.0.0       | stc-k8s-device-plugin V1.0.0 | 希姆计算 | 2021-09-01 | 初始版本。                                                   |

## 概述

希姆计算提供了Kubernetes设备插件stc-k8s-device-plugin，方便您在无需修改Kubernetes核心代码情况下，即可实现包含NPU设备的通用解决方案。部署stc-k8s-device-plugin后，在创建Pod时Kubernetes集群会自动申请使用NPU设备，并跟踪NPU设备的健康状况。

## 前提条件

- 安装Kubernetes，支持1.18、1.22、1.23和1.24版本，推荐使用1.18版本。

- 部署Kubernetes集群。

- 在主机上部署希姆计算异构环境。具体的步骤，请参见[希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。

  > 说明：目前在主机上有两种方式部署希姆计算异构环境。采取整包安装方式部署，所有配套软件都将安装在主机侧上。采取Docker安装方式部署，HPE驱动模块安装在主机上，其他模块安装在容器中。安装方式将影响后续创建Pod的方式。

- 联系希姆计算技术支持获取stc-k8s-device-plugin的工具包。

## 部署设备插件

1. 登录Kubernetes集群的master节点。

2. 解压stc-k8s-device-plugin工具包并进入插件目录。以工具包名称为stc-k8s-device-plugin-1.0.1.tgz为例：

   ```bash
   $ tar -xvzf stc-k8s-device-plugin-1.0.1.tgz
   $ cd stc-k8s-device-plugin-1.0.1
   $ ls
   Dockerfile  go.mod  go.sum  main.go  pod  README.md  stc-device-plugin.yaml
   ```

3. 构建设备插件镜像。目前提供两种方式：

   - 基于插件目录下的Dockerfile构建设备插件镜像，Dockerfile的详细内容，请参见*文件示例*章节。以为设备插件镜像添加标签stc/k8s-device-plugin为例，示例命令如下：

     ```bash
     $ docker build -t stc/k8s-device-plugin .
     Sending build context to Docker daemon  37.89kB
     Step 1/13 : FROM ubuntu:bionic
      ---> 71eaf13299f4
     Step 2/13 : RUN apt-get update     && apt-get install -y vim     && apt-get install -y coreutils     && apt-get install -y bash     && apt-get install -y wget
      ---> Using cache
      ---> e235bc4b7bcc
     Step 3/13 : RUN wget -nv -O - https://storage.googleapis.com/golang/go1.16.4.linux-amd64.tar.gz     | tar -C /usr/local -xz
      ---> Using cache
      ---> 2a9b91894b9b
     Step 4/13 : ENV GOPATH /go
      ---> Using cache
      ---> f598ffa7f6ee
     Step 5/13 : ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
      ---> Using cache
      ---> 50265c3b5d7d
     Step 6/13 : WORKDIR /gosrc/k8s-device-plugin
      ---> Using cache
      ---> e4d36dfbd928
     Step 7/13 : COPY * ./
      ---> Using cache
      ---> 43c58b782add
     Step 8/13 : RUN go version
      ---> Using cache
      ---> 84320c92e6a2
     Step 9/13 : RUN go env -w GOPROXY=https://goproxy.cn
      ---> Using cache
      ---> c637320a8d5d
     Step 10/13 : RUN go env -w GOSUMDB=off
      ---> Using cache
      ---> 1092caae484f
     Step 11/13 : RUN go build
      ---> Using cache
      ---> 02c79245297d
     Step 12/13 : RUN cp k8s-device-plugin /usr/bin/
      ---> Using cache
      ---> 961ce0f10d40
     Step 13/13 : CMD ["k8s-device-plugin"]
      ---> Using cache
      ---> f8edbe727b00
     Successfully built f8edbe727b00
     Successfully tagged stc/k8s-device-plugin:latest
     ```

   - 基于离线镜像包导入设备插件镜像。以导入stc-k8s-device-plugin-1_0_1.tar镜像包为例，示例命令如下：

     ```bash
     $ docker load -i stc-k8s-device-plugin-1_0_1.tar
     101b05ef38e1: Loading layer [==================================================>]  65.52MB/65.52MB
     c6432e482943: Loading layer [==================================================>]  108.2MB/108.2MB
     3dad1dfa0047: Loading layer [==================================================>]  394.4MB/394.4MB
     7737afced4be: Loading layer [==================================================>]   2.56kB/2.56kB
     977c0d882ff3: Loading layer [==================================================>]  36.35kB/36.35kB
     63f69879f16e: Loading layer [==================================================>]  4.096kB/4.096kB
     5e3217163bde: Loading layer [==================================================>]  3.584kB/3.584kB
     44f42d3c0c96: Loading layer [==================================================>]  264.9MB/264.9MB
     a51fa3855538: Loading layer [==================================================>]  13.26MB/13.26MB
     Loaded image: stc/k8s-device-plugin:latest
     ```
     
     > 说明：联系希姆计算技术支持获取离线镜像包。

4. 查看构建或导入好的设备插件镜像，即标签为stc/k8s-device-plugin的镜像。

   ```bash
   $ docker images | grep k8s-device-plugin
   REPOSITORY                                       TAG                             IMAGE ID       CREATED         SIZE
   stc/k8s-device-plugin                            latest                          f8edbe727b00   2 minutes ago   827MB                             3.4.3-0                         303ce5db0e90   3 years ago     288MB
   ```

5. 基于插件目录下的YAML文件在Kubernetes集群中部署设备插件，YAML文件的详细内容，请参见*文件示例*章节。

   > 说明：部署插件时采用创建DaemonSet的方式，新增节点上的NPU设备会自动加入资源池。

   ```bash
   $ kubectl apply -f stc-device-plugin.yaml
   daemonset.apps/stc-device-plugin-daemonset created
   ```

6. 检查部署结果。如果插件部署成功，查看pod时会出现名称以`stc-device-plugin-daemonset`开头的DaemonSet，且状态为Running。以master节点的主机名称为k8smasterdemo为例：

   ```bash
   $ kubectl get pods -n kube-system | grep stc-device-plugin
   NAME                                READY   STATUS    RESTARTS   AGE
   stc-device-plugin-daemonset-hlpgt   1/1     Running   0          7s
   ```

## 创建Pod并申请NPU设备

本章节演示如何在创建Pod时申请NPU设备，并将主机上部署的希姆计算异构环境映射到Pod。

> 说明：在创建Pod时，Pod镜像的OS和VM的OS需保持一致，否则可能因glibc、gcc等基础软件不匹配出现异常。

以基于Debian 10.6的Pod为例：

1. 进入插件目录stc-k8s-device-plugin-1.0.1。

2. 使用示例配置创建一个Pod。
   1. 构建Pod镜像，目前根据HPE安装方式的不同，可分为映射场景和非映射场景。
   
      若HPE采取整包方式安装，所有配套软件都安装在主机上。构建Pod镜像，可采用映射的方式获取HPE相关库。示例如下，Dockerfile的详细内容，请参见*文件示例*章节。
   
      ```bash
      $ docker build -t stc/k8s-pod-example pod/
      Sending build context to Docker daemon  4.096kB
      Step 1/5 : FROM debian:10.6
       ---> ef05c61d5112
      Step 2/5 : ENV LD_LIBRARY_PATH /usr/local/hpe/lib:$LD_LIBRARY_PATH
       ---> Using cache
       ---> 38b26155db40
      Step 3/5 : RUN ln -s /usr/local/hpe/bin/clang++ /usr/bin/stc-clang++
       ---> Using cache
       ---> 0011ae85fd0c
      Step 4/5 : RUN ln -s /usr/local/hpe/bin/stcc /usr/bin/stcc
       ---> Using cache
       ---> 94d2f9bcfefe
      Step 5/5 : RUN ln -s /usr/local/hpe/bin/lld /usr/bin/stc-ld.lld
       ---> Using cache
       ---> 35dac764fd1f
      Successfully built 35dac764fd1f
      Successfully tagged stc/k8s-pod-example:latest
      ```
   
      若HPE驱动模块安装在主机上，其他模块安装在容器中。构建Pod镜像，可采用非映射的方式，需要将HPE相关库安装在Pod中。构建Pod镜像示例如下，Dockerfile的详细内容，请参见*文件示例*章节。
   
      ```bash
      $ DOCKER_BUILDKIT=1 docker build -f Dockerfile -t stc/k8s-pod-example --build-arg URI=http://example.com --target dev .
      ```
   
   2. 查看构建好的Pod镜像，即标签为stc/k8s-pod-example的镜像。
   
      ```bash
      $ docker images | grep k8s-pod-example
      REPOSITORY                                       TAG                             IMAGE ID       CREATED         SIZE
      stc/k8s-pod-example                              latest                          35dac764fd1f   1 minutes ago   114MB
      ```
   
   3. 基于pod目录下YAML文件创建一个名为stc-example-pod的Pod，YAML文件的详细内容，请参见*文件示例*章节。
   
      ```bash
      $ kubectl apply -f pod/example.yaml
      pod/stc-example-pod created
      ```
   
3. 检查Pod的运行状态，确定状态为Running。

   ```bash
   $ kubectl get pods
   NAME              READY   STATUS    RESTARTS   AGE
   stc-example-pod   1/1     Running   0          10s
   ```

4. 在Pod中验证NPU设备可用。

   - 通过容器的Shell访问。以bash为例，打开一个bash终端后输入Shell命令即可。该示例过程中Kubernetes集群会将控制台输入发送到stc-example-pod中第一个容器的Shell，并将输出返回到控制台。如果成功访问到设备，会返回NPC的response。

     > 注意：采用映射方式构建的Pod，example文件下的文件在主机侧编译后，在Pod中可直接运行。采用非映方式构建的Pod，example文件下的文件需先在Pod中编译后才能运行。

     ```bash
     $ kubectl exec -it stc-example-pod -- bash
     root@stc-example-pod:/# cd /usr/local/hpe/example
     root@stc-example-pod:/usr/local/hpe/example# ./hello_world
     running hello_world......
     hello world from core 0/8.
     hello world from core 1/8.
     hello world from core 2/8.
     hello world from core 3/8.
     hello world from core 4/8.
     hello world from core 5/8.
     hello world from core 6/8.
     hello world from core 7/8.
     ```

   - 您也可以通过kubectl命令访问。如果成功访问到设备，会返回NPC的response。

     ```bash
     $ kubectl exec stc-example-pod -- /usr/local/hpe/example/hello_world
     running hello_world......
     hello world from core 0/8.
     hello world from core 1/8.
     hello world from core 2/8.
     hello world from core 3/8.
     hello world from core 4/8.
     hello world from core 5/8.
     hello world from core 6/8.
     hello world from core 7/8.
     ```


## 文件示例

### Dockerfile（构建设备插件镜像）

```Dockerfile
#  Copyright (c) 2019-2023 北京希姆计算科技有限公司 (Stream Computing Inc.)
#  All Rights Reserved.
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
FROM ubuntu:bionic
RUN apt-get update \
    && apt-get install -y vim \
    && apt-get install -y coreutils \
    && apt-get install -y bash \
    && apt-get install -y wget

RUN wget -nv -O - https://storage.googleapis.com/golang/go1.16.4.linux-amd64.tar.gz \
    | tar -C /usr/local -xz
ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH

WORKDIR /gosrc/k8s-device-plugin
COPY * ./
RUN go version
RUN go env -w GOPROXY=https://goproxy.cn
RUN go env -w GOSUMDB=off
RUN go build
RUN cp k8s-device-plugin /usr/bin/

CMD ["k8s-device-plugin"]
```

### stc-device-plugin.yaml（在Kubernetes集群中部署设备插件）

```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: stc-device-plugin-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: stc-device-plugin-ds
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: stc-device-plugin-ds
    spec:
      tolerations:
      - key: CriticalAddonsOnly
        operator: Exists
      - key: streamcomputing.com/npu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: stc/k8s-device-plugin
        imagePullPolicy: IfNotPresent
        name: stc-device-plugin-ctr
        securityContext:
          privileged: true
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
          - name: dev
            mountPath: /dev
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
        - name: dev
          hostPath:
            path: /dev
```

### Dockerfile（构建Pod镜像采用映射方式获取HPE相关库）

```Dockerfile
#  Copyright (c) 2019-2023 北京希姆计算科技有限公司 (Stream Computing Inc.)
#  All Rights Reserved.
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
FROM debian:10.6
ENV LD_LIBRARY_PATH /usr/local/hpe/lib:$LD_LIBRARY_PATH
RUN ln -s /usr/local/hpe/bin/clang++ /usr/bin/stc-clang++
RUN ln -s /usr/local/hpe/bin/stcc /usr/bin/stcc
RUN ln -s /usr/local/hpe/bin/lld /usr/bin/stc-ld.lld
```

### Dockerfile（构建Pod镜像采用非映射方式获取HPE相关库）

以STCRP1.5.1版本为例：

```Dockerfile
# syntax=docker/dockerfile:1

FROM scratch AS base

ENV INSTALL_DIR=/tmp

COPY requirements.txt $INSTALL_DIR/requirements.txt
COPY requirements_venv2.txt $INSTALL_DIR/requirements_venv2.txt
COPY requirements_extra.txt $INSTALL_DIR/requirements_extra.txt

FROM debian:buster AS dev

LABEL maintainer "streamcomputing.com <streamcomputing.com>"

ENV DEBIAN_FRONTEND noninteractive
RUN sed -i "s/deb\.debian\.org/mirrors.tuna.tsinghua.edu.cn/g; s/security\.debian\.org/mirrors.tuna.tsinghua.edu.cn/g" /etc/apt/sources.list \
    && apt-get update && apt-get install -y wget --no-install-recommends \
    && wget -q http://apt.streamcomputing.com/docker_sources_mirror.sh -O docker_sources_mirror.sh && bash docker_sources_mirror.sh \
    && apt-get update \
    && apt-get install -y libmariadb-dev python3 python3-pip python3-venv gnupg python3-dev make curl --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && python3 -m pip install --upgrade setuptools --no-cache-dir && python3 -m pip install --upgrade pip --no-cache-dir

ARG CACHEBUST=1
ARG TB_install_file=tb_debian10-1.12.0-cp37-cp37m-linux_x86_64.whl
ARG stc_hpaa=stc-hpaa-1.12.0-py37.tar.gz
ARG URI={URL containing tb whl and stc-hpaa tar.gz and hpe-repo deb}
ARG HPE_REPO=hpe-repo-debian10-1-7-local1.7.0_amd64.deb

ENV INSTALL_DIR=/tmp
ENV LANG=C.UTF-8

RUN wget -q ${URI}/${HPE_REPO} -O ${INSTALL_DIR}/${HPE_REPO} \
    && dpkg -i ${INSTALL_DIR}/${HPE_REPO} \
    && apt-key add /var/hpe-repo*/hpe.pub \
    && apt-get update \
    && apt-get install hpe-simple -y  \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf "${INSTALL_DIR:?}/"*

RUN --mount=type=bind,from=base,source=${INSTALL_DIR},target=${INSTALL_DIR},rw \
    python3 -m pip install -U -r ${INSTALL_DIR}/requirements.txt --no-cache-dir \ 
    && python3 -m pip install -U -r ${INSTALL_DIR}/requirements_extra.txt --no-cache-dir \ 
    && python3 -m pip install ${URI}/${TB_install_file} --no-cache-dir \
    && wget -q ${URI}/${stc_hpaa} -O /root/${stc_hpaa} \
    && python3 -m venv /root/venv2 \
    && /root/venv2/bin/python3 -m pip install -U pip --no-cache-dir \
    && /root/venv2/bin/python3 -m pip install -r ${INSTALL_DIR}/requirements_venv2.txt --no-cache-dir \
    && /root/venv2/bin/python3 -m pip install ${URI}/${TB_install_file} --no-cache-dir \
    && rm -rf "${INSTALL_DIR:?}/"*
```

### example.yaml（创建Pod）

Pod名为stc-example-pod，目前仅支持以一个NPU为最小粒度申请，并限制申请一个NPU设备。

```YAML
# example of a POD running sleep 100000 and allocated 1 NPU device

apiVersion: v1
kind: Pod
metadata:
  name: stc-example-pod
spec:
#  restartPolicy: OnFailure
  containers:
  - image: stc/k8s-pod-example
    imagePullPolicy: IfNotPresent
    name: stc-example-ctr
    command: ["sleep"]
    args: ["100000"]

    volumeMounts:
      - name: hpelink
        mountPath: /usr/local/hpe
    resources:
      limits:
        streamcomputing.com/npu: 1

  volumes:
    - name: hpelink
      hostPath:
        path: /usr/local/hpe
```
