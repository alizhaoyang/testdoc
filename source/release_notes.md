# 发布日志

<p class="attention">注意：这是一个需要注意的段落。</p>

<p class="danger">警告：危险，一定要看。</p>

 <p class="note">说明：这是一个需要说明的段落。</p>


## MACS 发布说明

发布日志中包含了 MACS 中各个组件当前版本以及历史版本的更新说明。

Note

Another use case for this is when you have a module with a C extension.

## 文档目标

```rst
.. note::
  本文档是 Moffett Software 的第二版的设计文档大纲. 目前并不是稳定版, 还需要大量的迭代.

```

```rst
.. warning::
  文档中可能存在短期内无法完成的目标, 请谨慎阅读.

```

 <p class="note">说明：这是一个需要说明的段落。</p>

## 定义

## 主要组件版本

| 组件                             | 名称              | 版本    | 支持架构 | 支持平台 |
| -------------------------------- | ----------------- |-------| -------- | -------- |
| SOLA Runtime Library             | libsola.so        | 3.5.0 | x86_64   | Linux    |
| SOLA Driver                      | N/A               | 3.5.0 | x86_64   | Linux    |
| MOFFETT Management Library       | libmfml.so        | 1.0.4 | x86_64   | Linux    |
| MOFFETT SMI                      | mx-smi            | 2.1.0 | x86_64   | Linux    |
| MOFFETT Qualification            | mx-qual           | 1.2.0 | x86_64   | Linux    |
| MOFFETT Firmware Tools           | mx-mft            | 1.0.1 | x86_64   | Linux    |
| MOFFETT MCU                      | N/A               | 4x08  | x86_64   | Linux    |
| MOFFETT Kubernetes Device Plugin | k8s-device-plugin | 0.1.0 | x86_64   | Linux    |

### 组件更新说明

#### MACS 1.1 更新说明

**SOLA Runtime**

- 新增特性
  - 所有 C++ 接口重构为 C 接口
- 修复问题
  - 修复打开 Profiler 某些情况下写文件会 crash 的问题

**SOLA Driver**

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

**MOFFETT Management Library**

- 新增特性
  - 增加获取 XID Error 的 API

**MOFFETT SMI**

- 新增特性
  - `--help`, `--version` 参数在驱动未加载时也可以运行
- 修复问题
  - `--help` 中 `device_id` 范围变为 0 ~ 31
  - 修复 `list` / `query` 命令参数重复时的显示问题

**MOFFETT Qualification**

- 修复问题
  - 修复 `stress` 命令没有权限创建文件引起的 crash 问题
  - `compute` 命令 `conv2d` 必须指定 `--sparsity` 和 `--iochannel` 参数
  - `compute` 命令 `multiply` 不允许指定 `--sparsity` 和 `--iochannel` 参数

**MOFFETT Kubernetes Device Plugin**

- 第一次发布墨芯 k8s-device-plugin, 支持在 Kubernetes 集群中使用墨芯 AI 加速卡设备。

#### MACS 1.0 更新说明

**SOLA Runtime**

- 第一次发布 SOLA Runtime API。

**SOLA Driver**

- 第一次发布 SOLA Driver。

**MOFFETT Management Library**

- 第一次发布 `MXML` 库。

**MOFFETT SMI**

- 第一次发布 `mx-smi` 工具。

**MOFFETT Qualification**

- 第一次发布 `mx-qual` 工具。

**MOFFETT Firmware Tools**

- 第一次发布 `mx-mft` 工具。
