# SOLA Runtime 示例程序

这个项目包含了 SOLA Runtime API 的示例代码，以及一些模型部署的 Demo，以帮助用户快速上手 SOLA Runtime API。

## 获取源码

可以从以下链接中获取源码：
```text
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-demo/sola-demo-3.5.0.tar.gz
```

## 目录与文件说明
```
├── common                  # 公用文件
├── data                    # 运行一些基础示例所需要的模型文件
├── inference               # 一个简单的通用推理框架示例
├── introduction            # SOLA Runtime API的基本用法介绍
├── models                  # SOLA Runtime API模型部署的demo
├── utils                   # SOLA Runtime API的一些实用工具
├── build.sh                # 编译脚本
├── install_dependencies.sh # 安装系统依赖脚本
├── prepare.sh              # 下载python环境依赖和系统依赖
├── CHANGELOG.md            # 更变日志
├── README.md               # 说明文档
├── LICENSE                 # 版权信息
└── VERSION                 # 版本信息
```

## 使用说明

### 系统要求

请在 SOLA Toolkit 支持的系统上使用本示例程序。

**注意：当前系统语言环境必须为英文，否则会执行错误，可以执行以下命令将当前shell环境改为英文：**
```shell
export LANG=en_US.UTF-8
export LC_ALL=en_US.UTF-8
export LANGUAGE=en_US.UTF-8
```

### 软件依赖

#### 基础系统依赖库（不包含模型部署示例的依赖）

请自行安装以下第三方依赖：
- `wget`
- `tar`
- `libgl1`
- `cmake` >= 3.10
- `g++` >= 7.x
- `gflags` >= 2.2
- `SOLA Toolkit` == 3.5.0

或执行以下脚本尝试一键安装：
```shell
sudo sh install_dependencies.sh
```

#### python 环境依赖

我们已经打包好了整个python虚拟环境，可以通过如下命令下载并激活：
```bash
./prepare.sh
source sola-demo-env/bin/activate
```

### 编译运行

示例程序分为两种，一种是Runtime API的示例，一种是模型部署的示例。

在根目录下执行 `build.sh`，会编译所有的示例程序，模型部署的示例在各自的目录下也有编译脚本，可以单独编译。

Runtime API示例程序的编译产物在根目录的 `build/bin` 下，模型示例程序的编译产物在各自的 `models/xxx/build` 下。

执行Runtime API的示例程序，可以直接在根目录下执行，如：
```shell
./build/bin/device_query
```

执行模型部署的示例程序，需要参考每个模型目录中的 `README.md` 。通常情况下，模型的部署都需要下载模型和数据文件，以及配置运行环境，所以基本都有 `prepare.sh`、`build.sh`、`run.sh`、`verify.sh` 这几个脚本。

`prepare.sh` 用于下载模型文件、数据预处理以及配置运行环境。

`build.sh` 用于编译模型部署的示例程序。

`run.sh` 用于执行模型部署的示例程序。

`verify.sh` 用于验证模型部署的示例程序的运行结果。
