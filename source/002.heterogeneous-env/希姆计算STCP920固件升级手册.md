# 希姆计算STCP920固件升级手册

## 版本历史

| **文档版本** | **对应产品版本**                                   | **作者** | **日期**   | **描述**                                        |
| ------------ | -------------------------------------------------- | -------- | ---------- | ----------------------------------------------- |
| V1.5.0       | NPU ctrl V1.3.4MCU V1.0.8                          | 希姆计算 | 2024-01-15 | 更新固件升级流程。                              |
| V1.4.1       | - NPU ctrl V1.2.3、NPU ctrl V1.3.3<br>- MCU V1.0.6 | 希姆计算 | 2022-12-14 | 增加NPU ctrl firmware新版本相关限制和流程说明。 |
| V1.3.1       | - NPU ctrl V1.2.2<br/>- MCU V1.0.6                 | 希姆计算 | 2022-12-05 | - 编辑优化。<br/>- 增加使用限制。               |
| V1.3.0       | - NPU ctrl V1.2.2<br/>- MCU V1.0.6                 | 希姆计算 | 2022-09-05 | 升级流程中添加步骤。                            |
| V1.2.0       | - NPU ctrl V1.2.2<br/>- MCU V1.0.6                 | 希姆计算 | 2022-08-03 | - 更新复位设备命令。<br/>- 编辑优化。           |
| V1.1.0       | Unknown                                            | 希姆计算 | 2022-02-14 | 增加板卡重启命令。                              |
| V1.0.0       | Unknown                                            | 希姆计算 | 2022-01-05 | 初始版本。                                      |

## 前提条件

- 已搭建好STCP920 AI推理卡的使用环境，具体操作，请参见[希姆计算异构环境安装指南](https://docs.streamcomputing.com/_/sharing/vSxLMI20nalGphdpXdEVoDg6JkUcfEkT?next=/zh/latest/)中的*管理硬件*和*安装HPE*章节。
- 已获取用于升级固件的文件。STCP920的固件包括NPU ctrl firmware和MCU firmware。

## 命令说明

使用stc-smi命令即可升级固件，同时请注意以下方面：

- 待升级固件：STCP920的固件包括NPU ctrl firmware和MCU firmware，支持分别升级，通过命令选项指定即可。指定NPU ctrl firmware的形式为`-u {file_name}.sdux`，指定MCU firmware的形式为`--mcu-upgrade {file_name}.sdux`。
- 固件升级文件：固件升级文件的后缀统一为sdux，文件请以最新稳定版为准。
- 操作权限：升级NPU crtl firmware时需要sudo权限。
- 待升级设备：升级时指定需要升级固件的NPU设备，指定NPU设备的形式为`-i {device_id}`。

## 使用限制

- 仅支持在宿主机上升级固件，禁止在VM上执行升级固件的步骤。
- 支持将STCP920 AI推理卡passthrough到基于KVM的VM，同时要求NPU ctrl firmware为V1.3.4或以上版本、HPE为V1.5.1或以上版本。另外，需要注意以下限制：
  - NPU ctrl要求使用PCIe类型的接口才能正确读取板卡信息，基于要求使用PCIe类型的接口，限制KVM必须使用q35类型的主板模拟，而i440类型的主板不可用。

## 升级流程

> 注意：由于服务器硬件存在差异，复位设备命令不能保证在所有机型上都能升级成功，这时需要断电重启服务器才能使升级生效。

1. 执行`stc-smi -q`命令，查看当前的固件版本信息。
2. 确认服务器上是否开启了smi_info.service或npu_exporter.service服务，若已开启，需要先关闭并禁用。

   ```bash
   $ sudo systemctl stop smi_info.service
   $ sudo systemctl stop npu_exporter.service
   $ sudo systemctl disable smi_info.service
   $ sudo systemctl disable npu_exporter.service
   ```
3. 执行`sudo stc-smi -u {file_name}.sdux -i {device_id}`命令，为指定NPU设备升级NPU ctrl firmware。
4. 执行`stc-smi --mcu-upgrade {file_name}.sdux -i {device_id}`命令，为指定NPU设备升级MCU firmware。
5. 在BMC上执行上下电操作，使NPU ctrl firmware升级和MCU firmware升级生效生效。
6. 执行`stc-smi -q`命令，查看NPU ctrl firmware和MCU firmware的固件版本信息，已更新到对应版本则代表升级成功。
7. 重启服务器后smi_info.service或npu_exporter.service服务不会自动启动，升级完成后，再打开服务。

   ```bash
   $ sudo systemctl enable smi_info.service
   $ sudo systemctl enable npu_exporter.service
   $ sudo systemctl start smi_info.service
   $ sudo systemctl start npu_exporter.servic
   ```
