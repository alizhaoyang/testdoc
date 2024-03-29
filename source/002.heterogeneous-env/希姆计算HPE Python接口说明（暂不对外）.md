# 希姆计算HPE Python接口说明

## 版本历史

| **版本** | **作者** | **日期**   | **描述**                                                     |
| -------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.4.0   | 希姆计算 | 2022-09-09 | 配套HPE V1.4.0提供文档。                                     |
| V1.3.0   | 希姆计算 | 2022-07-07 | 配套HPE V1.3.0发版文档。                                     |
| V1.2.1   | 希姆计算 | 2022-06-02 | - 整篇编辑优化。<br/>- 添加class hpe.hpert.api.KernelFlag。<br/>- 次版本号对齐软件发版。 |
| V1.0.0   | 希姆计算 | 2021-09-01 | 初始版本。                                                   |

## 概述

希姆计算提供了Python形式的主机端运行时接口（hpert）和性能数据采集接口stcpti（Stream Computing Profiling Tool Interface），方便您使用Python接口进行设备端内存访问、核函数执行、性能数据采集等操作。

> 说明：C++形式的主机端运行时库的详细信息，请参见*希姆计算异构编程手册*。C++形式的stcpti的详细信息，请参见*希姆计算stc-prof使用说明*。

## 前提条件

- 请确保主机已安装：
  - HPE。具体的步骤，请参见*希姆计算HPE安装指南*。
  - Python 3.6或以上版本。
  - NumPy库。
- 联系希姆计算销售人员获得HPE Python接口扩展库的安装包。

## 使用流程

1. 安装HPE Python接口扩展库。以安装包名称为hpe_python-1.2.0-cp37-cp37m-linux_x86_64.whl为例。

   ```bash
   $ pip3 install hpe_python-1.2.0-cp37-cp37m-linux_x86_64.whl
   ```

2. 编写并编译设备端程序。示例程序用C++实现，因此需要单独编译供后续Python实现的主机端程序读入。示例程序的作用是使用指定的核在数组间拷贝数据。编译时将copy.cc编译为copy.o，并用copy.o生成Fat Binary文件copy-fatbin.o。

   ```bash
   $ cat copy.cc
   #include <npurt.h>
   
   extern "C" {
   void copy(int *in, int *out, int num) {
       if (CoreID == 0){
           for(int i=0; i<num; i++)
                   out[i] = in[i];
       }
   }
   }
   
   $ stc-clang++ --target=riscv32npu -c copy.cc
   $ stc-ld.lld -flavor gnu copy.o /usr/local/hpe/riscv32npu/lib/libnpurt_hld.a -r -o copy-fatbin.o
   ```

3. 编写并运行主机端程序。示例程序用Python实现，其中读入了已经单独编译得到的设备端目标程序。示例程序的作用是使用主机端运行时接口从主机端向设备端拷贝100个整数，然后拷贝回主机端，同时使用性能数据采集接口获取过程中的性能数据。

   ```bash
   $ cat copy-example.py
   #!/usr/bin/env python
   # coding=utf-8
   
   import numpy as np
   import hpe.hpert.api as hpe
   import hpe.profiler.api as prof
   
   def main():
       '''
       copy 100 integers from host to device and then back to host
   
       '''
       DATA_NUM = 100
       try:
           print("version", hpe.version())
           hpe.detect()
           print('get device:', hpe.get_device())
   
           prof.init()
           prof.enable()
   
           # prepare host memory
           host_in = np.arange(DATA_NUM, dtype=np.int32)
           host_out = np.zeros(DATA_NUM, dtype=np.int32)
           # prepare device memory
           device_in = hpe.device_mem(host_in.nbytes)
           device_out = hpe.device_mem(host_out.nbytes)
   
           # move 100 integers from host to device
           device_in.copy_from_host(host_in)
   
           # 'hello' kernel function in hello.o moves 100 integers from device_in to device_out
           mod = hpe.module('copy-fatbin.o')
           mod.launch_kernel('copy', [device_in, device_out, DATA_NUM])
           hpe.synchronize()
   
           # move 100 integers from device to host
           device_out.copy_to_host(host_out)
           print("host_out:")
           print(host_out)
   
           prof.disable()
           perf_datas = prof.get_perf_datas()
           print("*************profiler data*************")
           print(perf_datas)
           prof.release()
   
       except hpe.CruntimeException as e:
           print(e)
   
   if __name__ == "__main__":
       main()
   
   $ python copy-example.py
   ```

