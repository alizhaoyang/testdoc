# 希姆计算HPE示例程序

## 版本历史

| **文档版本** | **对应产品版本** | **作者** | **日期**   | **说明**                              |
| ------------ | ---------------- | -------- | ---------- | ------------------------------------- |
| V1.0.1       | STCRP V1.2.0     | 希姆计算 | 2022-11-29 | - 验证hpe-example。<br />- 编辑优化。 |
| V1.0.0       | HPE V1.4.0       | 希姆计算 | 2022-08-26 | 初始版本。                            |

## 概述

为了方便您体验希姆计算异构编程环境中执行程序的效果，HPE中提供了若干示例程序（下文中简称为hpe-example）。在安装HPE后，hpe-example位于/usr/local/hpe/example下。

```bash
$ ls /usr/local/hpe/example
c++11_hpe.hc                  exclusive_devices_fifo.hc    kernel_parall_device.hc   matrix_multiply.hc
common_test.h                 exclusive_devices_stream.hc  kernel_parall_devices.hc  memcpy_global.hc
concurrent_kernels.hc         hello_world.hc               kernel_permanent.hc       memcpy_local.hc
concurrent_module_kernels.hc  highmem.hc                   kernel_serial.hc          memcpy_shared.hc
device                        host_device_kernel.hc        Makefile                  txt
device_attribute.hc           kernel_dependency.hc         matrix_convolution.hc
```

hpe-example涵盖了以下几类异构编程的典型操作：

- 基本语法：c++11_hpe.hc、host_device_kernel.hc

- 获取设备信息：device_attribute.hc、hello_world.hc

- 调用核函数：kernel_dependency.hc、kernel_parall_device.hc、kernel_parall_devices.hc、kernel_permanent.hc、kernel_serial.hc

- 使用stream：exclusive_devices_stream.hc

- 使用fifo：exclusive_devices_fifo.hc

- 并发执行：concurrent_kernels.hc、concurrent_module_kernels.hc

- 数据传输：highmem.hc、memcpy_global.hc 、memcpy_local.hc、memcpy_shared.hc

- 单独设备端程序：device下的hello.hc、kernel_fifo.hc、kernel.hc、kernel_sum.hc

- 数学计算：matrix_convolution.hc、matrix_multiply.hc

## hpe-example列表

各源文件的功能描述如下：

