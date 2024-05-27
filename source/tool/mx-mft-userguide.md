# mx-mft 用户手册

```rst
.. note::  
 
   这是一个高亮显示的注意事项。  
```



# **概述**

墨芯 AI 计算卡上的固件分为板载固件和动态加载的固件两个部分，其中板载固件又分为板载设备固件和板载 MCU 固件。mx-mft 是用于管理墨芯 AI 计算卡设备固件工具，您可使用 mx-mft 查询设备固件版本，给设备加载动态固件，重启设备，更新板载设备固件和 MCU 固件。

## 前提条件

在主机上安装 SOLA。具体的步骤，请参见《SOLA Toolkit 安装指南》。 安装 SOLA 后，mx-mft 的二进制文件位于`/usr/bin` 目录下。

## 命令说明

执行 `mx-mft --help` 命令获取 mx-mft 的使用方法： 

```Bash
$ mx-mft --help
mx-mft(V1.05) avaliable commands:
    mx-mft status <id> : show device status
    mx-mft boot <id> <firmware package>: load bootimage for devices
    mx-mft reboot <id> : reboot device
    mx-mft mcu-ota <S30_id> [<fw_file_path>]: ota S30 mcu fw
    mx-mft update <id> <firmware package>: update bootloader for devices

for command detail help, use "mx-mft -h <command>"
for mx-mft version information, use "mx-mft -v"
```

mx-mft 命令支持以下选项：

| **选项**                                 | **说明**                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| -v                                       | 查看 mx-mft 的版本信息                                       |
| status <device_id>                       | 显示设备的状态和版本信息。                                   |
| boot <device_id> <firmware package>      | 对设备加载固件。                                             |
| update <device_id> <firmware package> -f | 更新板卡固件。<br />**注意**：  您需要获取 root 权限后才能更新板卡固件。卡处于 bootloader 模式时才能更新卡的固件，如果卡已经加载动态固件，请使用`mx-mft reboot`命令使卡重新启动到 bootloader 模式。固件升级可以不加 `-f` 参数，固件降级和平级更新则需要增加`-f`参数。 |
| reboot <device_id>                       | 重启设备，使设备进入 bootloader 模式。                       |
| mcu-ota <device_id> [<fw_file_path>]     | 更新板卡上 MCU 的固件。<br />**注意**：仅 S30 计算卡支持此功能，墨芯其他计算卡不支持此功能。 |

> **说明**：
>
> - device_id：设备的 ID，参数可以类似是 0 或者 0-2（表示 0，1，2 这 3 个设备），或者是 all（表示所有的设备）
> - firmware package：墨芯发布的固件包压缩文件，例如 moffett-antoum-V3.52.14-20240327.tar.gz。
> - fw_file_path：墨芯发布的固件二进制文件的路径，例如/usr/local/sola/driver/firmware/ota-test/DMS-BC404X0007.bin

## 命令示例

### 查看设备状态

```Bash
$ mx-mft status 0
```

回显信息示例如下：

- **卡尚未加载固件状态下**

  ```Bash
  #其中Mode: Bootloader表示卡处于bootloader模式
  Device 0 info:
      SN: 9002348s21
      PN: 00S30-00A
      PCI bus: 0000:04:00.0
      Mode: Bootloader
      Bootloader version info:
          version: V12
          type: Release
          build date: Nov 27 2023
          active slot: A
  ```

- **卡已经加载固件状态下**

  ```Bash
  #其中Mode: Kernel UMD表示卡已经加载完成固件
  Device 0 info:
      SN: 9002348s21
      PN: 00S30-00A
      PCI bus: 0000:04:00.0
      Mode: Kernel UMD
      Firmware version: V1.0.14
  ```

### **加载设备的固件**

