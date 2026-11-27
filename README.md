# piconew

Pico SDK 工程生成器。通过交互式菜单或命令行参数创建可直接编译的 RP2040 / RP2350 工程骨架。

## 环境要求

| 依赖 | 版本 | 说明 |
|---|---|---|
| CMake | 3.13+ | SDK 最低要求 |
| arm-none-eabi-gcc | — | ARM 交叉编译工具链 |
| riscv32 裸机 GCC | — | 仅 `--arch riscv` 需要 |
| Pico SDK | 2.x | 1.x 不含 RP2350 支持 |
| bash | 4.0+ | 生成的脚本使用 `[[ ]]` 与数组 |
| picocom | — | 可选，`build.sh monitor` 使用 |

工具链与 SDK 的安装以官方仓库说明为准：<https://github.com/raspberrypi/pico-sdk>

Arch 系发行版安装工具链：

```bash
sudo pacman -S --needed arm-none-eabi-gcc arm-none-eabi-newlib cmake git python3
```

### 获取 SDK

```bash
git clone https://github.com/raspberrypi/pico-sdk.git --branch master ~/pico/pico-sdk
cd ~/pico/pico-sdk
git submodule update --init
```

`git submodule update --init` 为必需步骤，TinyUSB 等组件以子模块提供，未初始化时 USB 串口不可用。

也可用 AUR 包（路径为 `/usr/share/pico-sdk`）：

```bash
yay -S pico-sdk
```

### 环境变量

```bash
export PICO_SDK_PATH="$HOME/pico/pico-sdk"
```

写入 `~/.zshrc` 持久生效。验证：

```bash
ls "$PICO_SDK_PATH/pico_sdk_init.cmake"
```

### RISC-V 工具链（仅 `--arch riscv`）

SDK 按 `riscv32-pico-elf-gcc`、`riscv32-unknown-elf-gcc`、`riscv32-corev-elf-gcc`、`riscv-none-elf-gcc` 的顺序查找编译器，装任意一套并置于 `PATH` 或者设置`PICO_TOOLCHAIN_PATH`即可。官方预编译版本见 <https://github.com/raspberrypi/pico-sdk-tools/releases>，使用时设置：

```bash
export PICO_TOOLCHAIN_PATH=/path/to/riscv-toolchain-16/
```

## 安装

```bash
git clone https://github.com/WangYanbo51/piconew.git ~/code/piconew
mkdir -p ~/.local/bin
chmod +x ~/code/piconew/piconew
ln -sf ~/code/piconew/piconew ~/.local/bin/piconew
```

## 快速开始

```bash
piconew                 # 全部选项通过菜单选择
piconew blinky          # 指定工程名，其余菜单选择
piconew blinky --default   # 六项全部使用默认方案
```

六项全部指定则直接生成：

```bash
piconew blinky --board pico2 --arch arm --lang cpp --stdio usb --libs i2c,spi --starter blink
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
| `--arch` | `arm` \| `riscv` | 内核架构，缺省 `arm`，仅 RP2350 板支持 `riscv` |
| `--lang` | `c` \| `cpp` | 源文件语言 |
| `--stdio` | `usb` \| `uart` \| `both` \| `none` | 串口输出方式 |
| `--libs` | 逗号分隔 | 功能库，可留空 |
| `--starter` | `blink` \| `serial` \| `pio` \| `empty` | 起始代码模板 |
| `--default` | — | 六项全部使用默认方案，与上述选项互斥 |
| `--print-build-sh` | — | 只输出 `build.sh` 内容，用于升级已有工程 |
| `-h`, `--help` | — | 显示帮助 |

未指定的项逐一提问，全部指定则不提问。

## 交互式创建

第一步选择方案：

```
① 创建方案
  1) 上次的方案（Pico 2 / ARM / C++ / USB 串口 / 无库 / empty）
  2) 默认方案（Pico 2 / ARM / C++ / USB 串口 / 无库 / empty）
  3) 逐项选择
