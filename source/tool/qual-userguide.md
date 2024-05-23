# mx-qual 用户手册

mx-qual（Moffett Qualification）是一款基于 SOLA Runtime API 构建的设备质量测试工具，主要用于检测墨芯 SPU 设备的可用性、稳定性及性能等多项关键指标。

## 前提条件

在主机上安装 SOLA。具体的步骤，请参见《SOLA Toolkit 安装指南》。 安装 SOLA 后，mx-qual 的二进制文件位于`/usr/bin` 目录下。

## **命令说明**

### 基本命令

mx-qual 是一个命令行工具，您可通过执行 `mx-qual -h` 命令查看帮助信息。

```Bash
$ mx-qual -h
Moffett Quality Inspection Application  v1.4.0
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
  eyegraph                    Show pcie eye graph
```

> **注意**：mx-qual 以子命令的方式去执行相应的测试，子命令的使用方法可以通过执行 `mx-qual <sub_command> -h`命令查看。

### **子命令**

#### **list**

- 描述：列出指定的设备信息。

- 使用方式：

  ```Bash
    $ mx-qual list -h
    List all devices detected on the system
    Usage: mx-qual list [OPTIONS]
    
    Options:
      -h,--help                   Print this help message and exit
      -i,--index UINT:INT in [0 - 31] ...
                                  The device index you specified (default: all). Separate values with spaces.
                                  Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
  ```

- 参数说明：

  | **参数** | **说明**                                             |
    | -------- | ---------------------------------------------------- |
    | -i       | 指定设备的索引。如果不指定设备，则列出所有设备信息。 |

- 测试命令示例：

   ```Bash
    # 列出所有设备
    $ mx-qual list
    # 列出指定设备
    $ mx-qual list -i 0
    $ mx-qual list -i 0 1 2
    $ mx-qual list -i {0,1,2}
   ```

- 输出结果示例：

  ```Bash
  Device 0: "01S30-00A"
    Serial number:      2023243080074
    PCI Bus ID:         0000:8a:00.0
    Runtime version:    3.4.0
    Driver version:     3.4.0
    Firmware version:   1.0.14
  ```

#### **hardware_link**

- 描述：运行硬件链路测试，测试驱动和所有设备的通信链路是否正常。

- 使用方式：

  ```Bash
  $ mx-qual hardware_link -h
  Run hardware link test
  Usage: mx-qual hardware_link [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
  ```

- 测试命令示例：

   ```Bash
    $ mx-qual hardware_link
   ```

- 输出结果示例：

   ```Bash
    Test driver link... ok
    Test device count... ok
    Test device link... ok
    Test Result = PASS
   ```

#### **pcie_bandwidth**

- 描述：运行 PCIe 带宽测试，不指定设备时默认测试所有设备。

- 使用方式：

  ```Bash
    $ mx-qual pcie_bandwidth -h
  Run PCIe bandwidth test
  Usage: mx-qual pcie_bandwidth [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -i,--index UINT:INT in [0 - 31] ...
                                The device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
    -s,--sn TEXT                The device sn you specified.
  
    -d,--data_size UINT:INT in [32 - 100]
                                The transfer size (MB) you specified. (default 100MB)
  
    -l,--loop INT:INT in [1 - 100]
                                The number of test loop (default: 1)
  
    -f,--full_duplex            Enable full duplex mode
  ```

- 参数说明：

  | **参数** | **说明**                                         |
    | -------- | ------------------------------------------------ |
    | -i       | 指定设备的索引，不指定设备时默认测试所有设备     |
    | -s       | 指定设备的 SN 号                                 |
    | -d       | 指定测试的数据大小，单位为 MB，默认为 100MB      |
    | -l       | 指定测试的循环次数，默认为 1 次                  |
    | -f       | 默认进行半双工测试，使用`-f`时可以开启全双工测试 |

> **注意**：`-s`的优先级比`-i`高，若同时指定了`-i`和`-s`，则只测试`-s`指定的设备。

