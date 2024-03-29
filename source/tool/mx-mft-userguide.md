# mx-mft 用户文档

## 概述

MOFFETT SPU上的固件分为板载固件和动态加载的固件两个部分，其中板载固件又分为板载设备固件和板载MCU固件。

mx-mft是用于管理MOFFETT SPU 设备固件工具，管理员可使用mx-mft查询设备固件版本，给设备加载动态固件，重启设备，更新板载设备固件和MCU固件。

## mx-mft子命令

* `mx-mft status <device_id>`
  
   列出加速卡设备，显示设备的状态和版本信息

* `mx-mft boot <device_id> <firmware package>`
   
   对设备加载固件

* `mx-mft update <device_id> <firmware package>`
  
   更新板载固件

* `mx-mft reboot <device_id>`
  
   重启设备，使设备进入bootloader状态

* `mx-mft mcu-ota <device_id> [<fw_file_path>]`
  
   更新板卡上MCU的固件

`device_id`: 参数可以类似是0或者0-2（表示0,1,2这3个设备），或者是all（表示所有的设备）

`firmware package`: 参数指的是moffett发布的固件包压缩文件，其命名类似为moffett-antoum-V3.31.13-20231127.tar.gz

### 子命令说明

#### 1.  `mx-mft status <device id>`

 mx-mft status命令用于列出加速卡设备，以及显示设备的状态和版本信息

#### 卡尚未加载固件状态下

命令：

```
mx-mft status 0
```

输出：

```
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

其中Mode: Bootloader表示卡处于bootloader模式

##### 卡已经加载固件状态下

命令：

```
mx-mft status 0
```

输出：

```
Device 0 info:
    SN: 9002348s21
    PN: 00S30-00A
    PCI bus: 0000:04:00.0
    Mode: Kernel UMD
    Firmware version: V1.0.14
```

其中Mode: Kernel UMD表示卡已经加载完成固件

#### 2. 加载固件

`mx-mft boot <device id> <firmware package>`

mx-mft boot命令用于对卡设备加载固件

命令：

```
mx-mft boot 0-0 moffett-antoum-V3.31.13-20231127.tar.gz
```

输出：

```
Extracting boot image...
package boot image version info:
    version: V3.31
    build time: Nov 27 2023 15:42:06
boot image check OK, image size: 65.21MB

Device 0 info:
    SN: 9002348s21
    PN: 00S30-00A
    PCI bus: 0000:04:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V13
        type: Release
        build date: Nov 27 2023
        active slot: B

start loading boot image for device 0-0:
    device  0 (00S30-00A - 9002348s21): Boot image load success

total 1 device loading boot image, all success

```

#### 3. 重启设备

`mx-mft reboot <device id>`

mx-mft reboot命令用于重启设备，使设备进入bootloader状态

命令：

```
sudo mx-mft reboot 0
```

输出：

```
request device 0(pci bus:0:04:00.0) reboot success
wait devices rebooting...
remove devices from pci bus...
rescan all devices...
device 0 reboot success
```

注意:  需要使用root权限才能重启卡设备

#### 4. 更新固件

命令：

```shell
mx-mft update <device id> <firmware package>
```

mx-mft update命令用于更新板卡的固件

1. 需要使用root权限才能更新卡设备固件

2. 需要卡处于bootloader模式时才能更新卡的固件，如果卡已经加载动态固件，请使用mx-mft reboot 命令使卡重新启动到bootloader模式

3. 固件升级可以不加 -f 参数，固件降级和平级更新则需要增加 -f 参数

4. 升级完成后如果提示需要关闭服务器，则将服务器关闭再重启以使用新的固件

命令：

```
sudo mx-mft update 0-0 moffett-antoum-V3.31.13-20231127.tar.gz -f
```

#### 固件升级案例：

```bash
e.g. 固件升级
# 查询当前Firmware version
mx-qual list | grep "Firmware version"

# 重启设备，使设备进入bootloader状态
sudo mx-mft reboot all

# 更新板载的设备固件升级到v15版本
sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.15-20240106.tar.gz

