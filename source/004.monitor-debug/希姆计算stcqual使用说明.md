# 希姆计算stcqual使用说明

## 版本历史

| **文档版本** | **对应产品版本** | **作者** | **日期**   | **描述**                                                     |
| ------------ | ---------------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.3.4       | V1.3.12          | 希姆计算 | 2024-03-14 | 发布stcqual V1.3.12，验证文档内容。                          |
| V1.3.3       | V1.3.11          | 希姆计算 | 2024-01-22 | 发布stcqual V1.3.11，验证文档内容。                          |
| V1.3.2       | V1.3.10          | 希姆计算 | 2023-03-14 | 发布stcqual V1.3.10，验证文档内容。                          |
| V1.3.1       | V1.3.8           | 希姆计算 | 2022-12-27 | 发布stcqual V1.3.8，验证文档内容。                           |
| V1.3.0       | V1.3.7           | 希姆计算 | 2022-09-20 | 发布stcqual V1.3.7，增加`--high_temp_warn`参数，验证CentOS 7。 |
| V1.2.2       | HPE V1.4.0       | 希姆计算 | 2022-09-05 | 文档内容验证。                                               |
| V1.2.1       | HPE V1.3.0       | 希姆计算 | 2022-07-07 | 文档内容验证。                                               |
| V1.2.0       | HPE V1.2.0       | 希姆计算 | 2022-06-02 | 整篇编辑优化。<br/>添加示例的回显。<br/>补充排查依赖HPE驱动的程序的方法。<br/>补充重新安装HPE驱动的方法。<br/>补充`--prod_type`选项说明。<br/>补充常见问题。 |
| V1.1.1       | HPE V1.2.0       | 希姆计算 | 2022-04-11 | 编辑优化。                                                   |
| V1.1.0       | UnKnown          | 希姆计算 | 2021-12-10 | 增强PCIe带宽测试脚本和眼图工具。                             |
| V1.0.0       | UnKnown          | 希姆计算 | 2021-09-10 | 初始版本。                                                   |

## 概述

stcqual用于验证希姆计算NPU设备的功能和性能，支持测试PCIe带宽、DDR带宽、NPU算力、板卡功耗等，以及全面的系统级测试，并对应提供了测试用例。

> 说明：希姆计算软硬件产品相关的基本概念，请参见*[希姆计算基本概念](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)*。

本文中会提及以下驱动：

- stcdma.ko：stcqual提供的驱动，用于支持PCIe带宽测试。
- stcqual stc.ko：stcqual提供的驱动，用于支持除PCIe带宽以外的其他测试。
- standard stc.ko：HPE提供的驱动，是希姆计算异构编程环境包括的模块之一。

## 前提条件

- 主机配置满足以下要求：
  - Intel/AMD的64位CPU。
  - 系统内存大小不低于2GiB。
- 联系希姆计算技术支持获取stcqual工具包。
- 推荐以root用户登录主机，使用stcqual时需要访问一些系统目录。
- 主机已安装依赖包，以Ubuntu为例：

   ```bash
   $ sudo apt-get install bc vim-common numactl 
   ```

- 主机已安装编译C/C++程序所需的工具包，以Ubuntu为例：

   ```bash
   $ sudo apt-get install make
   $ sudo apt-get install build-essential
   ```

## 使用流程

以stcqual V1.3.12为例，使用流程如下：

1. 停止运行任何依赖HPE驱动的程序。在使用stcqual之前，NPU设备必须处于初始状态。
   1. 执行`lsmod | grep stc`命令查看正在依赖HPE驱动的程序。示例如下，stc模块正在被48个其他模块依赖。

      ```bash
      $ lsmod | grep stc
      stc                   512000  48
      ```

   2. 如果存在依赖HPE驱动的程序，可以执行`stc-smi`命令查看程序的Pid，然后视情况等待程序执行完成或者自行终止程序。示例如下，依赖HPE驱动的程序Pid为1328500。

      ```bash
      $ stc-smi
      +------------------------------------------------------------------------------+
      |    STC-SMI: 1.7.0                            Driver Version: 1.7.0           |
      +------------------------------------------------------------------------------+
      |  NPU        Name      Frequency|         Bus-Id        |            NPU-Util |
      |  Fan        Temp          Power|   ClusterCount        |  Memory Used /Total |
      +==============================================================================+
      |    0     STCP920          1000M|   0000:01:00.0        |                 0 % |
      |  N/A       33.6C   36.4W / 160W|              4        |     624.00M /16.00G |
      +------------------------------------------------------------------------------+
      
      +------------------------------------------------------------------------------+
      | Processes:                                                                   |
      |   NPU CLUSTER           Pid  processName                           MemoryUsed|
      +==============================================================================+
      |     0       0       1328500  python3                                  339.43M|
      +------------------------------------------------------------------------------+
      ```

