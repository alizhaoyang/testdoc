# 希姆计算stc-smi使用说明

## 版本历史

| **文档版本** | **对应产品版本** | **作者** | **日期**   | **描述**                                                     |
| ------------ | ---------------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.5.0       | HPE V1.7.0       | 希姆计算 | 2024-01-15 | 新增超温关卡功能。新增在升级固件时对固件文件的数字签名进行检查。 |
| V1.4.4       | HPE V1.6.2       | 希姆计算 | 2023-07-24 | 验证文档内容。                                               |
| V1.4.3       | HPE V1.6.0       | 希姆计算 | 2023-04-07 | 验证文档内容。                                               |
| V1.4.2       | HPE V1.5.0       | 希姆计算 | 2022-11-30 | 验证文档内容。                                               |
| V1.4.1       | HPE V1.4.0       | 希姆计算 | 2022-08-29 | 验证文档内容。编辑优化。                                     |
| V1.4.0       | HPE V1.3.0       | 希姆计算 | 2022-07-07 | 验证文档内容。                                               |
| V1.3.0       | HPE V1.2.2       | 希姆计算 | 2022-06-23 | 增加显示芯片利用率和SN的位置。                               |
| V1.2.0       | HPE V1.2.1       | 希姆计算 | 2022-06-02 | 整篇编辑优化。对齐最新的help message。                       |
| V1.1.1       | HPE V1.2.0       | 希姆计算 | 2022-04-11 | 验证文档内容。                                               |
| V1.1.0       | Unknown          | 希姆计算 | 2021-11-15 | 增加正常工作显示值。                                         |
| V1.0.0       | Unknown          | 希姆计算 | 2021-11-01 | 初始版本。                                                   |

## 概述

希姆计算的异构编程环境为开发、编译、运行异构程序提供了完整的工具链。stc-smi（Stream Computing System Management Interface）用于管理和监视希姆计算的NPU设备，包括查看设备信息、资源使用情况等。