## 主机端运行时接口（hpert）说明

### 接口调用要求

调用主机端运行时接口时，需要导入对应的库：

```Python
import hpe.hpert.api
```

> 说明：如果报错`ModuleNotFoundError：No module named 'hpe'`，请检查是否安装了HPE Python接口扩展库、是否进入了正确的Python环境。

主机端运行时接口提供以下功能：

- 设备管理：提供操作设备相关的功能，例如指定待使用的设备、获取设备信息等。
- 内存管理：提供操作内存相关的功能，例如分配/释放内存、拷贝内存数据等。
- 执行控制：提供执行目标程序相关的功能，例如指定运行配置、加载/卸载目标程序。
- 流管理：提供操作流相关的功能，例如创建/销毁流、创建/销毁事件、添加事件等。

> 说明：接口执行报错时，会通过异常（RuntimeAPIError）方式上报错误码，您可以通过返回的Error Message定位和排查问题。详细的Error Message及说明，请参见*class hpe.hpert.api.CruntimeException*章节。

### 设备管理

#### hpe.hpert.api.detect

函数描述：显示主机上NPU设备的信息，输出所有NPC Cluster的标识符、名称、PCI信息。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.detect()
```

函数参数：

None

函数返回值：

| **类型** | **描述**                                     |
| -------- | -------------------------------------------- |
| bool     | True：发现了NPU设备。 False：未发现NPU设备。 |

#### hpe.hpert.api.set_device

函数描述：指定执行设备端程序时使用的NPC Cluster。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.set_device(device_id)
```

函数参数：

| **名称**  | **输入/输出** | **类型** | **描述**                                                 |
| --------- | ------------- | -------- | -------------------------------------------------------- |
| device_id | 输入参数      | int      | NPC Cluster的标识符。如果不指定，默认使用NPC Cluster 0。 |

函数返回值：

None

#### hpe.hpert.api.get_device

函数描述：获取执行设备端程序时使用的NPC Cluster。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.get_device()
```

函数参数：

None

返回值：

| **类型** | **描述**              |
| -------- | --------------------- |
| int      | NPC Cluster的标识符。 |

#### hpe.hpert.api.synchronize

函数描述：等待当前进程的所有设备端操作执行结束。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.synchronize(device_id=None)
```

函数参数：

| **名称**  | **输入/输出** | **类型** | **描述**                                                     |
| --------- | ------------- | -------- | ------------------------------------------------------------ |
| device_id | 输入参数      | int      | NPC Cluster的标识符。默认值为None，代表等待默认NPC Cluster上的设备端操作执行结束。<br/>说明：如果已调用hpe.hpert.api.set_device指定NPC Cluster，则为该NPC Cluster为默认NPC Cluster；否则NPC Cluster 0为默认NPC Cluster。 |

返回值：

None

### 内存管理

#### hpe.hpert.api.device_mem

函数描述：在设备端全局内存的0GiB ~ 3GiB范围动态分配内存。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.device_mem(size, device_id=None)
```

函数参数：

| **名称**  | **输入/输出** | **类型** | **描述**                                                     |
| --------- | ------------- | -------- | ------------------------------------------------------------ |
| size      | 输入参数      | int      | 所需分配的内存大小，单位为字节。                             |
| device_id | 输入参数      | int      | NPC Cluster的标识符。默认值为None，代表等待默认NPC Cluster。<br/>说明：如果已调用hpe.hpert.api.set_device指定NPC Cluster，则为该NPC Cluster为默认NPC Cluster；否则NPC Cluster 0为默认NPC Cluster。 |

函数返回值：

| **类型**                | **描述**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| hpe.hpert.api.DeviceMem | hpe.hpert.api.DeviceMem的对象，包括了管理在设备端分配的一段内存所需的信息。 |

#### hpe.hpert.api.host_mem

函数描述：在主机端分配内存并设置为不会被换出的页锁定内存。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.host_mem(shape, dtype=numpy.float16)
```

函数参数：

| **名称** | **输入/输出** | **类型**    | **描述**                                |
| -------- | ------------- | ----------- | --------------------------------------- |
| shape    | 输入参数      | tuple       | 数据的形状。                            |
| dtype    | 输入参数      | numpy.dtype | 数据元素的类型，默认值为numpy.float16。 |