2. 解压stcqual工具包并进入工具目录。以工具包名称为stcqual-v1.3.12-e1fc4c9.tar.gz为例，工具目录中包含了若干目录和文件。

   ```bash
   $ tar -zxvf stcqual-v1.3.12-e1fc4c9.tar.gz
   $ cd stcqual
   $ ls
   build_info    npc_load.sh    npu.ddr_stress.sh  npu.slt.sh      pcie_common.sh  stcqual                      tools
   cases         npc_run.sh     npu.pcie_bw.sh     npu.thermal.sh  pvtdef.sh       stcqual.thermal.parallel.sh
   driver        npu.ai_net.sh  npu.pcie_eye.sh    npu.tops.sh     README.md       switch_freq.sh
   npc_clean.sh  npu.ddr_bw.sh  npu.power.sh       packages        result          system_prepare.sh
   ```
   
3. 选择测试时需要使用的内核驱动。stcqual单独提供了用于支持测试的驱动，而非使用HPE驱动，因此在执行测试前需要使用system_prepare.sh脚本切换驱动。

   > 说明：首次切换stcdma.ko或stcqual stc.ko时，会进行自动编译，下方示例简化了过程中输出的回显信息，看到DONE即代表编译完成。如果您需要重新编译驱动，可以删除./result目录下的stcdma.ko或stc.ko文件，然后重新运行system_prepare.sh脚本。

   - 切换stcdma.ko：

     ```bash
     $ ./system_prepare.sh 1
     Check numactl commands on this host...
     Remove standard stc.ko ...
     Load stcdma.ko to kernel...
     ...
     DONE
     ```

   - 切换stcqual stc.ko：

     ```bash
     $ ./system_prepare.sh 0
     Remove stcdma.ko ...
     Load stcqual stc.ko to kernel...
     ...
     DONE
     ```


4. 执行测试。stcqual为PCIe带宽、DDR带宽、NPU算力、板卡功耗、系统级测试等提供了测试用例，您可以通过命令方便地选择测试用例，详细说明，请参见*命令说明*和*命令示例*章节。测试完成后，会自动在log目录下生成日志文件。

   > 注意：部分测试可能耗时较长，例如系统级测试耗时约10分钟。在测试过程中请不要通过Ctrl+C等方式强制退出，否则会导致系统不稳定。

5. 完成测试后，切换回HPE驱动，恢复NPU设备至正常工作状态。

   ```bash
   $ sudo rmmod stc
   $ sudo modprobe stc 
   ```

## 文件说明

stcqual工具目录中包含了若干目录和文件，建议您关注的目录和文件如下：

| **名称**          | **类型** | **描述**                                                     | **文件示例**                                                 |
| ----------------- | -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| log               | 目录     | 在执行测试时自动生成的目录，保存测试过程中产生的日志。       | /stcqual/log/npu0-work/stcqual-slt-npu0-00P8K2860206090d-stc920-202209161634.log |
| result            | 目录     | 保存测试结果文件，例如DDR压力测试过程中输出的bin文件。       | /stcqual/result/npu0/npu0_ddr0_output_random.bin             |
| stcqual           | 文件     | 使用stcqual工具的入口，支持通过选项选择测试用例、输出形式等。 | stcqual                                                      |
| system_prepare.sh | 文件     | 用于切换驱动的脚本。                                         | system_prepare.sh                                            |
| README.md         | 文件     | stcqual工具的使用说明。                                      | README.md                                                    |

## 命令说明

执行`./stcqual -h`获取stcqual命令的使用方法：

