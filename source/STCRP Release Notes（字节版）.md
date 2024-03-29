# STCRP Release Notes

## 版本历史

| **文档版本** | **对应产品版本** | **作者** | **日期**   | **描述**                                                     |
| ------------ | ---------------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.8.0       | STCRP V1.5.1     | 希姆计算 | 2024-01-12 | 更新STCRP V1.5.1发版的信息。                                 |
| V1.7.0       | STCRP V1.5.0     | 希姆计算 | 2023-08-01 | 更新STCRP V1.5.0发版的信息。                                 |
| V1.6.0       | STCRP V1.4.1     | 希姆计算 | 2023-05-16 | 更新STCRP V1.4.1发版的信息。                                 |
| V1.5.0       | STCRP V1.4.0     | 希姆计算 | 2023-04-10 | 更新STCRP V1.4.0发版的信息。                                 |
| V1.4.0       | STCRP V1.3.1     | 希姆计算 | 2023-03-28 | 更新STCRP V1.3.1发版的信息，包括更新UOS V20安装包。          |
| V1.3.0       | STCRP V1.3.0     | 希姆计算 | 2023-01-16 | 更新STCRP V1.3.0发版的信息，包括更新麒麟V10安装包、TensorTurbo的模型算子支持信息等。 |
| V1.2.0       | STCRP V1.2.0     | 希姆计算 | 2022-12-21 | 增加远程源方式安装说明。                                     |
| V1.1.0       | STCRP V1.2.0     | 希姆计算 | 2022-11-30 | 更新STCRP V1.2.0发版的信息，包括增加CentOS 8安装包、更新HPE模块信息、更新TensorTurbo的模型算子支持信息等。 |
| V1.0.0       | STCRP V1.1.1     | 希姆计算 | 2022-09-28 | 初始版本。                                                   |

## 概述

希姆计算配套软件以STCRP的形式交付，包括异构编程环境、AI编译器，以及满足监控调试、部署集成等用途的工具。STCRP Release Notes记录了各软件的版本、功能变更、问题修复等信息。STCRP V1.5.1版本的信息如下：

