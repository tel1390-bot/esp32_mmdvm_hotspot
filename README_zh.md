# ESP32 MMDVM 热点固件（ESP32-S3 N16R8 适配版）

基于 [javastraat/esp32_mmdvm_hotspot](https://github.com/javastraat/esp32_mmdvm_hotspot) 的适配分支，专门为 **ESP32-S3-DevKitC-1（WROOM N16R8：16MB Flash + 8MB OPI PSRAM）** 编译验证，开箱即用。

> English README: [README.md](README.md)

## 功能特性

- **DMR**（网络→射频发射方向）、**POCSAG** 寻呼、**DAPNET** 完整支持
- 响应式 Web 管理界面，手机/电脑浏览器即可配置
- MQTT 集成、WireGuard VPN、SD 卡数据库、OTA 固件升级
- 默认接入 BrandMeister 等主流 DMR 服务器

## 适配内容（相对原项目）

| 项目 | 修改 |
|---|---|
| 时区 | `NTP_GMT_OFFSET_SEC` 由 GMT+1 改为 **GMT+8**（`28800`，北京时间） |
| 版本号 | 固件版本字符串为 `20260402_ESP32` |
| 板卡配置 | 官方 `esp32:esp32:mmdvm` 板定义（ESP32-S3 / 16MB / OPI PSRAM / 240MHz），与 N16R8 完全匹配，引脚无需改动 |
| 分区方案 | `ethelite_16MB`（16MB：8MB APP + OTA 双分区 + SPIFFS） |

## 硬件要求

### 必备

| 硬件 | 说明 |
|---|---|
| **ESP32-S3-DevKitC-1** | WROOM N16R8 模组（16MB Flash + 8MB PSRAM），其它 16MB Flash 的 S3 开发板也可 |
| **MMDVM 射频调制解调器模块** | MMDVM_HS / JumboSPOT / ZUMspot 等，负责 DMR 射频收发，ESP32 无法直接发射射频 |

### 接线（MMDVM 模块 ↔ 开发板）

| 信号 | 开发板 GPIO |
|---|---|
| ESP32 TX → 模块 RX | GPIO **43** |
| ESP32 RX ← 模块 TX | GPIO **44** |
| GND | GND |
|（可选）BOOT/RESET | GPIO 4 / 13 |

## 固件下载

直接下载编译好的固件（无需自己编译）：

👉 **[GitHub Releases](https://github.com/tel1390-bot/esp32_mmdvm_hotspot/releases)**

| 文件 | 用途 |
|---|---|
| `ESP32-S3-N16R8-merged-16MB-full-image.bin` | **16MB 完整镜像**，推荐，`write_flash 0x0` 一键写入 |
| `ESP32-S3-N16R8-app-ota.bin` | 应用固件，OTA 升级用 |

## 自己编译

依赖：arduino-cli、esp32 核心（3.3.12+）、库（Adafruit GFX / SSD1306 / PubSubClient / ArduinoJson / Sqlite3Esp32）。

```bash
# 安装 esp32 核心
arduino-cli config add board_manager.additional_urls https://espressif.github.io/arduino-esp32/package_esp32_index.json
arduino-cli core install esp32:esp32

# 安装依赖库
arduino-cli lib install "Adafruit GFX Library" "Adafruit SSD1306" "PubSubClient" "ArduinoJson" "Sqlite3Esp32"

# 注册板卡（复制 boards 定义到核心目录）
#  Windows: 复制 boards/boards.txt 到 %LOCALAPPDATA%\Arduino15\packages\esp32\hardware\esp32\<版本>\boards.local.txt
#           复制 boards/*.csv 到上述目录 tools\partitions\

# 编译
arduino-cli compile --fqbn "esp32:esp32:mmdvm:PartitionScheme=ethelite_16MB" .
```

## 烧录方法

### 一键脚本（Windows）

`flash.bat`（自动检测 ESP32-S3 端口，或 `flash.bat COM5` 指定端口）：
1. 全片擦除
2. 写入 16MB 完整镜像

### esptool 手动

```bash
esptool --port COMx --baud 921600 erase_flash
esptool --port COMx --baud 921600 write_flash 0x0 ESP32-S3-N16R8-merged-16MB-full-image.bin
```

> 提示：开发板如无法识别，按住 **BOOT** 键再插 USB 强制进入下载模式。

## 首次使用

1. 上电后热点启动 **WiFi AP**：SSID `DMR-Hotspot`，密码 `password`
2. 连接该 WiFi，浏览器访问 **http://192.168.4.1**
3. 在 Web 界面配置：WiFi 联网、DMR 模式、服务器（如 BrandMeister）、DMR ID、呼号、时隙/组呼等
4. 保存重启后即可工作

## 许可与合规提醒

- 本仓库遵循原项目 **Amateur Radio Non-Commercial License**（业余无线电非商业许可），详见 [license.txt](license.txt)
- DMR 热点发射属业余无线电业务，请**持业余电台执照并按规定操作**
- 该固件 DMR 目前仅支持**网络→射频**发射方向，射频→网络接收仍在开发中（POCSAG/DAPNET 完整可用）
