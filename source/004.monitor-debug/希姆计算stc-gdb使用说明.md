# 希姆计算stc-gdb使用说明

## 版本历史

| **文档版本** | **对应产品版本** | **作者** | **日期**   | **说明**                                                     |
| ------------ | ---------------- | -------- | ---------- | ------------------------------------------------------------ |
| V1.3.4       | HPE V1.7.0       | 希姆计算 | 2024-01-15 | 验证文档内容。                                               |
| V1.3.3       | HPE V1.6.2       | 希姆计算 | 2023-07-24 | 验证文档内容。                                               |
| V1.3.2       | HPE V1.6.0       | 希姆计算 | 2023-04-07 | 验证文档内容。                                               |
| V1.3.1       | HPE V1.5.0       | 希姆计算 | 2022-11-30 | 验证文档内容。                                               |
| V1.3.0       | HPE V1.4.0       | 希姆计算 | 2022-08-29 | 更新help回显，去掉未起作用的auto-switch、stc breakpoint。编辑优化。 |
| V1.2.0       | HPE V1.3.0       | 希姆计算 | 2022-07-07 | 支持设置硬件断点、观察点。编辑优化。                         |
| V1.1.0       | HPE V1.2.0       | 希姆计算 | 2022-04-11 | 更新编译命令和示例代码。整篇编辑优化。                       |
| V1.0.0       | Unknown          | 希姆计算 | 2021-09-13 | 初始版本。                                                   |



## 概述

stc-gdb（Stream Computing Debugger）是希姆计算推出的命令行工具，用于调试NPU异构程序。stc-gdb扩展了GDB（GNU Debugger），支持在Linux系统上调试主机端代码，控制运行在NPU上的程序。您可以使用stc-gdb方便地监视程序运行状态，获取和修改程序的中间运行结果，减轻程序开发过程中的调试工作量，提高开发效率。

