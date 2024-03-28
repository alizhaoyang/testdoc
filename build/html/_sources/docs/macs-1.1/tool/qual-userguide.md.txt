# mx-qual用户手册

## 概述
本文档介绍了 `mx-qual`(MOFFETT QUALIFICATION )工具的使用方法。`mx-qual` 是基于 `SOLA Runtime API` 实现的设备质量测试工具，主要用于检测设备的可用性、稳定性、性能等方面的指标。

## 使用方法

`mx-qual` 是一个命令行工具，可以通过 `mx-qual -h` 查看帮助信息。

```shell
Moffett Quality Inspection Application  v1.2.0
Usage: mx-qual [OPTIONS] SUBCOMMAND

Options:
  -h,--help                   Print this help message and exit
  --version                   Display program version information and exit

Subcommands:
  list                        List all devices detected on the system
  hardware_link               Run hardware link test
  pcie_bandwidth              Run PCIe bandwidth test
  memory_bandwidth            Run memory bandwidth test
  p2p                         Run peer to peer test
  compute                     Run computing power test
  stress                      Run stress test
  memtest                     Run hardware memory test
```

`mx-qual` 以子命令的方式去执行相应的测试，子命令的使用方法可以通过 `mx-qual <sub_command> -h` 查看。


### 子命令说明

#### list
```text
List devices
Usage: mx-qual list [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              The device index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {1,2,3}
```
列出指定的设备信息，如果不指定设备，则列出所有设备信息。

测试命令示例：
```shell
# 列出所有设备
mx-qual list
# 列出指定设备
mx-qual list -i 0
mx-qual list -i 0 1 2
mx-qual list -i {0,1,2}
```
输出结果示例：
```shell
Device 0: "01S30-00A"
  Serial number:      2023243080074
  PCI Bus ID:         0000:8a:00.0
  Runtime version:    3.4.0
  Driver version:     3.4.0
  Firmware version:   1.0.14
```

#### hardware_link
```text
Run hardware link test
Usage: mx-qual hardware_link [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
```
运行硬件链路测试，测试驱动和所有设备的通信链路是否正常。

测试命令示例：
```shell
mx-qual hardware_link
```

输出结果示例：
```text
Test driver link... ok
Test device count... ok
Test device link... ok
```

#### pcie_bandwidth
```text
Usage: mx-qual pcie_bandwidth [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              The device index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {1,2,3}
  -s,--sn TEXT                The device sn you specified.
                              
  -d,--data_size UINT:INT in [32 - 100]
                              The transfer size (MB) you specified. (default 100MB)
                              
  -l,--loop INT:INT in [1 - 100]
                              The number of test loop (default: 1)
                              
  -f,--full_duplex            Enable full duplex mode
```
运行 PCIe 带宽测试，不指定设备时默认测试所有设备，可以通过`-i`指定设备的`index`，也可以通过`-s`指定设备的`sn`，`-s`的优先级比`-i`高，若同时指定了`-i`和`-s`，则只测试`-s`指定的设备。

测试的数据大小可以通过`-d`指定，单位为MB，默认为100MB，测试的循环次数可以通过`-l`指定，默认为1次。默认进行半双工测试，使用`-f`可以开启全双工测试。

测试命令示例：
```shell
mx-qual pcie_bandwidth
mx-qual pcie_bandwidth -i 0
mx-qual pcie_bandwidth -i 0 1 2
mx-qual pcie_bandwidth -s 2023243080096
mx-qual pcie_bandwidth --sn=2023243080096
mx-qual pcie_bandwidth -i 0 -d 32 -l 10
mx-qual pcie_bandwidth -f -l 10
```

输出结果示例：
```text
PCIe Bandwidth Test
 Device id: [0]

 Host to Device Bandwidth
 Transfer Size: 100.000 MB, Bandwidth: 11.947 GB/s

 Device to Host Bandwidth
 Transfer Size: 100.000 MB, Bandwidth: 12.139 GB/s

Result = PASS
```

#### memory_bandwidth
```text
Run memory bandwidth test
Usage: mx-qual memory_bandwidth [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              The device index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {1,2,3}
  -s,--sn TEXT                The device sn you specified.
                              
  -d,--data_size UINT:INT in [32 - 100]
                              The transfer size (MB) you specified. (default 100MB)
                              
  -l,--loop INT:INT in [1 - 100]
                              The number of test loop (default: 1)
```
运行设备内存带宽测试，不指定设备时默认测试所有设备，可以通过`-i`指定设备的`index`，也可以通过`-s`指定设备的`sn`，`-s`的优先级比`-i`高，若同时指定了`-i`和`-s`，则只测试`-s`指定的设备。

测试的数据大小可以通过`-d`指定，单位为MB，默认为100MB，测试的循环次数可以通过`-l`指定，默认为1次。

测试命令示例：
```shell
mx-qual memory_bandwidth
mx-qual memory_bandwidth -i 0
mx-qual memory_bandwidth -i 0 1 2
mx-qual memory_bandwidth -s 2023243080096
mx-qual memory_bandwidth --sn=2023243080096
mx-qual memory_bandwidth -i 0 -d 32 -l 10
```

输出结果示例：
```text
Memory Bandwidth Test
 Device id: [0]

 Memory Read  Bandwidth
 Transfer Size: 100.000 MB, Bandwidth: 50.187 GB/s

 Memory Write Bandwidth
 Transfer Size: 100.000 MB, Bandwidth: 50.053 GB/s

Result = PASS
```

#### P2P

```bash
Run peer to peer test
Usage: mx-qual p2p [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              Devices index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
```

