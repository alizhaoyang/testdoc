<div align='center' >
    <h1>希姆计算 HPE v1.1.0 安装指南</h1>
    <br>
    Stream Computing Inc. <br>
    2021-12-20 
    <br><br><br><br><br><br><br><br><br><br>
</div>












#### 版权声明

本文档版权为北京希姆计算科技有限公司所有（Copyright️2019-2021）。未经本公司书面许可，任何单位或个人不得以任何形式或通过任何途径，包括使用影印、录制在内的电子或机械手段对本文档的部分或者全部进行复制或传播。

####  警告和承诺

我们尽最大努力使本文档尽可能的完善和准确，但疏漏和缺陷之处在所难免。北京希姆计算科技有限公司保留随时修改更新文档的权利。

####  反馈信息

您的反馈意见将使我们受益匪浅。如果您对本文档有什么疑问、意见或建议，请与我们联系，感谢您对我们的支持和帮助。


<div style="page-break-after:always;"></div>

# 1 介绍

HPE是北京希姆计算科技有限公司提供的异构编程环境，允许开发者使用C/C++高级编程语言开发异构程序，并像使用CPU一样，使用NPU来进行并行计算，可以显著提高计算性能。

### 1.1 系统需求

要使用HPE在您的系统中，您需要确保系统满足下列条件：

- 安装有HPE所支持的NPU
- 系统为HPE支持的Linux版本

HPE开发环境和主机开发环境紧密相关，包括主机的编译器和C运行时库等。所以，推荐使用被验证过的Linux发行版。在未被验证过的Linux发行版上使用HPE可能可以运行，但这并不总是能得到保障。

表一: HPE v1.1.0 Linux发行版验证列表

| 发行版       | 内核           | GLIBC     |
| ------------ | -------------- | --------- |
| Debian 9     | 4.9.0/4.19.117 | 2.24/2.28 |
| Debian 10    | 4.19.0/5.4.56  | 2.28      |
| Ubuntu 18.04 | 4.15.0/5.4.0   | 2.27      |
| Ubuntu 20.04 | 5.8.0/5.4.0    | 2.31      |
| CentOS 8     | 4.18.0         | 2.28      |

### 1.2 关于本文档

本文档需要读者熟悉Linux环境和基于命令行的C程序编译。读者不需要有HPE或并行计算经验。

> 注意：
> 很多本文档中使用的命令需要超级用户权限。在多数Linux发行版上，这将需要您使用root账户登录。在使能了sudo工具的系统上，用户可以添加sudo前缀获取超级用户权限。


# 2 HPE v1.1.0安装包介绍

希姆计算发布的HPE安装包已经支持目前主流Linux发行版系统(Debian,Ubuntu，CentOS等)，提供安装包如下：

- Debian9版本安装包：

  hpe-repo-debian9-1-1-local_1.1.0_amd64.deb

- Debian10版本安装包：

  hpe-repo-debian10-1-1-local_1.1.0_amd64.deb

- Ubuntu18.04版本安装包：

  hpe-repo-ubuntu1804-1-1-local_1.1.0_amd64.deb

- Ubuntu20.04版本安装包：

  hpe-repo-ubuntu2004-1-1-local_1.1.0_amd64.deb

- CentOS8版本安装包：

  hpe-repo-centos8-1-1-local_1.1.0_x86_64.rpm
  
> 注意：
> 希姆计算除提供以上标准版HPE版本外，还提供以mini-开头的运行板。运行版仅提供对希姆计算TensorTurbo的运行支持。

# 3 安装前准备

<div id = '1'>安装HPE前需要进行下列准备工作：</div>

- 验证系统安装有HPE支持的NPU
- 验证系统是被支持的Linux发行版
- 验证系统有正确的内核头文件和开发包
- 获得希姆计算HPE安装包

### 3.1 验证系统安装有希姆计算的NPU板卡

验证系统可以识别所安装的NPU板卡。例如对于STCP920，可以执行下面的命令行：

```
sudo lspci -vd 23e2: 
```

### 3.2 验证系统是被支持的Linux发行版

确定当前系统的发行版信息，可以使用下面的命令行：