> 说明：希姆计算软硬件产品相关的基本概念，请参见[希姆计算基本概念](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。

stc-gdb具有如下特性：

- 完全兼容GDB原生命令。
- ⽀持同时调试主机端和设备端的代码。
- ⽀持调试使用NPU单核和多核的程序。
- 支持对设备端代码进行源码级和指令级的单步调试。
- 支持attach机制直接跟踪运行中的程序，方便定位到设备端代码。
- 支持检查和修改程序所使用核的寄存器、变量或其他内存数据。

## 前提条件

- 在主机上部署希姆计算异构环境。具体的步骤，请参见[希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)。stc-gdb的二进制文件位于/usr/local/hpe/bin目录中。
- stc-gdb会将调试过程中产生的临时文件存储在/tmp目录中，因此请保证登录的用户拥有读写/tmp目录的权限。

## 使用流程

1. 编译获得含有调试信息的文件。在编译程序时添加`-g`选项，即可生成含有调试信息的⼆进制⽂件。示例如下，编译源文件matrix_multiply.hc，输出名为matrix_multiply的二进制文件，其中包含了调试信息。

   ```bash
   $ stcc --rtlib=compiler-rt matrix_multiply.hc -g -o matrix_multiply
   ```

2. 启动stc-gdb。在启动stc-gdb可以同时载入编译获得的二进制文件。示例如下，启动stc-gdb并载入名为matrix_multiply的二进制文件。

   ```bash
   $ stc-gdb matrix_multiply
   ```

   如果需要在调试程序时传入更多参数，可以在启动stc-gdb时添加`--args`选项并指定命令行参数。示例如下，启动stc-gdb并载入名为matrix_multiply的二进制文件，并传入命令行参数arg1和arg2。

   ```bash
   $ stc-gdb --args matrix_multiply arg1 arg2
   ```

3. 添加断点，然后运行程序开始调试。启动stc-gdb后，在需要查看调试信息的位置添加断点并运行程序即可。命中断点时程序暂停运行，您可以查看此时的调用堆栈、内存数据等信息。示例如下，分别在matmul函数处和程序第55行设置断点，然后运行程序开始调试。

   > 说明：支持指定函数、代码行、地址等添加断点，主机端代码和设备端代码均可，支持添加软件断点（break/b）、硬件断点（hbreak/hb）、观察点（watch）。

   ```bash
   (stc-gdb) b matmul
   (stc-gdb) b 55
   (stc-gdb) run
   ```

## 调试命令

### 命令类型

运行程序开始调试后，可以执行命令控制程序运行、获取相关的信息。stc-gdb支持的命令类别如下：

- GDB原生命令和选项：命名及使用方式和GDB定义保持一致，例如添加软件断点（break/b）、硬件断点（hbreak/hb）、观察点（watch）等。
- 希姆计算扩展命令和选项：希姆计算的扩展命令均使用stc作为前缀。

> 说明：希姆计算扩展命令仅在异构程序运行在设备端时可用，具体的调试过程请参见*调试示例*章节。

与GDB一致，您可以通过help命令查看命令说明。示例如下，查看扩展命令列表及stc focus命令的说明。

```bash
(stc-gdb) help stc
STC specific commands.

List of stc subcommands:

stc focus -- Set or show the currently controlled STC npu
stc info -- Show STC npu hardware info

Type "help stc" followed by stc subcommand name for full documentation.
Type "apropos word" to search for commands related to "word".
Command name abbreviations are allowed if unambiguous.
(stc-gdb) help stc focus
Set or show the currently controlled STC npu.
```

### stc focus

`stc focus`命令用于在调试程序时查看和管理使用的NPC。

- NPC坐标

  NPC（即core）是运行异构程序的最小单元。一张AI推理卡（device）包括多个cluster，每个cluster包括多个NPC。以STCP920为例，单张STCP920包括4个cluster，单cluster又包括8个NPC。在运行异构程序时可能需要使用多个NPC，为方便灵活操作每个NPC，规定使用[device x, cluster y, core z]的坐标形式标识唯一的NPC。

- 多核调试

  在NPC坐标的基础上，stc-gdb提供了focus机制，通过`stc focus`命令focus到指定的NPC后，该stc-gdb进程的命令只会作用在指定的NPC上，方便您调试多核程序。

`stc focus`命令的示例：

- 查看当前的focus，NPC坐标为[device 0, cluster 0, core 0]。

  ```bash
  (stc-gdb) stc focus
  [Focusing on logical device 0 cluster 0 core 0]
  ```

- 将focus从[device 0, cluster 0, core 0]切换到[device 0, cluster 0, core 2]。

  ```bash
  (stc-gdb) stc focus device 0 cluster 0 core 2
  [Switch from logical device 0 cluster 0 core 0 to logical device 0 cluster 0 core 2.]
  ```

- 如果未指定完整的NPC坐标信息，则参照当前使用中NPC的坐标信息补全，例如自动补全device和cluster。

  ```bash
  (stc-gdb) stc focus core 3
  [Switch from logical device 0 cluster 0 core 2 to logical device 0 cluster 0 core 3.]
  ```

### stc info

`stc info`命令用于查看当前device硬件相关的信息，包括cluster列表、NPC列表、NPC状态、当前focus的NPC等。

可能的NPC状态包括：

- BREAKPOINT：命中断点。
- INTERRUPT：因Ctrl+C等操作中断。
- SINGLESTEP：执行`n`等命令后单步调试。

`stc info`命令的示例：

```bash
(stc-gdb) stc info
device cluster core phy-core     pc       status    focus 
   0      0      0      0    0x1400180  BREAKPOINT    *   
   0      0      1      1    0x1400180  BREAKPOINT        
   0      0      2      2    0x1400180  BREAKPOINT        
   0      0      3      3    0x1400180  BREAKPOINT        
   0      0      4      4    0x1400180  BREAKPOINT        
   0      0      5      5    0x1400180  BREAKPOINT        
   0      0      6      6    0x1400180  BREAKPOINT        
   0      0      7      7    0x1400180  BREAKPOINT 
```

> 说明：focus列中的星号（*）表示当前使用的NPC为[device 0, cluster 0, core 0]。

### attach

stc-gdb兼容原生GDB的attach机制，您可以跟踪运行中的程序，直观了解设备端代码的执行情况。与GDB一致，stc-gdb支持在启动时和启动后attach到运行中程序所在的进程。

`attach`命令的示例：

- 启动stc-gdb时指定进程的pid。

  ```bash
  stc-gdb --pid $(pid of process)
  ```

- 启动stc-gdb后执行attach命令指定进程的pid。

  ```bash
  (stc-gdb) attach $(pid of process)
  ```

  > 说明：部分操作系统中需要sudo权限，请按照终端的提示进行提权操作。

## 调试示例

### 新创建进程并调试程序

启动stc-gdb后，在需要查看调试信息的位置添加断点并运行程序即可。命中断点时程序暂停运行，您可以使用查看此时的调用堆栈、内存数据等信息。

假设有示例异构程序文件matrix_multiply.hc，代码如下：

```c++
/*
 * Copyright (c) 2019-2021 北京希姆计算科技有限公司 (Stream Computing Inc.)
 * All Rights Reserved.
 *
 * NOTICE: All intellectual and technical information contained herein
 * are proprietary to Stream Computing Inc. Any unauthorized disemination,
 * copying or redistribution of this file via any medium is strictly prohibited,
 * unless you get a prior written permission or an applicable license agreement
 * from Stream Computing Inc.
 */
/*
 * This example uses internal instructions to do matrix multiply.
 */

#include <asm_macro.h>
#include <hpe.h>
#include <npurt.h>
#include <stdio.h>

// number of left matrix's col and right matrix's row
#define LCOL_RROW 8

// local_left * local_right = local_out
__device__ void matmul(__fp16 *local_out, __fp16 *local_left,
                       __fp16 *local_right) {
    int shape1, shape2;

    // do matrix multiply and result must be stored in IM buffer
    shape1 = DEFINE_SHAPE(LCOL_RROW, 1);
    shape2 = DEFINE_SHAPE(1, LCOL_RROW);
    CONFIG_VE_BC_CSR(shape1, shape2, 0, 0);
    memul_mm((__fp16 *)IM_BUFFER_START, local_left, local_right);

    // move result from IM buffer to local memory
    shape1 = DEFINE_SHAPE(1, 1);
    shape2 = 0;
    CONFIG_VE_CSR(shape1, shape2, 0, 0);
    mov_m(local_out, (__fp16 *)IM_BUFFER_START);
}

__global__ void matmul_kernel(__fp16 *global_out, __fp16 *global_left,
                              __fp16 *global_right) {
    __local__ __fp16 local_left[LCOL_RROW];
    __local__ __fp16 local_right[LCOL_RROW];
    __local__ __fp16 local_out;
    __shared__ __fp16 share_out[CoreNum];
    printf("CoreNum is %d\n", CoreNum);
    // copy right matrix to each core
    memcpy(local_right, global_right, LCOL_RROW * sizeof(__fp16));

    // copy a row of left matrix for each core
    memcpy(local_left, global_left + CoreID * LCOL_RROW,
           LCOL_RROW * sizeof(__fp16));
    // matrix multiply
    matmul(&local_out, local_left, local_right);

    // copy result in local memory of each core to shared memory
    memcpy(share_out + CoreID, &local_out, sizeof(__fp16));

    // sync to wait each of the core compute share_out data filled
    sync();

    if (CoreID == 0) {
        // copy result in share memory of each core to global memory
        memcpy(global_out, share_out, CoreNum * sizeof(__fp16));
    }
}

#define NCORE 8

int main(void) {
    __fp16 *dev_left, *dev_right, *dev_out;

    // left matrix data for 8 cores
    __fp16 host_left[8 * LCOL_RROW] = {
        0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, // row1
        1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, 1.0, // row2
        2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, 2.0, // row3
        3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, 3.0, // row4
        4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, 4.0, // row5
        5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, 5.0, // row6
        6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, 6.0, // row7
        7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, 7.0, // row8
    };
    // right matrix data
    __fp16 host_right[LCOL_RROW] = {1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0};

    // copy left matrix to device
    int mat_size_left = NCORE * LCOL_RROW * sizeof(__fp16);
    stcMalloc((void **)&dev_left, mat_size_left);
    stcMemcpy(dev_left, host_left, mat_size_left, stcMemcpyHostToDevice);

    // copy right matrix to device
    int mat_size_right = LCOL_RROW * sizeof(__fp16);
    stcMalloc((void **)&dev_right, mat_size_right);
    stcMemcpy(dev_right, host_right, mat_size_right, stcMemcpyHostToDevice);

    // allocate result buffer in device
    int mat_size_out = NCORE * sizeof(__fp16);
    __fp16 host_out[NCORE];
    stcMalloc((void **)&dev_out, mat_size_out);

    matmul_kernel<<<NCORE>>>(dev_out, dev_left, dev_right);
    stcDeviceSynchronize();

    // copy result from device to host
    stcMemcpy(host_out, dev_out, mat_size_out, stcMemcpyDeviceToHost);

    printf("matrix multiply result:");
    for (int i = 0; i < NCORE; i++)
        printf("%.1f, ", (float)(host_out[i]));
    printf("\n");

    stcFree(dev_left);
    stcFree(dev_right);
    stcFree(dev_out);
    return 0;
}
```

按照*使用流程*章节的步骤编译matrix_multiply.hc，获得名为matrix_multiply的二进制文件，即可启动stc-gdb开始调试，以在HPE V1.7.0中进行调试为例。

1. 启动stc-gdb并载入matrix_multiply。

   ```bash
   $ stc-gdb matrix_multiply
   STREAMCOMPUTING (R) STCNPU Debugger
   1.7.0 release.
   Portions Copyright (C) 2019-2021 STREAMCOMPUTING Inc.
   GNU gdb (GDB) 8.2.50.20181127-git
   Copyright (C) 2018 Free Software Foundation, Inc.
   License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   Type "show copying" and "show warranty" for details.
   This GDB was configured as "x86_64-pc-linux-gnu".
   Type "show configuration" for configuration details.
   For bug reporting instructions, please see:
   <http://www.gnu.org/software/gdb/bugs/>.
   Find the GDB manual and other documentation resources online at:
       <http://www.gnu.org/software/gdb/documentation/>.
   
   For help, type "help".
   Type "apropos word" to search for commands related to "word"...
   Reading symbols from matrix_multiply...
   
   warning: A handler for the OS ABI "GNU/Linux" is not built into this configuration
   of GDB.  Attempting to continue with the default riscv:rv32 settings.
   ```

2. 在核函数上添加断点，然后运行程序。

   1. 执行`break`（简写为`b`）命令添加函数断点。

      ```bash
      (stc-gdb) b matmul
      Breakpoint 1 at 0xc000001a: file matrix_multiply.hc, line 29.
      ```

   2. 添加代码行断点，然后执行`run`（简写为`r`）命令运行程序。

      ```bash
      (stc-gdb) b 55
      Breakpoint 2 at 0xc00001b6: file matrix_multiply.hc, line 55.
      (stc-gdb) run
      Starting program: /home/.../matrix_multiply
      [Thread debugging using libthread_db enabled]
      Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
      [Detaching after vfork from child process 3925344]
      warning: Probes-based dynamic linker interface failed.
      Reverting to original interface.
      ```

1. 命中断点后暂停运行程序，查看调用堆栈、focus等信息。
   1. 执行`backtrace`（简写为`bt`）命令查看调用堆栈。
   
      ```bash
      Breakpoint 2, matmul_kernel (global_out=0x1400100, global_left=0x1400000, global_right=0x1400080) at matrix_multiply.hc:55
      55          matmul(&local_out, local_left, local_right);
      (stc-gdb) bt
      #0  matmul_kernel (global_out=0x1400100, global_left=0x1400000, global_right=0x1400080) at matrix_multiply.hc:55
      ```
   
   2. 此时异构程序已经运行到设备端，可以执行扩展命令`stc focus`查看当前使用的NPC，或者使用`stc info`查看NPC状态等更多信息。
   
      ```bash
      (stc-gdb) stc focus
      [Focusing on logical device 0 cluster 0 core 0]
      (stc-gdb) stc info
      device cluster core phy-core     pc       status    focus 
         0      0      0      0    0x1400336  BREAKPOINT    *   
         0      0      1      1    0x1400336  BREAKPOINT        
         0      0      2      2    0x1400336  BREAKPOINT        
         0      0      3      3    0x1400336  BREAKPOINT        
         0      0      4      4    0x1400336  BREAKPOINT        
         0      0      5      5    0x1400336  BREAKPOINT        
         0      0      6      6    0x1400336  BREAKPOINT        
         0      0      7      7    0x1400336  BREAKPOINT        
      ```

4. 切换focus获取对应NPC的控制权，然后执行`continue`（简写为`c`）继续运行。

   ```bash
   (stc-gdb) stc focus device 0 cluster 0 core 2
   [Switch from logical device 0 cluster 0 core 0 to logical device 0 cluster 0 core 2.]
   (stc-gdb) c
   Continuing.
   ```

5. 命中断点后暂停运行程序，查看变量、内存数据等信息。

   1. 执行`print`（简写为`p`）命令打印变量的值。

      > 说明：stc-gdb支持打印半精度`__fp16`类型的变量。

      ```bash
      Breakpoint 1, matmul (local_out=0xc013fff0, local_left=0xc013ffd0, local_right=0xc013ffe0) at matrix_multiply.hc:29
      29          shape1 = DEFINE_SHAPE(LCOL_RROW, 1);
      (stc-gdb) p local_left
      $1 = (__fp16 *) 0xc013ffd0
      (stc-gdb) p *local_left
      $2 = 2

   2. 执行`examine`（简写为`x`）命令查看内存数据。

      ```bash
      (stc-gdb) x /8x local_left
      0xc013ffd0:     0x40004000      0x40004000      0x40004000      0x40004000
      0xc013ffe0:     0x40003c00      0x44004200      0x46004500      0x48004700
      ```

      

   3. 执行`info`（简写为`i`）命令查看寄存器的值。

      ```bash
      (stc-gdb) i registers pc
      pc                  0x140019a   0x140019a <matmul(half*, half*, half*)+26>
      (stc-gdb) i registers
      ra                  0x1400342   0x1400342 <matmul_kernel(half*, half*, half*)+258>
      sp                  0x10dff84   0x10dff84
      gp                  0x4af0      0x4af0
      tp                  0x140a9c0   0x140a9c0
      t0                  0xc0140000  -1072431104
      t1                  0x10        16
      t2                  0x0 0
      fp                  0x10dffb4   0x10dffb4
      s1                  0x2 2
      a0                  0x80001     524289
      a1                  0xc013ffd0  -1072431152
      a2                  0xc013ffe0  -1072431136
      a3                  0x10        16
      a4                  0x0 0
      a5                  0x1 1
      a6                  0x10dff58   17694552
      a7                  0x0 0
      s2                  0x34980     215424
      s3                  0x2b        43
      s4                  0x1400240   20972096
      s5                  0x26980     158080
      s6                  0xc1005c3c  -1056940996
      s7                  0x0 0
      s8                  0x10e0000   17694720
      s9                  0x140a9c0   21014976
      s10                 0xc1020004  -1056833532
      --Type <RET> for more, q to quit, c to continue without paging--
      ```

   4. 执行`info`（简写为`i`）命令查看扩展寄存器的值。

      ```bash
      (stc-gdb) i registers tid
      tid                 0x2 2
      (stc-gdb) i registers shape_s1
      shape_s1            0x800002    8388610
      ```

6. 执行`si`和`ni`命令进行汇编指令级单步调试，执行`s`、`n`命令进行源码级单步调试。

   ```bash
   (stc-gdb) si
   0x0140019e      29          shape1 = DEFINE_SHAPE(LCOL_RROW, 1);
   (stc-gdb) ni
   0x014001a0      29          shape1 = DEFINE_SHAPE(LCOL_RROW, 1);
   (stc-gdb) n
   30          shape2 = DEFINE_SHAPE(1, LCOL_RROW);
   (stc-gdb) s
   31          CONFIG_VE_BC_CSR(shape1, shape2, 0, 0);
   ```

7. 执行continue（简写为c）继续运行，命中断点后暂停运行程序，查看pc附近的汇编指令等信息。

   1. 命中断点后暂停运行程序。

      ```bash
      (stc-gdb) c
      Continuing.
      
      Breakpoint 2, matmul_kernel (global_out=0x1400100, global_left=0x1400000, global_right=0x1400080) at matrix_multiply.hc:55
      55          matmul(&local_out, local_left, local_right);
      ```

   2. 执行`disassemble`（简写为`disass`）命令查看当前pc附近的汇编指令。

      ```bash
      (stc-gdb) disass
      Dump of assembler code for function matmul_kernel(half*, half*, half*):
         0x01400240 <+0>:     addi    sp,sp,-16
         0x01400242 <+2>:     sw      a0,12(sp)
         0x01400244 <+4>:     sw      a1,8(sp)
         0x01400246 <+6>:     sw      a2,4(sp)
         0x01400248 <+8>:     sw      ra,0(sp)
         0x0140024a <+10>:    li      a1,48
         0x0140024e <+14>:    fmv.x.w a0,fs9
         0x01400252 <+18>:    auipc   ra,0x1
         0x01400256 <+22>:    jalr    1998(ra) # 0x1401a20 <check_local_memory>
         0x0140025a <+26>:    fmv.x.w a0,fs9
         0x0140025e <+30>:    addi    a0,a0,-48
         0x01400262 <+34>:    fmv.w.x fs9,a0
         0x01400266 <+38>:    lw      a0,12(sp)
         0x01400268 <+40>:    lw      a1,8(sp)
         0x0140026a <+42>:    lw      a2,4(sp)
         0x0140026c <+44>:    lw      ra,0(sp)
         0x0140026e <+46>:    addi    sp,sp,16
         0x01400270 <+48>:    addi    sp,sp,-64
         0x01400272 <+50>:    sw      ra,60(sp)
         0x01400274 <+52>:    sw      s0,56(sp)
         0x01400276 <+54>:    fsw     fs8,52(sp)
         0x01400278 <+56>:    fsw     fs11,48(sp)
         0x0140027a <+58>:    fmv.x.w a0,fs11
         0x0140027e <+62>:    mv      a0,a0
         0x01400280 <+64>:    fmv.w.x fs8,a0
         0x01400284 <+68>:    addi    s0,sp,64
         0x01400286 <+70>:    lw      a0,8(s0)
         0x01400288 <+72>:    lw      a0,4(s0)
      --Type <RET> for more, q to quit, c to continue without paging--
      ```

8. 执行`info`（简写`i`）命令查看已添加的断点，并执行`delete`（简写`d`）删除对应的断点。

   ```bash
   (stc-gdb) i b
   Num     Type           Disp Enb Address    What
   1       breakpoint     keep y   <MULTIPLE> 
           breakpoint already hit 1 time
   1.1                         y   0x0140019a in matmul(half*, half*, half*) at matrix_multiply.hc:29
   1.2                         n   0xc000001a in matmul(half*, half*, half*) at matrix_multiply.hc:29
   2       breakpoint     keep y   <MULTIPLE> 
           breakpoint already hit 2 times
   2.1                         y   0x01400336 in matmul_kernel(half*, half*, half*) at matrix_multiply.hc:55
   2.2                         n   0xc00001b6 in matmul_kernel(half*, half*, half*) at matrix_multiply.hc:55
   (stc-gdb) d 1
   (stc-gdb) i b
   Num     Type           Disp Enb Address    What
   2       breakpoint     keep y   <MULTIPLE> 
           breakpoint already hit 2 times
   2.1                         y   0x01400336 in matmul_kernel(half*, half*, half*) at matrix_multiply.hc:55
   2.2                         n   0xc00001b6 in matmul_kernel(half*, half*, half*) at matrix_multiply.hc:55
   (stc-gdb) d 2
   (stc-gdb) i b
   No breakpoints or watchpoints.
   ```

9. 执行`kill`（简写为`k`）强制结束进程并退出调试。

   ```bash
   (stc-gdb) k
   Kill the program being debugged? (y or n) y
   [Inferior 1 (process 357667) killed]
   ```

### attach已有进程并调试程序

假设有示例异构程序文件hello_world.hc，代码如下：

```c++
/*
 * Copyright (c) 2019-2021 北京希姆计算科技有限公司 (Stream Computing Inc.)
 * All Rights Reserved.
 *
 * NOTICE: All intellectual and technical information contained herein
 * are proprietary to Stream Computing Inc. Any unauthorized disemination,
 * copying or redistribution of this file via any medium is strictly prohibited,
 * unless you get a prior written permission or an applicable license agreement
 * from Stream Computing Inc.
 */
/*
 * This example uses NPURT API 'printf' to print device message in host
 * terminal.
 */
#include <asm_macro.h>
#include <hpe.h>
#include <npurt.h>
#include <stdio.h>

__global__ void hello(void) {
    while (1) {
        printf("hello world from core %d/%d.\n", CoreID, CoreNum);
    }
}

#define NCORE 8

int main(void) {

    printf("running hello_world......\n");

    hello<<<NCORE>>>();
    stcDeviceSynchronize();

    return 0;
}
```

假设hello_world当前已经在设备端运行，且pid为448655，以在HPE V1.7.0中进行调试为例。

1. 启动stc-gdb。

   ```bash
   $ sudo stc-gdb
   STREAMCOMPUTING (R) STCNPU Debugger
   1.7.0 release.
   Portions Copyright (C) 2019-2021 STREAMCOMPUTING Inc.
   GNU gdb (GDB) 8.2.50.20181127-git
   Copyright (C) 2018 Free Software Foundation, Inc.
   License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.
   Type "show copying" and "show warranty" for details.
   This GDB was configured as "x86_64-pc-linux-gnu".
   Type "show configuration" for configuration details.
   For bug reporting instructions, please see:
   <http://www.gnu.org/software/gdb/bugs/>.
   Find the GDB manual and other documentation resources online at:
       <http://www.gnu.org/software/gdb/documentation/>.
   
   For help, type "help".
   Type "apropos word" to search for commands related to "word".
   ```

2. 执行`attach`（简写为`at`）命令指定hello_world所在的进程，跟踪到hello_world。

   ```bash
   (stc-gdb) at 448655
   Attaching to process 448655
   Reading symbols from /home/.../hello_world...
   warning: A handler for the OS ABI "GNU/Linux" is not built into this configuration
   of GDB.  Attempting to continue with the default riscv:rv32 settings.
   
   Reading symbols from /usr/local/hpe/lib/libhpert.so.1.2...
   (No debugging symbols found in /usr/local/hpe/lib/libhpert.so.1.2)
   Reading symbols from /usr/local/hpe/lib/libprofiler_stc.so.1.2...
   (No debugging symbols found in /usr/local/hpe/lib/libprofiler_stc.so.1.2)
   Reading symbols from /usr/local/hpe/lib/libtracer_stc.so.1.2...
   (No debugging symbols found in /usr/local/hpe/lib/libtracer_stc.so.1.2)
   Reading symbols from /lib/x86_64-linux-gnu/libpthread.so.0...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libpthread.so.0)
   [Thread debugging using libthread_db enabled]
   Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
   Reading symbols from /lib/x86_64-linux-gnu/libstdc++.so.6...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libstdc++.so.6)
   Reading symbols from /lib/x86_64-linux-gnu/libm.so.6...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libm.so.6)
   Reading symbols from /lib/x86_64-linux-gnu/libc.so.6...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libc.so.6)
   Reading symbols from /lib/x86_64-linux-gnu/libgcc_s.so.1...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libgcc_s.so.1)
   Reading symbols from /lib64/ld-linux-x86-64.so.2...
   (No debugging symbols found in /lib64/ld-linux-x86-64.so.2)
   Reading symbols from /lib/x86_64-linux-gnu/libdl.so.2...
   (No debugging symbols found in /lib/x86_64-linux-gnu/libdl.so.2)
   0x01401288 in send_uart ()
   ```

3. 执行扩展命令`stc info`查看NPC状态等信息。

   ```bash
   (stc-gdb) stc info
   device cluster core phy-core     pc       status    focus 
      0      0      0      0    0x140125e   INTERRUPT    *   
      0      0      1      1    0x1401292   INTERRUPT        
      0      0      2      2    0x14011ac   INTERRUPT        
      0      0      3      3    0x14011ac   INTERRUPT        
      0      0      4      4    0x140118c   INTERRUPT        
      0      0      5      5    0x140125e   INTERRUPT        
      0      0      6      6    0x1401256   INTERRUPT        
      0      0      7      7    0x1401292   INTERRUPT        
   ```

4. 执行`break`（简写为`b`）命令为设备端代码添加断点，然后执行`continue`（简写为`c`）继续运行。命中断点后，暂停运行程序。

   ```bash
   (stc-gdb) b 22
   Breakpoint 1 at 0x140000c: file hello_world.hc, line 22.
   (stc-gdb) c
   Continuing.
   
   Breakpoint 1, hello () at hello_world.hc:22
   22              printf("hello world from core %d/%d.\n", CoreID, CoreNum);
   ```

5. 执行`quit`（`q`）命令退出调试。

   ```bash
   (stc-gdb) q
   A debugging session is active.
   
           Inferior 1 [process 448655] will be detached.
   
   Quit anyway? (y or n) y
   Detaching from program: /home/.../hello_world, process 448655
   [Inferior 1 (process 448655) detached]
   ```

## 常见问题

- 在attach已有进程时，建议sudo或root权限启动stc-gdb，否则可能出现以下报错：

  ```bash
  Attaching to process 448655
  Could not attach to process. Try again as the root user or the sudoers.
  ptrace: Operation not permitted.
  ```