```Bash
 #以实际文件路径和文件名为准
$ mx-mft boot 0-0 /usr/local/sola/driver/firmware/moffett-antoum*.tar.gz
Extracting boot image...
package boot image version info:
    version: Vxx
    build time: Nov 27 2023 15:42:06
boot image check OK, image size: 65.21MB

Device 0 info:
    SN: 9002348s21
    PN: 00S30-00A
    PCI bus: 0000:04:00.0
    Mode: Bootloader
    Bootloader version info:
        version: Vxx
        type: Release
        build date: Nov 27 2023
        active slot: B

start loading boot image for device 0-0:
    device  0 (00S30-00A - 9002348s21): Boot image load success

total 1 device loading boot image, all success
```

### 重启设备

重启设备，使设备进入 bootloader 状态。

> **注意**：  您需要获取 root 权限后才能重启卡设备。

```Bash
$ sudo mx-mft reboot 0
request device 0(pci bus:0:04:00.0) reboot success
wait devices rebooting...
remove devices from pci bus...
rescan all devices...
device 0 reboot success
```

### 更新板卡固件

```Bash
$ sudo mx-mft update 0-0 /usr/local/sola/driver/firmware/moffett-antoum* -f 
```

### 重启服务器

固升级完成后您需要重启服务器以使用新的固件。

```Bash
$ sudo systemctl restart mx-daemon
```

### 查看固件版本

```Bash
$ mx-qual list | grep "Firmware version"
```

## 使用示例

### **升级固件**

1. 查询当前的固件版本。

    ```Bash
    $ mx-qual list | grep "Firmware version"
    ```

2.  重启设备，使设备进入 bootloader 状态。

   ```Bash
   $ sudo mx-mft reboot all
   ```

3. 更新板载的设备固件升级到 v15 版本。

   ```Bash
   #以实际文件名为准
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.15-*.tar.gz
   ```

4. 重启加载驱动服务。

    ```Bash
    $ sudo systemctl restart mx-daemon
    ```

5. 等待 1 分钟左右后，查询新的固件版本。

   ```Bash
   $ mx-qual list | grep "Firmware version"
   ```

   完整流程示例如下：

   ```Bash
   ### 参考结果：操作前后
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.14
     ...
     Firmware version:   1.0.14
   
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.15-*.tar.gz
   Package bootloader version info:
       version: V15
       type: V14 OTA TEST
       build time: Jan  6 2024 11:24:35
   
   Device 0 info:
       SN: 2023513080278    PN: 00S30-00A
       PCI bus: 0000:51:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V14
           type: Release
           build date: Dec 28 2023        active slot: A
   ...
   
   Device 23 info:
       SN: 2023483080243    PN: 00S30-00A
       PCI bus: 0000:e6:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V14
           type: Release
           build date: Dec 28 2023        active slot: B
   
   Start updating bootloader for device 0-23:
       device  0 (00S30-00A - 2023513080278): bootloader update success, version is V0.15 now
       ...
       device 23 (00S30-00A - 2023483080243): bootloader update success, version is V0.15 now
   
   Total 24 device update bootloader, all success
   
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.15
     ...
     Firmware version:   1.0.15
   ```

### **降级固件**

1. 查询当前的固件版本。

   ```Bash
   $ mx-qual list | grep "Firmware version"
   ```

2. 重启设备，使设备进入 bootloader 状态。

   ```Bash
   $ sudo mx-mft reboot all
   ```

3. 更新板载的设备固件降级到 v13 版本。

   ```Bash
   #以实际文件路径和文件名为准
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.13-*.tar.gz  -f
   ```

4. 重启加载驱动服务。

   ```Bash
   $ sudo systemctl restart mx-daemon
   ```

