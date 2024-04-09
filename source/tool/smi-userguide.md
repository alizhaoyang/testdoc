# mx-smi用户手册

## 简介

MOFFETT System Management Interface (mx-smi) 是一个基于 MOFFETT Management Library (MXML) 开发的命令行工具，用于管理和监控 MOFFETT SPU 设备 (SPU devices)。

用户可以通过此工具查询 SPU 设备的状态，并在拥有所需权限下修改某些状态和设置。

MX-SMI 目前支持在安装了 SOLA 工具包 (SOLA Toolkit) 的 Linux 系统上运行，并以纯文本的形式将相关信息输出到标准输出中。

| **子命令** | **说明**                                                     |
| ---------- | ------------------------------------------------------------ |
| list       | 以设备 (Device) 为单位输出概览信息，若是单卡多设备的产品（例如 S30），会标注位于同一张卡的设备。选项列表如下：<无选项> : 默认输出所有设备的概览信息。-i, –index : 用于指定设备id (device index)，可使用以下格式输入：空格分隔 : -i 0 1 2花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2} |
| query      | 输出设备详细信息。选项列表如下：<无选项> : 默认输出所有设备信息-i, –index : 用于指定设备id (device index)，可使用以下格式输入：空格分隔 : -i 0 1 2花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}-d, –display : 用于指定查询类型，并输出相关信息。可接受的参数 (不限定大小写)如下：MEMORY:设备内存信息UTILIZATION - 使用率信息TEMPERATURE - 温度信息POWER - 功率信息FREQUENCY - 频率信息VOLTAGE - 电压信息PIDS - 进程信息ECC - ECC信息 |
| select     | 按指定的内容和顺序，输出设备属性信息。默认第一行展示输出的属性名。后续每一行为用户指定的设备的属性信息。选项列表如下：-f, –field : **必选项** 用于指定需要输出的属性值，会按指定的顺序输出可接受的参数列表 (不限定大小写)：timestamp - 输出当前时间戳，格式为 “2006-01-02 15:04:05”driver_version - 驱动版本sola_version - SOLA版本count - 检测到的设备（Devices）数量index - 设备ID （根据Bus ID生成）name - 设备名称（产品名称）cores - 设备核心数量firmware_version - 固件版本mcu_version - MCU版本serial - 序列号(同一张卡拥有相同序列号，不同卡的序列号不同)uuid - UUID (每个设备全局唯一ID)board - 主板IDpci.bus - 16进制输出 PCI buspci.device - 16进制输出 PCI devicepci.domain - 16进制输出 PCI domainpci.bus_id - 16进制输出 PCI bus id，格式为 “domain:bus:device.function”pci.device_id - 16进制输出 PCI vendor device idpci.sub_device_id - 16进制输出 PCI Sub System idmemory.total - SPU总内存(单位:MiB)memory.reserved - SPU预留内存(单位:MiB)memory.used - SPU已使用内存(单位:MiB)memory.free - SPU总空闲内存(单位:MiB)utilization - 使用率(各核心上的平均使用率)ecc.errors.corrected.volatile.device_memory - 已纠正的易失性设备内存错误数量ecc.errors.corrected.aggregate.device_memory - 已纠正的聚合设备内存错误数量ecc.errors.uncorrected.volatile.device_memory - 未纠正的易失性设备内存错误数量ecc.errors.uncorrected.aggregate.device_memory - 未纠正的聚合设备内存错误数量temperature - 温度(单位:℃)power.draw - 当前功率(单位:W)power.limit - 最大功率(单位:W)frequency - 频率(单位:MHz)volatile - 电压(单位:mV)可使用以下格式输入空格分隔 : -q index pci.bus_id花括号中用逗号分隔 (以列表的形式传入) : -q {index,pci.bus_id}-i, –index : 用于指定设备id (device index)，可使用以下格式输入空格分隔 : -i 0 1 2花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}–noheader : 不输出属性名表头 |
| reboot     | 重启指定的设备并加载固件。执行此操作需要获取sudo权限。选项列表如下：选项组 target devices ： 以下选项至少传入一项-i, –index : 用于指定设备id (device index)，可使用以下格式输入：空格分隔 : -i 0 1 2花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}–all : 直接指定所有设备 |

