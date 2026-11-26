# piconew

Pico SDK 工程生成器。通过交互式菜单或命令行参数创建可直接编译的 RP2040 / RP2350 项目骨架。

## 功能

- 交互式创建工程，五组选项：开发板、源文件语言、串口输出方式、功能库、起始代码
- 记忆上一次成功创建时使用的配置，支持一键复用
- 生成的工程开箱即可编译，无需修改构建脚本
- 配套 `build.sh` 提供编译、烧录、复位、串口监视、清理
- `.pio` 文件由 SDK 在编译期自动调用 pioasm 转为头文件
- 新增源文件无需修改 `CMakeLists.txt`

## 环境要求

| 依赖 | 版本 | 说明 |
|---|---|---|
| Linux | — | 已在 Manjaro（Arch 系）验证 |
| bash | 4.0+ | 生成的脚本使用 `[[ ]]` 与数组 |
| CMake | 3.13+ | Pico SDK 最低要求 |
| arm-none-eabi-gcc | — | 交叉编译工具链 |
| Pico SDK | 2.x | 通过 `PICO_SDK_PATH` 指定 |
| picocom | — | 可选，用于 `build.sh monitor` |
| udisksctl | — | 可选，用于自动挂载 BOOTSEL 设备 |

Pico SDK 2.x 为 RP2350 支持的必要条件，1.x 版本无法编译 Pico 2 固件。

## 安装

```bash
git clone git@github.com:WangYanbo51/piconew.git ~/code/piconew
mkdir -p ~/.local/bin
ln -sf ~/code/piconew/piconew ~/.local/bin/piconew
chmod +x ~/code/piconew/piconew
```

未配置 SSH 密钥时，将仓库地址替换为 `https://github.com/WangYanbo51/piconew.git`。

使用符号链接而非复制文件，可使 `git pull` 的更新立即生效。

确认 `~/.local/bin` 已包含在 `PATH` 中：

```bash
echo $PATH | tr ':' '\n' | grep local
```

若未包含，在 shell 配置文件中追加：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## 快速开始

交互式创建，全部选项通过菜单选择：

```bash
piconew
```

指定工程名，其余选项通过菜单选择：

```bash
piconew blinky
```

跳过全部提问，直接生成：

```bash
piconew blinky --board pico2 --lang cpp --stdio usb --libs i2c,spi --starter blink --yes
```

进入工程目录编译：

```bash
cd ~/pico/blinky
./build.sh
```

## 命令行参数

| 参数 | 取值 | 说明 |
|---|---|---|
| `--board` | `pico2` \| `pico2_w` \| `pico` \| `pico_w` | 目标开发板 |
| `--lang` | `c` \| `cpp` | 源文件语言 |
| `--stdio` | `usb` \| `uart` \| `both` \| `none` | 串口输出方式 |
| `--libs` | 逗号分隔的库名列表 | 功能库，可留空 |
| `--starter` | `blink` \| `serial` \| `pio` \| `empty` | 起始代码模板 |
| `--yes` | — | 不对缺失项提问，使用默认值填充 |
| `--print-build-sh` | — | 仅输出 `build.sh` 内容，用于升级已有工程 |
| `-h`, `--help` | — | 显示帮助 |

未通过命令行指定的选项，在交互模式下会依次提问；配合 `--yes` 时使用默认值。

## 创建方案

交互式创建的第一步为选择方案：

```
① 创建方案
  1) 上次的方案（Pico 2 / C++ / USB 串口 / 无库 / empty）
  2) 默认方案（Pico 2 / C++ / USB 串口 / 无库 / empty）
  3) 逐项选择
```

| 选项 | 行为 |
|---|---|
| 上次的方案 | 复用上一次成功创建时的五项配置。无历史记录时不显示此项 |
| 默认方案 | Pico 2 / C++ / USB 串口 / 无功能库 / 空模板 |
| 逐项选择 | 依次询问五组选项 |

选择「上次的方案」或「默认方案」后不再逐项提问，直接生成工程。

默认方案同时作为 `--yes` 模式下缺失参数的填充值。

## 选项参考

### 开发板

| 值 | 型号 |
|---|---|
| `pico2` | Pico 2（RP2350） |
| `pico2_w` | Pico 2 W（RP2350，带无线模块） |
| `pico` | Pico（RP2040） |
| `pico_w` | Pico W（RP2040，带无线模块） |

### 串口输出方式

| 值 | 说明 |
|---|---|
| `usb` | 通过 USB CDC 输出，无需额外硬件 |
| `uart` | 通过 UART0（GP0/GP1）输出，需 USB-TTL 转换模块 |
| `both` | 同时启用 USB 与 UART |
| `none` | 不启用，节省 Flash 占用 |

### 功能库

多个库以逗号分隔，例如 `--libs i2c,spi`。

| 值 | 链接的库 | 用途 |
|---|---|---|
| `i2c` | `hardware_i2c` | I2C 总线 |
| `spi` | `hardware_spi` | SPI 总线 |
| `pwm` | `hardware_pwm` | PWM 输出 |
| `adc` | `hardware_adc` | 模拟输入 |
| `uart` | `hardware_uart` | 额外 UART 通道 |
| `multicore` | `pico_multicore` | 双核编程 |
| `float` | `pico_float` | 硬件浮点 |
| `pio` | `hardware_pio` | PIO 可编程 IO |