```bash
$ cd stcqual
$ ./stcqual -h
------------------------------------------------------------
STCQual - A qualify toolset for StreamComputing NPU product.
------------------------------------------------------------

Option list：
  -h, --help:                Print usage
  -v, --version:             STCQual version info
  -l, --list:                Show list of currently detected NPUs on the host.
  -t, --test:                Specify the test to be run [1-8]. Default value is 8
  -p, --pcibus:              Specify the PCI Slot bus ID for the test
  -i, --index:               Specify the NPU ID for the test
      --loops:               Provide number of loops for specified test
      --prod_id:             Provide product id for log file
      --prod_type:           Provide product type for stcqual test: [920, 908]
      --cust_time:           Provide a customized timestamp for log file
      --high_temp_warn:      Specify a customized high temperature warning threshold

------------------------------------------------
Test list:
        [1]  PCIe bandwidth test
        [2]  PCIe eye data test
        [3]  DDR bandwidth test
        [4]  Computing power (TOPS) test
        [5]  AI net performance test
        [6]  Thermal test
        [7]  Power test
        [8]  System Level Test (SLT) - sequence order
```

stcqual命令支持以下选项：

| **选项**           | **描述**                                                     |
| ------------------ | ------------------------------------------------------------ |
| `-h, --help`       | 查看stcqual支持的选项和测试集。                              |
| `-v, --version`    | 查看stcqual的版本信息。                                      |
| `-l, --list`       | 查看主机中检测到的NPU设备。                                  |
| `-t, --test`       | 通过编号指定测试集，默认值为8。取值范围为1~8。               |
| `-p, --pcibus`     | 在使用指定PCI总线的NPU设备上执行测试。                       |
| `-i, --index`      | 在指定NPU上执行测试。                                        |
| `--loops`          | 指定循环测试的次数，默认值为1。                              |
| `--prod_id`        | 生成日志文件时，在文件名中添加自定义信息。                   |
| `--prod_type`      | 用于指定产品类型，默认为920，产品类型会影响测试时使用的时钟频率。取值范围：<br/>920：代表STCP920，测试时使用的时钟频率为1000M。<br/>908：代表STCP908，测试时使用的时钟频率为900M。 |
| `--cust_time`      | 生成日志文件时，在文件名中添加指定的时间戳。                 |
| `--high_temp_warn` | 在板卡温度超过温度阈值时会触发高温警告并停止测试，默认情况下，用户版stcqual的温度阈值为85℃，工厂版stcqual的温度阈值为80℃。您可以通过该参数自定义温度阈值。<br/>说明：仅支持在用户版stcqual使用该参数。此外，仅test case 5~8中会检测板卡温度，因此仅test case 5~8中该参数才会生效。 |

stcqual支持以下测试集：

| **测试集编号** | **测试集名称**                           | **描述**                                                     |
| -------------- | ---------------------------------------- | ------------------------------------------------------------ |
| 1              | PCIe bandwidth test                      | PCIe带宽测试，通过PCIe DMA通道在主机端内存和设备端内存之间传输数据，根据传输的最大数据量和传输时间计算出PCIe的带宽。 |
| 2              | PCIe eye data test                       | PCIe眼图数据测试，逐个lane打印眼图数据，并根据若干个检测点，汇总判断眼图质量是否满足要求。 |
| 3              | DDR bandwidth test                       | DDR带宽测试，通过NPU内部的sysDMA通道在设备端DDR与LLB之间传输数据，根据传输的最大数据量和传输时间（基于clock数）计算出DDR的带宽。 |
| 4              | Computing power (TOPS) test              | NPU算力测试，运行指定的MAC运算，根据运算消耗的时间（基于clock数）评估NPU的算力。 |
| 5              | AI Net performance test                  | AI网络性能测试，顺序执行ResNet50、ResNet50 INT8、BERT，根据执行各个网络时所处理的数据量和消耗的时间，计算出在NPU上执行AI网络的性能。 |
| 6              | Thermal test                             | 发热测试，在所有NPU上运行高功耗用例，后台线程定期轮询板卡温度。检测到温度过高时，则记录一次error到结果文件。 |
| 7              | Power test                               | 功耗测试，在后台运行高功耗用例，并在前台监控功耗状况。       |
| 8              | System Level Test (SLT) - sequence order | 系统级测试（SLT），顺序执行10项测试，全面验证NPU的功能和性能，包括scan_sram、ddr_stress、pld、sdma、l1_im、mac、mac_x8、resnet50_x8、resnet50、bert。 |

## 命令示例

以stcqual V1.3.12为例，演示几类命令的执行结果。

> 说明：大部分命令回显信息较多，因此示例仅展示部分回显信息。

### 查看NPU设备

查看主机中检测到的NPU设备，回显内容和驱动类型有关：