- 测试命令示例：

  ```Bash
    # 测试所有设备
    $ mx-qual pcie_bandwidth
    # 通过 device id 测试指定设备
    $ mx-qual pcie_bandwidth -i 0
    $ mx-qual pcie_bandwidth -i 0 1 2
    # 通过设备SN号来测试指定设备
    $ mx-qual pcie_bandwidth -s 2023243080096
    $ mx-qual pcie_bandwidth --sn=2023243080096
    # 使用不同的参数测试
    $ mx-qual pcie_bandwidth -i 0 -d 32 -l 10
    $ mx-qual pcie_bandwidth -f -l 10
  ```

- 输出结果示例：

  ```Bash
    $ mx-qual pcie_bandwidth -i 0
    PCIe Bandwidth Test (half duplex mode)
     Device id: [0]
    
     Host to Device Bandwidth
     Transfer Size: 100.000 MB, Bandwidth: 12.389 GB/s
    
     Device to Host Bandwidth
     Transfer Size: 100.000 MB, Bandwidth: 12.369 GB/s
    
    Test Result = PASS
  ```

#### **memory_bandwidth**

- 描述：运行设备内存带宽测试。

- 使用方式：

  ```Bash
   $ mx-qual memory_bandwidth -h
  Run memory bandwidth test
  Usage: mx-qual memory_bandwidth [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -i,--index UINT:INT in [0 - 31] ...
                                The device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
    -s,--sn TEXT                The device sn you specified.
  
    -d,--data_size UINT:INT in [32 - 100]
                                The transfer size (MB) you specified. (default 100MB)
  
    -l,--loop INT:INT in [1 - 100]
                                The number of test loop (default: 1)
  ```

> **注意**：`-s`的优先级比`-i`高，若同时指定了`-i`和`-s`，则只测试`-s`指定的设备。

- 参数说明：

  | **参数** | **说明**                                     |
    | -------- | -------------------------------------------- |
    | -i       | 指定设备的索引，不指定设备时默认测试所有设备 |
    | -s       | 指定设备的 SN 号                             |
    | -d       | 指定测试的数据大小，单位为 MB，默认为 100MB  |
    | -l       | 指定测试的循环次数，默认为 1 次              |

- 测试命令示例：

  ```Bash
  # 测试所有设备
  $ mx-qual memory_bandwidth
  # 通过 device id 测试指定设备
  $ mx-qual memory_bandwidth -i 0
  $ mx-qual memory_bandwidth -i 0 1 2
  # 通过 SN 测试指定设备
  $ mx-qual memory_bandwidth -s 2023243080096
  $ mx-qual memory_bandwidth --sn=2023243080096
  # 使用不同参数测试
  $ mx-qual memory_bandwidth -i 0 -d 32 -l 10
  ```

- 输出结果示例：

  ```Bash
  $ mx-qual memory_bandwidth -i 0
  Memory Bandwidth Test
   Device id: [0]
  
   Memory Read  Bandwidth
   Transfer Size: 100.000 MB, Bandwidth: 61.361 GB/s
  
   Memory Write Bandwidth
   Transfer Size: 100.000 MB, Bandwidth: 61.567 GB/s
  
  Test Result = PASS
  ```

#### **P2P**

- 描述：运行 `peer-to-peer` 带宽测试。

- 使用方式：

  ```Bash
   $ mx-qual p2p -h
  Run peer to peer test
  Usage: mx-qual p2p [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -d,--device UINT:INT in [0 - 31] ...
                                Device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
    -c,--card UINT:INT in [0 - 31] ...
                                Card index you specified. Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1}
  
  ```

- 参数说明：

  | **参数** | **说明**                                                     |
    | -------- | ------------------------------------------------------------ |
    | -d       | 可通过该参数指定运行 p2p 测试的设备（device），不指定设备时默认测试所有设备 |
    | -c       | 可通过该参数指定运行 p2p 测试的卡（card）                    |

  > **说明**：如果不指定设备或卡，则默认测试所有设备。`-c`的优先级比`-d`高。