> 说明：希姆计算软硬件产品相关的基本概念，请参见[希姆计算基本概念](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。

## 前提条件

在主机上部署希姆计算异构环境。具体的步骤，请参见[希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。

安装HPE后，stc-smi的二进制文件位于/usr/local/hpe/bin目录中。

## 命令说明

执行`stc-smi --help`获取stc-smi命令的使用方法：

```bash
$ stc-smi --help
Usage: stc-smi [OPTIONS]
  -h, --help           Print this help, and exit.
  <no arguments>       Display a summary of NPU devices connected to the system.
  --version            Print version number.
  -l, --list           Display NPU devices list.

DISPLAY OPTIONS:
  -q, --query          Display all info of all the NPU devices, the following options can be selected.
    [optional]
    -i, --id=          Specify the NPU device to be operated.
    -c, --cluster=     Specify cluster number.
    -p, --pci          Display NPU device pci info.
    --task             Display running tasks info.

  --query-npu=         Query information of the NPU devices, select the option from query list.
    [query list]
    index              Zero based index of the NPU.
    name               The official product name of the NPU.
    serial             This number matches the serial number physically printed on each board. It is a globally unique immutable alphanumeric value.
    power.draw         The last measured power draw for the entire board, in watts.
    temperature.npu    Core NPU temperature. in degrees C.
    utilization.npu    Percent of time over the past sample period during which one or more kernels was executing on the NPU.The sample period is 1 second.
    memory.total       Total installed NPU memory.
    memory.used        Total memory allocated by active contexts.
    [mandatory]
    --format=          Comma separated list of format options:
                       csv - comma separated values (MANDATORY).

  -r, --reset          Reset specified NPU device.
    -i, --id=          Specify the NPU device to be operated (MANDATORY).

  -m, --monitor        Set monitor related parameters.
    -i, --id=          Specify the NPU device to be operated (MANDATORY).
    --enable           Enable the temperature and voltage monitor.
    --disable          Disable the temperature and voltage monitor.
    --temperature=         Temperature threshold for alarm.
    --shutdown-threshold=  Temperature threshold for shutdown.
    --base=            Lower limit of voltage alarm value (Must set with -o, --offset).
    -o, --offset=      Voltage alarm value interval value (Must set with -b, --base).

  -S, --save           Save local config as default config.
    -i, --id=          Specify the NPU device to be operated (MANDATORY).

  -u, --upgrade=       Upgrade specified NPU ctrl firmware of specified device. This operation requires root rights.
    -i, --id=          Specify the NPU device to be operated (MANDATORY).

  --mcu-upgrade=       Upgrade specified MCU firmware of specified device. This operation requires root rights.
    -i, --id=          Specify the NPU device to be operated (MANDATORY).
```

stc-smi命令支持以下选项：

- 获取信息的类型

  | **选项**       | **描述**                                                     |
  | -------------- | ------------------------------------------------------------ |
  | `-h, --help`   | 显示stc-smi命令的帮助信息并退出，包括支持的选项以及描述。    |
  | <no arguments> | 显示连接到系统的NPU设备的摘要信息，包括设备索引、频率、功耗、内存使用、进程等。 |
  | `--version`    | 显示stc-smi的版本信息。                                      |
  | `-l, --list`   | 显示NPU设备列表。                                            |

- 显示信息的方式

  | **选项**        | **描述**                                                     |
  | --------------- | ------------------------------------------------------------ |
  | `-q, --query`   | 显示NPU设备的所有信息，可以与以下选项联合使用：<br/>- `-i, --id=`：指定要查看的NPU设备，后跟NPU设备ID。例如，`stc-smi -q -i 0`查看NPU设备0。<br/>- `-c, --cluster=`：指定要查看的NPC Cluster，后跟NPC Cluster ID。例如，`stc-smi -q -c 0`查看所有NPU设备的NPC Cluster 0。<br/>- `-p, --pci`：显示NPU设备的PCI总线相关信息，包括Bus ID、板卡标识符、传输速率等。例如，`stc-smi -q -i 0 -p`查看NPU设备0的PCI总线相关信息。<br/>- `--task`：显示任务信息，例如`stc-smi -q -i 0 -c 0 --task`查看NPU设备0的NPC Cluster 0上的任务。 |
  | `--query-npu=`  | 查询NPU设备的指定信息，需要同时指定信息项（`--query-npu=`）和显示格式（`--format=`）。如果希望查询多项信息，以英文逗号分隔即可，例如`stc-smi --query-npu=index,name --format=csv`查询NPU设备的索引和产品名称。 支持查询的信息项如下：<br/>- `index`：NPU设备的索引，从0开始。<br/>- `name`：NPU的产品名称。<br/>- `serial`：全球唯一的不可变的字母数字编号，与实际印刷在每块板上的序列号相匹配。<br/>- `power.draw`：整个板卡的功耗，单位为W。<br/>- `temperature.npu`：NPU核心的温度，单位为℃。<br/>- `utilization.npu`：在上个采样时间（1秒）内，NPU上执行核函数所占采样时间的百分比。<br/>- `memory.total`：NPU全局内存的总大小。<br/>- `memory.used`：运行任务的上下文占用的全局内存大小，包括软件栈占用的固定内存、运行代码占用的内存等。<br/>支持的显示格式如下：<br/>- `csv`：逗号分隔格式。 |
  | `-r, --reset`   | 重置指定的NPU设备，可以与以下选项联合使用：<br/>- `-i, --id=`：指定要重置的NPU设备。 |
  | `-m, --monitor` | 设置监视器相关参数，可以与以下选项联合使用：<br /> `-i, --id=`：指定要监控的NPU设备。<br />`--enable`：开启温度和电压监视器。<br />`--disable`：关闭温度和电压监视器。<br />`--temperature=`：温度告警阈值。<br />`--shutdown-threshold=`：关卡温度阈值。<br />`--base=`：电压报警值下限（必须用配合`-o，--offset`设置)。<br />`-o, --offset=`：电压报警值间隔值（必须用配合`-b，--base`设置)。 |
  | `-S, --save`    | 保存当前监视器配置为默认配置。可以与以下选项联合使用：<br /> `-i, --id=`：指定要保存配置的NPU设备。 |
  | `-u, --upgrade` | 升级指定设备的NPU ctrl firmware固件，需要有root权限。要求与以下选项联合使用：<br /> `-i, --id=`：指定要升级的NPU设备。 |
  | `--mcu-upgrade` | 升级指定设备的MCU固件，需要有root权限。要求与以下选项联合使用：<br /> `-i, --id=`：指定要升级的NPU设备。 |