```bash
$ ./system_prepare.sh 0
Remove stcdma.ko ...
Load stcqual stc.ko to kernel...
DONE
$ ./stcqual --list
NPU: 0
        STCP920         BUS-ID: 0:03:00.0 (01PSF600000d0709)
```

示例为检测到一个NPU设备，产品类型为STCP920，NPU设备所使用PCI总线的ID为`03:00.0`。

### 在所有NPU上测试

在所有NPU上执行PCIe眼图数据测试：

```bash
$ ./system_prepare.sh 0
stcqual stc.ko is already loaded...
$ ./stcqual --test 2
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
Core(s) per socket:  1
Socket(s):           8
NUMA node(s):        1
Vendor ID:           AuthenticAMD
CPU family:          23
Model:               49
Model name:          AMD EPYC-Rome Processor
Stepping:            0
CPU MHz:             2794.748
BogoMIPS:            5589.49
Virtualization:      AMD-V
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            512K
L3 cache:            16384K
NUMA node0 CPU(s):   0-7
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clzero xsaveerptr wbnoinvd arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------
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
--------------------------------
stc device voltage info vsoc:840 mv vmac:720 mv
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

################################################################
               STCP920 [pcie_eye] TEST ON NPU0 BEGIN
#################################################################

Start pcie eyesurf on NPU0:  lane_num=16, test_loops=1
*********************eyesurf NPU0 round0***************************
eyesurf NPU0 lane0: [PASS]
...
```

### 在指定NPU上测试

- 在NPU 0上执行PCIe带宽测试：

```bash
$ ./system_prepare.sh 1
Check numactl commands on this host...
Remove stcqual stc.ko ...
Load stcdma.ko to kernel...
DONE
$ ./stcqual --test 1 --index 0
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   40 bits physical, 48 bits virtual
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              1
Core(s) per socket:              1
Socket(s):                       8
NUMA node(s):                    1
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           49
Model name:                      AMD EPYC-Rome Processor
Stepping:                        0
CPU MHz:                         2794.748
BogoMIPS:                        5589.49
Virtualization:                  AMD-V
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       256 KiB
L1i cache:                       256 KiB
L2 cache:                        4 MiB
L3 cache:                        128 MiB
NUMA node0 CPU(s):               0-7
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Full AMD retpoline, IBPB conditional, IBRS_FW, STIBP disabled, RSB filling
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl xtopology cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero xsaveerptr arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------

################################################################
               STCP920 [pcie_bw] TEST ON NPU0 BEGIN
#################################################################


current_link_speed = 16 Gbps
current_link_width = 16 lane
test command is: TEST_H2D_CMD=0x40000021 TEST_D2H_CMD=0x40000012 TEST_BID_CMD=0x40000023
PCIE bandwidth test START -- [LOOP 0]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  70 Gbps                 103 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

################################################################
               STCP920 [pcie_bw] TEST ON NPU0 [PASS]
#################################################################
```

- 在NPU 0上执行PCIe眼图数据测试：

```bash
$ ./system_prepare.sh 0
Remove stcdma.ko ...
Load stcqual stc.ko to kernel...
DONE
$ ./stcqual --test 2 --index 0
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   40 bits physical, 48 bits virtual
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              1
Core(s) per socket:              1
Socket(s):                       8
NUMA node(s):                    1
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           49
Model name:                      AMD EPYC-Rome Processor
Stepping:                        0
CPU MHz:                         2794.748
BogoMIPS:                        5589.49
Virtualization:                  AMD-V
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       256 KiB
L1i cache:                       256 KiB
L2 cache:                        4 MiB
L3 cache:                        128 MiB
NUMA node0 CPU(s):               0-7
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Full AMD retpoline, IBPB conditional, IBRS_FW, STIBP disabled, RSB filling
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl xtopology cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero xsaveerptr arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------
NPU: 0
        Product Name: STCP920
        Sn: 01PSF600000d0709
        Chip count: 1
        Temperature: 30.00C
        Fan speed: N/A
        Power: 30.87W
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
        Driver: 1.4.1
        Chip version: 20200102
--------------------------------
stc device voltage info vsoc:840 mv vmac:720 mv
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

################################################################
               STCP920 [pcie_eye] TEST ON NPU0 BEGIN
#################################################################

Start pcie eyesurf on NPU0:  lane_num=16, test_loops=1
*********************eyesurf NPU0 round0***************************
eyesurf NPU0 lane0: [PASS]
...
```

### 在使用指定PCI总线的NPU上执行测试