函数返回值：

| 类型          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| numpy.ndarray | numpy.ndarray的对象，包括了管理在主机端分配的一段内存所需的信息。 |

#### hpe.hpert.api.host_pinned

函数描述：将主机端的内存设置为不会被换出的页锁定内存。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.host_pinned(array)
```

函数参数：

| **名称** | **输入/输出** | **类型**      | **描述**                            |
| -------- | ------------- | ------------- | ----------------------------------- |
| array    | 输入参数      | numpy.ndarray | 锁定numpy.ndarray对象的数据页内存。 |

函数返回值：

None

#### hpe.hpert.api.DeviceMem.offset_address

函数描述：获取设备端的偏移地址。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.DeviceMem.offset_address(offset)
```

函数参数：

| **名称** | **输入/输出** | **类型** | **描述**                                                     |
| -------- | ------------- | -------- | ------------------------------------------------------------ |
| offset   | 输入参数      | int      | 相对hpe.hpert.api.DeviceMem对象设备端基址的偏移量，单位为字节。 |

返回值：

| **类型** | **描述**           |
| -------- | ------------------ |
| int      | 设备端的偏移地址。 |

#### hpe.hpert.api.DeviceMem.copy_from_host

函数描述：将主机端numpy.ndarray对象内存的数据拷贝到设备端内存。

函数类型：stream取值为0时是同步函数；stream取值非0时是异步函数。

函数定义：

```Python
hpe.hpert.api.DeviceMem.copy_from_host(host_array, device_offset=0, host_offset=0, size=None, stream=0)
```

函数参数：

| **名称**      | **输入/输出** | **类型**             | **描述**                                                     |
| ------------- | ------------- | -------------------- | ------------------------------------------------------------ |
| host_array    | 输入参数      | numpy.ndarray        | 主机端numpy.ndarray对象。                                    |
| device_offset | 输入参数      | int                  | 设备端内存地址上的偏移量，单位为字节。默认值为0，代表无偏移，从hpe.hpert.api.DeviceMem对象设备端基址开始存放数据。 |
| host_offset   | 输入参数      | int                  | 主机端内存地址上的偏移量，单位为字节。默认值为0，代表无偏移，从host_array内存的基址开始拷贝数据。 |
| size          | 输入参数      | int                  | 待拷贝内存的大小，单位为字节。默认值为None，代表按整个host_array的大小拷贝数据。 |
| stream        | 输入参数      | hpe.hpert.api.Stream | 流标识符，默认值为0。                                        |

函数返回值：

None

#### hpe.hpert.api.DeviceMem.copy_to_host

函数描述：将设备端内存的数据拷贝到主机端numpy.ndarray对象的内存。

函数类型：stream取值为0时是同步函数；stream取值非0时是异步函数。

函数定义：

```Python
hpe.hpert.api.DeviceMem.copy_to_host(host_array, device_offset=0, host_offset=0, size=None, stream=0)
```

函数参数：

| **名称**      | **输入/输出** | **类型**             | **描述**                                                     |
| ------------- | ------------- | -------------------- | ------------------------------------------------------------ |
| host_array    | 输入参数      | numpy.ndarray        | 主机端numpy.ndarray对象。                                    |
| device_offset | 输入参数      | int                  | 设备端内存地址上的偏移量，单位为字节。默认值为0，代表无偏移，从hpe.hpert.api.DeviceMem对象设备端基址开始拷贝数据。 |
| host_offset   | 输入参数      | int                  | 主机端内存地址上的偏移量，单位为字节。默认值为0，代表无偏移，从host_array内存的基址开始存放数据。 |
| size          | 输入参数      | int                  | 拷贝字节大小。默认值为None，代表按整个host_array的大小拷贝数据。 |
| stream        | 输入参数      | hpe.hpert.api.Stream | 流标识符，默认值为0。                                        |

函数返回值：

None

### 执行控制

#### hpe.hpert.api.module

