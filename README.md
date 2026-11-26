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

## 准备构建环境

piconew 生成的工程依赖交叉编译工具链与 Pico SDK 完成构建，首次使用前需完成以下配置。

### 安装工具链

Arch 系发行版：

```bash
sudo pacman -S --needed arm-none-eabi-gcc arm-none-eabi-newlib cmake git python3
```

`arm-none-eabi-newlib` 提供标准 C 库实现，缺失时链接阶段会报 `undefined reference`。

### 获取 Pico SDK

从源码获取（推荐，便于固定或切换版本）：

```bash
git clone https://github.com/raspberrypi/pico-sdk.git --branch master ~/pico/pico-sdk
cd ~/pico/pico-sdk
git submodule update --init
```

`git submodule update --init` 为必需步骤。TinyUSB、BTstack、mbedtls、lwIP 等组件以子模块形式提供，未初始化时 USB 串口功能不可用，配置阶段输出以下警告：

```
TinyUSB submodule has not been initialized; USB support will be unavailable
```

也可使用 Arch 系 AUR 提供的软件包：

```bash
yay -S pico-sdk
```

该方式安装路径为 `/usr/share/pico-sdk`，需相应调整下一节的 `PICO_SDK_PATH`。该包不在官方仓库中，`pacman -S pico-sdk` 会提示找不到目标。

SDK 版本必须为 2.x。1.x 版本不包含 RP2350 支持，无法编译 Pico 2 固件。

### 配置环境变量

```bash
export PICO_SDK_PATH="$HOME/pico/pico-sdk"
```

写入 shell 配置文件以持久生效（bash 用户改为 `~/.bashrc`）：

```bash
echo 'export PICO_SDK_PATH="$HOME/pico/pico-sdk"' >> ~/.zshrc
source ~/.zshrc
```

验证配置：

```bash
ls "$PICO_SDK_PATH/pico_sdk_init.cmake"
```

路径正确时该命令列出文件；若报 `No such file or directory`，说明变量为空或路径有误。

## 安装 piconew

```bash
git clone git@github.com:WangYanbo51/piconew.git ~/code/piconew
mkdir -p ~/.local/bin
ln -sf ~/code/piconew/piconew ~/.local/bin/piconew
chmod +x ~/code/piconew/piconew
```

未配置 SSH 密钥时，将仓库地址替换为 `https://github.com/WangYanbo51/piconew.git`。

## 快速开始

交互式创建，全部选项通过菜单选择：

```bash
piconew
```

指定工程名，其余选项通过菜单选择：

```bash
piconew blinky
```

五项全部指定，直接生成：

```bash
piconew blinky --board pico2 --lang cpp --stdio usb --libs i2c,spi --starter blink
```

五项全部采用默认方案：

```bash
piconew blinky --default
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
| `--default` | — | 五项全部使用默认方案，不与上述选项参数同时使用 |
| `--print-build-sh` | — | 仅输出 `build.sh` 内容，用于升级已有工程 |
| `-h`, `--help` | — | 显示帮助 |

参数与提问行为的关系：

| 指定情况 | 行为 |
|---|---|
| 未指定任何选项 | 交互式选择创建方案 |
| 指定部分选项 | 指定的项直接采用，其余项逐一提问 |
| 指定全部五项 | 不提问，直接生成 |
| 仅指定 `--default` | 不提问，五项全部采用默认方案 |

`--default` 与选项参数互斥。同时出现时命令报错退出，因为「全部使用默认方案」与「逐项指定取值」是两种互斥的意图。

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

命令行下的 `--default` 等价于选择「默认方案」，同样跳过全部提问。

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
| `pio` | `hardware_pio` | PIO 可编程 IO，仅链接库，不生成 `.pio` 文件 |

### 起始代码

| 值 | 内容 |
|---|---|
| `blink` | 板载 LED 闪烁，串口输出计数 |
| `serial` | 串口回显，用于收发测试 |
| `pio` | PIO 方波发生器，含 `.pio` 示例，同时链接 `hardware_pio` |
| `empty` | 仅包含最小的程序骨架 |

功能库选 `pio` 与起始代码选 `pio` 相互独立：前者只链接 `hardware_pio`，供程序自行调用 PIO API，不产生任何 `.pio` 文件；后者额外生成 `src/square_wave.pio` 与 `pico_generate_pio_header` 指令。起始代码选 `empty` 而仍需 PIO 时，只添加功能库即可，不会出现多余的示例文件。

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

起始代码选择 `pio` 时，`CMakeLists.txt` 包含：

```cmake
pico_generate_pio_header(${PROJECT_NAME} ${CMAKE_CURRENT_LIST_DIR}/src/square_wave.pio)
```

编译期由 SDK 调用 pioasm 生成 `build/square_wave.pio.h`。修改 `.pio` 文件后重新编译即生效，生成的头文件不应手工修改。

仅在功能库中勾选 `pio` 而不使用该起始代码时，不生成上述指令，需自行编写 `.pio` 文件并添加对应行。

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
