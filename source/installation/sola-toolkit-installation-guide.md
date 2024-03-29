# SOLA Toolkit 安装指南

## 1. 简介

`SOLA Toolkit` 是墨芯软件开发工具包（以下简称 SOLA），它提供了驱动、运行时库以及一系列的工具，便于用户开发和调试在墨芯 AI 加速卡上的应用程序。

本指南将向您展示如何安装 SOLA 开发工具包并检查其是否正确运行。

### 1.1 系统要求

要在您的系统上使用 SOLA，需要满足软硬件环境依赖：

* 可识别的墨芯 AI 加速卡
* 支持的 Linux 版本及 gcc 编译器工具链
* SOLA Toolkit

SOLA 开发环境依赖于与主机开发环境的紧密集成，包括主机编译器和 C 运行时库，因此仅符合 SOLA 开发环境要求的 Linux 发行版被支持。

下表列出了支持的 Linux 发行版。

| 发行版 | 架构 |  默认内核版本 | 默认 gcc 版本 | GLIBC 版本 |
| --- | --- | --- |-----------|----------|
| Ubuntu 18.04 | x86_64 | 4.15.0 | 7.5.0     | 2.27     |
| Ubuntu 20.04 | x86_64 | 5.4.0 | 9.3.0     | 2.31     |
| Debian 10 | x86_64 | 4.19.0 | 8.3.0     | 2.28     |
| Debian 11 | x86_64 | 5.10.0 | 10.2.1    | 2.31     |
| RHEL 8.x | x86_64 | 4.18.0 | 8.3.1     | 2.28     |
| RHEL 9.x | x86_64 | 5.4.0 | 10.2.1    | 2.34     |

## 2. 安装前操作

在 Linux 上安装 SOLA 工具包和驱动程序之前，必须执行一些操作：

- 验证系统是否具有支持 SOLA 的墨芯 AI 加速卡
- 验证系统是否正在运行受支持的 Linux 版本
- 验证系统是否已安装 gcc
- 验证系统是否安装了正确的内核头文件和开发包
- 下载 SOLA 工具包
- 处理不同方式的安装冲突

### 2.1 验证系统是否具有支持 SOLA 的墨芯 AI 加速卡

验证系统是否具有支持 SOLA 的墨芯 AI 加速卡，可以通过 `lspci` 命令，如果您的系统上没有该命令，请自行安装 `pciutils` 软件包。

然后在命令行中执行以下命令：

```bash
lspci | grep 1f36
```
若系统中存在墨芯 AI 加速卡，那么命令将会返回设备的 PCI 信息。

### 2.2 验证系统是否正在运行受支持的 Linux 版本

SOLA 仅在某些特定的 Linux 发行版上受支持，这些在 SOLA 工具包发行说明中列出。

要确定您正在运行的发行版和版本号，请在命令行中执行以下命令：

```bash
uname -m && cat /etc/*release
```

您应该看到类似于以下内容的输出：

```bash
x86_64
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic
```

### 2.3 验证系统是否已安装 gcc

安装 SOLA 工具包需要 gcc 编译器，它通常作为 Linux 安装的一部分进行安装，并且在大多数情况下，与受支持的 Linux 版本一起安装的 gcc 版本可以正常工作。

要验证系统上安装的 gcc 版本，请在命令行中执行以下命令：

```bash
gcc --version
```

安装 SOLA 至少需要 **gcc 6.x** 及以上的版本。

### 2.4 验证系统是否安装了正确的内核头文件和开发包

SOLA 驱动程序要求在安装驱动程序时以及重建驱动程序时安装内核运行版本的内核头文件和开发包。例如，如果您的系统运行内核版本 3.17.4-301，则必须安装 3.17.4-301 内核头文件和开发包。
为了避免安装过程中版本不匹配的问题，在安装 SOLA 之前以及每次更改内核版本时，请手动确保安装正确版本的内核头文件和开发包。

可以通过运行以下命令找到系统正在运行的内核版本：
    
```bash
uname -r
```

