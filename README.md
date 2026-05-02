# ESP32-S3 NT35510 800x480 示例

本仓库是一个面向 ESP32-S3 + NT35510 屏幕的示例集合，包含 Arduino IDE 版本、ESP-IDF 版本、CH340 串口驱动、屏幕规格书和一个预编译亮屏测试固件。

## 项目内容

| 路径 | 内容 |
| --- | --- |
| `Code/esp32-s3-nt35510-arduino-ide/` | Arduino IDE 示例和所需库压缩包 |
| `Code/esp32-s3-nt35510-idf-example/` | ESP-IDF 示例工程 |
| `Code/ESP32S3-800X480-亮屏测试固件.bin` | 预编译亮屏测试固件 |
| `CH340驱动/CH341SER.EXE` | Windows CH340/CH341 USB 串口驱动 |
| `屏幕规格书/` | BY-T40VB19R3C 屏幕规格书和 NT35510 数据手册 |

## 硬件信息

从源码和规格书可确认的关键信息：

- 主控：ESP32-S3
- LCD 模组：BY-T40VB19R3C
- LCD 驱动 IC：NT35510
- 屏幕规格：3.97 英寸 TFT，480(RGB) x 800 dots
- 示例显示方向：800 x 480 横屏
- LCD 接口：示例使用 16-bit Intel 8080/i80 并口
- 色深：RGB565 / 16 bpp
- 触摸：规格书为 4 线电阻触摸，但当前示例未接入触摸功能
- SD 卡：SPI 模式

### LCD 引脚

| 功能 | ESP32-S3 GPIO |
| --- | --- |
| LCD D0-D15 | GPIO0-GPIO15 |
| LCD WR/PCLK | GPIO18 |
| LCD CS | GPIO17 |
| LCD DC | GPIO16 |
| LCD RST | GPIO20 |
| LCD BL | GPIO38，高电平点亮 |
| 其他控制 | GPIO19，示例中上电后拉高 |

### SD 卡引脚

| 功能 | ESP32-S3 GPIO |
| --- | --- |
| SCK/CLK | GPIO40 |
| MISO | GPIO39 |
| MOSI | GPIO41 |
| CS/SS | GPIO42 |

## Arduino IDE 示例

目录：`Code/esp32-s3-nt35510-arduino-ide/`

当前项目截图中使用的 Arduino 开发板包为 `esp32 by Espressif Systems 3.0.7`。`libraries.zip` 内包含：

- `lvgl-8.4.0/`
- `lv_conf.h`

使用步骤：

1. 安装 Arduino IDE。
2. 在 Boards Manager 中安装 `esp32 by Espressif Systems`，建议使用仓库截图中的 `3.0.7`。
3. 解压 `Code/esp32-s3-nt35510-arduino-ide/libraries.zip`。
4. 将 `lvgl-8.4.0` 和 `lv_conf.h` 放到 Arduino 的 `libraries` 目录下，参考同目录的 `库文件.png`。
5. 打开并上传示例：
   - `lvgl-example/lvgl-example.ino`：初始化 NT35510 屏幕并运行 `lv_demo_widgets()`
   - `sd-example/sd-example.ino`：测试 SD 卡挂载、目录、读写、重命名、删除和性能

`lvgl-example` 使用 2 MHz i80 PCLK、16-bit 总线、800 x 480 分辨率，并调用：

```cpp
esp_lcd_panel_swap_xy(panel_handle, true);
esp_lcd_panel_mirror(panel_handle, true, false);
```

## ESP-IDF 示例

目录：`Code/esp32-s3-nt35510-idf-example/`

子目录 README 标明建议使用 ESP-IDF `v5.3.1`；预编译固件信息中也显示 `ESP-IDF: v5.3.1-dirty`。如使用其他 IDF 版本构建失败，优先切换到 `v5.3.1`。

工程依赖：

- 目标芯片：`esp32s3`
- LVGL：`~8.4.0`
- PSRAM：启用，默认 80 MHz
- CPU：240 MHz
- LVGL POSIX 文件系统：启用，盘符 `S`
- LVGL 图片支持：启用 PNG 和 SJPG；当前 UI 扫描 `.jpg` / `.sjpg`

构建前请确认配置为 NT35510 + 16-bit 总线。当前源码实际使用 D0-D15，因此配置应为：

- `Example Configuration -> i80 LCD controller model -> NT35510`
- `Example Configuration -> NT35510 Data Width -> 16`
- `CONFIG_EXAMPLE_LCD_I80_BUS_WIDTH=16`

可用命令：

```sh
cd Code/esp32-s3-nt35510-idf-example
idf.py set-target esp32s3
idf.py menuconfig
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

如果想直接套用仓库中已有的 NT35510/16-bit 配置，可先参考或恢复 `sdkconfig.old` 中的相关配置；当前 `sdkconfig.old` 里记录的是：

```text
CONFIG_EXAMPLE_LCD_I80_CONTROLLER_NT35510=y
CONFIG_EXAMPLE_LCD_NT35510_DATA_WIDTH_16=y
CONFIG_EXAMPLE_LCD_I80_BUS_WIDTH=16
```

### ESP-IDF 示例运行效果

`main/i80_controller_example_main.c` 会：

1. 拉高 GPIO19。
2. 挂载 SD 卡到 `/sdcard`。
3. 初始化 16-bit i80 LCD 总线和 NT35510。
4. 初始化 LVGL。
5. 运行 `example_lvgl_demo_ui()`。

`main/lvgl_demo_ui.c` 会扫描 SD 卡根目录中的第一个 `.jpg` 或 `.sjpg` 文件，并通过 LVGL 居中显示。如果没有 SD 卡、没有图片或解码失败，屏幕会显示对应错误信息。

使用图片显示功能时，把 `.jpg` 或 `.sjpg` 文件放到 SD 卡根目录即可。

## 预编译亮屏固件

文件：`Code/ESP32S3-800X480-亮屏测试固件.bin`

通过 `esptool image-info` 识别到的信息：

- 芯片：ESP32-S3
- 镜像大小：578720 bytes
- Flash mode：DIO
- Flash freq：80 MHz
- Flash size：2 MB
- ESP-IDF：v5.3.1-dirty
- 编译时间：2024-09-09 23:24:03

二次开发建议优先使用 Arduino 或 ESP-IDF 示例重新编译；该二进制主要用于快速验证屏幕能否点亮。

## 注意事项

- Windows 下如无法识别串口，可安装 `CH340驱动/CH341SER.EXE`。
- 仓库中存在若干 `Zone.Identifier` 附带文件，是从 Windows 拷贝产生的元数据文件，不参与构建。
- ESP-IDF 示例源码按 NT35510 和 16-bit i80 总线编写；若 menuconfig 里选成 ST7789 或 8-bit，总线引脚和源码会不匹配。
- 当前示例没有实现触摸读取，屏幕触摸部分需要自行补充驱动和引脚配置。
