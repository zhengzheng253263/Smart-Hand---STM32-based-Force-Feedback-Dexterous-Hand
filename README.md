# Smart-Hand---STM32-based-Force-Feedback-Dexterous-Hand
一个低成本、可维护的灵巧手项目。通过薄膜压力传感器（FSR）实现抓握力闭环控制， 拇指固定对掌 + 其余手指随力度实时跟随，模拟人手的自然抓握动作。
## 核心特点 | Features

- **STM32主控**：基于 STM32F103C8T6 (LQFP-48)，运行频率 72MHz。
- **多路PWM控制**：使用 TIM2 的 CH1/CH3/CH4 通道，独立控制 3 个舵机。
- **力反馈闭环**：通过薄膜压力传感器 (FSR) 采集信号，经 ADC1 采样后，实现“力度越大，握紧越紧”的实时跟随。
- **定制PCB**：独立完成原理图与 PCB Layout，集成电源、舵机接口和传感器接口，已打样验证。
- **模块化3D打印结构**：支持快速拆装，方便维护和升级。

## 演示视频 | Demo

 https://www.bilibili.com/video/BV16CeQ6eEKQ/?share_source=copy_web&vd_source=082ebc51ae8051110e422990226c6a00
## 硬件架构 | Hardware

| 模块 | 型号 | 说明 |
| 主控 | STM32F103C8T6 | 核心控制，72MHz |
| 舵机 | MG90S × 2 | 拇指转动、大小拇指张握 |
| 舵机 | MG946 × 1 | 主抓握 |
| 传感器 | FSR 薄膜压力传感器 | 分压电路接入 ADC1_IN1 |
| 电源 | 6V 磷酸铁锂电池 | 独立供电，与主控共地 |
| 结构 | 3D打印 + 拉簧 + 销轴 | 模块化设计 |

> 如需查看PCB原理图和3D打印文件，请见 `Hardware/` 目录。

## 软件架构 | Software

- **开发环境**：STM32CubeIDE + STM32CubeMX (HAL库)
- **调试器**：ST-Link V2

**核心控制链路：**
`FSR -> 分压电路 -> PA1 -> ADC1 -> fsrValue -> 映射公式 -> pulse -> TIM2_CHx -> 舵机`

**核心代码逻辑：**
```c
// 拇指固定对掌，其余两指随力度实时跟随
if (fsrValue < 2500) {
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, 1500); // 拇指固定
    uint32_t pulse = 500 + ((uint32_t)(2500 - fsrValue) * 1900 / 2500);
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_3, pulse); // 主抓握
    __HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_4, pulse); // 大小拇指
}
## 踩坑记录 | Debug Log

| 问题现象 | 根本原因 | 解决方案 |
| 程序烧录后不运行 | 时钟树HCLK仅8MHz | CubeMX调整PLL x9，HCLK = 72MHz |
| HAL_Delay卡死 | SysTick未正确配置 | 改用软件循环延时临时规避，后修复时钟 |
| 多路PWM引脚冲突 | TIM2_CH2默认PA1与ADC冲突 | 改用TIM2_CH4 (PA3) |
| FSR读数不变 | 分压电路缺少下拉电阻 | 增加10kΩ下拉电阻，形成完整分压电路 |
| 舵机抖动无力 | 电池内阻大，供电不足 | 改用6V磷酸铁锂独立供电，与主控共地 |
| PCB网络标签错乱 | EDA软件网络标签优先级高于GND符号 | PCB手动修正网络，防止短路 |

## 未来计划 | Future Work

- [1] 增加 ESP32 视觉模块，通过 UART 与 STM32 通信，实现“识别-抓取”闭环。
- [2] 引入状态机逻辑，区分“未接触 / 接触未抓稳 / 抓稳”三种状态。
- [3] 设计八足运输机器人移动平台（爬坡、楼梯、第一对足取物）。

## 作者 | Author

- **GitHub**：[@郑正253263](https://github.com/郑正253263)
- **B站**：【宇宙逃离A计划的个人空间-哔哩哔哩】 https://b23.tv/yFxUruE

---
