# SOLA Toolkit 安装指南

## **概述**

`SOLA Toolkit` 是墨芯软件开发工具包（以下简称 SOLA），它提供了驱动、运行时库以及一系列的工具，便于用户开发和调试在墨芯 AI 计算卡上的应用程序。本文将为您展示安装 SOLA 的方式及安装过程中的注意事项。

## **前提条件**

### 服务器已经安装墨芯 AI 计算卡

您可在目标环境中执行以下命令，查看系统中是否成功安装墨芯 AI 计算卡，以墨芯 S30 为例：

```Bash
$ lspci | grep 1f36
#如果系统中存在墨芯S30，那么将会返回设备的PCI信息。
36:00.0 Processing accelerators: Device 1f36:7030
37:00.0 Processing accelerators: Device 1f36:7030
38:00.0 Processing accelerators: Device 1f36:7030
```

如果您的系统上没有`lspci`命令，请执行以下命令安装 `pciutils` 软件包：

<p class="attention">注意：执行以下命令时，需获取 sudo 权限。</p>

> **注意：**执行以下命令时，需获取 sudo 权限。

```Bash
# Ubuntu/debian 
$ apt-get install pciutils
# RHEL/CentOS/fedora
$ dnf install pciutils
```

### **系统正在运行受支持的 Linux** **版本**

SOLA 开发环境依赖于与主机开发环境的紧密集成，包括主机编译器和 C 运行时库，因此仅符合 SOLA 开发环境要求的 Linux 发行版被支持。目前 SOLA 支持的操作系统版本如下表所示：

| **操作系统版本** | **架构** | **默认内核版本** | **默认** **gcc** **版本** | **GLIBC** **版本** |
| ---------------- | -------- | ---------------- | ------------------------- | ------------------ |
| Ubuntu 18.04     | x86_64   | 4.15.0           | 7.5.0                     | 2.27               |
| Ubuntu 20.04     | x86_64   | 5.4.0            | 9.4.0                     | 2.31               |
| Debian 10        | x86_64   | 4.19.0           | 8.3.0                     | 2.28               |
| Debian 11        | x86_64   | 5.10.0           | 10.2.1                    | 2.31               |
| RHEL 8.2/8.3     | x86_64   | 4.18.0           | 8.3.1                     | 2.28               |
| RHEL 9.3         | x86_64   | 5.4.0            | 10.2.1                    | 2.34               |
| Kylin V10        | x86_64   | 4.19             | 7.3.0                     | 2.28               |

在安装 SOLA 前，请执行以下命令查看操作系统版本是否属于支持的操作系统版本。

```Bash
$ uname -m && cat /etc/*release
```

### 系统已安装 GCC

SOLA 工具包依赖于 GCC 编译器，且要求 GCC 版本大于等于 gcc 6.x。 您可执行以下命令，查看系统安装的 GCC 版本：

```Bash
  $ gcc --version
```

### 系统已安装正确的内核头文件和开发包

SOLA 驱动程序要求在安装驱动程序时以及重建驱动程序时安装内核运行版本的内核头文件和开发包。例如，如果您的系统运行内核版本 3.17.4-301，则必须安装 3.17.4-301 内核头文件和开发包。 为了避免安装过程中版本不匹配的问题，在安装 SOLA 之前以及每次更改内核版本时，请手动确保安装正确版本的内核头文件和开发包。

您可通过执行以下命令查看系统正在运行的内核版本：

```Bash
$ uname -r
```

如需安装与当前运行的内核版本一致的内核头文件，请执行以下命令：

> **注意：**若无法通过包管理器获取对应内核的版本，请务必另行下载对应版本并安装对应版本。

- **Ubuntu/Debian**

  ```Bash
  $ sudo apt-get install linux-headers-$(uname -r)
  ```

- **RHEL 8 /RHEL 9/ CentOS 8/CentOS 9**

  ```Bash
  sudo dnf install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
  ```

### 安装其它第三方依赖

安装 SOLA，您还需安装以下第三方依赖：

- dkms
- libelf-dev （Debian/Ubuntu） 或 elfutils-libelf-devel （RHEL/CentOS）
- kmod
- make
- tar

### 下载 SOLA 工具包

目前我们提供三种类型的软件包，您可按需取用，这三种软件包分别是：

- （**推荐使用**）特定于发行版的 RPM 包或 Deb 包：可与发行版的本机包管理系统交互。
- 独立于发行版的 Runfile 包：可以在更广泛的 Linux 发行版上工作，但不会更新发行版的本机包管理系统。

下载方式如下：

> **注意：**
>
> - 如果您使用的是 Ubuntu 系统或 Debian 系统，请下载 Debian 软件包或 Runfile 软件包。
> - 如果您使用的是 RHEL 系统或 CentOS 系统，请下载 RPM 软件包或 Runfile 软件包 。

- 执行以下命令，下载 Runfile 包。

  ```Bash
   wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola_3.6.0_x86_64.run
  ```

- 执行以下命令，下载 RPM 包。

  ```Bash
   wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola-3.6.0-1.x86_64.rpm
  ```

- 执行以下命令，下载 Deb 包。

  ```Bash
   wget https://moffett-oss-bucket01.oss-cn-shenzhen.aliyuncs.com/sola-toolkits/sola_3.6.0_amd64.deb
  ```