## 命令示例

本章节以在HPE V1.7.0中stc-smi命令为例演示执行效果。

### 显示摘要信息

1. 运行示例目标程序。

   ```bash
   $ cat hello_world.hc
   #include <npurt.h>
   #include <hpe.h>
   #include <unistd.h>
   
   #define NCORE 8
   
   int *global_data;
   
   __global__ void hello(void) {
       while (1) {
           printf("hello world from core %d/%d.\n", CoreID, CoreNum);
       }
   }
   
   int main(void) {
       stcMalloc((void **)&global_data, 1024 * 1024);
       hello<<<NCORE>>>();
       sleep(10);
       stcFree(global_data);
       stcDeviceSynchronize();
       return 0;
   }
   $ stcc --rtlib=compiler-rt hello_world.hc -o hello_world
   $ ./hello_world
   ```

2. 查看摘要信息。

   ```bash
   $ stc-smi
   +------------------------------------------------------------------------------+
   |    STC-SMI: 1.7.0                            Driver Version: 1.7.0           |
   +------------------------------------------------------------------------------+
   |  NPU        Name     Frequency |         Bus-Id |                   NPU-Util |
   |  Sta        Temp         Power |   ClusterCount |        Memory Used / Total |
   +==============================================================================+
   |    0     STCP920         1000M |   0000:03:00.0 |                        0 % |
   |   on         29C    30W / 160W |              4 |           624.00M / 16.00G |
   +------------------------------------------------------------------------------+
   
   +------------------------------------------------------------------------------+
   | Processes:                                                                   |
   |  NPU  CLUSTER          Pid                          ProcessName   MemoryUsed |
   +==============================================================================+
   | There is no process.                                                         |
   +------------------------------------------------------------------------------+
   ```

   各项信息的描述如下：

   | **项目**           | **示例值**      | **描述**                                               |
   | ------------------ | --------------- | ------------------------------------------------------ |
   | STC-SMI            | 1.7.0           | stc-smi工具的版本信息。                                |
   | Driver Version     | 1.7.0           | NPU Driver的版本信息。                                 |
   | NPU                | 0               | NPU设备的索引。                                        |
   | Name               | STCP920         | NPU设备的产品名称。                                    |
   | Frequency          | 1000M           | NPU设备的频率信息。                                    |
   | Sta                | N/A             | NPU设备板卡状态。                                      |
   | Temp               | 36.6℃           | NPU设备核心的温度信息，合理范围为25℃～85℃。            |
   | Power              | 33.7W / 160W    | NPU设备的功耗信息，包括实时功耗/额定功耗。             |
   | Bus-Id             | 0:65:00.0       | NPU设备所使用PCI总线的ID。                             |
   | NPU-Util           | 25 %            | NPU设备的使用率，即使用的NPC占NPU设备内所有NPC的比例。 |
   | ClusterCount       | 4               | NPU设备内NPC Cluster的数量。                           |
   | Memory Used /Total | 624.05M /16.00G | NPU设备的内存信息，包括已用内存/总内存的大小。         |
   | NPU                | 0               | 执行中任务所使用NPU设备的ID。                          |
   | CLUSTER            | 0               | 执行中任务所使用NPC Cluster的ID。                      |
   | Pid                | 583900          | 执行中任务的系统进程ID。                               |
   | processName        | hello_world     | 执行中任务的系统进程名称。                             |
   | MemoryUsed         | 45.62K          | 执行中任务使用的内存大小。                             |

   示例摘要信息中，NPU设备的Memory Used为624.05M，其中624M是软件栈固定使用的内存大小，剩下的则是运行目标程序使用的内存，即Processes中的MemoryUsed。

### 显示工具版本

```bash
$ stc-smi --version
stc-smi version: 1.7.0-g42a0300
```

示例结果显示版本号为1.7.0-g42a0300。

### 显示设备列表

```bash
$ stc-smi --list
NPU: 0
        STCP920         BUS-ID: 0:65:00.0 (00PCK08700050605)
```

示例结果显示检测到一个NPU设备，产品名称为STCP920，所使用PCI总线的ID为0:65:00.0，设备序列号为00PCK08700050605。