SOLA 驱动程序在安装时还依赖一些第三方的软件包，请根据您的发行版安装以下软件包：
- tar
- make
- kmod
- libelf-dev (Debian/Ubuntu) 或 elfutils-libelf-devel (RHEL/CentOS)
- dkms

### 2.5 下载 SOLA 工具包

SOLA 工具包包含创建、构建和运行 SOLA 应用程序所需的 SOLA 驱动程序、运行时库、头文件、工具和其他资源，选择您正在使用的平台并下载相应的 SOLA 工具包。

**Runfile 软件包**
```bash
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola_3.5.0_x86_64.run
```

**RPM 软件包**
```shell
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola-3.5.0-1.x86_64.rpm
```

**Deb 软件包**
```shell
https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola_3.5.0_amd64.deb
```

### 2.6 选择安装方式
可以使用两种不同的安装机制之一来安装 SOLA 工具包：特定于发行版的软件包（RPM 和 Deb 包）或独立于发行版的软件包（runfile 包）。

独立于发行版的软件包的优点是可以在更广泛的 Linux 发行版上工作，但不会更新发行版的本机包管理系统。特定于发行版的软件包会与发行版的本机包管理系统交互，建议尽可能使用特定于发行版的软件包。

### 2.7 处理不同方式的安装冲突

特定于发行版的软件包和独立于发行版的软件包底层实现是同源的，所以安装和卸载都会互相影响，如果您同时安装了两种不同方式的安装包，在卸载时可能会影响本机的包管理工具的使用，所以建议在安装时保持一种方式。

## 3. 安装 SOLA 工具包

SOLA 可以使用多种方式安装，取决于您使用的 Linux 发行版，以下介绍不同发行版的安装方式。

- Ubuntu
- Debian
- RHEL 8 / CentOS 8
- RHEL 9 / CentOS 9

### 3.1 Ubuntu
在 Ubuntu 上安装 SOLA 时，您可以选择 Runfile 软件包和 Debian 软件包，他们都仅作为本地安装程序提供。

#### 3.1.1 准备安装

- 确保已经执行了安装前操作
- 内核头文件和开发包已经安装

  执行以下命令，安装与当前运行的内核版本一致的内核头文件：
  ```bash
  sudo apt-get install linux-headers-$(uname -r)
  ```
  注意：若无法通过包管理器获取对应内核的版本，请务必另行下载对应版本并安装对应版本。

- 第三方依赖已经安装
  - tar
  - make
  - kmod
  - libelf-dev
  - dkms

#### 3.1.2 使用 Debian 软件包安装
在命令行中执行以下命令：
```bash
sudo dpkg -i sola_<version>_<architecture>.deb
dpkg -l | grep sola
```

#### 3.1.3 使用 Runfile 软件包安装
在命令行中执行以下命令：
```bash
sudo sh sola_<version>_<architecture>.run --driver
```

#### 3.1.4 验证安装
请参考验证安装这一章节。

### 3.2 Debian
在 Debian 上安装 SOLA 时，您可以选择 Runfile 软件包和 Debian 软件包，他们都仅作为本地安装程序提供。

#### 3.2.1 准备安装

- 确保已经执行了安装前操作
- 内核头文件和开发包已经安装

  执行以下命令，安装与当前运行的内核版本一致的内核头文件：
  ```bash
  sudo apt-get install linux-headers-$(uname -r)
  ```
  注意：若无法通过包管理器获取对应内核的版本，请务必另行下载对应版本并安装对应版本。

- 第三方依赖已经安装
  - tar
  - make
  - kmod
  - libelf-dev
  - dkms

#### 3.2.2 使用 Debian 软件包安装
在命令行中执行以下命令：
```bash
sudo dpkg -i sola_<version>_<architecture>.deb
dpkg -l | grep sola
```

#### 3.2.3 使用 Runfile 软件包安装
在命令行中执行以下命令：
```bash
sudo sh sola_<version>_<architecture>.run --driver
```

#### 3.2.4 验证安装
请参考验证安装这一章节。

