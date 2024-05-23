# mx-smi 用户手册

## **概述**

mx-smi（MOFFETT System Management Interface）是一款基于 MOFFETT Management Library （MXML） 开发的命令行工具，旨在实现对墨芯 AI 计算卡的高效管理与精准监控。您只需通过这一便捷工具，即可轻松查询墨芯 AI 计算卡的实时状态，并在具备相应权限的条件下，对设备的部分状态及设置进行灵活调整。

## 前提条件

在主机上安装 SOLA。具体的步骤，请参见《SOLA Toolkit 安装指南》。 安装 SOLA 后，mx-smi 的二进制文件位于`/usr/bin` 目录下。

## 命令说明

### 命令格式

```Bash
$ mx-smi [选项] [子命令]
```

### 基本命令

执行 `mx-smi --help` 获取 mx-smi 命令的使用方法： 

```Bash
$ mx-smi --help
Moffett System Management Interface Application  v2.3.0
Usage: mx-smi [OPTIONS] [SUBCOMMAND]

Options:
  -h,--help                   Print this help message and exit
  --version                   Display program version information and exit

Subcommands:
  list                        Draw chart to show summary of devices.
  query                       Display devices details.
  select                      Print properties you explicit specified.
  reboot                      Reboot device
```

### **子命令**

mx-smi 提供多种子命令查看或操作设备，子命令使用说明详见下文。

#### **子命令 list**

以设备 （Device） 为单位输出概览信息，若是单卡多设备的产品（例如 S30），会标注位于同一张卡的设备。选项列表如下：

- <无选项> : 默认输出所有设备的概览信息，包括设备索引、频率、功耗、内存使用、进程等信息。。
- -i， –index : 用于输出指定设备 id 的概览信息。可使用以下格式输入：
  - 空格分隔，例如 -i 0 1 2
  - 花括号中用逗号分隔 ，例如 -i {0，1，2}

#### **子命令** **query**

输出设备详细信息。选项列表如下：

- <无选项> : 默认输出所有设备的详细信息。

- -i， –index : 用于指定设备 id 的详细信息。可使用以下格式输入：

  - 空格分隔，例如 -i 0 1 2
  - 花括号中用逗号分隔，例如 -i {0，1，2}

- -d， –display : 用于查询设备的指定信息。可接受的参数不限定大小写，参数列表如下：

  | **参数**    | **说明**                |
    | ----------- | ----------------------- |
    | MEMORY      | 设备内存信息            |
    | UTILIZATION | 设备使用率信息          |
    | TEMPERATURE | 设备的温度信息          |
    | POWER       | 设备的功耗信息          |
    | FREQUENCY   | 设备的频率信息          |
    | VOLTAGE     | 设备的电压信息          |
    | PIDS        | 执行中任务的系统进程 ID |
    | ECC         | ECC 信息                |

  可使用以下格式输入：

  -  空格分隔，例如 -d MEMORY POWER

  -  花括号中用逗号分隔，例如-d {MEMORY，POWER}

#### **子命令 select**

按指定的内容和顺序，输出设备的属性信息。