### 显示指定设备的信息

您可以根据需要查看指定NPU设备的信息：

- 显示设备的所有信息。

  ```bash
  $ stc-smi -q
  NPU: 0
          Product name: STCP920
          Product brand: STREAMCOMPUTING
          Sn: 01PSF60000170610
          Chip count: 1
          Temperature:   29C
          AlertTemperature:   90C
          ShutdownTemperature:   90C
          Status: on
          Power:   30W
          Cluster count: 4
          Frequency: 1000M
          Bus: 0000:03:00.0
          Vendor: 23e2 Device: 0100
          Current link speed: 16.0 GT/s PCIe
          Max link speed: 16.0 GT/s PCIe
          Current link width: 16
          Max link width: 16
          Write bytes: 0B
          Read bytes: 0B
          HPE version: 1.7.0
          Driver: 1.7.0
          Chip version: 20200102
          MCU firmware: 1.0.8
          NPU ctrl firmware: 1.3.4
                  Cluster 0:
                  Frequency: 1000M        Core count: 8   DMA count: 2    Util: 0 %       Status: WORK    Memory Used / Total: 156.00M / 4.00G
                  Npc status:      IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE
                  Cluster 1:
                  Frequency: 1000M        Core count: 8   DMA count: 2    Util: 0 %       Status: WORK    Memory Used / Total: 156.00M / 4.00G
                  Npc status:      IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE
                  Cluster 2:
                  Frequency: 1000M        Core count: 8   DMA count: 2    Util: 0 %       Status: WORK    Memory Used / Total: 156.00M / 4.00G
                  Npc status:      IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE
                  Cluster 3:
                  Frequency: 1000M        Core count: 8   DMA count: 2    Util: 0 %       Status: WORK    Memory Used / Total: 156.00M / 4.00G
                  Npc status:      IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE    IDLE
  ```

- 显示NPC Cluster 0的信息。

  ```bash
  $ stc-smi -q -i 0 -c 0
  NPU: 0
                  Cluster 0:
                  Frequency: 1000M        Core count: 8   DMA count: 2    Util: 100 %     Status: WORK    Memory Used/Total: 156.05M/4.00G
                  Npc status:      RUNNING         RUNNING         RUNNING         RUNNING         RUNNING         RUNNING   RUNNING         RUNNING
                      Cluster: 0  Process: hello_world    Pid: 584348     Global mem used: 45.62K
  ```

  各项的含义如下：

  | **项目**            | **示例值**       | **描述**                                                     |
  | ------------------- | ---------------- | ------------------------------------------------------------ |
  | Product Name        | STCP920          | NPU设备的产品名称。                                          |
  | Product Brand       | STREAMCOMPUTING  | NPU设备的品牌名称。                                          |
  | Sn                  | 01PSF60000170610 | NPU设备的SN。每个NPU设备的SN是唯一的，不会重复。             |
  | Chip count          | 1                | NPU设备的数量。                                              |
  | Temperature         | 29C              | NPU设备核心的温度信息，合理范围为25℃～85℃。                  |
  | AlertTemperature    | 90C              | NPU设备告警温度。                                            |
  | ShutdownTemperature | 90C              | NPU设备温度。                                                |
  | Status              | on               | NPU设备状态。                                                |
  | Power               | 30W              | NPU设备的实时功耗，额定功耗为160W。                          |
  | Cluster count       | 4                | NPU设备内NPC Cluster的数量。                                 |
  | Frequency           | 1000M            | NPU设备的频率信息。                                          |
  | Bus                 | 0000:03:00.0     | NPU设备所使用PCI总线的ID。                                   |
  | Vendor              | 23e2             | PCIe厂商的ID。                                               |
  | Current link speed  | 16.0 GT/s PCIe   | PCIe设备当前的连接速度，视主机的PCIe插槽而定，可能值包括16GT/S（GEN4）、8GT/S（GEN3）、5GT/S （GEN2）。 |
  | Max link speed      | 16.0 GT/s PCIe   | PCIe设备支持的最高连接速度，视主机的PCIe插槽而定，可能值包括16GT/S（GEN4）、8GT/S（GEN3）、5GT/S （GEN2）。 |
  | Current link width  | 16               | PCIe设备当前连接的通道数。                                   |
  | Max link width      | 16               | PCIe设备支持连接的最大通道数。                               |
  | Write bytes         | 0B               | PCIe设备的写数据大小统计，数值动态变化。                     |
  | Read bytes          | 0B               | PCIe设备的读数据大小统计，数值动态变化。                     |
  | HPE version         | 1.7.0            | 当前主机安装HPE异构编程环境的版本信息。                      |
  | Driver              | 1.7.0            | NPU Driver的版本信息。                                       |
  | Chip version        | 20200102         | STCP920芯片的产品版本信息。                                  |
  | MCU firmware        | 1.0.8            | MCU固件的版本信息。                                          |
  | NPU ctrl firmware   | 1.3.4            | NPU ctrl固件的版本信息。                                     |
  | Cluster             | 0                | NPC Cluster的ID。                                            |
  | Frequency           | 1000M            | NPC Cluster的频率。                                          |
  | Core count          | 8                | 当前NPC Cluster中NPC的数量。                                 |
  | DMA count           | 2                | 当前NPC Cluster中DMA通道的数量。                             |
  | Util                | 0 %              | 当前NPC Cluster的使用率，即使用的NPC占当前NPC Cluster内所有NPC的比例。 |
  | Status              | WORK             | 当前NPC Cluster的状态信息，可能值： - WORK：正常工作状态。 - DEAD：错误状态。 |
  | Memory Used/Total   | 156.00M/4.00G    | 当前NPC Cluster的内存信息，包括已用内存/总内存的大小。       |
  | Npc status          | IDLE             | 当前NPC Cluster中各个NPC的状态信息，可能值：<br/>- IDLE：闲置中，未执行任务。<br/>- RUNNING：正在执行任务。<br/>- STOP：停止使用NPC。例如终止运行中的任务时，会先让NPC进入STOP状态，然后再复位设备进入IDLE状态。<br/>- NPC-EXCEPTION：软件错误导致NPC异常。<br/>- UNKNOWN：未知错误导致NPC异常。 |