函数描述：加载设备端目标程序。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.module(image, device_id=None)
```

函数参数：

| **名称**  | **输入/输出** | **类型**   | **描述**                                                     |
| --------- | ------------- | ---------- | ------------------------------------------------------------ |
| image     | 输入参数      | str或bytes | 要加载的目标程序文件路径或内存字节串。                       |
| device_id | 输入参数      | int        | NPC Cluster的标识符。默认值为None，代表使用默认NPC Cluster。<br/>说明：如果已调用hpe.hpert.api.set_device指定NPC Cluster，则为该NPC Cluster为默认NPC Cluster；否则NPC Cluster 0为默认NPC Cluster。 |

函数返回值：

| **类型**             | **描述**                                                     |
| -------------------- | ------------------------------------------------------------ |
| hpe.hpert.api.Module | hpe.hpert.api.Module的对象，包括了管理加载到设备端的程序所需的信息。 |

#### hpe.hpert.api.Module.launch_kernel

函数描述：执行核函数，支持通过参数指定核函数的运行配置。

函数类型：异步函数

函数定义：

```Python
hpe.hpert.api.Module.launch_kernel(kernel_name, kernel_params=[], stream=0, core_num=8, flag=KernelFlag.stcKernelFlagNone)
```

函数参数：

| **名称**      | **输入/输出** | **类型**             | **描述**                                                     |
| ------------- | ------------- | -------------------- | ------------------------------------------------------------ |
| kernel_name   | 输入参数      | str                  | 核函数名称。                                                 |
| kernel_params | 输入参数      | list或tuple          | 核函数的参数数组。                                           |
| stream        | 输入参数      | hpe.hpert.api.Stream | 流标识符，默认值为0。                                        |
| core_num      | 输入参数      | int                  | 在一个NPC Cluster上并行执行核函数时所使用NPC的数量，默认值为8。 |
| flags         | 输入参数      | KernelFlag           | 核函数的执行标志，默认值为KernelFlag.stcKernelFlagNone，代表无运行标志。详细的运行标志含义，请参见*class hpe.hpert.api.KernelFlag*章节。 |

函数返回值：

None

### 流管理

#### hpe.hpert.api.stream

函数描述：创建一个流。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.stream(device_id=None)
```

函数参数：

| **名称**  | **输入/输出** | **类型** | **描述**                                                     |
| --------- | ------------- | -------- | ------------------------------------------------------------ |
| device_id | 输入参数      | int      | NPC Cluster的标识符。默认值为None，代表使用默认NPC Cluster。<br/>说明：如果已调用hpe.hpert.api.set_device指定NPC Cluster，则为该NPC Cluster为默认NPC Cluster；否则NPC Cluster 0为默认NPC Cluster。 |

函数返回值：

| **类型**             | **描述**                                             |
| -------------------- | ---------------------------------------------------- |
| hpe.hpert.api.Stream | hpe.hpert.api.Stream的对象，包括了管理流所需的信息。 |

#### hpe.hpert.api.Stream.synchronize

函数描述：等待当前流上的所有操作执行结束。如果核函数执行异常退出，输出触发异常时核函数的调用栈。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.Stream.synchronize()
```

函数参数：

None

函数返回值：

None

#### hpe.hpert.api.event

函数描述：创建一个事件。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.event(device_id=None)
```

函数参数：

| **名称**  | **输入/输出** | **类型** | **描述**                                                     |
| --------- | ------------- | -------- | ------------------------------------------------------------ |
| device_id | 输入参数      | int      | NPC Cluster的标识符。默认值为None，代表使用默认NPC Cluster。<br/>说明：如果已调用hpe.hpert.api.set_device指定NPC Cluster，则为该NPC Cluster为默认NPC Cluster；否则NPC Cluster 0为默认NPC Cluster。 |

函数返回值：

| **类型**            | **描述**                                              |
| ------------------- | ----------------------------------------------------- |
| hpe.hpert.api.Event | hpe.hpert.api.Event的对象，包括了管理事件所需的信息。 |

#### hpe.hpert.api.Event.synchronize

函数描述：等待当前事件执行完成。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.Event.synchronize()
```

函数参数：

None

函数返回值：

None

#### hpe.hpert.api.Event.record

函数描述：在指定流的当前运行点添加该事件。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.Event.record(stream=0)
```

函数参数：

| **名称** | **输入/输出** | **类型**             | **描述**                                                |
| -------- | ------------- | -------------------- | ------------------------------------------------------- |
| stream   | 输入参数      | hpe.hpert.api.Stream | 流标识符，默认值为0，代表在所有流的当前运行点添加事件。 |

