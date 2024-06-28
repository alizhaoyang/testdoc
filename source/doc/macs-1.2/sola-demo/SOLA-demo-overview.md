# SOLA Runtime 示例程序

## 概述

这个项目旨在为用户提供 SOLA Runtime API 的实用示例代码以及模型部署的详细案例，从而协助用户迅速掌握并高效运用 SOLA Runtime API，实现项目的快速部署与实施。

## 准备工作

### **安装 SOLA**

安装 SOLA 的详细步骤，请参见《SOLA Toolkit 安装指南》。

### 设置操作系统语言环境为英文

执行本示例程序必须在系统语言环境为英文的环境下进行，否则会出现执行错误。您执行以下命令将当前系统语言环境环境更改为英文：

```Bash
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
```

### **获取**源码

执行以下命令，获取源码文件并解压。以获取 sola-demo-3.6.0 为例：

```Bash
$ sudo wget  https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/sola-demo-3.6.0.tar.gz
$ tar -zxvf sola-demo-3.6.0.tar.gz
$ cd sola-demo-3.6.0
```

解压后的目录与文件结构如下所示：

```Bash
├── common                  # 公用文件
├── data                    # 运行一些基础示例所需要的模型文件
├── inference               # 一个简单的通用推理框架示例
├── introduction            # SOLA Runtime API的基本用法介绍
├── models                  # SOLA Runtime API模型部署的示例
├── utils                   # SOLA Runtime API的一些实用工具
├── build.sh                # 编译脚本
├── install_dependencies.sh # 安装系统依赖脚本
├── prepare.sh              # 下载python环境依赖和系统依赖
├── CHANGELOG.md            # 更变日志
├── README.md               # 说明文档
├── LICENSE                 # 版权信息
└── VERSION                 # 版本信息
```

### 安装基本**软件依赖**

执行以下命令，安装第三方基础软件依赖：

```Bash
$ sudo sh install_dependencies.sh
```

您也可以自行安装安装以下第三方基础软件依赖：

- wget
- tar
- libgl1
- cmake >= 3.10
- g++ >= 7.x
- gflags >= 2.2
- SOLA Toolkit >= 3.6.0

### 安装 **Python 环境**依赖

我们已经打包好了整个 Python 虚拟环境，您可在进入解压后的源码文件后，执行以下命令下载并激活 Python 虚拟环境：

```Bash
$ ./prepare.sh
$ source sola-demo-env/bin/activate
```

## 后续步骤

准备工作完成后，您可以直接进入 models 文件夹，选择相应的模型目录，并执行对应的模型部署操作。部署模型的步骤，请参见各个模型的部署手册。

## 部署模型流程

> 说明： 在每个模型文件目录下，我们为以下每个步骤都提供了对应的脚本，您可以直接使用。

1. 下载模型文件、数据预处理以及配置运行环境：**prepare.sh。**
2. 编译模型：**build.sh**。
3. 执行模型部署：**run.sh** 。
4. 验证模型部署的运行结果：**verify.sh**。

## **更多信息**

示例程序分为两种，一种是 Runtime API 的示例，一种是模型部署的示例。

Runtime API 示例程序的编译产物在根目录的 `build/bin` 下，模型示例程序的编译产物在各自的 `models/{model_name}/build` 目录下。

您可在根目录下执行以下命令来运行 Runtime API 的示例程序：

```Bash
$ ./build/bin/device_query
```