```
uname -m && cat /etc/*release
```

### 3.3 验证系统有正确的内核头文件

hpe驱动安装要求kernel headers与系统内核匹配。例如，如果系统内核是5.4.0-77，那么需要安装5.4.0-77的内核头文件。

##### 3.3.1 Debian/Ubuntu

可以使用下列命令行查看系统是否具有内核匹配的头文件：

```
sudo dpkg --get-selections | grep $(uname -r)
```

如果没有可以使用下列命令行进行安装：

```
sudo apt-get install linux-headers-$(uname -r)
```

##### 3.3.2 CentOS

可以使用下列命令行查看系统是否具有内核匹配的头文件：

```
sudo yum list | grep kernel-headers
```

如果没有可以使用下列命令行进行安装：

```
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

### 3.4 获得希姆计算HPE安装包

请联系希姆计算销售人员获得HPE安装包。


# 4 使用包管理器安装HPE

包管理器安装方式与系统的包管理系统对接。HPE的deb或rpm 包是存储库包。存储库包会在系统上安装包含安装包的本地存储库。然后通知包管理器在哪里可以找到实际的安装包，但不会安装它们。安装过程由几个步骤组成，不同的系统命令行不同。

### 4.1 Debian/Ubuntu

##### 4.1.1 [执行安装前准备](#1)

##### 4.1.2 安装存储库及GPG密钥

```
sudo dpkg -i hpe-repo-{OS}-1-0-local_1.0.0_amd64.deb
sudo apt-key add /var/hpe-repo-{OS}-1-0-local/hpe.pub
```

##### 4.1.3 运行apt-get安装HPE的相关组件

```
sudo apt update
sudo apt install hpe -y
```

安装完成后, 所有文件会被放置在/usr/local/hpe/下.

##### 4.1.4 查看是否安装成功（示例随版本不同可能不同） 

```
lsmod | grep stc
stc 352256 0
```

##### 4.1.5 查看hpe目录（示例随版本不同可能不同） 

```
ls /usr/local/hpe/ 
bin example include lib libexec modules riscv32npu share version.txt
```

##### 4.1.6 试用HPE

```
cd /usr/local/hpe/example
make
./hello_world
```

将看到如下输出:

```
hello world from core 0/8.
hello world from core 1/8.
hello world from core 2/8.
hello world from core 3/8.
hello world from core 4/8.
hello world from core 5/8.
hello world from core 6/8.
hello world from core 7/8.
```

### 4.2 CentOS

##### 4.2.1 [执行安装前准备](#1)

##### 4.2.2 安装存储库

```
sudo yum install -y hpe-repo-centos8-1-1-local_1.1.0_amd64.deb
```

##### 4.2.3 运行yum安装HPE的相关组件

```
sudo yum update
sudo yum install hpe -y
```

安装完成后, 所有文件会被放置在/usr/local/hpe/下.

##### 4.2.4 查看是否安装成功（示例随版本不同可能不同） 

```
lsmod | grep stc
```
输出为:
```
stc 352256 0
```

##### 4.2.5 查看hpe目录（示例随版本不同可能不同） 

```
/usr/local/hpe/ 
```
输出为：
```
bin example include lib libexec modules riscv32npu share version.txt
```
##### 4.2.6 试用HPE

```
cd /usr/local/hpe/example
make
./hello_world
```

将看到如下输出:

```
hello world from core 0/8.
hello world from core 1/8.
hello world from core 2/8.
hello world from core 3/8.
hello world from core 4/8.
hello world from core 5/8.
hello world from core 6/8.
hello world from core 7/8.
```

# 5 卸载HPE

使用下列步骤可以从您的系统中卸载HPE。不同的操作系统命令行不同：

### 5.1 Debian/Ubuntu

```
sudo apt-get --purge remove "stc-*" "hpe-*" "hpe" -y
```

如果要清理卸载，可以执行：

```
sudo apt-get autoremove
```

### 5.2 CentOS

```
sudo yum remove  "stc-*" "hpe-*" "hpe" -y
```

如果要清理卸载，可以执行：

```
sudo yum clean all
```