函数返回值：

None

#### hpe.hpert.api.Stream.wait

函数描述：当前流需要等指定事件被置为完成状态后再开始处理请求。

函数类型：同步函数

函数定义：

```Python
hpe.hpert.api.Stream.wait(event)
```

函数参数：

| **名称** | **输入/输出** | **类型**            | **描述**     |
| -------- | ------------- | ------------------- | ------------ |
| event    | 输入参数      | hpe.hpert.api.Event | 事件标识符。 |

函数返回值：

None

### 数据类型

#### class hpe.hpert.api.Module

数据描述：管理加载到设备端的程序。

#### class hpe.hpert.api.DeviceMem

数据描述：管理在设备端分配的一段内存。

#### class hpe.hpert.api.Stream

数据描述：管理流信息。

#### class hpe.hpert.api.Event

数据描述：管理事件信息。

#### class hpe.hpert.api.CruntimeException

数据描述：管理接口执行报错时返回的Error Message。支持返回的Error Message及描述如下所示：

| **Error Code**                  | **Error Message**                                            | **描述**                                                     |
| ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| stcErrorInvalidValue            | parameters is not within an acceptable range of values       | 一个或多个参数的取值超出了有效值范围。                       |
| stcErrorInvalidDevice           | invalid NPC Cluster device ID                                | 使用了无效的NPC Cluster ID。                                 |
| stcErrorHostMemoryAllocation    | fail to allocate host memory                                 | 在主机端分配内存失败。                                       |
| stcErrorDeviceMemoryAllocation  | fail to allocate device memory                               | 在设备端分配内存失败。                                       |
| stcErrorInvalidDevicePointer    | invalid device memory pointer                                | 使用了无效的设备端内存地址。                                 |
| stcErrorLinkFailure             | fail to link device image                                    | 设备端目标程序链接失败。                                     |
| stcErrorInvalidKernel           | invalid kernel name                                          | 使用了无效的核函数名称。                                     |
| stcErrorInvalidImage            | invalid device image                                         | 设备端目标程序不可用。<br/>说明：设备端目标程序对应fat binary，而设备端目标模块则对应具体的binary。 |
| stcErrorNoImage                 | no device image                                              | 设备端目标程序不存在。                                       |
| stcErrorInvalidModule           | invalid module                                               | 设备端目标模块不可用。<br/>说明：设备端目标程序对应fat binary，而设备端目标模块则对应具体的binary。 |
| stcErrorNoModule                | no module                                                    | 设备端目标模块不存在。                                       |
| stcErrorInvalidStream           | invalid stream                                               | 流不可用。                                                   |
| stcErrorInvalidEvent            | invalid event                                                | 事件不可用。                                                 |
| stcErrorInvalidFifo             | invalid fifo                                                 | 队列不可用。                                                 |
| stcErrorDeviceImageException    | device image run into exception                              | 设备端目标程序运行时出现异常。                               |
| stcErrorSyscallFailure          | system call failure                                          | 主机端的系统调用失败。                                       |
| stcErrorForkForbidden           | forbid to call runtime API, if parent process called runtime API | 父进程已调用过运行时接口，禁止子进程再次调用。               |
| stcErrorInvalidCoreNum          | invalid core number                                          | 使用了无效的核数量。                                         |
| stcErrorGDBFailure              | stcgdb can't trace current process                           | stc-gdb跟踪失败，无法获取调试信息。                          |
| stcErrorDriverMismatch          | mismatch between version of hpert and version of stc-drv     | NPU驱动版本和NPU设备不匹配。                                 |
| stcErrorDeviceBreakdown         | device break down                                            | NPU设备出现故障，无法继续使用。                              |
| stcErrorNoFreeNpu               | no free NPU device                                           | 没有可用的NPU设备。                                          |
| stcErrorNpuNotAcquired          | the npu is not acquired                                      | NPU设备未被占用，但无法访问该NPU设备。                       |
| stcErrorInvalidNpuTask          | invalid NPU task                                             | NPU任务不可用。                                              |
| stcErrorInvalidImageDataSection | too many data section in device image                        | 设备端目标程序包含了太多可写数据段，导致程序无法正常运行。   |
| stcErrorNpuTaskRunning          | NPU task is running with the active NPU module               | NPU任务正在使用NPU模块，不可以卸载该NPU模块。                |
| stcErrorDriverFailure           | PCIe driver internal failure                                 | NPU驱动错误。                                                |