### 起始代码

| 值 | 内容 |
|---|---|
| `blink` | 板载 LED 闪烁，串口输出计数 |
| `serial` | 串口回显，用于收发测试 |
| `pio` | PIO 方波发生器，含 `.pio` 示例 |
| `empty` | 仅包含最小的程序骨架 |

## 生成的项目结构

```
~/pico/<工程名>/
├── CMakeLists.txt          已配置开发板、功能库、串口方式
├── src/
│   ├── main.c | main.cpp   起始代码
│   └── square_wave.pio     仅 PIO 模板生成
├── build.sh                编译 / 烧录 / 复位 / 串口监视 / 清理
├── .gitignore
└── README.md               记录本项目所选配置
```

## build.sh 用法

| 命令 | 行为 |
|---|---|
| `./build.sh` | 编译 |
| `./build.sh flash` | 编译并烧录。板子未处于 BOOTSEL 时自动复位 |
| `./build.sh reset` | 仅将板子复位至 BOOTSEL，不编译 |
| `./build.sh monitor` | 打开串口监视（别名 `mon`、`serial`） |
| `./build.sh clean` | 删除 `build` 目录 |

`monitor` 通过 USB VID `2e8a` 定位串口设备，避免选中其他 `/dev/ttyACM*`。退出 picocom 为依次按下 `Ctrl+A`、`X`。

烧录流程按以下顺序判断板子状态：

1. 存在 `RP2350` / `RPI-RP2` 挂载点：直接写入 `.uf2`
2. 不存在挂载点但存在串口设备：依次尝试 `picotool reboot`、1200 波特率、pyserial 三种方式复位，等待 U 盘出现后写入
3. 两者均不存在：提示手动进入 BOOTSEL

BOOTSEL 设备存在但未挂载时，调用 `udisksctl` 自动挂载；该步骤失败时输出手动挂载命令。

## 构建配置说明

### 源文件收集

`CMakeLists.txt` 使用 `file(GLOB ...)` 收集 `src/` 下全部 `.c` 与 `.cpp` 文件。新增源文件放入该目录后重新编译即可生效，无需修改构建脚本。

新增功能库时，在 `target_link_libraries` 中追加库名。

### PIO 头文件

选择 PIO 相关选项后，`CMakeLists.txt` 包含：

```cmake
pico_generate_pio_header(${PROJECT_NAME} ${CMAKE_CURRENT_LIST_DIR}/src/square_wave.pio)
```

编译期由 SDK 调用 pioasm 生成 `build/square_wave.pio.h`。修改 `.pio` 文件后重新编译即生效，生成的头文件不应手工修改。

### 开发板设置顺序

生成的 `CMakeLists.txt` 将 `PICO_BOARD` 赋值置于 `include(pico_sdk_import.cmake)` 之前：

```cmake
if(NOT PICO_BOARD)
    set(PICO_BOARD pico2)
endif()
include($ENV{PICO_SDK_PATH}/external/pico_sdk_import.cmake)
```

该顺序为必需。`pico_sdk_import.cmake` 在引入时即加载平台配置，若在其后设置 `PICO_BOARD`，变量已被默认值 `pico` 占用，条件不成立，最终生成 RP2040 固件。

调整该顺序后必须删除 `build` 目录重新配置，`CMakeCache.txt` 中缓存的 `PICO_BOARD` 不会自动更新。

## 配置持久化

上一次使用的配置保存在 `~/.config/piconew/last.conf`，纯文本，五个键各占一行：

```
BOARD=pico2
LANG=cpp
STDIO=usb
LIBS=
STARTER=empty
```

该文件可直接编辑；删除后等同于清除历史记录。仅在工程创建成功时写入。

## 环境变量

| 变量 | 默认值 | 说明 |
|---|---|---|
| `PICO_ROOT` | `~/pico` | 工程存放根目录 |
| `PICO_SDK_PATH` | `~/pico/pico-sdk` | Pico SDK 路径 |
| `XDG_CONFIG_HOME` | `~/.config` | 配置文件父目录 |

若系统通过包管理器安装了 `pico-sdk`，`PICO_SDK_PATH` 可能指向 `/usr/share/pico-sdk`，生成器遵循该变量的实际取值。同时存在多份 SDK 时，建议仅保留一份，避免修改源码后未生效。

## 构建验证

以下配置已实机编译通过（Pico SDK 2.3.1，`PICO_BOARD=pico2`）：

| 配置 | 产物 |
|---|---|
| `--lang c --stdio usb --libs i2c,spi --starter blink` | 62 KB，含 `hardware_i2c`、`hardware_spi` |
| `--lang cpp --stdio both --starter pio` | 67 KB，`.pio` 自动汇编成功 |

生成固件的芯片标识为 RP2350（family `0xe48bff57`，载入地址 `0x10ffff00`）。