## 命令手册

* 命令格式：mx-smi [选项] [子命令]

### 基本命令

* <无选项> : 以SPU卡为单位输出概览表格

```TEXT
> mx-smi
Tue Feb 27 10:05:56 2024
╭──────────────────────────────────────────────────────────────────────────────╮
│MOFFETT-SMI 2.1.0       Driver Version 3.5.0       SOLA Version 3.5.0         │
╰──────────────────────────────────────────────────────────────────────────────╯
 Card
┌──────────────────────────────────────────────────────────────────────────────┐
│Index Name Freq.  Voltage Temp. Pwr Util  Bus ID   Memory-Usage       SN      │
├──────────────────────────────────────────────────────────────────────────────┤
│Card0 S30  700MHz  930mV   44C  6W   0%  0:36:00.0 0MiB/61440MiB 2023073080040│
├──────────────────────────────────────────────────────────────────────────────┤
│Card1 S30  700MHz  930mV   45C  5W   0%  0:3b:00.0 0MiB/61440MiB 2023423080188│
├──────────────────────────────────────────────────────────────────────────────┤
│Card2 S30  700MHz  930mV   44C  6W   0%  0:40:00.0 0MiB/61440MiB 2023423080195│
├──────────────────────────────────────────────────────────────────────────────┤
│Card3 S30  700MHz  930mV   47C  5W   0%  0:45:00.0 0MiB/61440MiB 2023423080208│
├──────────────────────────────────────────────────────────────────────────────┤
│Card4 S30  700MHz  930mV   49C  4W   0%  0:9c:00.0 0MiB/61440MiB 2023423080176│
├──────────────────────────────────────────────────────────────────────────────┤
│Card5 S30  700MHz  930mV   46C  3W   0%  0:a1:00.0 0MiB/61440MiB 2023073080034│
├──────────────────────────────────────────────────────────────────────────────┤
│Card6 S30  700MHz  925mV   49C  4W   0%  0:a6:00.0 0MiB/61440MiB 2023423080193│
├──────────────────────────────────────────────────────────────────────────────┤
│Card7 S30  700MHz  930mV   46C  5W   0%  0:ac:00.0 0MiB/61440MiB 2023073080042│
└──────────────────────────────────────────────────────────────────────────────┘
 Processes
┌──────────────────────────────────────────────────────────────────────────────┐
│index   PID   Process Name                                        Memory Usage│
├──────────────────────────────────────────────────────────────────────────────┤
│  [No running processes found]                                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### 表头释义

**注：以卡为单位展示时，Index/Name/SN 同一张卡的值相同。Pwr/Memory-Usage，展示的值为卡中所有设备的总和。其他项均展示卡中 Bus ID 最低的设备对应的信息**

* Card （卡信息）
  * Index        - 卡编号
  * Name         - 卡名 （同一张卡一样）
  * Freq.        - 频率
  * Voltage      - 电压
  * Temp.        - 温度
  * Pwr          - 功率
  * Util         - 使用率 （负载）
  * Bus ID       - 总线ID，格式为 "domain:bus:device.function"
  * Memory-Usage - 内存使用情况，格式为 "usage / total" （已使用 / 总量）
  * SN           - 序列号 （同一张卡一样）
* Processes （进程信息）
  * index        - 运行该进程的卡编号
  * PID          - 进程ID
  * Process Name - 进程名称，最大展示长度为32个字符，超出长度后以 ... 省略开头的字符
  * Memory Usage - 进程占用的内存大小

#### 选项列表

* -h,--help : 输出命令/子命令帮助信息

* --version : 输出版本信息

### 子命令 list

以设备 (Device) 为单位输出概览表格，若是单卡多设备的产品（如 S30），会标注位于同一张卡的设备

```TEXT
> mx-smi list -i {0,1,2,3,4,5}
Tue Feb 27 10:06:58 2024
╭──────────────────────────────────────────────────────────────────────────────╮
│MOFFETT-SMI 2.1.0        Driver Version 3.5.0        SOLA Version 3.5.0       │
╰──────────────────────────────────────────────────────────────────────────────╯
 Devices