# 重启加载驱动服务
sudo systemctl restart mx-daemon

# 等待2-3分钟，查询更新结果
mx-qual list | grep "Firmware version"


### 参考结果：操作前后
$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.14
  ...
  Firmware version:   1.0.14

$ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.15-20240106.tar.gz
Package bootloader version info:
    version: V15
    type: V14 OTA TEST
    build time: Jan  6 2024 11:24:35

Device 0 info:
    SN: 2023513080278
    PN: 00S30-00A
    PCI bus: 0000:51:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V14
        type: Release
        build date: Dec 28 2023
        active slot: A
...

Device 23 info:
    SN: 2023483080243
    PN: 00S30-00A
    PCI bus: 0000:e6:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V14
        type: Release
        build date: Dec 28 2023
        active slot: B

Start updating bootloader for device 0-23:
    device  0 (00S30-00A - 2023513080278): bootloader update success, version is V0.15 now
    ...
    device 23 (00S30-00A - 2023483080243): bootloader update success, version is V0.15 now

Total 24 device update bootloader, all success

$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.15
  ...
  Firmware version:   1.0.15
```

### 固件降级案例：

```bash
e.g. 固件降级
# 查询当前Firmware version
mx-qual list | grep "Firmware version"

# 重启设备，使设备进入bootloader状态
sudo mx-mft reboot all

# 更新板载的设备固件降级到v13版本
sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.13-20240106.tar.gz -f

# 重启加载驱动服务
sudo systemctl restart mx-daemon

# 等待2-3分钟，查询更新结果
mx-qual list | grep "Firmware version"


## 参考结果：操作前后
$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.15
  ...
  Firmware version:   1.0.15

$ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.13-20240106.tar.gz -f
Package bootloader version info:
    version: V13
    type: V14 OTA TEST
    build time: Jan  6 2024 11:14:51

Device 0 info:
    SN: 2023513080278
    PN: 00S30-00A
    PCI bus: 0000:51:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V15
        type: Release
        build date: Jan  6 2024
        active slot: A
...

Device 23 info:
    SN: 2023483080243
    PN: 00S30-00A
    PCI bus: 0000:e6:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V15
        type: Release
        build date: Jan  6 2024
        active slot: B

Start updating bootloader for device 0-23:
    device  0 (00S30-00A - 2023513080278): bootloader update success(without reboot), version is V13 now
    ...
    device 23 (00S30-00A - 2023483080243): bootloader update success(without reboot), version is V13 now

Total 24 device update bootloader, all success


Important: Please power off this server to use device new firmware

$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.13
  ...
  Firmware version:   1.0.13
```

#### 固件平级案例：

```bash
e.g. 固件平级
# 查询当前Firmware version
mx-qual list | grep "Firmware version"

# 重启设备，使设备进入bootloader状态
sudo mx-mft reboot all

# 更新板载的设备固件降级到v13版本
sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.13-20240106.tar.gz -f

# 重启加载驱动服务
sudo systemctl restart mx-daemon

# 等待2-3分钟，查询更新结果
mx-qual list | grep "Firmware version"


### 参考结果：操作前后
$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.13
  ...
  Firmware version:   1.0.13

$ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.13-20240106.tar.gz -f
Package bootloader version info:
    version: V13
    type: V14 OTA TEST
    build time: Jan  6 2024 11:14:51

Device 0 info:
    SN: 2023513080278
    PN: 00S30-00A
    PCI bus: 0000:51:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V13
        type: Release
        build date: Jan  6 2024
        active slot: A
...

Device 23 info:
    SN: 2023483080243
    PN: 00S30-00A
    PCI bus: 0000:e6:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V13
        type: Release
        build date: Jan  6 2024
        active slot: B

Start updating bootloader for device 0-23:
    device  0 (00S30-00A - 2023513080278): bootloader update success(without reboot), version is V13 now
    ...
    device 23 (00S30-00A - 2023483080243): bootloader update success(without reboot), version is V13 now

Total 24 device update bootloader, all success


Important: Please power off this server to use device new firmware

