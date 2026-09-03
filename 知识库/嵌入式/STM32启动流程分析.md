---
title: "最通俗易懂的STM32完整启动流程分析"
date: 2026-07-28
tags: [知识库, 嵌入式, STM32, 启动流程, 中断向量表]
source: "微信公众号 无际单片机编程"
---

# 最通俗易懂的STM32完整启动流程分析

> 从上电复位到进入main函数，三步拆解STM32启动全流程。

## 摘要

本文详细分析STM32从上电复位到进入main函数的完整启动流程。包括上电复位后CPU从0x00000000开始执行，中断向量表的加载，堆栈指针设置，SystemInit时钟配置，以及__main完成C运行时初始化后跳转到用户main函数的全过程。通过形象比喻帮助理解每个步骤的作用。

## 三大步骤总览

1. **上电复位**：CPU从0x00000000开始跑代码
2. **系统初始化**：设置堆栈、准备数据、配置时钟
3. **跳转main**：进入用户程序

## 一、复位和启动文件

### 1. 复位触发方式
- **上电复位**：刚插电源
- **外部复位**：RESET引脚
- **软件复位**：程序触发

不管哪种，CPU都从0x00000000开始，即Reset_Handler入口。

### 2. 三种启动模式（BOOT0/BOOT1引脚）

| 启动模式 | 地址 | 用途 |
|---------|------|------|
| Flash启动 | 0x08000000 | 最常见，正常运行 |
| 系统内存启动 | 0x1FFF0000 | Bootloader/固件更新 |
| SRAM启动 | 0x20000000 | 调试用 |

Flash启动时，0x00000000映射到0x08000000。

### 3. 启动文件的作用
- 定义中断向量表
- 设置堆栈
- 初始化系统（调用SystemInit）
- 跳转到C环境（__main → main）

## 二、上电第一步：中断向量表

### __Vectors 中断向量表

位于Flash起始地址0x08000000，存储关键地址：

```
__Vectors
  DCD  __initial_sp    ; 栈顶地址（第1个字）
  DCD  Reset_Handler   ; 复位处理程序（第2个字）
  DCD  NMI_Handler     ; 不可屏蔽中断
  DCD  HardFault_Handler ; 硬件错误
  ...
```

DCD = Define Constant Data，存储32位地址数据，不是可执行指令。

### 执行顺序

1. CPU从0x08000000读第1个字 → **__initial_sp** → 加载到SP寄存器
2. CPU从0x08000004读第2个字 → **Reset_Handler** → 跳转执行

## 三、__initial_sp 堆栈定义

```asm
Stack_Size EQU 0x00000800    ; 堆栈大小 = 2KB

AREA    STACK, NOINIT, READWRITE, ALIGN=3
Stack_Mem  SPACE  Stack_Size ; 分配2KB空间
__initial_sp                 ; 栈顶标号（高地址）
```

- **NOINIT**：不初始化（留空）
- **READWRITE**：可读可写
- **ALIGN=3**：8字节对齐
- **Stack_Mem**：栈底（低地址），SP不会低于此地址
- **__initial_sp**：栈顶（高地址），SP初始值，压栈时向下移动

## 四、Reset_Handler 启动流程

```asm
Reset_Handler PROC
  EXPORT  Reset_Handler  [WEAK]
  IMPORT  SystemInit
  IMPORT  __main
  LDR     R0, =SystemInit
  BLX     R0              ; 调用SystemInit（配置时钟72MHz等）
  LDR     R0, =__main
  BX      R0              ; 跳转到__main（C运行时入口）
  ENDP
```

1. **SystemInit**：ST库函数，初始化时钟、外设
2. **__main**：C运行时库入口
   - 复制.data段到SRAM（初始化全局变量）
   - 清零.bss段（未初始化全局变量）
   - 跳转到用户main函数

## 形象比喻

| 步骤 | 比喻 |
|------|------|
| 上电/复位 | 闹钟响了，你醒了 |
| 设置__initial_sp | 整理床铺，准备放东西 |
| 跳转Reset_Handler | 开始起床流程 |
| 调用SystemInit | 洗漱、穿衣服（准备硬件） |
| 跳转__main | 吃早餐（准备软件环境） |
| 调用main | 出门上班（运行你的任务） |

## 原文链接

## 个人想法

理解启动流程对于Bootloader开发和调试至关重要。很多初学者遇到"程序跑飞"、"进入HardFault"等问题，往往是因为不理解启动流程。建议结合启动文件（startup_stm32f103.s）实际阅读一遍，印象会更深刻。

## 原文链接

https://mp.weixin.qq.com/s/yRaUjmGrXkeL5-GWJDrECA