5. 等待 1 分钟左右后，查询新的固件版本。

   ```Bash
   $ mx-qual list | grep "Firmware version"
   ```

    完整流程示例如下：

    ```Bash
    ## 参考结果：操作前后
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.15
     ...
     Firmware version:   1.0.15
   
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*13-*.tar.gz  -f
   Package bootloader version info:
       version: V15
       type: V14 OTA TEST
       build time: Jan  6 2024 11:14:51
   
   Device 0 info:
       SN: 2023513080278    PN: 00S30-00A
       PCI bus: 0000:51:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V15
           type: Release
           build date: Jan  6 2024        active slot: A
   ...
   
   Device 23 info:
       SN: 2023483080243    PN: 00S30-00A
       PCI bus: 0000:e6:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V15
           type: Release
           build date: Jan  6 2024        active slot: B
   
   Start updating bootloader for device 0-23:
       device  0 (00S30-00A - 2023513080278): bootloader update success(without reboot), version is V13 now
       ...
       device 23 (00S30-00A - 2023483080243): bootloader update success(without reboot), version is V13 now
   
   Total 24 device update bootloader, all success
   
   
   Important: Please power off this server to use device new firmware
   
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.13
     ...
     Firmware version:   1.0.13

### **平级固件**

基于上文的降级固件，进行以下步骤：

1. 查询当前的固件版本。

   ```Bash
   $ mx-qual list | grep "Firmware version"
   ```

2. 重启设备，使设备进入 bootloader 状态。

   ```Bash
   $ sudo mx-mft reboot all
   ```

3. 更新板载的设备固件降级到 v13 版本。

   ```Bash
   #以实际文件路径和文件名为准
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.13-*.tar.gz -f
   ```

4. 重启加载驱动服务。

   ```Bash
   $ sudo systemctl restart mx-daemon
   ```

5. 等待 1 分钟左右后，查询新的固件版本。

   ```Bash
      $ mx-qual list | grep "Firmware version"
   ```

   完整流程示例如下：

   ```Bash
      # e.g. 固件平级
   
   ### 参考结果：操作前后
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.13
     ...
     Firmware version:   1.0.13
   
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.13-*tar.gz -f
   Package bootloader version info:
       version: V13
       type: V14 OTA TEST
       build time: Jan  6 2024 11:14:51
   
   Device 0 info:
       SN: 2023513080278    PN: 00S30-00A
       PCI bus: 0000:51:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V13
           type: Release
           build date: Jan  6 2024        active slot: A
   ...
   
   Device 23 info:
       SN: 2023483080243    PN: 00S30-00A
       PCI bus: 0000:e6:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V13
           type: Release
           build date: Jan  6 2024        active slot: B
   
   Start updating bootloader for device 0-23:
       device  0 (00S30-00A - 2023513080278): bootloader update success(without reboot), version is V13 now
       ...
       device 23 (00S30-00A - 2023483080243): bootloader update success(without reboot), version is V13 now
   
   Total 24 device update bootloader, all success
   
   
   Important: Please power off this server to use device new firmware
   
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.13
     ...
     Firmware version:   1.0.13
   ```

### **更新固件至最新版本**

1. 查询当前的固件版本。

   ```Bash
   $ mx-qual list | grep "Firmware version"
   ```

2. 重启所有设备，使设备进入 bootloader 状态。

    ```Bash
    $ sudo mx-mft reboot all
    ```

3. 更新板载的设备固件升级到最新版本。

   > **说明**：
   >
   >    - 该过程约耗时 3-5 分钟，请耐心等候。
   >    - moffett-antoum-V3.41.14-20240112.tar.gz 仅代表本例，实际以`/usr/local/sola/driver/firmware/`内的同类型文件为准。

    ```Bash
    $ sudo mx-mft update all /usr/local/sola/driver/firmware/moffett-antoum-*.14-*.tar.gz -f
    ```

4.  断电关机。

   ```Bash
   $ sudo shutdown -P now
   ```

5. 在关机后等待 1分钟左右后进行开机，并查询新的固件版本。

    ```Bash
      $ mx-qual list | grep "Firmware version"
    ```

   完整流程示例如下：

   ```Bash
   # e.g. 固件更新至最新版本
   
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.13
     ...
     Firmware version:   1.0.13
   
   $ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-*.14-*.tar.gz
   Package bootloader version info:
       version: V12
       type: V12 OTA TEST
       build time: Jan  6 2024 11:24:35
   
   Device 0 info:
       SN: 2023513080278    PN: 00S30-00A
       PCI bus: 0000:51:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V12
           type: Release
           build date: Dec 28 2023        active slot: A
   ...
   
   Device 23 info:
       SN: 2023483080243    PN: 00S30-00A
       PCI bus: 0000:e6:00.0
       Mode: Bootloader
       Bootloader version info:
           version: V12
           type: Release
           build date: Dec 28 2023        active slot: B
   
   Start updating bootloader for device 0-23:
       device  0 (00S30-00A - 2023513080278): bootloader update success, version is V0.14 now
       ...
       device 23 (00S30-00A - 2023483080243): bootloader update success, version is V0.14 now
   
   Total 24 device update bootloader, all success
   
   $ mx-qual list | grep "Firmware version"  Firmware version:   1.0.14
     ...
     Firmware version:   1.0.14
   ```
   
   

### **降级板卡 MCU** **固件**

> **注意：**
>
> - 仅在 S30 板卡上支持板卡 MCU 的升级和降级。
> - 降级板卡固件时，S30 的 MCU version 必须在 4X05 及以上版本。
> - 板卡需要在加载动态固件后才能更新 MCU 的固件，如果板卡尚未加载动态固件，请使用`mx-mft boot` 命令加载卡的固件。

下文以升级 MCU 版本为例，演示了更新板卡 MCU 固件的流程：

1. 查询设备 0 1 2 的 MCU 版本。

   ```Bash
   $ mx-smi select -f mcu_version -i 0 1 2
   mcu_version
   4X08
   4X08
   4X08
   ```

2. 导入固件二进制文件，对设备 0 1 2 的 mcu 版本降级为 4X07。

   ```Bash
   #以实际文件路径为准
   $ mx-mft mcu-ota 0-2 /usr/local/sola/driver/firmware/ota-test/DMS-BC404X0007.bin
   device 0 mcu ota updating... 
   device 0 mcu ota success! 
   device 1 mcu ota updating... 
   deivce 1 mcu ota success! 
   device 2 mcu ota updating... 
   deivce 2 mcu ota success! 
   ```

3. 降级流程结束后执行以下命令，查看当前 MCU 版本。

   ```Bash
   $ mx-smi select -f mcu_version -i 0 1 2
   mcu_version
   4X07
   4X07
   4X07
   ```

###  升级板卡 MCU 固件

> **注意：**
>
> - 仅在 S30 板卡上支持板卡 MCU 的升级和降级。
> - 升级板卡固件时，S30 的 MCU version 必须在 4X04 及以上版本。
> - 板卡需要在加载动态固件后才能更新 mcu 的固件，如果板卡尚未加载动态固件，请使用`mx-mft boot` 命令加载卡的固件。

1. 查询设备 0 1 2 的 MCU 版本。

   ```Bash
   $ mx-smi select -f mcu_version -i 0 1 2
   mcu_version
   4X07
   4X07
   4X07
   ```

2. 执行以下命令，升级板卡 MCU 固件。

   ```Bash
   ## 对设备0 1 2的mcu版本升级为 4X08，默认不带DMS-xxx.bin参数，则使用当前环境自带的最新版本.
   $ mx-mft mcu-ota 0-2
   device 0 mcu ota updating... 
   device 0 mcu ota success! 
   device 1 mcu ota updating... 
   deivce 1 mcu ota success! 
   device 2 mcu ota updating... 
   deivce 2 mcu ota success! 
   ```

3. 升级流程结束后执行以下命令，查看当前 MCU 版本，确认是否升级成功。

   ```Bash
   ## 查询设备0 1 2的mcu版本，前后对比可知，mcu版本从4X07升级为4X08
   $ mx-smi select -f mcu_version -i 0 1 2
   mcu_version
   4X08
   4X08
   4X08
   ```