- 测试命令示例：

  ```Bash
    # 按照 device 的维度测试所有 device 之间的 p2p 性能
    mx-qual p2p
    # 按照 device 的维度测试指定 device 之间的 p2p 性能
    mx-qual p2p -d 0 1 2
    # 按照 card 的维度测试指定 card 之间的 p2p 性能
    mx-qual p2p -c 0 1
  ```

- 输出结果示例：

  - 以 device 为维度

    ```Plain
     $ mx-qual p2p -d 0 1 2
    P2P Connectivity Matrix
        D/D      0      1      2
        0        0      1      1
        1        1      0      1
        2        1      1      0
    
    Unidirectional P2P Bandwidth Matrix (GB/s)
        D/D      0      1      2
        0     0.00  10.97  10.95
        1    10.93   0.00  10.94
        2    10.94  10.95   0.00
    
    Bidirectional P2P Bandwidth Matrix (GB/s)
        D/D      0      1      2
        0     0.00  21.96  21.89
        1    21.96   0.00  21.89
        2    21.89  21.89   0.00
    
    P2P Latency Matrix (ms)
        SPU      0      1      2
        0     0.00   3.07   3.06
        1     3.09   0.00   3.06
        2     3.06   3.06   0.00
    
        CPU      0      1      2
        0     0.00   8.63   8.60
        1     8.63   0.00   8.60
        2     8.71   8.63   0.00
    
    Test Result = PASS
    ```

  - 以 card 为维度：

    ```Plain
      $ mx-qual p2p -c 0 1
    P2P Connectivity Matrix
        C/C      0      1
        0        0      1
        1        1      0
    
    Unidirectional P2P Bandwidth Matrix (GB/s)
        C/C      0      1
        0     0.00  14.11
        1    14.09   0.00
    
    Bidirectional P2P Bandwidth Matrix (GB/s)
        C/C      0      1
        0     0.00  28.14
        1    28.14   0.00
    
    P2P Latency Matrix (ms)
        SPU      0      1
        0     0.00   7.14
        1     7.15   0.00
    
        CPU      0      1
        0     0.00  21.49
        1    21.43   0.00
    
    Test Result = PASS
    ```

#### **stress**

- 描述：运行压力测试，可以对设备的内存和计算单元进行压测

- 使用方式：

  ```Bash
    $ mx-qual stress -h
  Run stress test
  Usage: mx-qual stress [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -i,--index UINT:INT in [0 - 31] ...
                                The device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
    -t,--time INT:INT in [2 - 100000]
                                Number of minutes consumed in a single stress test. (default: 2)
                                The deviation is subject to the influence of the machine.
    -l,--load INT:INT in [0 - 100]
                                Pressure test load, ranging from 0% to 100% (default: 100%)
  ```

- 参数说明：

  | **参数** | **说明**                                            |
    | -------- | --------------------------------------------------- |
    | -i       | 指定设备的索引，不指定设备时默认测试所有设备。      |
    | -t       | 指定循环测试一次的时间，默认为 2 分钟，单位为分钟。 |
    | -l       | 指定压测负载，范围从 0-100，默认是 100%负载。       |

  > **说明：**在运行前的一分钟，系统会进行设备预热，期间不会进行设备状态的监控。待预热完成后，系统将开始每隔一秒监控设备的温度、功率和利用率，确保数据的实时更新。同时，系统会在当前目录下自动生成`mx-qual-stress.log`文件，以便后续对压测数据进行详细分析。压测任务结束后，系统会输出整个压测过程的综合信息，方便您全面了解压测情况。

- 测试命令示例：

  ```Bash
  # 默认压测2分钟
  mx-qual stress
  # 压测60分钟
  mx-qual stress -t 60
  # 使用50%的负载压测60分钟
  mx-qual stress -t 60 -l 50
  ```

- 输出结果示例：

  ```Bash
    device  temp.cur  temp.avg  power.cur  power.avg  util.cur  util.avg    
  0       68        68        65         66         99        99        
  1       67        66        65         65         99        100       
  2       69        68        67         66         99        99        
  
  
  ===================================================================================================
  Summary
  ===================================================================================================
  device  temp.min  temp.max  temp.avg  power.min  power.max  power.avg  util.min  util.max  util.avg
  0       66        68        68        65         68         66         99        99        99       
  1       65        67        66        63         67         65         99        100       100      
  2       67        69        68        65         68         66         99        99        99       
  
  Test Result = PASS
  ```

