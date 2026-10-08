# 阶段起点：Waveshare P4 7B 借鉴上游 v2.11.2

## 目的

在不破坏当前可工作的 Waveshare 7B 门铃/TTS 版本的前提下，建立一个干净的上游分支，先恢复 7B 硬件 Profile，再按验证结果选择性移植上游改进。

本阶段是离线开发阶段：不连接 COM3、不刷写设备、不回滚、不擦除 Flash、不读取设备 Flash，也不发送门铃或视频测试流。

## 固定起点

| 项目 | 值 |
| --- | --- |
| 上游仓库 | `jtenniswood/espcontrol` |
| 上游版本 | `v2.11.2` |
| 上游提交 | `86d61c8c` |
| 开发分支 | `feature/waveshare-p4-7b-support` |
| Profile 提交 | `ec89138e` |
| 独立工作树 | `D:\\Temp\\espcontrol-waveshare-p4-7b-profile-20261009` |
| 当前产品工作树 | `D:\\nas\\espcontrol-waveshare7b` |
| 当前产品基线提交 | `f366dc4` |

当前产品工作树存在未提交的音频、门铃和 7B 修改。本阶段不在该工作树上继续编辑，也不撤销其中任何修改。

## Waveshare 7B 硬件事实

- 主控：ESP32-P4，360 MHz，32 MB Flash。
- 屏幕：1024x600 MIPI DSI，Waveshare 7B 面板。
- 触摸：GT911，I2C SDA/SCL 为 GPIO7/GPIO8，复位 GPIO22，中断 GPIO21。
- 背光：GPIO32，低电平有效，当前产品配置限制最大功率为 0.8。
- C6 网络协处理器：ESP32-C6 over SDIO。
- SDIO：CMD/CLK/D0-D3 = GPIO19/18/14/15/16/17，复位 GPIO54，高电平有效，20 MHz。
- 调试/串口：USB-Serial/JTAG，当前实机门禁使用 COM3。

上游 `guition-esp32-p4-jc1060p470` 可作为同尺寸 P4 UI 和 C6/SDIO 的参考，但其 Profile 标注为 16 MB Flash，不能直接当作 Waveshare 7B 固件使用。

## 本阶段范围

### 第一条移植切片：硬件 Profile

只处理以下内容：

- 新建 `waveshare-esp32-p4-7b` 设备入口和设备硬件定义。
- 将上游 v2.11.2 的公共配置接入 7B Profile。
- 保留 32 MB Flash、Waveshare MIPI DSI、GT911、GPIO32 背光和 C6/SDIO 参数。
- 建立普通 Wi-Fi 配置和后续 Ethernet 配置的编译入口。
- 完成离线配置检查、构建和产物哈希记录。

### 后续可选切片：图片资源管理

硬件 Profile 单独通过后，才移植上游已经验证过的图片相关改进：

- 忽略 `entity_picture=None`，避免无效地址进入缓存。
- 图片刷新/停止播放时释放资源。
- 图片刷新失败后的恢复路径。
- artwork 地址恢复和显示状态保护。

这些改动必须单独提交和单独验证，不能与硬件 Profile 混在一个提交中。

## 明确不在本阶段移植

- 当前 7B 的 `esp_audio_stack`、AEC、双麦、语音唤醒和 STT/TTS 固件逻辑。
- 门铃快照、TTS 自动化和 Home Assistant 实体命名。
- H264/MJPEG 播放器、视频传输和 Frigate 联动。
- 用户 Wi-Fi、摄像头地址、HA 地址、密码或任何私密配置。
- 直接覆盖上游音频代码。上游近期音频/语音取舍与当前 7B TTS 目标不同。

## 验收门禁

1. 新工作树保持干净，阶段提交可独立回退。
2. 不出现 Wi-Fi 凭据、摄像头地址和 HA 私密信息。
3. ESPHome 配置检查通过。
4. 普通 Wi-Fi 配置完成离线构建并记录真实退出码；Ethernet 入口仅完成结构接入，尚未单独宣称构建通过。
5. `git diff --check`、相关单元测试和构建产物 SHA256 复核通过。
6. 在用户明确要求前，不进行任何 COM3 或设备操作。

## 本次离线构建结果