$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.13
  ...
  Firmware version:   1.0.13
```

### 固件更新至最新版本案例：

```bash
e.g. 固件更新至最新版本
# 查询当前Firmware version
mx-qual list | grep "Firmware version"

# 重启设备，使设备进入bootloader状态
sudo mx-mft reboot all

# 更新板载的设备固件升级到最新版本: 约耗时3-5分钟，请耐心等候。
# moffett-antoum-V3.41.14-20240112.tar.gz 仅代表本例，
# 实际以/usr/local/sola/driver/firmware/内的同类型文件为准。

sudo mx-mft update all /usr/local/sola/driver/firmware/moffett-antoum-V3.41.14-20240112.tar.gz -f

# 断电关机
sudo shutdown -P now

# 开机等待2-3分钟，查询更新结果
mx-qual list | grep "Firmware version"


## 参考结果：操作前后
$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.12
  ...
  Firmware version:   1.0.12

$ sudo mx-mft update all /usr/local/sola/driver/firmware/ota-test/moffett-antoum-V3.40.15-20240106.tar.gz
Package bootloader version info:
    version: V12
    type: V12 OTA TEST
    build time: Jan  6 2024 11:24:35

Device 0 info:
    SN: 2023513080278
    PN: 00S30-00A
    PCI bus: 0000:51:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V12
        type: Release
        build date: Dec 28 2023
        active slot: A
...

Device 23 info:
    SN: 2023483080243
    PN: 00S30-00A
    PCI bus: 0000:e6:00.0
    Mode: Bootloader
    Bootloader version info:
        version: V12
        type: Release
        build date: Dec 28 2023
        active slot: B

Start updating bootloader for device 0-23:
    device  0 (00S30-00A - 2023513080278): bootloader update success, version is V0.14 now
    ...
    device 23 (00S30-00A - 2023483080243): bootloader update success, version is V0.14 now

Total 24 device update bootloader, all success

$ mx-qual list | grep "Firmware version"
  Firmware version:   1.0.14
  ...
  Firmware version:   1.0.14
```

#### 5. 更新板卡 MCU 固件

命令：

```
mx-mft mcu-ota <device_id> <mcu_firmware_package>
```

mx-mft mcu-ota命令用于更新板卡上MCU的固件




**注意事项：**

- 需要确认S30的MCU version是否为V4X05或以上版本，*****其他版本和S4计算卡不支持此功能***
- 需要卡已经加载动态固件后才能更新mcu的固件，如果卡未加载动态固件，请使用mx-mft boot 命令加载卡的固件

命令：

```
mx-mft mcu-ota 0-2 DMS-BC404X0004.bin
```

输出：

```
e.g.
## 查询设备0 1 2的mcu版本
$ mx-smi select -f mcu_version -i 0 1 2
mcu_version
4X05
4X05
4X05


## 对设备0 1 2的mcu版本降级为 4X04
$ mx-mft mcu-ota 0-2 /usr/local/sola/driver/firmware/ota-test/DMS-BC404X0004.bin
device 0 mcu ota updating... 
device 0 mcu ota success! 
device 1 mcu ota updating... 
deivce 1 mcu ota success! 
device 2 mcu ota updating... 
deivce 2 mcu ota success! 


## 查询设备0 1 2的mcu版本，前后对比可知，mcu版本从4X05降为4X04
$ mx-smi select -f mcu_version -i 0 1 2
mcu_version
4X04
4X04
4X04


## 对设备0 1 2的mcu版本升级为 4X05，默认不带DMS-xxx.bin参数，则使用当前环境自带的最新版本
$ mx-mft mcu-ota 0-2
device 0 mcu ota updating... 
device 0 mcu ota success! 
device 1 mcu ota updating... 
deivce 1 mcu ota success! 
device 2 mcu ota updating... 
deivce 2 mcu ota success! 


## 查询设备0 1 2的mcu版本，前后对比可知，mcu版本从4X04升为4X05
$ mx-smi select -f mcu_version -i 0 1 2
mcu_version
4X05
4X05
4X05
```

