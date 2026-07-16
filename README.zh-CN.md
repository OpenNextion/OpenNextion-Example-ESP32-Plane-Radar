# OpenNextion ESP32 Plane Radar

[![English](https://img.shields.io/badge/lang-English-blue)](./README.md)
[![中文](https://img.shields.io/badge/lang-中文-red)](./README.zh-CN.md)

<p align="center">
  <img src="docs/images/opennextion-esp32-plane-radar-demo.jpg" alt="OpenNextion ESP32 Plane Radar 在 OpenNextion 显示屏上的运行效果" width="820">
</p>

OpenNextion ESP32 Plane Radar 是一个面向 OpenNextion ESP32-S3 矩形屏的桌面飞机雷达固件，可以直接刷写使用。它基于优秀的 [ESP32 Plane Radar](https://github.com/MatixYo/ESP32-Plane-Radar) 项目，并增加了 OpenNextion 开发板支持、竖屏矩形 UI 布局、Wi-Fi 配网流程，以及已支持 OpenNextion 显示屏的 release 固件。

你可以把它作为一个小型 ESP32 ADS-B 雷达显示器放在桌面上，通过 Wi-Fi 配网页面设置自己的位置，然后在紧凑的雷达 UI 上查看附近飞机。

## 支持的显示屏

公开的 `v0.1.0` release 支持两款 OpenNextion 竖屏显示屏：

| 显示屏型号 | 尺寸 | 分辨率 | 方向 | PlatformIO env | 状态 |
| --- | --- | --- | --- | --- | --- |
| [ONX3248G035][onx3248g035] | 3.5 英寸 | 320 x 480 | 竖屏 | `onx3248g035` | 已验证 |
| [ONX2432G028][onx2432g028] | 2.8 英寸 | 240 x 320 | 竖屏 | `onx2432g028` | 已验证 |

请不要把为某一款显示屏构建的固件刷写到另一款显示屏上。

## 背景

我发现了一个很优秀的飞机雷达项目：[ESP32 Plane Radar](https://github.com/MatixYo/ESP32-Plane-Radar)。原项目主要面向圆形屏幕设计，但我手上只有几块矩形的 OpenNextion ESP32 显示屏，所以决定把 Plane Radar 移植到这些矩形屏幕上。

OpenNextion 开发板使用 ESP32-S3 作为主控制器，并为 DIY 项目提供了实用的硬件资源，包括原理图、板级文件，以及用于外壳或支架设计的机械参考文件。

这个分支保留原始雷达行为，并增加已支持 OpenNextion 开发板所需的硬件适配。

## 3D 打印外壳

我也为已支持的 OpenNextion 显示屏尺寸设计了简单的 3D 打印外壳，并计划发布到 MakerWorld。需要的人可以免费下载和打印。

每个外壳都是为对应 OpenNextion 显示屏准备的简易桌面外壳。撕下显示屏边缘的胶带，将显示屏压入打印外壳中，并用边缘胶固定到位。

MakerWorld 项目链接：

- 3.5 英寸 ONX3248G035 外壳：待补充
- 2.8 英寸 ONX2432G028 外壳：待补充

## 显示内容

雷达 UI 保留原项目的圆形飞机雷达作为主要视觉元素，并适配到 OpenNextion 竖向矩形显示屏上。

- 围绕你配置的经纬度显示附近飞机
- 支持 5 km、10 km、15 km 和 25 km 量程预设
- 在数据可用时显示航班号、机型、高度和航向指示
- 超出当前雷达范围的飞机会以方位点显示在雷达外圈附近
- 底部紧凑信息面板显示量程、飞机数量、更新时间和最近飞机信息
- 可选显示来自嵌入 OurAirports 数据的主要机场跑道叠加层

2.8 英寸和 3.5 英寸布局分别进行了调整，使雷达在两种尺寸上都保持为主要 UI 元素。

## 基础使用

### 首次 Wi-Fi 配网

首次启动会创建名为 `PlaneRadar-Setup` 的 AP 并打开配网页面。

1. 使用手机或电脑连接到 `PlaneRadar-Setup`。
2. 打开 `http://plane-radar.local` 或 `http://192.168.4.1`。
3. 输入你的家庭 Wi-Fi 信息。
4. 输入用于雷达中心点的纬度和经度。
5. 保存并等待设备重启进入雷达界面。

设备加入局域网后，同一个配置页面仍可通过 `http://plane-radar.local` 或串口日志中显示的设备 IP 地址访问。你可以在这里更新 Wi-Fi、雷达位置、单位和跑道叠加显示设置。

### BOOT 按键

Plane Radar 使用 BOOT 按键执行快捷操作。

| 操作 | 效果 |
| --- | --- |
| 短按 | 切换量程预设：5 -> 10 -> 15 -> 25 km，并保存到 flash |
| 长按 3 秒 | 清除 Wi-Fi、位置和单位设置；重启进入配网页面 |
| 配网 / 启动时长按 | 强制清除凭据并进入配网页面 |

在 OpenNextion ESP32-S3 目标上，BOOT 按键使用 GPIO `0`。

## 固件下载和刷写

请从 GitHub Releases 页面下载固件。`v0.1.0` release 为每款已支持显示屏提供一个合并后的完整刷写固件：

| 显示屏型号 | 方向 | 固件文件 | 刷写地址 |
| --- | --- | --- | --- |
| [ONX3248G035][onx3248g035] | 竖屏 | `opennextion-esp32-plane-radar-v0.1.0-onx3248g035.bin` | `0x0` |
| [ONX2432G028][onx2432g028] | 竖屏 | `opennextion-esp32-plane-radar-v0.1.0-onx2432g028.bin` | `0x0` |

将匹配的合并固件刷写到地址 `0x0`：

```sh
python -m esptool --chip esp32s3 -p /dev/cu.wchusbserial1110 -b 921600 write_flash \
  0x0 ./opennextion-esp32-plane-radar-v0.1.0-onx3248g035.bin
```

请根据你的开发板替换串口和固件文件名。

对于这个 release，推荐使用完整固件刷写。除非 OTA 流程单独验证，否则不提供 OTA 固件下载。

## 当前移植工作

这个版本基于 ESP32 Plane Radar，重点为两款已验证 OpenNextion 显示屏提供板级支持。

这个分支的主要改动：

- 为 ONX3248G035 和 ONX2432G028 提供独立 PlatformIO board 目标
- 为已支持显示屏提供板级 LovyanGFX 面板配置
- 为 3.5 英寸和 2.8 英寸屏幕提供竖屏矩形雷达布局
- 为矩形屏空间增加底部紧凑信息面板
- 支持 BOOT 按键切换量程和重置配置
- 生成可从地址 `0x0` 简单完整刷写的合并固件

OpenNextion 开发板带有触摸和 SD 卡硬件，但当前固件尚未使用它们。

## 当前验证状态

### 显示验证

<p align="center">
  <img src="docs/images/opennextion-esp32-plane-radar-validation.jpg" alt="OpenNextion ESP32 Plane Radar 在 OpenNextion 显示屏上的验证效果" width="720">
</p>

- ONX3248G035 已在真实硬件上验证
- ONX2432G028 已在真实硬件上验证
- Wi-Fi 配网流程已在 OpenNextion 目标上验证
- 雷达显示和 ADS-B 刷新已在硬件上进行视觉验证
- BOOT 短按切换量程和长按重置已支持
- 当前固件尚未使用触摸和 SD 卡

### 固件验证矩阵

图例：✅ 已验证 / ⚠️ 部分验证或依赖硬件环境 / ⏳ 未使用

| 开发板 | 构建 | 启动 | 显示 | Wi-Fi 配网 | 雷达 UI | 触摸 | SD 卡 | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ONX3248G035 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ⏳ 未使用 | ⏳ 未使用 | 3.5 英寸 ST7796U 显示屏 |
| ONX2432G028 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ✅ 已验证 | ⏳ 未使用 | ⏳ 未使用 | 2.8 英寸 ST7789 显示屏 |

## 文档

- [构建、刷写和合并固件](docs/BUILD_AND_FLASH.md)

## 路线图

后续计划：

- 在可行情况下，让这个 OpenNextion 分支持续跟进上游 ESP32 Plane Radar
- OpenNextion 可打印外壳准备好后补充链接
- 继续在已支持硬件上验证显示、Wi-Fi、ADS-B 和 UI 行为
- 评估未来是否使用板载触摸和 SD 卡硬件

## 致谢

本项目基于 ESP32 Plane Radar。感谢原作者和相关开源项目。

- ESP32 Plane Radar: https://github.com/MatixYo/ESP32-Plane-Radar
- OpenNextion open source projects: https://github.com/OpenNextion
- OpenNextion board documentation: https://nextion.tech/wiki/

## 许可证

本项目保留上游 ESP32 Plane Radar 的许可证条款。详情见 `LICENSE`。第三方库可能有各自的许可证声明。

## 免责声明

本项目不是官方上游 ESP32 Plane Radar 项目。

Plane Radar 显示来自公开 ADS-B 来源的飞机数据，并依赖网络可用性、DNS、Wi-Fi 质量、本地配置和第三方服务可用性。刷写和使用第三方固件存在风险。请在理解风险后使用。本项目不对设备损坏、数据丢失、网络连接问题、不准确或延迟的飞机数据、法律后果或其他任何使用后果负责。

[onx3248g035]: https://nextion.tech/wiki/onx3248g035/
[onx2432g028]: https://nextion.tech/wiki/onx2432g028/