```

默认方案为 Pico 2 / ARM / C++ / USB 串口 / 无功能库 / 空模板，等价于 `--default`。`上次的方案` 无历史记录时不显示。RP2040 板（`pico`、`pico_w`）跳过架构提问。

## 选项参考

| 选项 | 取值说明 |
|---|---|
| 开发板 | `pico2`、`pico2_w` 为 RP2350；`pico`、`pico_w` 为 RP2040 |
| 架构 | `arm` 对应 `rp2350-arm-s`，`riscv` 对应 `rp2350-riscv`，RP2040 为 `rp2040` |
| 串口 | `usb`（USB CDC）、`uart`（UART0，GP0/GP1）、`both`、`none` |
| 功能库 | `i2c` `spi` `pwm` `adc` `uart` `multicore` `float` `pio`，多个以逗号分隔 |
| 起始代码 | `blink` 闪灯、`serial` 串口回显、`pio` 方波、`empty` 空骨架 |

`pio` 作为功能库只链接 `hardware_pio`，不生成 `.pio` 文件；作为起始代码才生成 `src/square_wave.pio`。两者相互独立。

UF2 芯片标识：RP2350 ARM-S 为 `0xe48bff59`，RP2350 RISC-V 为 `0xe48bff5a`，RP2040 为 `0xe48bff56`。`0xe48bff57` 是 `ABSOLUTE_FAMILY_ID`，用于 ABS 块，与芯片无关。

## 项目结构

```
~/pico/<工程名>/
├── CMakeLists.txt          已配置开发板、平台、功能库、串口方式
├── src/
│   ├── main.c | main.cpp   起始代码
│   └── square_wave.pio     仅 PIO 模板生成
├── build.sh                编译 / 烧录 / 复位 / 串口监视 / 工程信息 / 清理
├── .gitignore
└── README.md
```

## build.sh

| 命令 | 行为 |
|---|---|
| `./build.sh` | 编译 |
| `./build.sh flash` | 编译并烧录，板子未处于 BOOTSEL 时自动复位 |
| `./build.sh reset` | 仅复位至 BOOTSEL，不编译 |
| `./build.sh monitor` | 串口监视（别名 `mon`、`serial`），`Ctrl+A` 后按 `X` 退出 |
| `./build.sh info` | 打印当前工程信息 |
| `./build.sh clean` | 删除 `build` 目录 |

`info` 输出示例：

```
工程：pwm_breath
路径：/home/user/pico/pwm_breath
开发板：pico2
平台：rp2350-arm-s（ARM）
串口：USB
链接库：pico_stdlib hardware_pwm
源文件：main.cpp
SDK：/usr/share/pico-sdk（2.3.1）
工具链：arm-none-eabi-gcc 16.2.0
产物：build/pwm_breath.uf2（60K，RP2350 ARM-S，2026-11-26 21:19）
串口设备：未检测到
```

字段均从 `CMakeLists.txt` 与磁盘文件现场读取。产物芯片标识与平台不符时给出告警，用于发现 `build/` 中是旧配置编出的固件。`info`、`monitor`、`reset`、`clean` 不要求 SDK 就位。

烧录顺序：已挂载 `RP2350` / `RPI-RP2` 则直接写入；否则依次用 `picotool reboot`、1200 波特率、pyserial 复位后写入；均失败则提示手动进入 BOOTSEL。设备存在但未挂载时调用 `udisksctl` 自动挂载。

## 说明

**板型与平台设置在 `include(pico_sdk_import.cmake)` 之前。** SDK 在引入时即加载平台配置，若在其后设置，`PICO_BOARD` 已被默认值 `pico` 占用，会静默生成 RP2040 固件。改动后必须删除 `build` 目录重新配置。

**`src/` 下源文件由 `file(GLOB ...)` 收集**，新增文件无需修改 `CMakeLists.txt`；新增功能库需在 `target_link_libraries` 中追加库名。

**配置持久化于 `~/.config/piconew/last.conf`**，六个键各占一行，仅在创建成功时写入，删除后等同于清除历史。旧版五键配置缺少 `ARCH` 时按 `arm` 处理。

| 环境变量 | 默认值 |
|---|---|
| `PICO_ROOT` | `~/pico` |
| `PICO_SDK_PATH` | `~/pico/pico-sdk` |
| `XDG_CONFIG_HOME` | `~/.config` |