- -f， –field : **必选项**，用于指定需要输出的信息类型，并按指定的顺序输出。可接受的参数不限定大小写，参数列表如下：

  | **参数**                                       | **说明**                                                    |
    | ---------------------------------------------- | ----------------------------------------------------------- |
    | timestamp                                      | 输出当前时间戳。例如 “2024-01-02 15:04:05”                  |
    | driver_version                                 | 驱动的版本                                                  |
    | sola_version                                   | SOLA 版本                                                   |
    | count                                          | 检测到的设备（Devices）数量                                 |
    | index                                          | 设备 ID （根据 Bus ID 生成）                                |
    | name                                           | 设备名称（产品名称）                                        |
    | cores                                          | 设备核心数量                                                |
    | firmware_version                               | 固件版本                                                    |
    | mcu_version                                    | MCU 版本                                                    |
    | serial                                         | 序列号（同一张卡拥有相同序列号，不同卡的序列号不同）        |
    | uuid                                           | UUID （每个设备全局唯一 ID）                                |
    | board                                          | 主板 ID                                                     |
    | pci.bus                                        | 16 进制输出 PCI bus                                         |
    | pci.device                                     | 16 进制输出 PCI device                                      |
    | pci.domain                                     | 16 进制输出 PCI domain                                      |
    | pci.bus_id                                     | 16 进制输出 PCI bus id，格式为 “domain:bus:device.function” |
    | pci.device_id                                  | 16 进制输出 PCI vendor device id                            |
    | pci.sub_device_id                              | 16 进制输出 PCI Sub System id                               |
    | memory.total                                   | SPU 总内存（单位：MiB）                                     |
    | memory.reserved                                | SPU 预留内存（单位：MiB）                                   |
    | memory.used                                    | SPU 已使用内存（单位：MiB）                                 |
    | memory.free                                    | SPU 总空闲内存（单位：MiB）                                 |
    | utilization                                    | 使用率（各核心上的平均使用率）                              |
    | ecc.errors.corrected.volatile.device_memory    | 已纠正的易失性设备内存错误数量                              |
    | ecc.errors.corrected.aggregate.device_memory   | 已纠正的聚合设备内存错误数量                                |
    | ecc.errors.uncorrected.volatile.device_memory  | 未纠正的易失性设备内存错误数量                              |
    | ecc.errors.uncorrected.aggregate.device_memory | 未纠正的聚合设备内存错误数量                                |
    | temperature                                    | 设备的温度，单位为摄氏度（℃）                               |
    | power.draw                                     | 设备的当前功率                                              |
    | power.limit                                    | 设备的最大功率，单位为瓦特（W）                             |
    | frequency                                      | 设备的频率，单位为兆赫兹（MHz）                             |
    | volatile                                       | 设备的电压，单位为毫伏（mV）                                |

  您可使用以下格式输入：

  -  空格分隔，例如-f index pci.bus_id

  -  花括号中用逗号分隔，例如-f {index，pci.bus_id}

- -i， –index : 用于指定设备 id （device index），可使用以下格式输入：

  - 空格分隔，例如 -i 0 1 2
  - 花括号中用逗号分隔，例如 -i {0，1，2}

- –noheader : 不输出属性名表头

#### **子命令 reboot**

重启指定的设备并加载固件。

> **注意**：执行 reboot 命令需要获取 sudo 权限。

选项列表如下：

| **选项**   | **说明**                                                     |
| ---------- | ------------------------------------------------------------ |
| –all       | 重启所有设备。                                               |
| -i, –index | 通过指定设备的 index 来重启指定的设备。输入格式示例如下：空格分隔 : -i 0 1 2花括号中用逗号分隔 ，例如 -i {0，1，2} |

## 命令示例

### 显示摘要信息

以 SPU 卡为单位显示摘要信息。

```Bash
$ mx-smi
Mon Apr 15 17:54:26 2024
╭──────────────────────────────────────────────────────────────────────────────────╮
│MOFFETT-SMI 2.3.0         Driver Version 3.5.2         SOLA Version 3.5.3         │
╰──────────────────────────────────────────────────────────────────────────────────╯
 Card
┌──────────────────────────────────────────────────────────────────────────────────┐
│Index  Name Freq.  Voltage Temp. Pwr  Util  Bus ID    Memory-Usage        SN      │
├──────────────────────────────────────────────────────────────────────────────────┤
│Card1  S30  700MHz  930mV   53C  110W 99%  0:2d:00.0 111MiB/61440MiB 2023243080104│
├──────────────────────────────────────────────────────────────────────────────────┤
│Card2  S30  700MHz  930mV   55C  100W 99%  0:3a:00.0 111MiB/61440MiB 2023243080099│
└──────────────────────────────────────────────────────────────────────────────────┘
 Processes
┌──────────────────────────────────────────────────────────────────────────────────┐
│Index    PID   Process Name                                           Memory Usage│
├──────────────────────────────────────────────────────────────────────────────────┤
│ 1:0    31863  mx-qual                                                       37MiB│
│ 1:1    31863  mx-qual                                                       37MiB│
│ 1:2    31863  mx-qual                                                       37MiB│
│ 2:3    31863  mx-qual                                                       37MiB│
│ 2:4    31863  mx-qual                                                       37MiB│
│ 2:5    31863  mx-qual                                                       37MiB│
└──────────────────────────────────────────────────────────────────────────────────┘
```

