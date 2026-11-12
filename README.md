# STM32 PWM SG90 舵机控制

基于 STM32F103C8T6 和 STM32 HAL 的舵机控制练习。通过 TIM3 输出 PWM 控制 SG90 的目标位置，使用两个按键切换档位，并用 RGB LED 区分增加、减少和端点回绕。

## 功能与操作

上电后从最低档开始。按键按下后更新目标位置，程序等待松手后继续读取按键。

| 操作 | 目标位置变化 | LED 提示 |
| --- | --- | --- |
| 按 KEY1 | 增加一档（标称 +45°） | 绿灯 |
| 按 KEY2 | 减少一档（标称 −45°） | 红灯 |
| 在最高档按 KEY1 | 回到最低档 | 蓝灯 |
| 在最低档按 KEY2 | 回到最高档 | 蓝灯 |

程序写入 `500、1000、1500、2000、2500 µs` 五档目标脉宽。按 0°–180° 线性换算，这五档可理解为标称 `0°、45°、90°、135°、180°`。程序不读取实际角度，实际行程以所用舵机为准。更新 PWM 后，相应 LED 亮约 200 ms，提示按键指令已处理。

## 引脚与接线

| 信号 | STM32 引脚 | 配置 |
| --- | --- | --- |
| 舵机 PWM 信号 | PB0 | TIM3_CH3，复用推挽输出 |
| KEY1 | PA0 | 下拉输入，按下为高电平 |
| KEY2 | PC13 | 下拉输入，按下为高电平 |
| RGB LED（红 / 绿 / 蓝） | PA1 / PA2 / PA3 | GPIO 输出，低电平点亮 |

舵机的电源端使用符合模块规格的电源，地线与 STM32 的 GND 相连。PB0 仅接舵机的控制信号端。接线时以实物的引脚标识为准。

## PWM 参数

当前 [TIM3 配置](Core/Src/tim.c) 使用 72 MHz 的定时器时钟，预分频值为 `72 - 1`，每个计数对应 1 µs。自动重装值 ARR 为 `65535`，因此当前 PWM 周期是 **65.536 ms**（约 **15.26 Hz**）。[主程序](Core/Src/main.c) 通过更新 CCR3 调整高电平脉宽。

如果所用舵机要求 **20 ms（50 Hz）** 的控制周期，在保持 1 µs 计数的前提下，可在 CubeMX 中将 TIM3 的 Counter Period 设置为 `20000 - 1`，再生成代码。脉宽端点和实际角度仍需按舵机规格与实物表现校准。

## 构建与下载

工程使用 STM32CubeMX、ARM GNU 工具链（`arm-none-eabi-gcc`）、CMake 3.22 或以上版本以及 Ninja。可在 VS Code 打开仓库根目录并选择 **Debug** 预设，或在工具已可从命令行调用时执行：

```sh
cmake --preset Debug
cmake --build --preset Debug
```

生成的程序为 `build/Debug/TIM_PWM_Re.elf`。配置好 CMSIS-DAP 与 OpenOCD 后即可通过 SWD 下载，也可以使用其他支持 STM32F103 的 SWD 工具。若重新生成代码，保持 CubeMX 的 **SYS → Debug → Serial Wire**，以便继续使用 SWD 调试。

## 代码入口

- [Core/Src/main.c](Core/Src/main.c)：按键轮询、档位步进与回绕、PWM 更新和 LED 提示。
- [Core/Src/tim.c](Core/Src/tim.c)：TIM3 和 PB0 的 PWM 初始化。
- [Core/Src/gpio.c](Core/Src/gpio.c)：按键和 RGB LED 的 GPIO 初始化。
- [TIM_PWM_Re.ioc](TIM_PWM_Re.ioc)：CubeMX 外设配置。
