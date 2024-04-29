# Asus ROG Strix Z690-A Gaming Wifi D4 + i7-14700KF + RX 6600

## 硬件信息
- CPU： Intel i7-14700KF
- 主板： Asus ROG Strix Z690-A Gaming Wifi D4
- 内存： 2x32GB DDR4 3200MHz
- 显卡： AMD Radeon RX 6600
- 有线网卡： Intel I225-V
- 无线网卡： Intel AX201
- 声卡： Realtek USB2.0 Audio

## 工作情况
<img width="280" alt="image" src="https://github.com/vimt/Asus-ROG-Strix-Z690-A-Gaming-Wifi-D4-i7-14700KF-RX6600/assets/10686631/863a2c21-ca68-47fe-9e84-3046217594d0">

- MacOS Sonoma 14.3.1
- Opencore: 0.9.7

### What works
- 无线网络
- 声音
- USB
- HiDPI
- 睡眠

### What doesn't work
- AirDrop：不行
- AirPlay：有的设备可以，有的不行 (Apple TV)

## 配置过程

### BIOS settings
参考 [Asus Z690 ProArt Creator WiFi (Thunderbolt 4) + i7-12700K + AMD RX 6800 XT](https://www.tonymacx86.com/threads/asus-z690-proart-creator-wifi-thunderbolt-4-i7-12700k-amd-rx-6800-xt.318311/) ，有详细的 BIOS 设置截屏

关闭
- Fast Boot
- Secure Boot
- USB Configuration - Serial Port Configuration
- Onboard Devices Configuration - Serial Port Configuration
- F1 错误等待
- CSM
- CFG Lock：没找到，通过 CfgLock.efi 关闭

开启
- VT-x
- VT-d（可以开着，通过 `DisableIoMapper` 控制）
- Above 4G decoding
- XHCI Hand-off
- XMP
- Re-Size BAR Support（配合 OC `ResizeAppleGpuBars = 0`）


## 安装
我仔细阅读了 [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)，严格按照教程进行了配置，并参考了大量其他人的配置。  

配置好后，一直卡在 `[EB|#LOG:EXITBS:START]` 。参考了 [TroubleShooting](https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/kernel-issues.html#stuck-on-eb-log-exitbs-start) 各种方法都试过了，都没有解决。

在我几乎要放弃的时候，十分感谢 `MaLd0n` 发布的 [Hackintosh EFI Folders with OpenCore MOD](https://www.olarila.com/topic/25111-hackintosh-efi-folders-with-opencore-mod/) 帖子

我下载了他为 `Desktop AlderLake and RaptorLake` 准备的 EFI，没有任何调整，直接安装成功！感谢 `MaLd0n` ！ 
(TODO: 为什么 `MaLd0n` 就可以运行？）

## 安装完的优化

### 无线网卡
感谢 [zxystd](https://github.com/zxystd) 的工作，让我们可以直接使用 Intel 的无线网卡。

添加 `AirportItlwm.kext`

### 启动时多次重启问题
启动系统时，`OpenCore` 刷完日志应该进入 MacOS 登陆界面。但实际经常刷完代码后系统需要再重启一次才能进入登陆界面。

解决方案：去掉 `boot-args` 的 `-v` 参数（不刷日志，显示苹果进度条）


### CPU性能问题
MacOS 对 Intel 大小核的支持不好。搜索一番，总结通常需要使用两个 kext：
- [CPUFriend](https://github.com/acidanthera/CPUFriend)：调整 CPU 睿频。
- [CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild)：将小核视为超线程，让系统优先使用大核。

但实际我都没有使用：

**为什么 没有使用 CPUFriend：**

在 `iMacPro1,1` 下，用 `CPU-S` 看睿频是 700-5500。不需要额外设置睿频

**为什么没有使用 CpuTopologyRebuild：**

安装了 `CpuTopologyRebuild` 之后，通过
```shell
sysctl -n hw.logicalcpu
sysctl -n hw.physicalcpu
```
看到实际变成了 12核心28线程，而 14700KF 实际架构是 8P12E，应该是 8核心28线程。感觉有异常实际未使用。

（如果同学有建议请提 issue）

### USB定制
使用 `USBToolBox` 在 Windows 上定制 USB。

但实际定制错了，后面版有4个USB口用不了。暂时也不影响使用，先不管了，后续有需要再改。

（其实也不好改了，现在已经 15个端口用满了，再改的话只能考虑禁用一些口的 USB2.0 了）

定制完成之后，删除 `USBInjectAll.kext` 和 `XHCI-unsupported.kext`

### 蓝牙
根据 [IntelBluetoothFirmware的文档](https://openintelwireless.github.io/IntelBluetoothFirmware/Installation.html)

添加以下 kext 后正常工作
- IntelBluetoothFirmware.kext
- BlueToolFixup.kext
- IntelBTPatcher.kext

### 睡眠问题
跟着 [opencore文档](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html#preparations) 调整
```shell
sudo pmset autopoweroff 0  # 禁止休眠
sudo pmset powernap 0  # 禁止定期唤醒
sudo pmset standby 0  # 禁止睡眠和休眠之间切换
sudo pmset proximitywake 0  # 禁止iphone/watch唤醒
sudo pmset tcpkeepalive 0  # 禁止TcpKeepAlive 防止每2小时唤醒一次

# 查看 wake sleep 日志
pmset -g log | grep -e "Sleep.*due to" -e "Wake.*due to"  
# 查看电源管理设置
pmset -g
# 查看
pmset -g assertions
```

（🌟：在调整过程中，ChatGPT 极大帮助我解释一些不懂的命令/日志

添加 `SSDT-GPRW` 禁用USB唤醒，只允许电源键唤醒

做了以上调整之后，睡眠唤醒工作正常。

但发现有其他问题：每隔1小时，电脑会自动唤醒一次，但显示器不亮。

通过 `pmset -g log | grep -e "Sleep.*due to" -e "Wake.*due to"` 命令，发现这个功能叫 DarkWake

具体日志：
```log
2024-03-18 09:04:05 +0800 Wake Requests       	[*process=powerd request=CSPNEvaluation deltaSecs=3612 wakeAt=2024-03-18 10:04:17] [process=powerd request=UserWake deltaSecs=24536 wakeAt=2024-03-18 15:53:01 info="com.apple.alarm.user-invisible-com.apple.calaccessd.travelEngine.periodicRefreshTimer,386"]
...
2024-03-18 10:04:24 +0800 DarkWake            	DarkWake from Normal Sleep [CDN] : due to RTC/Maintenance Using AC (Charge:0%) 45 secs
...
2024-03-18 10:05:09 +0800 Sleep               	Entering Sleep state due to 'Maintenance Sleep':TCPKeepAlive=disabled Using AC (Charge:0%)
```

添加 `Disable RTC wake scheduling` 这个 Patch。 

### 有线网卡
`MaLd0n` 的EFI 带了 `AppleIGC.kext` 直接能用，并未特殊调整。

另外 [一个post](https://www.tonymacx86.com/threads/gigabyte-z490-vision-d-thunderbolt-3-i5-10400-amd-rx-580.298642/page-1067#post-2377077) 说 `Sonoma` 不需要 `AppleIGC.kext`，开启 `VT-d` 功能可以直接驱动。尝试了确实可以。

但我还是选择 `AppleIGC.kext`（`AppleVTD` 功能开启之后据说会有其他异常）

### 声音
这个主板找不到具体的声卡型号，就是 `Realtek USB2.0 Audio`。

`MaLd0n` 的 EFI 未特殊调整可以直接使用。没有再调整。

### SMBios
EFI 默认是 `iMacPro1,1`，纠结要不要换 `MacPro7,1` 可以得到更好的性能。看了一些帖子感觉差距不大，未调整。