- ESPHome：`2026.9.1`。
- ESP-IDF：`5.5.5`。
- 构建并行度：`3`。
- 配置检查：通过；随后使用同一份本地未跟踪凭据完成完整编译，凭据未进入 Git，编译后已删除本地 `secrets.yaml`。
- 完整编译：成功，Ninja `1756/1756`，退出码为 `0`。
- 内存摘要：DIRAM 使用 `371184 / 576464`（`64.39%`）；Flash 使用 `5241756 / 15663104`（`33.5%`）。

### 刷写参数

| 项目 | 值 |
| --- | --- |
| Flash | `32MB` |
| 模式/频率 | `dio` / `40m` |
| bootloader | `0x2000` |
| partition table | `0x8000` |
| app | `0x20000` |

这里的 partition table 地址是上游 Profile 使用的 `0x8000`。旧 Waveshare 产品构建曾使用 `0x9000`，两者不能混用；本阶段产物不能直接作为旧产品回滚镜像或硬件刷写方案。

### 产物 SHA256

| 文件 | 字节数 | SHA256 |
| --- | ---: | --- |
| `bootloader.bin` | `23632` | `5ED143B3369221548BD03E327B6EDDA4ADB35D5F35642A7B7317814BA88601E3` |
| `partition-table.bin` | `3072` | `DFDA1F18324B6ADC69F0014CBB96CD2DEB28CFC604E32A16844F2A0A0ABC5FA2` |
| `espcontrol-waveshare-7b.bin` | `5242160` | `8E91EE2BB223ED9BAAE3ACF9EEEB90E1305AECEFF528B11DC1ECD3DFEADE2281` |
| `firmware.factory.bin` | `5373232` | `3DE07FB4197BC1AFAEAF94BF99A1DF651CAF6F502B2F9BC146F7DCA887A3A2EB` |

本次仅完成离线配置检查和构建；未连接 COM3、未刷写、未回滚、未擦除 Flash、未读取设备 Flash、未发送门铃或视频测试流。

## 第一条上游改进：旋转补偿轴

上游 `4b9aa4bc` 修复了 7 英寸屏在旋转选择变化后，封面图播放按钮、图标和文字沿错误轴缩放的问题。该提交原本只覆盖 Guition 7 英寸 Profile；本分支将相同的最小逻辑加入 Waveshare 7B 的 `apply_screen_rotation`，并把 Waveshare 7B 纳入同一回归测试。

- 实现提交：`754265fa`。
- 生产改动：在 `cover_art_apply_responsive_layout` 之前设置 `90/270` 为纵向补偿轴，其余旋转为横向补偿轴。
- 未改动：音频、TTS、门铃、H264/MJPEG、Frigate、Wi-Fi 凭据和 COM3 操作。
- 静态红绿测：通过；修改前确认缺少轴更新，修改后确认轴更新位于布局调用之前。
- Python 测试语法检查：通过。
- ESPHome 配置检查：通过。
- ESPHome 增量编译：通过，分区大小检查通过。
- 宿主 C++ 回归测试：未执行；当前 Windows 环境没有 `c++`、`g++` 或 `clang++` 可执行文件，不能把该测试标记为通过。

### 本条改进后的产物

| 文件 | 字节数 | SHA256 |
| --- | ---: | --- |
| `bootloader.bin` | `23632` | `5ED143B3369221548BD03E327B6EDDA4ADB35D5F35642A7B7317814BA88601E3` |
| `partition-table.bin` | `3072` | `DFDA1F18324B6ADC69F0014CBB96CD2DEB28CFC604E32A16844F2A0A0ABC5FA2` |
| `espcontrol-waveshare-7b.bin` | `5242176` | `20F82AD96F61B0301475B1C3849FEE1A397AA725EFA21EB89F58D43A98375300` |
| `firmware.factory.bin` | `5373248` | `3F757CBEBC07DF6F69628F3ACFD7FC5639B315116BFAC2A2B3EFCFA58651F932` |

## 当前结论

阶段起点采用“上游 v2.11.2 + 独立 Waveshare 7B Profile + 产品功能后续选择性移植”的路线。当前可工作的门铃/TTS版本仍是实机基线；本分支首先证明硬件 Profile 能在上游代码上离线构建，再进入图片资源改进。
