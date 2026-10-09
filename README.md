# EspControl Waveshare 7B

中文文档 | [English](README.en.md)

这是一个基于 [EspControl](https://github.com/jtenniswood/espcontrol) 的 Waveshare ESP32-P4-WIFI6-Touch-LCD-7B 适配与案例项目。

项目目标不是复制上游固件，而是保留 EspControl 的网页配置、Home Assistant 集成和卡片系统，同时为 Waveshare 7B 建立独立硬件 Profile，并逐步加入经过验证的家庭场景功能。

## 当前状态

已完成：

- 建立 Waveshare 7B 独立硬件 Profile。
- 适配 ESP32-P4、32MB Flash、1024x600 MIPI-DSI 屏幕。
- 适配 GT911 触摸、GPIO32 背光和 ESP32-C6 SDIO Wi-Fi/BLE 协处理器。
- 基于上游 v2.11.2 完成离线配置检查和 ESP-IDF 编译。
- 移植 7 英寸屏旋转时的封面图按钮、图标和文字补偿修复。
- 保留独立分支、阶段文档和可复现构建记录。

正在规划：

- 升级到上游 v2.12.0，并逐项保留 Waveshare 7B 差异。
- 中文固件界面和字体适配。
- Home Assistant 摄像头、门铃快照和 TTS 案例整理。
- ESPHome Device Builder 的构建管理方式。
- 评估 7 英寸家用型与 4.3 英寸部署型 Profile 的差异，并规划可复用的共用能力。
- 4.3 英寸 Profile 尚未实施或实机验证；如计划商业发布，需先确认上游许可证及授权要求。

尚未作为本分支稳定功能合并：

- 当前产品线中的语音唤醒、STT 和双向对讲。
- H264/MJPEG 播放器和视频传输逻辑。
- 真实设备刷写和长期实机验收。

## 硬件 Profile

| 项目 | 参数 |
| --- | --- |
| 主控 | ESP32-P4，360MHz |
| Flash | 32MB |
| 显示屏 | 1024x600 MIPI-DSI |
| 触摸 | GT911，SDA GPIO7，SCL GPIO8，RESET GPIO22，INT GPIO21 |
| 背光 | GPIO32，低电平有效，最大功率限制 80% |
| Wi-Fi/BLE | ESP32-C6 over SDIO |
| SDIO | CMD/CLK/D0-D3 = GPIO19/18/14/15/16/17 |
| C6 复位 | GPIO54，高电平有效 |
| SDIO 频率 | 20MHz |

## 目录

```text
devices/waveshare-esp32-p4-7b/   Waveshare 7B ESPHome Profile
dev-docs/stages/                  阶段记录和构建证据
components/                      EspControl 和硬件组件
common/                          共享设备配置
product/v2/translations/         固件翻译源文件
```

## 本地编译

当前开发入口：

```text
devices/waveshare-esp32-p4-7b/dev.yaml
```

需要本地创建未跟踪的 `secrets.yaml`：

```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
```

示例编译命令：

```powershell
esphome config devices/waveshare-esp32-p4-7b/dev.yaml
$env:NINJAFLAGS = "-j3"
esphome compile devices/waveshare-esp32-p4-7b/dev.yaml
```

本项目阶段默认只做离线构建。没有经过硬件门禁前，不要直接执行 `esphome run`。

## ESPHome Device Builder

ESPHome Device Builder 可以作为编译和设备管理入口，但 Git 仓库仍是源码主线。
建议流程：

```text
GitHub 仓库 -> Device Builder -> 编译/下载/OTA -> Home Assistant
```

不要把真实 Wi-Fi、RTSP、Home Assistant 或 SSH 凭据提交到仓库。

## 分支和上游

- `main`：个人项目稳定线。
- `feature/waveshare-p4-7b-support`：当前 Waveshare 7B 适配线。
- `feature/upstream-v2.12`：后续上游升级线。
- `feature/chinese-firmware`：后续中文固件翻译线。

本地远程仓库：

```text
origin   https://github.com/ericho1978/espcontrol-waveshare-7b.git
upstream https://github.com/jtenniswood/espcontrol.git
```

同步上游时，应在独立分支中逐项合并和验证，不要直接覆盖 Waveshare 7B Profile 或当前产品功能。

## 汉化计划

固件固定文字来自：

```text
product/v2/translations/strings.en.txt
```

中文翻译将使用独立的 `strings.zh-cn.txt`，再通过项目脚本生成：

```powershell
python scripts/build.py i18n
```

不要直接编辑生成文件 `components/espcontrol/i18n_generated.h`。中文字体还需要补充可显示中文字符的字体资源，并单独进行 Flash、内存和屏幕显示验证。

## 许可证和上游声明

本项目包含来自上游 EspControl 的代码和资源，必须同时遵守仓库中的 `LICENSE` 与 `NOTICE`。上游项目使用 PolyForm Noncommercial License 1.0.0；商业部署或销售前需要取得相应许可。

上游项目：

- https://github.com/jtenniswood/espcontrol
- https://jtenniswood.github.io/espcontrol/

## 安全边界

仓库不应包含：

- `secrets.yaml`
- 真实 Wi-Fi 密码
- 摄像头 RTSP 用户名和密码
- Home Assistant、SSH 或 Docker 凭据
- 带真实配置的设备备份
- 未脱敏的运行日志和截图

这是一个可复现的硬件适配与产品化案例项目，不代表所有功能已经完成实机验收。
