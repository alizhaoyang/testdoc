# MACS 发布日志

## 概述

墨芯高级计算系统（MOFFETT Advanced Computing System，简称MACS)以SOLA Tookit的形式交付，它提供了驱动、运行时库以及一系列的工具，便于用户开发和调试在墨芯 AI 加速卡上的应用程序。MACS发布日志记录了SOLA各组件的版本、功能变更、问题修复等信息。

## 主要组件版本

| **组件**                         | **名称**          | **版本** | **支持架构** | **支持平台** |
| -------------------------------- | ----------------- | -------- | ------------ | ------------ |
| SOLA Runtime Library             | libsola.so        | 3.6.0    | x86_64       | Linux        |
| SOLA Driver                      | N/A               | 3.6.0    | x86_64       | Linux        |
| SOLA Firmware                    | N/A               | 3.6.0    |              |              |
| MOFFETT Management Library       | libmfml.so        | 1.05     | x86_64       | Linux        |
| MOFFETT SMI                      | mx-smi            | 2.3.0    | x86_64       | Linux        |
| MOFFETT Qualification            | mx-qual           | 1.4.0    | x86_64       | Linux        |
| MOFFETT Firmware Tools           | mx-mft            | 1.09     | x86_64       | Linux        |
| MOFFETT UBoot Firmware           | N/A               | 1.0.14   | x86_64       | Linux        |
| MOFFETT MCU                      | N/A               | 4x011    | x86_64       | Linux        |
| MOFFETT Kubernetes Device Plugin | k8s-device-plugin | 0.1.0    | x86_64       | Linux        |

## MACS 1.2 更新说明

### **SOLA Runtime**

- 重构大模型 trigger 方案
- 支持大模型 data pipeline
- 支持 4 core split 模式下内存只由 core 0 输出
- 支持大模型 profile
- 支持获取设备的 SN、board_id
- 优化日志

### **SOLA Driver**

- 支持通过 PCIe switch 测试设备眼图
- 支持 UCE 计数
- 修复个别 AMD 芯片服务器无法识别卡问题
- 曙光服务器和麒麟系统的适配
- 设备启动恢复电压值为 default 值

### **MOFFETT Qualification**

- 优化 help 信息
- 优化 stress 功能，支持任意负载

### 其它

- 安装包增加 EULA（End User License Agreement）
- Runfile 安装增加依赖检查

#### MACS 1.1.4 更新说明

**其它**

- Deb、RPM安装包结构规范
- 预编译二进制增加 Full RELRO、PIE、Canary 等安全编译选项

## MACS 1.1.3 更新说明

### **SOLA Driver**

- 增加开机时 PCIe 速率检测，发现降速后自动恢复

### **MOFFETT Qualification**

- 将 `p2p` 可选参数改为 `-d` 和 `-c`，分别表示以 device 为单位或以 card 为单位进行测试
- 修复 `p2p` 测试双向带宽计算错误的 bug
- 修复 `p2p` 测试 latency 波动不合理的 bug
- 优化 `compute` 测试，展示在不同稀疏倍率下的算力
- `compute` 测试增加可选参数 `-i`，用于指定测试的设备
- `stress` 测试删除可选参数 `--loop`，调整可选参数 `--time` 的数值范围
- `stress` 测试增加可选参数 `--load`，表示负载程度，支持0%、50%、100%负载
- `stress` 改为同时进行计算和内存测试

**MOFFETT** **SMI**

- 以 board id 作为卡展示模式下的 id
- 修复某些情况下，展示的 device 和实际可用的数量不一致的问题
- 优化 `reboot` 在 device 无法访问时的错误信息

## **MACS 1.1.2 更新说明**

### **SOLA Driver**

- 修复reboot时电压偶现为0的问题
- 裁减固件大小，将开机加载固件优化到20秒

### **MOFFETT Qualification**

- 修复执行路径无写入权限导致的crash

### **其它**

- moffett-bug-report.sh 需要root权限，以收集PCIe的详细信息

## **MACS 1.1 更新说明**

### **SOLA Runtime**

- 新增特性
  - 所有 C++ 接口重构为 C 接口
- 修复问题
  - 修复打开 Profiler 某些情况下写文件会 crash 的问题

### **SOLA Driver**

- 修复问题
  - 整机功耗调整为 250W
  - Idle 整机功耗调整到 30W 以内
  - 修复通过 BMC 监控 GPU 温度值跳变问题
  - 修复启动过程中，BMC 监控到 GPU 温度值为 0 的问题
  - 修复 mx-smi 概率性读到的功耗为 0 的问题
  - 修复个别卡偶现 subsys 加载固件失败的问题
  - 修复个别服务器 bloom 程序执行失败的问题
  - 修复个别服务器 p2p 程序执行失败的问题
  - 修复个别服务器超温降频后无法恢复成原始频率的问题

### **MOFFETT Management Library**

- 新增特性
  - 增加获取 XID Error 的 API

### **MOFFETT SMI**

- 新增特性
  - `--help`, `--version` 参数在驱动未加载时也可以运行
- 修复问题
  - `--help` 中 `device_id` 范围变为 0 ~ 31
  - 修复 `list` / `query` 命令参数重复时的显示问题

### **MOFFETT Qualification**

- 修复问题
  - 修复 `stress` 命令没有权限创建文件引起的 crash 问题
  - `compute` 命令 `conv2d` 必须指定 `--sparsity` 和 `--iochannel` 参数
  - `compute` 命令 `multiply` 不允许指定 `--sparsity` 和 `--iochannel` 参数

### **MOFFETT Kubernetes Device Plugin**

- 第一次发布墨芯 k8s-device-plugin, 支持在 Kubernetes 集群中使用墨芯 AI 加速卡设备。

## **MACS 1.0 更新说明**

### **SOLA Runtime**

- 第一次发布 SOLA Runtime API。

### **SOLA Driver**

- 第一次发布 SOLA Driver。

### **MOFFETT Management Library**

- 第一次发布 `MXML` 库。

### **MOFFETT SMI**

- 第一次发布 `mx-smi` 工具。

### **MOFFETT Qualification**

- 第一次发布 `mx-qual` 工具。

### **MOFFETT Firmware Tools**

- 第一次发布 `mx-mft` 工具。