### 3.3 RHEL 8 / CentOS 8
在 Redhat 8 或 CentOS 8 上安装 SOLA 时，您可以选择 Runfile 软件包和 RPM 软件包，他们都仅作为本地安装程序提供。

#### 3.3.1 准备安装

- 确保已经执行了安装前操作
- 内核头文件和开发包已经安装

  执行以下命令，安装与当前运行的内核版本一致的内核开发包和内核头文件：
  ```bash
  sudo dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
  ```
  注意：若无法通过包管理器获取对应内核的版本，请务必另行下载对应版本并安装对应版本。

- 第三方依赖已经安装
  - tar
  - make
  - kmod
  - elfutils-libelf-devel
  - dkms
  
  某些软件包仅在第三方存储库中可以访问，例如 EPEL。在安装前，必须将任何此类第三方存储库添加到包管理器存储库数据库中，否则缺少依赖项将导致安装无法继续。要启用 EPEL 存储库，请执行以下命令：
  ```bash
  sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
  ```

#### 3.3.2 使用 RPM 软件包安装
在命令行中执行以下命令：
```bash
sudo rpm -i sola-<version>.<architecture>.rpm
rpm -qi sola
```

#### 3.3.3 使用 Runfile 软件包安装
在命令行中执行以下命令：
```bash
sudo sh sola_<version>_<architecture>.run --driver
```

#### 3.3.4 验证安装
请参考验证安装这一章节。

### 3.4 RHEL 9 / CentOS 9
在 Redhat 9 或 CentOS 9 上安装 SOLA 时，您可以选择 Runfile 软件包和 RPM 软件包，他们都仅作为本地安装程序提供。

#### 3.4.1 准备安装

- 确保已经执行了安装前操作
- 内核头文件和开发包已经安装

  执行以下命令，安装与当前运行的内核版本一致的内核开发包和内核头文件：
  ```bash
  sudo dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
  ```
 注意：若无法通过包管理器获取对应内核的版本，请务必另行下载对应版本并安装对应版本。

- 第三方依赖已经安装
  - tar
  - make
  - kmod
  - elfutils-libelf-devel
  - dkms
  
  某些软件包仅在第三方存储库中可以访问，例如 EPEL。在安装前，必须将任何此类第三方存储库添加到包管理器存储库数据库中，否则缺少依赖项将导致安装无法继续。要启用 EPEL 存储库，请执行以下命令：
  ```bash
  sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
  ```

#### 3.4.2 使用 RPM 软件包安装
在命令行中执行以下命令：
```bash
sudo rpm -i sola-<version>.<architecture>.rpm
rpm -qi sola
```

#### 3.4.3 使用 Runfile 软件包安装
在命令行中执行以下命令：
```bash
sudo sh sola_<version>_<architecture>.run --driver
```

#### 3.4.4 验证安装
请参考验证安装这一章节。

## 4. 验证安装

安装完成后，默认安装在 `/usr/local/sola-<version>` 下，同时会自动创建一个 `/usr/local/sola` 的软链接。

`/etc/ld.so.conf.d/sola.conf` 文件会被自动创建，同时执行 `ldconfig` 命令。

以下工具安装在 `/usr/bin` 下：

- mx-smi
- mx-qual
- mx-mft
- mx-daemon
- sola-uninstall
- moffett-bug-report.sh

执行 `mx-smi` 查看系统中的墨芯 AI 加速卡的设备信息。

## 5. 卸载 SOLA Toolkit

卸载 SOLA Toolkit 时会检测当前系统上所有被安装在默认路径下的 SOLA Toolkit 版本，并提供给用户选择。执行卸载会同时卸载 SOLA Toolkit 的所有组件，包括驱动程序、工具和库。根据安装方式的不同，卸载方法也有所不同，以下列出不同安装方式的卸载方法。

### 使用 RPM 软件包安装

在命令行中执行以下命令：
```bash
sudo rpm -e sola
```

### 使用 Deb 软件包安装

在命令行中执行以下命令：
```bash
sudo dpkg -r sola
```

### 使用 Runfile 软件包安装

在命令行中执行以下命令：
```bash
sudo sola-uninstall
```