#### **compute**

- 描述：运行算力测试。

- 使用方式：

  ```Bash
  $ mx-qual compute -h
  Run computing power test
  Usage: mx-qual compute [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -i,--index UINT:INT in [0 - 31] ...
                                The device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
  ```

> **说明**：运行算力测试时，可以通过 `-i` 参数指定测试的设备。如果不指定设备，则默认对所有设备进行测试 。测试结果将以卡为单位进行展示，并包含在不同稀疏倍率下的性能数据。需要注意的是，具有相同序列号 （SN） 的设备被视为同一张卡。

- 参数说明：

  | **参数** | **说明**                                             |
    | -------- | ---------------------------------------------------- |
    | -i       | 指定设备的索引。如果不指定设备，则列出所有设备信息。 |

- 测试命令示例：

  ```Bash
  $ mx-qual compute 
  ```

- 输出结果示例：

  ```Plain
   INT8
    1x sparsity:
      SN: 2023243080074, actual:   87.64 TOPS, target:   88.47 TOPS, Utilization: 99.06%
      SN: 2023243080096, actual:   87.64 TOPS, target:   88.47 TOPS, Utilization: 99.06%
    2x sparsity:
      SN: 2023243080074, actual:  173.77 TOPS, target:  176.95 TOPS, Utilization: 98.21%
      SN: 2023243080096, actual:  173.62 TOPS, target:  176.95 TOPS, Utilization: 98.12%
    4x sparsity:
      SN: 2023243080074, actual:  347.35 TOPS, target:  353.89 TOPS, Utilization: 98.15%
      SN: 2023243080096, actual:  347.55 TOPS, target:  353.89 TOPS, Utilization: 98.21%
    8x sparsity:
      SN: 2023243080074, actual:  695.91 TOPS, target:  707.79 TOPS, Utilization: 98.32%
      SN: 2023243080096, actual:  695.91 TOPS, target:  707.79 TOPS, Utilization: 98.32%
    16x sparsity:
      SN: 2023243080074, actual: 1377.36 TOPS, target: 1415.58 TOPS, Utilization: 97.30%
      SN: 2023243080096, actual: 1374.20 TOPS, target: 1415.58 TOPS, Utilization: 97.08%
    32x sparsity:
      SN: 2023243080074, actual: 2662.64 TOPS, target: 2831.16 TOPS, Utilization: 94.05%
      SN: 2023243080096, actual: 2662.64 TOPS, target: 2831.16 TOPS, Utilization: 94.05%
  BF16
    1x sparsity:
      SN: 2023243080074, actual:   43.82 TOPS, target:   44.24 TOPS, Utilization: 99.05%
      SN: 2023243080096, actual:   43.82 TOPS, target:   44.24 TOPS, Utilization: 99.05%
    2x sparsity:
      SN: 2023243080074, actual:   86.77 TOPS, target:   88.47 TOPS, Utilization: 98.08%
      SN: 2023243080096, actual:   86.80 TOPS, target:   88.47 TOPS, Utilization: 98.11%
    4x sparsity:
      SN: 2023243080074, actual:  174.13 TOPS, target:  176.95 TOPS, Utilization: 98.41%
      SN: 2023243080096, actual:  174.23 TOPS, target:  176.95 TOPS, Utilization: 98.46%
    8x sparsity:
      SN: 2023243080074, actual:  348.76 TOPS, target:  353.89 TOPS, Utilization: 98.55%
      SN: 2023243080096, actual:  348.76 TOPS, target:  353.89 TOPS, Utilization: 98.55%
    16x sparsity:
      SN: 2023243080074, actual:  698.35 TOPS, target:  707.79 TOPS, Utilization: 98.67%
      SN: 2023243080096, actual:  698.35 TOPS, target:  707.79 TOPS, Utilization: 98.67%
    32x sparsity:
      SN: 2023243080074, actual: 1371.04 TOPS, target: 1415.58 TOPS, Utilization: 96.85%
      SN: 2023243080096, actual: 1374.20 TOPS, target: 1415.58 TOPS, Utilization: 97.08%
     Test Result = PASS
  ```