┌──────────────────────────────────────────────────────────────────────────────┐
│Index Name Freq.  Voltage Temp. Pwr Util  Bus ID   Memory-Usage       SN      │
├──────────────────────────────────────────────────────────────────────────────┤
│  +   S30                                                        2023073080040│
│  0        700MHz  930mV   44C  2W   0%  0:36:00.0 0MiB/20480MiB              │
│  1        700MHz  925mV   44C  2W   0%  0:37:00.0 0MiB/20480MiB              │
│  2        700MHz  930mV   44C  2W   0%  0:38:00.0 0MiB/20480MiB              │
├──────────────────────────────────────────────────────────────────────────────┤
│  +   S30                                                        2023423080188│
│  3        700MHz  930mV   45C  2W   0%  0:3b:00.0 0MiB/20480MiB              │
│  4        700MHz  925mV   48C  1W   0%  0:3c:00.0 0MiB/20480MiB              │
│  5        700MHz  925mV   47C  2W   0%  0:3d:00.0 0MiB/20480MiB              │
└──────────────────────────────────────────────────────────────────────────────┘
 Processes
┌──────────────────────────────────────────────────────────────────────────────┐
│index   PID   Process Name                                        Memory Usage│
├──────────────────────────────────────────────────────────────────────────────┤
│  [No running processes found]                                                │
└──────────────────────────────────────────────────────────────────────────────┘
```

#### 表头释义

**注：以设备为单位展示时，若为单卡多设备产品（如 S30），有独立的一行展示同一张卡的卡名和序列号。**

* Devices （设备信息）
  * Index        - device index （设备ID）
  * Name         - 卡名
  * Freq.        - 频率
  * Voltage      - 电压
  * Temp.        - 温度
  * Pwr          - 功率
  * Util         - 使用率 （负载）
  * Bus ID       - 总线ID，格式为 "domain:bus:device.function"
  * Memory-Usage - 内存使用情况，格式为 "usage / total" （已使用 / 总量）
  * SN           - 序列号 （同一张卡一样）
* Processes （进程信息）
  * index        - 运行该进程的设备ID
  * PID          - 进程ID
  * Process Name - 进程名称，最大展示长度为32个字符，超出长度后以 ... 省略开头的字符
  * Memory Usage - 进程占用的内存大小

#### 选项列表

* <无选项> : 默认输出所有设备概览

* -i, --index : 用于指定设备id (device index)，可使用以下格式输入
  * 空格分隔 : -i 0 1 2
  * 花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}

### 子命令 query

输出设备详细信息

```
> mx-smi query -i 0
Timestamp             : Tue Feb 27 10:07:27 2024
Driver Version        : 3.5.0
SOLA Version          : 3.5.0

Attached Devices      : 24

Device 0
    Product Name      : 00S30-00A
    SPU Cores Number  : 4
    FW Version        : 1.0.14
    MCU Version       : 4X08
    Serial Number     : 2023073080040
    UUID              : 01000000-0000-0000-0000-00AKS0211812
    Board Id          : 0x001
    PCI
        Bus           : 0x36
        Device        : 0x00
        Domain        : 0x0000
        Bus Id        : 0:36:00.0
        Device Id     : 0x70301f36
        Sub System Id : 0x70001f36
    Memory Usage
        Total         : 20480 MiB
        Reserved      : 4484 MiB
        Used          : 0 MiB
        Free          : 15996 MiB
    Utilization
        Core 0        : 0 %
        Core 1        : 0 %
        Core 2        : 0 %
        Core 3        : 0 %
        Max           : 0 %
        Min           : 0 %
        Avg           : 0 %
    Temperature       : 53 C
    Power
        Power Draw    : 2 W
        Power Limit   : 83 W
    Frequency         : 700 MHz
    Voltage           : 930 mV
    Processes