#### class hpe.hpert.api.KernelFlag

数据描述：设置核函数的运行标志。支持设置的运行标志如下所示：

| **枚举定义**                       | **枚举值** | **说明**                                            |
| ---------------------------------- | ---------- | --------------------------------------------------- |
| stcKernelFlagNone                  | 0          | NPC执行核函数前后均处理DCache，可以视为无运行标志。 |
| stcKernelFlagInputDataBypassDcache | 1          | NPC执行核函数前，不需要处理DCache。                 |
| stcKernelFlagOutputDataBypassDache | 2          | NPC执行核函数后，不需要处理DCache。                 |
| stcKernelFlagIODataBypassDache     | 3          | NPC执行核函数前后，均不需要处理DCache。             |

## 性能数据采集（stcpti）接口说明

### 接口调用要求

调用性能数据采集接口时，需要导入对应的库：

```Python
import hpe.profiler.api
```

> 说明：如果报错`ModuleNotFoundError：No module named 'hpe'`，请检查是否安装了HPE Python接口扩展库、是否进入了正确的Python环境。

### 接口说明

#### hpe.profiler.api.init

函数描述：初始化性能数据采集的上下文。

函数类型：同步函数

函数定义：

```Python
hpe.profiler.api.init()
```

函数参数：

None

函数返回值：

| **类型** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| enum     | 如果成功，返回hpe.profiler.capi_ret.STC_PROFILER_SUCCESS。 如果失败，返回hpe.profiler.capi_ret.STC_PROFILER_ERROR。 |

#### hpe.profiler.api.enable

函数描述：开始采集目标程序的性能数据。

函数类型：同步函数

函数定义：

```Python
hpe.profiler.api.enable()
```

函数参数：

None

函数返回值：

| **类型** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| enum     | 如果成功，返回hpe.profiler.capi_ret.STC_PROFILER_SUCCESS。 如果失败，返回hpe.profiler.capi_ret.STC_PROFILER_ERROR。 |

#### hpe.profiler.api.get_perf_datas

函数描述：获取性能数据采集的结果。

函数类型：同步函数

函数定义：

```Python
hpe.profiler.api.get_perf_datas()
```

函数参数：

None

函数返回值：

| **类型**                         | **描述**                                                     |
| -------------------------------- | ------------------------------------------------------------ |
| hpe.profiler.api.ProfilingResult | hpe.profiler.api.ProfilingResult的对象，包括了性能数据采集的结果。 |

#### hpe.profiler.api.disable

函数描述：停止采集目标程序的性能数据。

函数类型：同步函数

函数定义：

```Python
hpe.profiler.api.disable()
```

函数参数：

None

函数返回值：

| **类型** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| enum     | 如果成功，返回hpe.profiler.capi_ret.STC_PROFILER_SUCCESS。 如果失败，返回hpe.profiler.capi_ret.STC_PROFILER_ERROR。 |

#### hpe.profiler.api.release

函数描述：释放性能数据采集的上下文。

函数类型：同步函数

函数定义：

```Python
hpe.profiler.api.release()
```

函数参数：

None

函数返回值：

| **类型** | **描述**                                                     |
| -------- | ------------------------------------------------------------ |
| enum     | 如果成功，返回hpe.profiler.capi_ret.STC_PROFILER_SUCCESS。 如果失败，返回hpe.profiler.capi_ret.STC_PROFILER_ERROR。 |

### 数据类型

#### class hpe.profiler.api.ProfilingResult

数据描述：性能数据采集结果，保存了所有核函数的性能数据，包括所有核函数在每个NPC上的性能数据和sysDMA相关的性能数据。

#### class hpe.profiler.api.KernelFunction

数据描述：单个核函数的性能数据，保存了单个核函数使用NPC的性能数据，包括单个核函数在每个NPC上的性能数据，包括MCU指令、VME指令、MME指令、MTE指令等的性能信息，例如cycle数等。

#### hpe.profiler.capi_ret.STC_PROFILER_SUCCESS

数据描述：性能数据采集接口执行成功。

#### hpe.profiler.capi_ret.STC_PROFILER_ERROR

数据描述：性能数据采集接口执行失败。