在使用指定PCI总线（地址为`03:00.0`）的NPU上执行DDR带宽测试：

> 说明：如果使用`./stcqual --list`得到的BUS-ID，无需包括开头（例如`0:`）和末尾（例如`(01P9MV530002030a)`），否则会出现报错。

```bash
$ ./system_prepare.sh 0
stcqual stc.ko is already loaded...
$ ./stcqual --list
NPU: 0
        STCP920         BUS-ID: 0:03:00.0 (01PSF600000d0709)
$ ./stcqual --test 3 --pcibus 03:00.0
Convert pci_bus_id: 03:00.0 to npu_id: 0
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
Core(s) per socket:  1
Socket(s):           8
NUMA node(s):        1
Vendor ID:           AuthenticAMD
CPU family:          23
Model:               49
Model name:          AMD EPYC-Rome Processor
Stepping:            0
CPU MHz:             2794.754
BogoMIPS:            5589.50
Virtualization:      AMD-V
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            512K
L3 cache:            16384K
NUMA node0 CPU(s):   0-7
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clzero xsaveerptr wbnoinvd arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------
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
--------------------------------
stc device voltage info vsoc:840 mv vmac:720 mv
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

################################################################
               STCP920 [ddr_bw] TEST ON NPU0 BEGIN
#################################################################

...
```

### 自定义温度阈值

执行发热测试并设置温度阈值为50℃：

```bash
$ ./system_prepare.sh 0
stcqual stc.ko is already loaded...
$ ./stcqual --test 6 --high_temp_warn 50
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
Core(s) per socket:  1
Socket(s):           8
NUMA node(s):        1
Vendor ID:           AuthenticAMD
CPU family:          23
Model:               49
Model name:          AMD EPYC-Rome Processor
Stepping:            0
CPU MHz:             2794.754
BogoMIPS:            5589.50
Virtualization:      AMD-V
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            512K
L3 cache:            16384K
NUMA node0 CPU(s):   0-7
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clzero xsaveerptr wbnoinvd arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------
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
--------------------------------
stc device voltage info vsoc:840 mv vmac:720 mv
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

################################################################
               STCP920 [thermal] TEST ON NPU0 BEGIN
#################################################################

...
```

### 循环测试

循环执行5次PCIe带宽测试：

```bash
$ ./system_prepare.sh 1
Check numactl commands on this host...
Remove stcqual stc.ko ...
Load stcdma.ko to kernel...
DONE
$ ./stcqual --test 1 --loops 5
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   40 bits physical, 48 bits virtual
CPU(s):                          8
On-line CPU(s) list:             0-7
Thread(s) per core:              1
Core(s) per socket:              1
Socket(s):                       8
NUMA node(s):                    1
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           49
Model name:                      AMD EPYC-Rome Processor
Stepping:                        0
CPU MHz:                         2794.748
BogoMIPS:                        5589.49
Virtualization:                  AMD-V
Hypervisor vendor:               KVM
Virtualization type:             full
L1d cache:                       256 KiB
L1i cache:                       256 KiB
L2 cache:                        4 MiB
L3 cache:                        128 MiB
NUMA node0 CPU(s):               0-7
Vulnerability Itlb multihit:     Not affected
Vulnerability L1tf:              Not affected
Vulnerability Mds:               Not affected
Vulnerability Meltdown:          Not affected
Vulnerability Spec store bypass: Mitigation; Speculative Store Bypass disabled via prctl and seccomp
Vulnerability Spectre v1:        Mitigation; usercopy/swapgs barriers and __user pointer sanitization
Vulnerability Spectre v2:        Mitigation; Full AMD retpoline, IBPB conditional, IBRS_FW, STIBP disabled, RSB filling
Vulnerability Srbds:             Not affected
Vulnerability Tsx async abort:   Not affected
Flags:                           fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl xtopology cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 xsaves clzero xsaveerptr arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------

################################################################
               STCP920 [pcie_bw] TEST ON NPU0 BEGIN
#################################################################


current_link_speed = 16 Gbps
current_link_width = 16 lane
test command is: TEST_H2D_CMD=0x40000021 TEST_D2H_CMD=0x40000012 TEST_BID_CMD=0x40000023
PCIE bandwidth test START -- [LOOP 0]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  70 Gbps                 103 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

PCIE bandwidth test START -- [LOOP 1]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  70 Gbps                 103 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

PCIE bandwidth test START -- [LOOP 2]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  70 Gbps                 103 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

PCIE bandwidth test START -- [LOOP 3]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  64 Gbps                 97 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

PCIE bandwidth test START -- [LOOP 4]

Result:
--------------------------------------------------------
                        HOST2NPU                NPU2HOST
Theoretical             256 Gbps                256 Gbps
Actual                  64 Gbps                 103 Gbps
--------------------------------------------------------

---------- PCIE bandwidth test is DONE ----------

################################################################
               STCP920 [pcie_bw] TEST ON NPU0 [PASS]
#################################################################
```