> **注意**：特定于发行版的软件包和独立于发行版的软件包底层实现是同源的，所以安装和卸载都会互相影响，如果您同时安装了两种不同类型的安装包，在卸载时可能会影响本机的包管理工具的使用，所以建议您保持同一种类型的安装包的使用。

## **安装步骤**

SOLA 可以使用多种方式安装，具体取决于您所使用的 Linux 发行版。下文将为您详细介绍几种主流的发行版的安装方式。

<p class="attention">注意：使用 Debian 软件包或 RPM 软件包安装时，默认您同意最终用户许可协议，协议内容可在[墨芯官网文档中心](https://docs.moffettai.com/)查看。
使用 Runfile 软件包安装时，需要您手动同意最终用户许可协议。</p>

> **注意**：
>
> - 使用 Debian 软件包或 RPM 软件包安装时，默认您同意最终用户许可协议，协议内容可在[墨芯官网文档中心](https://docs.moffettai.com/)查看。
> - 使用 Runfile 软件包安装时，需要您手动同意最终用户许可协议。

### **Ubuntu/Debian**

#### **使用 Debian 软件包安装（推荐使用）**

在命令行中执行以下命令：

```Bash
#请将安装包名替换为您实际下载的安装包名
$ sudo dpkg -i sola_<version>_<architecture>.deb
$ dpkg -l | grep sola
```

#### **使用 Runfile 软件包**安装

在命令行中执行以下命令：

```Bash
#请将安装包名替换为您实际下载的安装包名
$ sudo sh sola_<version>_<architecture>.run --driver
```

### **RHEL 8 /RHEL 9/ CentOS 8/CentOS 9**

#### **使用 RPM 软件包安装（推荐使用）**

在命令行中执行以下命令：

```Bash
#请将安装包名替换为您实际下载的安装包名
$ sudo rpm -U sola-<version>.<architecture>.rpm
$ rpm -qi sola
```

#### **使用 Runfile 软件包安装**

在命令行中执行以下命令：

```Bash
#请将安装包名替换为您实际下载的安装包名
$ sudo sh sola_<version>_<architecture>.run --driver
```

## **验证安装结果**

安装 SOLA 后，可以执行 `mx-smi` 命令，可以返回系统中 SOLA 的版本信息和墨芯 AI 计算卡的设备信息，若版本信息与安装的版本一致，则表明安装成功。

安装完成后，SOLA 默认安装在 `/usr/local/sola-<version>` 下，同时会自动创建一个 `/usr/local/sola` 的软链接。同时，以下可执行程序会被安装在 `/usr/bin` 下，方便您使用。

| **可执行程序**        | **说明**               |
| --------------------- | ---------------------- |
| mx-smi                | 管理和监控设备的工具   |
| mx-qual               | 系统和设备质量检测工具 |
| mx-mft                | 设备固件管理工具       |
| mx-daemon             | 自动加载固件的守护进程 |
| sola-uninstall        | 卸载程序               |
| moffett-bug-report.sh | 故障收集工具           |

## **卸载 SOLA** **Toolkit**

执行卸载命令会同时卸载 SOLA Toolkit 的所有组件，包括驱动程序、工具和库。您可根据不同的安装方式，选择不同的卸载步骤：

- 如果您是使用 RPM 软件包方式安装的 SOLA ，可执行以下命令卸载 SOLA：

  ```Bash
   $ sudo rpm -e sola
  ```

  > **注意**：如果在您的系统中执行卸载命令时，系统提示存在多个版本的 SOLA Toolkit，您可以通过执行 `sudo rpm -e sola-<version>.<architecture>` 命令来单独卸载指定版本的 SOLA Toolkit。这个命令会将指定版本的 SOLA Toolkit 从 RPM 数据库中移除。请注意，只有在最后一个版本被卸载时，RPM 才会执行 SOLA Toolkit 的卸载脚本，彻底清理所有已安装版本的 SOLA Toolkit 相关文件和设置。

  

  <p class="attention">注意：如果在您的系统中执行卸载命令时，系统提示存在多个版本的 SOLA Toolkit，您可以通过执行 sudo rpm -e sola-<version>.<architecture> 命令来单独卸载指定版本的 SOLA Toolkit。这个命令会将指定版本的 SOLA Toolkit 从 RPM 数据库中移除。请注意，只有在最后一个版本被卸载时，RPM 才会执行 SOLA Toolkit 的卸载脚本，彻底清理所有已安装版本的 SOLA Toolkit 相关文件和设置。</p>

- 如果您是使用 Deb 软件包方式安装的 SOLA ，可执行以下命令卸载 SOLA：

  ```Bash
   $ sudo dpkg -r sola
  ```

- 如果您是使用 Runfile 软件包方式安装的 SOLA ，可执行以下命令卸载 SOLA：

  ```Bash
   # 默认会检测当前系统上所有被安装在默认路径下的SOLA版本，您可在出现的提示界面通过上下键和空格键选中要卸载的 SOLA 版本，然后使用 Enter 键卸载所选择的 SOLA Toolkit。
   $ sudo sola-uninstall
   # 指定卸载当前版本
   $ sudo sola-uninstall --current
   # 指定卸载所有版本
   $ sudo sola-uninstall --all
  ```