运行 `peer-to-peer` 带宽测试，不指定设备则默认测试所有设备，可以通过`-i`指定运行设备的 `index`，所有有效设备会两两配对进行测试。

测试命令示例：

```bash
mx-qual p2p
mx-qual p2p -i 0 1 2
```

输出结果示例：

```bash
P2P Connectivity Matrix
    D/D      0      1      2
    0        0      1      1
    1        1      0      1
    2        1      1      0

Unidirectional P2P Bandwidth Matrix (GB/s)
    D/D      0      1      2
    0     0.00  10.96  10.96
    1    10.94   0.00  10.79
    2    10.91  10.92   0.00

Bidirectional P2P Bandwidth Matrix (GB/s)
    D/D      0      1      2
    0     0.00  11.38  11.37
    1    11.38   0.00  11.34
    2    11.37  11.34   0.00

P2P Latency Matrix (us)
    SPU      0      1      2
    0     0.00 121.66 115.52
    1   101.10   0.00 105.69
    2   126.04 137.34   0.00

    CPU      0      1      2
    0     0.00 125.62 118.89
    1   103.60   0.00 109.61
    2   129.72 141.06   0.00
```

#### stress

```text
Run stress test
Usage: mx-qual stress [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              The device index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {1,2,3}
  -t,--time INT:INT in [2 - 1440]
                              Number of minutes consumed in a single stress test. (default: 2)
                              The deviation is subject to the influence of the machine.
  -l,--loop INT:INT in [1 - 99999]
                              The number of test loop, each loop takes approximately 2 minutes, or the time specified by --time. (default: 1)
```
运行压力测试，不指定设备时默认测试所有设备，可以通过`-i`指定设备的`index`。测试的循环次数和一次的时间可以通过`-l`和`-t`指定，默认执行一次循环，一次循环持续2分钟。若需要测试一小时，那么可以指定`-l 30`或者`-t 60`。

运行的前一分钟，会先让设备预热，不会监控设备状态。一分钟后，会监控设备的温度、功率和利用率，每隔一秒刷新一次，同时还会在当前目录下生成 `mx-qual-stress.log` 文件，可以用于后续分析。压测完成后，会输出压测过程中的整体信息。

测试命令示例：
```shell
# 压测2分钟
mx-qual stress
# 压测60分钟
mx-qual stress -l 30
```

输出结果示例：
```text
device  temp.cur  temp.avg  power.cur  power.avg  util.cur  util.avg
0       68        65        79         76         99        98        
1       71        68        77         76         99        98        
2       70        67        80         78         99        98        
3       72        69        79         78         99        98        
4       69        66        83         77         99        98        
5       67        64        79         77         99        98
```

#### compute
```text
Run computing power test
Usage: mx-qual compute [OPTIONS] op dtype

Positionals:
  op        REQUIRED          Support op: conv2d / multiply

  dtype     REQUIRED          Support type: int8 / bf16
                              note: multiply only support bf16

Options:
  -h,--help                   Print this help message and exit
  --sparsity INT:{8,16,32}    Support: 8 / 16 /32
                              Only used by conv2d
  --iochannel INT:{256,512}   Support: 256, 512
                              Only used by conv2d
```
运行算力测试。可以测试的算子类型有`conv2d`和`multiply`，可以通过`conv2d`和`multiply`指定。每个算子类型可配置的参数不同。
 * `conv2d` 
  * 支持两种数据类型: `int8` 和 `bf16`
  * 支持配置以下参数：
    1. sparsity : 8 / 16 / 32
    2. iochannel : 256 / 512 (ichannel = ochannel)
 * `multiply` 
  * 仅支持一种数据类型: `bf16`

测试命令示例：
```shell
mx-qual compute conv2d int8 --sparsity 8 --iochannel 256
mx-qual compute multiply bf16
```

输出结果示例：
```text
SPU INT8 Target TOPS@8xsparsity: 707.788800 TOPS.
Actual (TensorCore) TOPS@8xSparsity: 697.528523
Actual latency: 0.023750 ms
Utilization: 98.55%

VPU BF16 Target TFLOPS: 9.830400 TFLOPS.
Actual (TensorCore) TFLOPS: 9.084315
Actual latency: 0.003166 ms
Utilization: 92.41%
```


#### memtest
```text
Run hardware memory test
Usage: mx-qual memtest [OPTIONS]

Options:
  -h,--help                   Print this help message and exit
  -i,--index UINT:INT in [0 - 31] ...
                              The device index you specified (default: all). Separate values with spaces.
                              Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {1,2,3}
  -t,--type UINT:INT in [0 - 10] ...
                              The memory test type you specified (default: 7)
                                type 0          [Walking 1 bit]
                                type 1          [Own address test]
                                type 2          [Moving inversions, ones&zeros]
                                type 3          [Moving inversions, 8 bit pat]
                                type 4          [Moving inversions, random pattern]
                                type 5          [Block move, 64 moves]
                                type 6          [Moving inversions, 32 bit pat]
                                type 7          [Random number sequence]
                                type 8          [Modulo 20, random pattern]
                                type 9          [Bit fade test]
                                type 10         [Memory stress test]

  -l,--loop INT:INT in [1 - 100]
                              The number of test loop (default: 1)
```
这是一个参考`MemTest86`实现的用于测试硬件内存稳定性和可靠性的工具。

可以通过`-i`指定设备的`index`，默认运行所有设备。可以通过`-t`指定测试的类型，可以指定多个类型，类型的取值范围为0-10，具体的类型含义可以参考`MemTest86`的说明，默认是7。可以通过`-l`指定测试的循环次数，默认为1次。

测试命令示例：
```shell
mx-qual memtest
mx-qual memtest -i 0 -t 0

```