### 查询设备的指定信息

您可以根据需要查看NPU设备的指定信息：

```bash
$ stc-smi --query-npu=index,utilization.npu,memory.used --format=csv
index, utilization.npu [%], memory.used [MiB]
0, 25 %, 625 MiB

$ stc-smi --query-npu=index,temperature.npu,power.draw --format=csv      
index, temperature.npu, power.draw [W]
0, 36, 33.55 W
```

> 说明：`--query-npu`支持指定的信息项，请参见*命令说明*章节。

按照逗号分隔格式显示查询的信息。各项的含义如下：

| **项目**        | **示例值** | **描述**                                               |
| --------------- | ---------- | ------------------------------------------------------ |
| index           | 0          | NPU设备的索引。                                        |
| utilization.npu | 25 %       | NPU设备的利用率，即使用的NPC占NPU设备内所有NPC的比例。 |
| memory.used     | 625 MiB    | NPU设备已使用内存的大小。                              |
| temperature.npu | 36         | NPU设备核心的温度信息。                                |
| power.draw      | 33.55 W    | NPU设备的实时功耗。                                    |

### 设置关卡温度阈值

为防止板卡长期在超过设计的最高温度下运行，从而影响板卡的使用寿命。您可以设置关卡温度阈值。当板卡的温度达到了关卡温度阈值时，将会清理当前板卡所有任务并设置为关卡状态。

1. 温度监控功能默认不开启，需要先开启监控功能：

   ```bash
   #开启监控功能
   $ stc-smi -m --enable -i 0
   ```

2. 设置NPU设备0的关卡温度阈值为90度：

   ```bash
   #设置
   $ stc-smi -m --shutdown-threshold=90 -i 0
   ```

如果板卡因超过温度阈值自动关卡，在重新使用板卡时您需要手动开卡才能解除关卡状态：

```Bash
$ stc-smi --startup -i 0 
```

### 复位设备

在NPU设备状态异常时，您可以尝试复位设备。复位NPU设备0的命令示例如下：

> 警告：执行设备复位命令前，请确认设备上没有正在执行的任务。复位是针对整个NPU设备的，而非单个NPC Cluster。

```bash
$ stc-smi -r -i 0
```