> 说明：Runtime API指主机端运行时API，NPURT API指设备端运行时API，详细的API列表和说明，请参见[希姆计算异构编程手册](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。

| **源文件名称**               | **功能描述**                                                 |
| ---------------------------- | ------------------------------------------------------------ |
| c++11_hpe.hc                 | 使用HPE API（C++11）统计txt文件中x/y/z/w字符的数量。         |
| common_test.h                | 示例异构程序需要的一些通用声明。                             |
| concurrent_kernels.hc        | 演示并发执行时stream的用法，并介绍如何使用stcStreamWaitEvent函数同步有依赖关系的HPE stream。 |
| concurrent_module_kernels.hc | 演示使用目标程序模块相关的Runtime API。                      |
| device                       | 包括在希姆计算NPU上执行的设备端核函数示例：<br>- hello.hc：调用printf（NPURT API）在主机端打印设备信息。<br/>- kernel_fifo.hc：演示在设备端调用fifo_push和fifo_pop（NPURT API）。<br/>- kernel.hc：演示简单的带参数设备端函数。<br/>- kernel_sum.hc：演示简单的带参数设备端函数，实现elementwise加法。 |
| device_attribute.hc          | 获取一个设备的属性信息。                                     |
| exclusive_devices_fifo.hc    | 调用NPU API通过核函数参数演示fifo的用法。                    |
| exclusive_devices_stream.hc  | 调用printf（NPU API）在主机端打印设备信息，并调用NPU API通过核函数参数演示stream的用法。 |
| hello_world.hc               | 调用printf（NPU API）在主机端打印设备信息。                  |
| highmem.hc                   | 调用memcpy（NPURT API）从shared memory向global high memory（高端内存，3GiB ~ 4GiB范围）传输数据。 |
| host_device_kernel.hc        | 同时在设备端和主机端代码中调用双边函数（同时用__host__和__device__修饰的函数）。 |
| kernel_dependency.hc         | 调用两个stream中有依赖关系的核函数，并打印core 0中的信息。   |
| kernel_parall_device.hc      | 在一个设备中并行调用核函数，并打印core 0中的信息。streamID用作parallelID，parallelID相同的核函数串行执行，在硬件资源充足时parallelID不同的核函数并行执行。 |
| kernel_parall_devices.hc     | 在两个设备中并行调用两个核函数。                             |
| kernel_permanent.hc          | 调用stcFifo*（Runtime API）、fifo_*（NPURT API）向permanent核函数传输数据。 |
| kernel_serial.hc             | 串行调用一个stream中的核函数，并打印core 0中的信息。         |
| Makefile                     | 编译/usr/local/hpe/example下的异构程序。                     |
| matrix_convolution.hc        | 使用内部指令完成矩阵卷积计算。                               |
| matrix_multiply.hc           | 使用内部指令完成矩阵乘计算。                                 |
| memcpy_global.hc             | 调用memcpy（NPURT API）从global/local memory向global memory传输数据。 |
| memcpy_local.hc              | 调用memcpy（NPURT API）从global memory向local memory传输数据。 |
| memcpy_shared.hc             | 调用memcpy（NPURT API）从global/local memory向shared memory传输数据。 |
| txt                          | 包括一个war_and_peace.txt文件，供c++11_hpe.hc统计x/y/z/w字符的数量。 |

## 执行hpe-example

1. 进入/usr/local/hpe/example文件夹，并编译hpe-example。

   ```bash
   $ cd /usr/local/hpe/example
   $ make
   stcc -DNCORE=8 c++11_hpe.hc --rtlib=compiler-rt -o c++11_hpe
   stcc -DNCORE=8 concurrent_kernels.hc --rtlib=compiler-rt -o concurrent_kernels
   stcc -DNCORE=8 concurrent_module_kernels.hc --rtlib=compiler-rt -o concurrent_module_kernels
   stcc -DNCORE=8 device_attribute.hc --rtlib=compiler-rt -o device_attribute
   stcc -DNCORE=8 exclusive_devices_fifo.hc --rtlib=compiler-rt -o exclusive_devices_fifo
   stcc -DNCORE=8 exclusive_devices_stream.hc --rtlib=compiler-rt -o exclusive_devices_stream
   stcc -DNCORE=8 hello_world.hc --rtlib=compiler-rt -o hello_world
   stcc -DNCORE=8 highmem.hc --rtlib=compiler-rt -o highmem
   stcc -DNCORE=8 host_device_kernel.hc --rtlib=compiler-rt -o host_device_kernel
   stcc -DNCORE=8 kernel_dependency.hc --rtlib=compiler-rt -o kernel_dependency
   stcc -DNCORE=8 kernel_parall_device.hc --rtlib=compiler-rt -o kernel_parall_device
   stcc -DNCORE=8 kernel_parall_devices.hc --rtlib=compiler-rt -o kernel_parall_devices
   stcc -DNCORE=8 kernel_permanent.hc --rtlib=compiler-rt -o kernel_permanent
   stcc -DNCORE=8 kernel_serial.hc --rtlib=compiler-rt -o kernel_serial
   stcc -DNCORE=8 matrix_convolution.hc --rtlib=compiler-rt -o matrix_convolution
   stcc -DNCORE=8 matrix_multiply.hc --rtlib=compiler-rt -o matrix_multiply
   stcc -DNCORE=8 memcpy_global.hc --rtlib=compiler-rt -o memcpy_global
   stcc -DNCORE=8 memcpy_local.hc --rtlib=compiler-rt -o memcpy_local
   stcc -DNCORE=8 memcpy_shared.hc --rtlib=compiler-rt -o memcpy_shared
   stc-clang -DNCORE=8 --shc-device-only device/hello.hc -c -o device/hello.o
   stc-clang -DNCORE=8 --shc-device-only device/kernel_fifo.hc -c -o device/kernel_fifo.o
   stc-clang -DNCORE=8 --shc-device-only device/kernel.hc -c -o device/kernel.o
   stc-clang -DNCORE=8 --shc-device-only device/kernel_sum.hc -c -o device/kernel_sum.o
   ```

2. 执行示例程序，以c++11_hpe和hello_world为例。

   ```bash
   $ ./c++11_hpe
   get file path: ./txt/war_and_peace.txt
   running kernel_count_xyzw_frequency......
   Read 3223503 byte corpus from ./txt/war_and_peace.txt
   counted 107310 instances of 'x', 'y', 'z', or 'w' in "./txt/war_and_peace.txt"
   $ ./hello_world
   running hello_world......
   hello world from core 0/8.
   hello world from core 1/8.
   hello world from core 2/8.
   hello world from core 3/8.
   hello world from core 4/8.
   hello world from core 5/8.
   hello world from core 6/8.
   hello world from core 7/8.
   ```

## hpe-example执行效果

### c++11_hpe

```bash
$ ./c++11_hpe
get file path: ./txt/war_and_peace.txt
running kernel_count_xyzw_frequency......
Read 3223503 byte corpus from ./txt/war_and_peace.txt
counted 107310 instances of 'x', 'y', 'z', or 'w' in "./txt/war_and_peace.txt"
```

### concurrent_kernels

```bash
$ ./concurrent_kernels
nstreams: 9
kernel parall_func runs with parallel ID 3.
kernel parall_func runs with parallel ID 5.
kernel parall_func runs with parallel ID 6.
kernel parall_func runs with parallel ID 7.
kernel parall_func runs with parallel ID 7.
kernel parall_func runs with parallel ID 8.
kernel parall_func runs with parallel ID 0.
kernel parall_func runs with parallel ID 2.
kernel sum output result 38.
Measured time for example = 0.077s
Passed: npu sum: 38, cpu_sum: 38 expect equal
```

### concurrent_module_kernels

```bash
$ ./concurrent_module_kernels
nstreams: 9
get file path: ./device/kernel_sum.o
kernel parall_func runs with parallel ID 2.
kernel parall_func runs with parallel ID 4.
kernel parall_func runs with parallel ID 6.
kernel parall_func runs with parallel ID 6.
kernel parall_func runs with parallel ID 7.
kernel parall_func runs with parallel ID -5.
kernel parall_func runs with parallel ID 0.
kernel parall_func runs with parallel ID 2.
kernel sum output result 22.
Measured time for example = 0.078s
Passed: npu sum: 22, cpu_sum: 22 expect equal
```

### device（hello.o、kernel_fifo.o、kernel.o、kernel_sum.o）

单独编译的设备端程序需要在主机端程序读入后执行，编写主机端程序的例子，请参见[希姆计算异构编程手册](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)的*进程独占NPU*章节。

### device_attribute

```bash
$ ./device_attribute
device 0 attribute list:
  device name: STCP920
  hardware version of chip: 538968322
  hardware version of board: 0
  device number in a chip: 4
  core number in a device: 8
  size of share memory in a device: 0x2000
  size of global memory in a device: 0xc00
  number of concurrent kernels in a device: 1
  PCIe bus ID of a chip: 0x1
  PCIe device ID of a chip: 256
  version of device frimware: 0
  version of device driver: 17039360
```

### exclusive_devices_fifo

```bash
$ ./exclusive_devices_fifo
get file path: ./device/kernel_fifo.o
running kernel_fifo......
Acquired Npu Num: 1
Loadeded Npu Moudule Num: 1
output 1:2:3:4
```

### exclusive_devices_stream

```bash
$ ./exclusive_devices_stream
get file path: ./device/hello.o
running simple_hello_world......
hello world from core 0/8.
hello world from core 1/8.
hello world from core 2/8.
hello world from core 0/8.
hello world from core 3/8.
hello world from core 0/8.
hello world from core 4/8.
hello world from core 1/8.
hello world from core 0/8.
hello world from core 5/8.
hello world from core 2/8.
hello world from core 1/8.
hello world from core 6/8.
hello world from core 1/8.
hello world from core 3/8.
hello world from core 2/8.
hello world from core 4/8.
hello world from core 2/8.
hello world from core 3/8.
hello world from core 7/8.
hello world from core 5/8.
hello world from core 3/8.
hello world from core 6/8.
hello world from core 7/8.
hello world from core 4/8.
hello world from core 4/8.
hello world from core 5/8.
hello world from core 5/8.
hello world from core 6/8.
hello world from core 7/8.
hello world from core 6/8.
hello world from core 7/8.
get file path: ./device/kernel.o
running simple_input_output......
output 1:2:3:4
```

### hello_world

```bash
$ ./hello_world
running hello_world......
hello world from core 0/8.
hello world from core 1/8.
hello world from core 2/8.
hello world from core 3/8.
hello world from core 4/8.
hello world from core 5/8.
hello world from core 6/8.
hello world from core 7/8.
```

### highmem

```bash
$ ./highmem
-------- HighMEM TEST BEGIN --------
prepare host data:
76543210 76543210 76543210 76543210 76543210 76543210 76543210 76543210 
Check host dst result:
76543210 76543210 76543210 76543210 76543210 76543210 76543210 76543210 
---- Static allocate test ----
check dst result: 
12345678 12345678 12345678 12345678 12345678 12345678 12345678 12345678 

---- TEST DONE ----
```

### host_device_kernel

无报错代表双边函数在主机端和设备端均执行成功。

```bash
$ ./host_device_kernel
```

### kernel_dependency

```bash
$ ./kernel_dependency
kernel dep1_func runs.
kernel dep2_func runs.
```

### kernel_parall_device

```bash
$ ./kernel_parall_device
kernel parall_func runs with parallel ID 0.
kernel parall_func runs with parallel ID 1.
kernel parall_func runs with parallel ID 0.
environment variable STC_SET_DEVICES: 0
device: 0, device core count: 8
```

### kernel_parall_devices

```bash
$ ./kernel_parall_devices
kernel parall_func runs in device 0.
kernel parall_func runs in device 1.
```

### kernel_permanent

```bash
$ ./kernel_permanent
0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 
```

### kernel_serial

```bash
$ ./kernel_serial
kernel serial1_func runs.
kernel serial2_func runs.
kernel serial2_func runs.
kernel serial1_func runs.
kernel serial1_func runs.
kernel serial2_func runs.
```

### matrix_convolution

```bash
$ ./matrix_convolution
matrix convolution result:
12.0, 12.0, 12.0, 12.0, 
24.0, 24.0, 24.0, 24.0, 
36.0, 36.0, 36.0, 36.0, 
48.0, 48.0, 48.0, 48.0, 
60.0, 60.0, 60.0, 60.0, 
72.0, 72.0, 72.0, 72.0, 
84.0, 84.0, 84.0, 84.0, 
96.0, 96.0, 96.0, 96.0, 
```

### matrix_multiply

```bash
$ ./matrix_multiply
matrix multiply result:0.0, 36.0, 72.0, 108.0, 144.0, 180.0, 216.0, 252.0, 
```

### memcpy_global

```bash
$ ./memcpy_global
core 4: copy direction [global -> global] data [0x1400 -> 0x1400].
core 4: copy direction [global -> global] data [0x2400 -> 0x2400].
core 4: copy direction [global -> global] data [0x3400 -> 0x3400].
core 4: copy direction [global -> global] data [0x4400 -> 0x4400].
core 1: copy direction [global -> global] data [0x1100 -> 0x1100].
core 1: copy direction [global -> global] data [0x2100 -> 0x2100].
core 1: copy direction [global -> global] data [0x3100 -> 0x3100].
core 1: copy direction [global -> global] data [0x4100 -> 0x4100].
core 0: copy direction [global -> global] data [0x1000 -> 0x1000].
core 0: copy direction [global -> global] data [0x2000 -> 0x2000].
core 0: copy direction [global -> global] data [0x3000 -> 0x3000].
core 0: copy direction [global -> global] data [0x4000 -> 0x4000].
core 2: copy direction [global -> global] data [0x1200 -> 0x1200].
core 2: copy direction [global -> global] data [0x2200 -> 0x2200].
core 2: copy direction [global -> global] data [0x3200 -> 0x3200].
core 2: copy direction [global -> global] data [0x4200 -> 0x4200].
core 3: copy direction [global -> global] data [0x1300 -> 0x1300].
core 3: copy direction [global -> global] data [0x2300 -> 0x2300].
core 3: copy direction [global -> global] data [0x3300 -> 0x3300].
core 3: copy direction [global -> global] data [0x4300 -> 0x4300].
core 6: copy direction [global -> global] data [0x1600 -> 0x1600].
core 6: copy direction [global -> global] data [0x2600 -> 0x2600].
core 6: copy direction [global -> global] data [0x3600 -> 0x3600].
core 6: copy direction [global -> global] data [0x4600 -> 0x4600].
core 5: copy direction [global -> global] data [0x1500 -> 0x1500].
core 5: copy direction [global -> global] data [0x2500 -> 0x2500].
core 5: copy direction [global -> global] data [0x3500 -> 0x3500].
core 5: copy direction [global -> global] data [0x4500 -> 0x4500].
core 7: copy direction [global -> global] data [0x1700 -> 0x1700].
core 7: copy direction [global -> global] data [0x2700 -> 0x2700].
core 7: copy direction [global -> global] data [0x3700 -> 0x3700].
core 7: copy direction [global -> global] data [0x4700 -> 0x4700].
core 0: copy direction [local -> global] data [0x10 -> 0x0].
core 1: copy direction [local -> global] data [0x11 -> 0x1].
core 2: copy direction [local -> global] data [0x12 -> 0x2].
core 3: copy direction [local -> global] data [0x13 -> 0x3].
core 4: copy direction [local -> global] data [0x14 -> 0x4].
core 5: copy direction [local -> global] data [0x15 -> 0x5].
core 6: copy direction [local -> global] data [0x16 -> 0x6].
core 7: copy direction [local -> global] data [0x17 -> 0x7].
Passed: item 0, expect:2001, result: 2001
Passed: item 1, expect:2101, result: 2101
Passed: item 2, expect:2201, result: 2201
Passed: item 3, expect:2301, result: 2301
Passed: item 4, expect:2401, result: 2401
Passed: item 5, expect:2501, result: 2501
Passed: item 6, expect:2601, result: 2601
Passed: item 7, expect:2701, result: 2701
Passed: item 0, expect:2000, result: 2000
Passed: item 1, expect:2100, result: 2100
Passed: item 2, expect:2200, result: 2200
Passed: item 3, expect:2300, result: 2300
Passed: item 4, expect:2400, result: 2400
Passed: item 5, expect:2500, result: 2500
Passed: item 6, expect:2600, result: 2600
Passed: item 7, expect:2700, result: 2700
get string: no error
```

### memcpy_shared

```bash
$ ./memcpy_shared
core 0: copy direction [global -> shared] data [0x1000 -> 0x1000].
core 0: copy direction [global -> shared] data [0x2000 -> 0x2000].
core 1: copy direction [global -> shared] data [0x1100 -> 0x1100].
core 1: copy direction [global -> shared] data [0x2100 -> 0x2100].
core 2: copy direction [global -> shared] data [0x1200 -> 0x1200].
core 2: copy direction [global -> shared] data [0x2200 -> 0x2200].
core 3: copy direction [global -> shared] data [0x1300 -> 0x1300].
core 3: copy direction [global -> shared] data [0x2300 -> 0x2300].
core 4: copy direction [global -> shared] data [0x1400 -> 0x1400].
core 4: copy direction [global -> shared] data [0x2400 -> 0x2400].
core 5: copy direction [global -> shared] data [0x1500 -> 0x1500].
core 5: copy direction [global -> shared] data [0x2500 -> 0x2500].
core 6: copy direction [global -> shared] data [0x1600 -> 0x1600].
core 6: copy direction [global -> shared] data [0x2600 -> 0x2600].
core 7: copy direction [global -> shared] data [0x1700 -> 0x1700].
core 7: copy direction [global -> shared] data [0x2700 -> 0x2700].
core 0: copy direction [local -> shared] data [0x10 -> 0x10].
core 1: copy direction [local -> shared] data [0x11 -> 0x11].
core 2: copy direction [local -> shared] data [0x12 -> 0x12].
core 3: copy direction [local -> shared] data [0x13 -> 0x13].
core 4: copy direction [local -> shared] data [0x14 -> 0x14].
core 5: copy direction [local -> shared] data [0x15 -> 0x15].
core 6: copy direction [local -> shared] data [0x16 -> 0x16].
core 7: copy direction [local -> shared] data [0x17 -> 0x17].
```