#### **memtest**

- 描述：进行硬件内存相关的测试，主要用于测试硬件内存的稳定性和可靠性。

- 使用方式：

  ```Bash
    $ mx-qual memtest -h
  Run hardware memory test
  Usage: mx-qual memtest [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -i,--index UINT:INT in [0 - 31] ...
                                The device index you specified (default: all). Separate values with spaces.
                                Or give a list of elements, separated by commas and enclosed in curly brackets e.g. {0,1,2}
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

- 参数说明：

  | **参数** | **说明**                                                     |
    | -------- | ------------------------------------------------------------ |
    | -i       | 指定设备的索引，不指定时默认运行所有设备。                   |
    | -t       | 指定测试的类型，类型的取值范围为 0～10。默认是 7，您也可以可以指定多个类型，具体的类型含义请参见[ MemTest86 的说明](https://www.memtest86.com/tech_individual-test-descr.html)。 |
    | -l       | 指定测试的循环次数，默认为 1 次。                            |

- 测试命令示例：

  ```Bash
  $ mx-qual memtest
  $ mx-qual memtest -i 0 -t 0
  ```

- 输出结果示例：

  ```Plain
  Device[0] running test 4 / 4 ... passed
  Test Result = PASS
  ```

#### eyegraph

- 描述：查看 PCIe 信道的眼图。

- 使用方式：

  ```Bash
  $ mx-qual eyegraph --help
  Show pcie eye graph, must be root user to run this cmd
  Usage: mx-qual eyegraph [OPTIONS]
  
  Options:
    -h,--help                   Print this help message and exit
    -s,--bdf TEXT REQUIRED      BDF(example: 0000:01:00.1) of the pcie switch device you specified
                                you can find switch device bdf by lspci cmd:
                                lspci -D | grep PMC | grep Memory | grep 4068
  
    -l,--lane UINT:INT in [0 - 15]
                                The lane id of eye graph you want to show (default: 0)
  ```

- 参数说明：

     | **参数** | **说明**                                                     |
     | -------- | ------------------------------------------------------------ |
     | -s       | 指定要查看眼图的设备，可以通过如下命令，查看系统中可用设备（BDF 格式）：`lspci -D | grep PMC | grep Memory | grep 4068`。 |
     | -l       | 指定要查看的链路的 lane id                                   |

- 测试命令示例：

  > **注意**：查看眼图需要 root 权限。

   ```Bash
    $ sudo mx-qual eyegraph -s 0000:01:00.1 -l 0
   ```

- 输出结果示例：

  ```bash
   $ sudo mx-qual eyegraph -s 0000:01:00.1 -l 0
  
  eye graph for logic port 0(0000:01:00.0), lane id is 0
  it is a upstream port
  start fetch eye pixels, total pixels count 6592
  fetch 496 eye pixels,  6096 pixels to be fetched
  fetch 496 eye pixels,  5600 pixels to be fetched
  fetch 496 eye pixels,  5104 pixels to be fetched
  fetch 496 eye pixels,  4608 pixels to be fetched
  fetch 496 eye pixels,  4112 pixels to be fetched
  fetch 496 eye pixels,  3616 pixels to be fetched
  fetch 496 eye pixels,  3120 pixels to be fetched
  fetch 496 eye pixels,  2624 pixels to be fetched
  fetch 496 eye pixels,  2128 pixels to be fetched
  fetch 496 eye pixels,  1632 pixels to be fetched
  fetch 496 eye pixels,  1136 pixels to be fetched
  fetch 496 eye pixels,   640 pixels to be fetched
  fetch 496 eye pixels,   144 pixels to be fetched
  fetch 144 eye pixels,     0 pixels to be fetched
  fetch eye pixel end
         0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3 3 3 3 3 3 3 3 3 4 4 4 4 4 4 4 4 4 4 5 5 5 5 5 5 5 5 5 5 6 6 6 6
       T 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
     V
    255  1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 3 4 4 4 5 5 5 5 5 . . 5 5 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    250  1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 4 5 5 5 5 5 . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    245  1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 5 . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    240  1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 5 . . . . . . . 5 5 5 5 5 4 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    235  1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    230  1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 5 . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1
    225  1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1
    220  1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 5 . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1
    215  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1
    210  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
    205  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
    200  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
    195  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1
    190  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 5 . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1
    185  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1
    180  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1 1 1 1 1 1
    175  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . 5 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1
    170  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1
    165  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1
    160  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
    155  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
    150  1 1 1 1 1 1 1 1 2 2 2 2 3 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
    145  1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1 1 1 1
    140  1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
    135  1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
    130  1 1 1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
    125  1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1 1 1
    120  1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
    115  1 1 1 1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
    110  1 1 1 1 1 1 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
    105  1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1 1
    100  1 1 1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1
     95  1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1
     90  1 1 1 1 1 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1
     85  1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1
     80  1 1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
     75  1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
     70  1 1 1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
     65  1 1 1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1
     60  1 1 1 2 2 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1
     55  1 1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1
     50  1 1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1
     45  1 1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1
     40  1 1 1 2 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1
     35  1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 1 1 1 1
     30  1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 1 1 1 1
     25  1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1 1
     20  1 1 2 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 2 2 2 2 1 1 1
     15  1 1 2 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1
     10  1 1 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1
      5  1 1 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 1 1 1
      0  1 1 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1
     -5  1 1 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1
    -10  1 1 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1
    -15  1 1 2 2 2 3 3 4 4 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 2 2 2 2 1 1 1
    -20  1 1 2 2 2 3 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1
    -25  1 1 2 2 2 2 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1 1
    -30  1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 1 1 1 1
    -35  1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 1 1 1 1
    -40  1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1
    -45  1 1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1
    -50  1 1 1 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1
    -55  1 1 1 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1
    -60  1 1 1 2 2 2 2 3 3 3 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1
    -65  1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
    -70  1 1 1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
    -75  1 1 1 1 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1
    -80  1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1
    -85  1 1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1
    -90  1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1
    -95  1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1
   -100  1 1 1 1 1 2 2 2 2 2 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1
   -105  1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
   -110  1 1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
   -115  1 1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1
   -120  1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1
   -125  1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
   -130  1 1 1 1 1 1 1 2 2 2 2 3 3 3 4 5 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
   -135  1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1
   -140  1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
   -145  1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
   -150  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1
   -155  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 5 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 4 3 3 2 2 2 1 1 1 1 1 1 1 1 1 1
   -160  1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1
   -165  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1
   -170  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 3 2 2 2 1 1 1 1 1 1 1 1 1 1 1
   -175  1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1
   -180  1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 . . . . . . . . . . . . . . . . . . . . 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1
   -185  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . . . 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
   -190  1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 . . . . . . . . . . . . . . . . 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
   -195  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 . . . . . . . . . . . . . . . 5 5 5 4 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1
   -200  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 . . . . . . . . . . . . . 5 5 5 5 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1
   -205  1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 4 5 5 5 5 . . . . . . . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1
   -210  1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 4 4 5 5 5 5 5 5 . . . . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -215  1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 . . . . . . . . . 5 5 5 5 4 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -220  1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 5 . . . . . . 5 5 5 5 4 4 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -225  1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 . . . . 5 5 5 5 5 4 4 3 3 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -230  1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 3 3 3 3 4 4 4 5 5 5 5 5 . . 5 5 5 5 5 4 4 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -235  1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 3 3 3 3 4 4 4 5 5 5 5 . 5 5 5 5 5 4 4 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -240  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 3 3 3 3 3 4 4 5 5 5 5 5 5 5 4 4 3 3 3 3 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -245  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 3 3 3 3 4 4 5 5 5 5 5 4 3 3 3 3 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -250  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 3 3 3 3 4 4 5 5 4 4 3 3 3 2 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
   -255  1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 3 3 3 3 4 4 4 3 3 3 2 2 2 2 2 2 2 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
  
  ```