### 在日志文件名中添加产品类型

在NPU 0上执行SLT测试，生成的日志文件名中包括产品类型：

```bash
$ ./system_prepare.sh 0
Remove stcdma.ko ...
Load stcqual stc.ko to kernel...
DONE
$ ./stcqual -t8 -i0 --prod_id stc920
stcqual version:
--------------------
stcqual version: v1.3.12

HOST SYSTEM INFO:
--------------------
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              8
On-line CPU(s) list: 0-7
Thread(s) per core:  1
Core(s) per socket:  1
Socket(s):           8
NUMA node(s):        1
Vendor ID:           AuthenticAMD
CPU family:          23
Model:               49
Model name:          AMD EPYC-Rome Processor
Stepping:            0
CPU MHz:             2794.754
BogoMIPS:            5589.50
Virtualization:      AMD-V
Hypervisor vendor:   KVM
Virtualization type: full
L1d cache:           32K
L1i cache:           32K
L2 cache:            512K
L3 cache:            16384K
NUMA node0 CPU(s):   0-7
Flags:               fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm rep_good nopl cpuid extd_apicid tsc_known_freq pni pclmulqdq ssse3 fma cx16 sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm cmp_legacy svm cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw topoext perfctr_core ssbd ibrs ibpb stibp vmmcall fsgsbase tsc_adjust bmi1 avx2 smep bmi2 rdseed adx smap clflushopt clwb sha_ni xsaveopt xsavec xgetbv1 clzero xsaveerptr wbnoinvd arat npt nrip_save umip rdpid arch_capabilities

Current test is running on NPU0:
--------------------------------
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
--------------------------------
stc device voltage info vsoc:840 mv vmac:720 mv
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

################################################################
               STCP920 [slt] TEST ON NPU0 BEGIN
#################################################################

######## Picking Test [scan_sram] RUN ON [NPU0] ########
Sample count [1]  Wait time [1]
256+0 records in
256+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 0.16502 s, 1.6 GB/s
256+0 records in
256+0 records out
268435456 bytes (268 MB, 256 MiB) copied, 0.939705 s, 286 MB/s

Current SOC Freq:
stc device soc west frequency 1000
stc device soc east frequency 1000
stc device ddr frequency 3733

-------- TEST [scan_sram] START on NPU[0] --------

...
```

测试完成后，log目录下的日志文件名称示例为stcqual-slt-npu0-01PSF60000170610-stc920-202403131427.log。

> 说明：SLT测试耗时约10分钟，回显信息也较多，因此示例仅展示部分回显信息。

## 常见问题

- 如果在切换驱动时出现以下报错，说明存在依赖HPE驱动的程序，请参考*使用流程*章节中的步骤排查。

```bash
$ ./system_prepare.sh 0
Remove standard stc.ko ...
rmmod: ERROR: Module stc is in use

### Failed to remove old stc.ko
$ ./system_prepare.sh 1
Check numactl commands on this host...
Remove standard stc.ko ...
rmmod: ERROR: Module stc is in use

### Failed to remove old stc.ko
```

- 在指定PCI总线时，如果使用`./stcqual --list`得到的BUS-ID，无需包括开头（例如`0:`）和末尾（例如`(00P8K2860206090d)`），否则会出现报错。

```bash
$./stcqual --list
NPU: 0
        STCP920         BUS-ID: 0:04:00.0 (00P8K2860206090d)
$ ./stcqual --test 2 --pcibus 0:04:00.0
#### Invalid pci bus address. ####
$ ./stcqual --test 2 --pcibus 0:04:00.0 (00P8K2860206090d)
-bash: syntax error near unexpected token `('
$ lspci -d:0100
00:01.0 VGA compatible controller: Red Hat, Inc. QXL paravirtual graphic card (rev 04)
04:00.0 Processing accelerators: Device 23e2:0100
```