```

#### 选项列表

* <无选项> : 默认输出所有设备信息

* -i, --index : 用于指定设备id (device index)，可使用以下格式输入
  * 空格分隔 : -i 0 1 2
  * 花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}

* -d, --display : 用于指定查询类型，并输出相关信息
  * 可接受的参数列表 (不限定大小写)：
    * MEMORY      - 内存信息
    * UTILIZATION - 使用率信息
    * TEMPERATURE - 温度信息
    * POWER       - 功率信息
    * FREQUENCY   - 频率信息
    * VOLTAGE     - 电压信息
    * PIDS        - 进程信息
  * 可使用以下格式输入
    * 空格分隔 : -d MEMORY POWER
    * 花括号中用逗号分隔 (以列表的形式传入) : -d {MEMORY,POWER}

### 子命令 select

按指定的内容和顺序，输出设备属性信息

默认第一行展示输出的属性名。后续每一行为用户指定的设备的属性信息

```
> mx-smi select -f {index,pci.bus_id,board} -i 3 2 1
index, pci.bus_id, board
3, 0:3b:00.0, 2
2, 0:38:00.0, 1
1, 0:37:00.0, 1
```

#### 选项列表

* -f, --field : **必选项** 用于指定需要输出的属性值，会按指定的顺序输出
  * 可接受的参数列表 (不限定大小写)：
    * timestamp         - 输出当前时间戳，格式为 "2006-01-02 15:04:05"
    * driver_version    - 驱动版本
    * sola_version      - SOLA版本
    * count             - 检测到的设备（Devices）数量
    * index             - 设备ID （根据Bus ID生成）
    * name              - 设备名称（产品名称）
    * cores             - 设备核心数量
    * firmware_version  - 固件版本
    * mcu_version       - MCU版本
    * serial            - 序列号(同一张卡拥有相同序列号，不同卡的序列号不同)
    * uuid              - UUID (每个设备全局唯一ID)
    * board             - 主板ID
    * pci.bus           - 16进制输出 PCI bus 
    * pci.device        - 16进制输出 PCI device
    * pci.domain        - 16进制输出 PCI domain
    * pci.bus_id        - 16进制输出 PCI bus id，格式为 "domain:bus:device.function"
    * pci.device_id     - 16进制输出 PCI vendor device id
    * pci.sub_device_id - 16进制输出 PCI Sub System id
    * memory.total      - SPU总内存(单位:MiB)
    * memory.reserved   - SPU预留内存(单位:MiB)
    * memory.used       - SPU已使用内存(单位:MiB)
    * memory.free       - SPU总空闲内存(单位:MiB)
    * utilization       - 使用率(各核心上的平均使用率)
    * temperature       - 温度(单位:℃)
    * power.draw        - 当前功率(单位:W)
    * power.limit       - 最大功率(单位:W)
    * frequency         - 频率(单位:MHz)
    * volatile          - 电压(单位:mV)
  * 可使用以下格式输入
    * 空格分隔 : -q index pci.bus_id
    * 花括号中用逗号分隔 (以列表的形式传入) : -q {index,pci.bus_id}

* -i, --index : 用于指定设备id (device index)，可使用以下格式输入
  * 空格分隔 : -i 0 1 2
  * 花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}

* --noheader : 不输出属性名表头

### 子命令 reboot

重启指定的设备并加载固件，需要sudo权限

**注意：请确保同一时间仅有一个 reboot 操作在执行**

```
> sudo mx-smi reboot --all
check devices status...
wait devices rebooting...
package boot image version info:
    version: V3.50
    build time: Feb 24 2024 18:48:53
reboot finished: success 24, failed 0, skip 0
```

#### 选项列表

* 选项组 target devices ： 以下选项至少传入一项
  * -i, --index : 用于指定设备id (device index)，可使用以下格式输入
    * 空格分隔 : -i 0 1 2
    * 花括号中用逗号分隔 (以列表的形式传入) : -i {0,1,2}
  * --all : 直接指定所有设备