- Card （卡信息）

   > **注意：**以卡为单位展示时，Index/Name/SN 同一张卡的值相同。Pwr/Memory-Usage，展示的值为卡中所有设备的总和。其他项均展示卡中 Bus ID 最低的设备对应的信息。

  | **参数**     | **说明**                                                     |
    | ------------ | ------------------------------------------------------------ |
    | Index        | 卡的索引                                                     |
    | Name         | 卡名 （同一张卡一样）                                        |
    | Freq.        | 卡的频率                                                     |
    | Voltage      | 卡的电压                                                     |
    | Temp.        | 卡的温度                                                     |
    | Pwr          | power，卡的功率                                              |
    | Util         | 卡的使用率 （负载）                                          |
    | Bus ID       | 总线 ID，格式为 “domain:bus:device.function”                 |
    | Memory-Usage | 卡的内存使用情况，格式为 “usage / total” ，即（已使用内存 / 内存总量） |
    | SN           | 卡的序列号，是卡的唯一标识 。                                |

- Processes （进程信息）

   | **参数**     | **说明**                                                     |
    | ------------ | ------------------------------------------------------------ |
    | index        | 运行该进程的卡编号以及对应的设备 ID，输出格式为：（卡编号：设备 ID） |
    | PID          | 进程 ID。                                                    |
    | Process Name | 进程名，最大展示长度为 32 个字符，超出长度后以 … 省略开头的字符 |
    | Memory Usage | 进程占用的内存大小                                           |

### 显示工具版本

```Bash
$ mx-smi --version
2.3.0
```

### 显示设备列表

```Bash
$ mx-smi list -i {0,1,2,3,4,5}
Mon Apr 15 17:48:06 2024
╭───────────────────────────────────────────────────────────────────────────────╮
│MOFFETT-SMI 2.3.0        Driver Version 3.5.2        SOLA Version 3.5.3        │
╰───────────────────────────────────────────────────────────────────────────────╯
 Devices
┌───────────────────────────────────────────────────────────────────────────────┐
│Index  Name Freq.  Voltage Temp. Pwr Util  Bus ID   Memory-Usage       SN      │
├───────────────────────────────────────────────────────────────────────────────┤
│  +    S30                                                        2023073080040│
│  0         700MHz  930mV   44C  2W   0%  0:2d:00.0 0MiB/20480MiB              │
│  1         700MHz  925mV   44C  2W   0%  0:2e:00.0 0MiB/20480MiB              │
│  2         700MHz  930mV   44C  2W   0%  0:2f:00.0 0MiB/20480MiB              │
├───────────────────────────────────────────────────────────────────────────────┤
│  +    S30                                                        2023423080188│
│  3         700MHz  930mV   45C  2W   0%  0:3a:00.0 0MiB/20480MiB              │
│  4         700MHz  925mV   48C  1W   0%  0:3b:00.0 0MiB/20480MiB              │
│  5         700MHz  925mV   47C  2W   0%  0:3c:00.0 0MiB/20480MiB              │
└───────────────────────────────────────────────────────────────────────────────┘
 Processes
┌───────────────────────────────────────────────────────────────────────────────┐
│Index    PID   Process Name                                        Memory Usage│
├───────────────────────────────────────────────────────────────────────────────┤
│   [No running processes found]                                                │
└───────────────────────────────────────────────────────────────────────────────┘
```

> **注意**：以设备为单位展示时，若为单卡多设备产品（例如 S30），有独立的一行展示同一张卡的卡名和序列号。

- Devices （设备信息）

   | **参数**     | **说明**                                                     |
    | ------------ | ------------------------------------------------------------ |
    | Index        | 设备的索引                                                   |
    | Name         | 卡名                                                         |
    | Freq.        | 设备的频率                                                   |
    | Voltage      | 设备的电压                                                   |
    | Temp.        | 设备的温度                                                   |
    | Pwr          | power，设备的功率                                            |
    | Util         | 设备的使用率 （负载）                                        |
    | Bus ID       | 总线 ID，格式为 “domain:bus:device.function”                 |
    | Memory-Usage | 设备的内存使用情况，格式为 “usage / total” （已使用 / 总量） |
    | SN           | 卡的序列号，是卡的唯一标识 。                                |

