# piconew — Pico SDK 工程生成器

一条命令生成带选项的工程骨架，生成即可编译。

## 安装

```bash
git clone git@github.com:WangYanbo51/piconew.git ~/code/piconew
mkdir -p ~/.local/bin
ln -sf ~/code/piconew/piconew ~/.local/bin/piconew
chmod +x ~/code/piconew/piconew
```

没配 SSH key 的话，把地址换成 `https://github.com/WangYanbo51/piconew.git`。

用软链接而不是 `cp`，这样 `git pull` 更新后立刻生效，不用再复制一次。

确认 `~/.local/bin` 在 PATH 里：

```bash
echo $PATH | tr ':' '\n' | grep local
```

没有的话在 `~/.zshrc` 里加一行：

```bash
export PATH="$HOME/.local/bin:$PATH"
```

## 用法

最简单的方式，全程菜单选择：

```bash
piconew
```

指定名字，其余菜单选：

```bash
piconew blinky
```

一步到位，不提问：

```bash
piconew blinky --board pico2 --lang cpp --stdio usb --libs i2c,spi --starter blink --yes
```

## 创建方案：上次 / 默认 / 逐项

交互式创建时，第一步先问方案，不用每次都重选：

```
① 创建方案
  1) 上次的方案（Pico 2 / C++ / USB 串口 / 无库 / empty）
  2) 默认方案（Pico 2 / C++ / USB 串口 / 无库 / empty）
  3) 逐项选择
```

- **上次的方案** —— 复用上一次成功创建时的五项设置。第一次用还没有记录，这一项不显示
- **默认方案** —— Pico 2 / C++ / USB 串口 / 不选功能库 / 空模板
- **逐项选择** —— 依次问开发板、语言、串口、功能库、起始代码

记录存在 `~/.config/piconew/last.conf`，纯文本，五项各一行，想改直接编辑，删掉就等于清空记录：

```
BOARD=pico2
LANG=cpp
STDIO=usb
LIBS=
STARTER=empty
```

`--yes` 时的缺省值就是默认方案，所以 `piconew foo --yes` 等价于 Pico 2 + C++ + USB + 无库 + 空模板。

## 五个选项

**① 开发板**

| 值 | 说明 |
|---|---|
| `pico2` | Pico 2（RP2350） |
| `pico2_w` | Pico 2 W（带 WiFi） |
| `pico` | Pico（RP2040） |
| `pico_w` | Pico W（RP2040 + WiFi） |

**② 语言** — `c` 或 `cpp`

**③ 串口输出**

| 值 | 说明 |
|---|---|
| `usb` | 插数据线就能 printf，最常用 |
| `uart` | 走 GP0/GP1，需要 USB-TTL 模块 |
| `both` | 两者都开 |
| `none` | 不开，省空间 |

**④ 功能库**（逗号分隔，可多选）

| 值 | 加的库 | 用途 |
|---|---|---|
| `i2c` | `hardware_i2c` | I2C 传感器 |
| `spi` | `hardware_spi` | SPI 屏幕 / SD 卡 |
| `pwm` | `hardware_pwm` | 调光 / 电机 |
| `adc` | `hardware_adc` | 读模拟电压 |
| `uart` | `hardware_uart` | 额外串口 |
| `multicore` | `pico_multicore` | 双核分工 |
| `float` | `pico_float` | 硬件浮点 |
| `pio` | `hardware_pio` | PIO 可编程 IO |

**⑤ 起始代码**

| 值 | 内容 |
|---|---|
| `blink` | 闪灯 + 串口打印计数 |
| `serial` | 串口回显，测收发 |
| `pio` | PIO 方波发生器，含 `.pio` 示例 |
| `empty` | 空模板，只剩骨架 |

## 生成的目录结构

```
~/pico/你的工程名/
├── CMakeLists.txt          配置好板子/库/串口，一般不用改
├── src/
│   ├── main.c              起始代码
│   └── square_wave.pio     仅 PIO 模板才有
├── build.sh                编译 / 烧录 / 清理
├── .gitignore
└── README.md               记录了本项目选了什么
```

## 用起来

```bash
cd ~/pico/你的工程名
./build.sh          # 编译
./build.sh flash    # 编译并烧录
./build.sh monitor  # 看串口输出
./build.sh clean    # 清理
```

`monitor` 会自动按 USB VID `2e8a` 找串口（不会误抓蓝牙之类的 `/dev/ttyACM*`），可以直接写 `./build.sh mon`。退出是先按 `Ctrl+A`、松手，再按 `X`。

## 加代码不用改 CMakeLists

`CMakeLists.txt` 里用的是 `file(GLOB ...)` 收集 `src/` 下所有 `.c` 和 `.cpp`。

想拆文件就直接往 `src/` 里丢，重新 `./build.sh` 就编进去。需要新增库时，往 `target_link_libraries` 后面加一行。

## PIO 不用手工转头文件

选了 PIO 相关选项后，`CMakeLists.txt` 里会有这行：

```cmake
pico_generate_pio_header(${PROJECT_NAME} ${CMAKE_CURRENT_LIST_DIR}/src/square_wave.pio)
```

编译时 SDK 自动调用 pioasm 生成 `build/square_wave.pio.h`。改了 `.pio` 重新 `./build.sh` 就行，不像 Arduino 那边要手工跑一遍。

## 环境变量

| 变量 | 默认 | 说明 |
|---|---|---|
| `PICO_ROOT` | `~/pico` | 工程存放根目录 |
| `PICO_SDK_PATH` | `~/pico/pico-sdk` | SDK 路径 |

如果系统里装了 `pico-sdk` 包，`PICO_SDK_PATH` 可能指向 `/usr/share/pico-sdk`，脚本会自动跟随这个变量。

## 已验证

以下组合实机编译通过（SDK 2.3.1，`-DPICO_BOARD=pico2`）：

- `--lang c --stdio usb --libs i2c,spi --starter blink` → 62 KB
- `--lang cpp --stdio both --starter pio` → 67 KB，`.pio` 自动汇编成功