| **软件**    | **版本** | **说明**                                                     | **相关文档**                                                 |
| ----------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| HPE         | V1.7.0   | 异构编程环境，安装HPE后您可以使用C/C++语言开发异构程序，方便地使用NPU进行并行计算。 | [希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/) |
| TensorTurbo | V1.12.0  | AI编译器，将来自不同框架的模型编译为希姆计算NPU上的可执行文件。 | [希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/) |
| stc-hpaa    | V1.12.0  | 精度分析工具。                                               | [希姆计算精度分析工具使用说明](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/) |
| Gem5        | V1.3.3   | 性能分析工具。                                               | [希姆计算性能分析工具使用说明（字节版）](https://docs.streamcomputing.com/_/sharing/xTEF1yKpcQBieujQ4kc20DzVnrxztKta?next=/zh/bd-6mon-acceptance/) |
| STC_DDK     | V1.3.0   | 基于TensorTurbo开发的模型编译和部署工具，通过简洁的接口即可将基于主流框架实现的模型编译为在希姆计算推理卡上执行的格式，并执行模型。 | [希姆计算STC_DDK使用说明](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/) |

支持的配套软件安装方式以及对应动作如下：

| **安装方式** | **安装包类型**                                               | **希姆计算动作**                                             | **客户动作**                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 整包方式     | HPE deb/rpm包安装、TensorTurbo以及其他工具whl包安装；所有包均支持本地源/远程源安装。 | 针对本地源安装，发送离线安装包；针对远程源安装，维护对外提供URL中的软件包。<br />提供安装文档。 | 如果使用本地源，接收并自行管理离线安装包，然后参考文档安装；如果使用远程源，参考文档安装即可。 |
| Docker方式   | HPE驱动模块deb/rpm包安装；deb/rpm包支持本地源/远程源安装。<br />HPE非驱动模块、TensorTurbo以及其他工具Docker安装；Docker安装相关的需要获取离线安装包。 | HPE驱动模块：针对本地源安装，发送离线安装包；针对远程源安装，维护对外提供URL中的软件包。<br />Docker安装相关：发送离线安装包，离线安装包中包括了Dockerfile示例。<br />提供安装文档。 | HPE驱动模块，如果使用本地源，接收并自行管理离线安装包，然后参考文档安装；如果使用远程源，参考文档安装即可。<br />Docker安装相关，自行定制Dockerfile并在HTTP Server上管理离线安装包，然后参考文档安装。 |

在Docker方式中，配套软件的模块被区分为Host侧和Docker侧，方便您后续区分升级不同类型的配套软件，维护起来更加灵活。

## HPE V1.7.0

### HPE模块版本

#### 驱动模块

| **模块**          | **版本信息** | **安装HPE后文件位置**                                  | **说明**                                                     |
| ----------------- | ------------ | ------------------------------------------------------ | ------------------------------------------------------------ |
| stc-dkms          | 1.7.0        | ko文件位于/lib/modules/$(uname -r)/updates/dkms/stc.ko | 为了适应不同内核版本，驱动基于DKMS发布版本避免手动编译的繁琐。stc-drv（Stream Computing Driver）是用于主机端与设备端交互的设备驱动。 |
| stc-kernel-common | 1.7.0        | 文件位于目录/lib/udev/rules.d/70-stc.drv.rules         | 自动添加设备节点规则。                                       |

#### 非驱动模块

| **模块**      | **版本信息** | **安装HPE后文件位置**                                        | **说明**                                                     |
| ------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| hpert         | 1.2.7        | 头文件位于/usr/local/hpe/include目录 <br />库文件位于/usr/local/hpe/lib目录 | 供主机端程序使用的接口，用于主机端与设备端的交互。           |
| hpert-dev     | 1.2.7        | 头文件位于/usr/local/hpe/riscv32npu/include目录 <br />库文件位于/usr/local/hpe/riscv32npu/lib目录 | 供设备端程序使用的接口，用于设备端信息打印、内存拷贝等操作。 |
| stcc          | 1.7.0        | 库文件位于/usr/local/hpe/lib目录 <br />可执行文件位于/usr/local/hpe/bin目录 | 异构程序编译器。                                             |
| stc-smi       | 1.7.0        | 可执行文件位于/usr/local/hpe/bin目录                         | 设备管理工具，用于管理和监视设备。                           |
| stc-prof      | 1.2.7        | 可执行文件stc-prof位于/usr/local/hpe/bin目录 <br />头文件位于/usr/local/hpe/include目录 <br />共享库文件位于/usr/local/hpe/lib目录 | 异构程序性能分析工具。                                       |
| stc-gdb       | 1.7.0        | 可执行文件位于/usr/local/hpe/bin目录                         | 异构程序调试工具。                                           |
| stc-vprof     | 1.7.0        | 可执行文件stc-vprof位于/usr/local/hpe/bin目录 <br />配置文件位于/usr/local/hpe/conf目录 <br />共享库文件位于/usr/local/hpe/java目录 | 异构程序可视化性能分析工具。                                 |
| hpe-example   | 1.7.0        | hc源文件和Makefile位于/usr/local/hpe/example目录             | 供您体验在异构环境中执行效果的示例程序。                     |
| libstc-common | 1.0.0        | 头文件位于/usr/local/hpe/include目录                         | 解决整数线性规划（Integer Linear Programming，ILP）问题的C++库。 |

### HPE操作系统支持

- Ubuntu 18.04 
- Ubuntu 20.04 
- Ubuntu 22.04
- Debian 9 
- Debian 10 
- CentOS 7
- CentOS 8
- 麒麟V10
- UOS V20
- BC-Linux V8.2
- BC-Linux V22.10
- RHEL 9.0

### HPE固件兼容

- MCU firmware V1.0.8
- NPU ctrl firmware V1.3.4

### HPE功能新增

- stc-smi
  - 实现超温关卡功能。
  - 升级时对固件做签名检查。
- 支持PCIe P2P和设置32核的HS group。

### HPE解决的问题

- stc-smi
  - 修复reset无法清除sudo或shell脚本启动的进程的问题。
  - 修复复位reset和shutdown功能关闭或重启一个NPU时，会把其他单板进程全部kill的问题。
  - 完善smi显示信息的格式。
- 其他
  - 修复ctrl-c测试npc复位后概率状态异常的问题。
  - 修复HPE hello_world在虚拟机环境下首次执行失败的问题。
  - 修复HPE升级时example目录丢失，卸载后处理脚本有删除的问题。
  - 修复ctrl-c测试中reset发生后core不能进入idle，而进入了SYNCING状态的问题。
  - 修复执行ctrl-c测试后，有进程挂死在NPU上，重启NPU后进程还在，而hello_world不能执行的问题。
  - 修复NPU上正在有程序运行时，进行复位(-r/R)操作后dmesg报错，但运行的进程仍然存在的问题。

### HPE使用限制

- 必须确保操作系统的kernel和kernel headers版本匹配。
- 如果您的服务器上已经安装了HPE，建议先卸载干净，然后再开始安装新的版本。

### HPE已知问题

- 如果报错`Failed to check for processor microcode upgrades`，可以卸载needrestart，然后重装HPE。
- 如果出现`SSL error:0909006C:PEM routines:get_name:no start line: ../crypto/pem/pem_lib.c:745`，可以打开系统UEFI中的secure_boot选项。
- 在CentOS 7 OS环境下stc-prof抓取不到stcGetErrorString接口，因此会导致统计的调用次数与其他OS环境不一致。