- Processes （进程信息）

   | **参数**     | **说明**                                                     |
    | ------------ | ------------------------------------------------------------ |
    | index        | 运行该进程的设备 ID                                          |
    | PID          | 进程 ID                                                      |
    | Process Name | 进程名称，最大展示长度为 32 个字符，超出长度后以 … 省略开头的字符 |
    | Memory Usage | 进程占用的内存大小                                           |

### 显示指定设备的信息

```Bash
$ mx-smi list -i 0
Mon Mar  4 17:48:06 2024
╭───────────────────────────────────────────────────────────────────────────────╮
│MOFFETT-SMI 2.3.0        Driver Version 3.5.1        SOLA Version 3.5.0        │
╰───────────────────────────────────────────────────────────────────────────────╯
 Devices
┌───────────────────────────────────────────────────────────────────────────────┐
│Index  Name Freq.  Voltage Temp. Pwr Util  Bus ID   Memory-Usage       SN      │
├───────────────────────────────────────────────────────────────────────────────┤
│  +    S30                                                        2023073080040│
│  0         700MHz  930mV   44C  2W   0%  0:2d:00.0 0MiB/20480MiB              │
└───────────────────────────────────────────────────────────────────────────────┘
 Processes
┌───────────────────────────────────────────────────────────────────────────────┐
│Index    PID   Process Name                                        Memory Usage│
├───────────────────────────────────────────────────────────────────────────────┤
│   [No running processes found]                                                │
└───────────────────────────────────────────────────────────────────────────────┘
```

### **查询**设备的详细信息

```Bash
$ mx-smi query -i 0
Timestamp                             : Mon Apr 15 18:01:32 2024
Driver Version                        : 3.5.2
SOLA Version                          : 3.5.3

Attached Devices                      : 6

Device 0
    Product Name                      : 01S30-00A
    SPU Cores Number                  : 4
    FW Version                        : 1.0.14
    MCU Version                       : 4X08
    Serial Number                     : 2023243080104
    UUID                              : 01000000-0000-0000-0000-00AKS0216498
    Board Id                          : 0x001
    PCI
        Bus                           : 0x2d
        Device                        : 0x00
        Domain                        : 0x0000
        Bus Id                        : 0:2d:00.0
        Device Id                     : 0x70301f36
        Sub System Id                 : 0x70001f36
    Memory Usage
        Total                         : 20480 MiB
        Reserved                      : 4484 MiB
        Used                          : 0 MiB
        Free                          : 15996 MiB
    Utilization
        Core 0                        : 0 %
        Core 1                        : 0 %
        Core 2                        : 0 %
        Core 3                        : 0 %
        Max                           : 0 %
        Min                           : 0 %
        Avg                           : 0 %
    ECC Errors
        Volatile
            Device Memory Corrected   : 0
            Device Memory Uncorrected : 0
        Aggregate
            Device Memory Corrected   : 0
            Device Memory Uncorrected : 0
    Temperature                       : 43 C
    Power
        Power Draw                    : 2 W
        Power Limit                   : 83 W
    Frequency                         : 700 MHz
    Voltage                           : 930 mV
    Processes
```

### 查询设备的指定信息

默认第一行展示输出的属性名。后续每一行为用户指定的设备的属性信息。

```Bash
$ mx-smi select -f {index,pci.bus_id,board} -i 3 2 1
index, pci.bus_id, board
3, 0:3a:00.0, 2
2, 0:2f:00.0, 1
1, 0:2e:00.0, 1
```

### **重启设备**

在 SPU 设备状态异常时，您可以尝试复位设备。复位 SPU 所有设备的命令示例如下： 

> **注意**：
>
> - 重启指定的设备并加载固件，需要获取 sudo 权限。
> - 同一时间仅能执行一个重启操作。

```Bash
$ sudo mx-smi reboot --all
check devices status...
wait devices rebooting...
package boot image version info:
    version: V3.51
    build time: Mar  4 2024 14:27:08
reboot finished: success 6, failed 0, skip 0
```

## 其他扩展语法test你好啊

### Markdown 中直接插入 rst

````markdown
```rst
<your raw rst text here>
```
````

举例

````markdown
```rst
.. note::
  This is a note.
```
````

```rst
.. note::
  This is a note.
```

### 
