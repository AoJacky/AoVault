---
title: "一文理解单片机BootLoader的前世今生"
date: 2026-07-28
tags: [知识库, 嵌入式, Bootloader, IAP, ISP, OTA, STM32]
source: "微信公众号 汽车电子嵌入式 / ADAS与ECU之吾见"
---

# 一文理解单片机BootLoader的前世今生

> 从烧录方式演进到Bootloader设计实例，再到OTA和分散烧录，全面讲解BL的原理与实现。

## 1. 烧录方式的更新迭代

### 1.1 古老的烧录方式
51单片机时代，需专用烧录器+高压写入ROM，成本高、操作繁琐。

### 1.2 ISP与ICP
- **ISP**（In System Programming）：单片机需驻留程序与上位机通信，需要最小系统可运行
- **ICP**（In Circuit Programming）：MCU作为可读写存储电路，不需要预置程序、不需要可运行状态
- AT89S51 + 并口下载线降低了入门门槛

### 1.3 串口ISP
- STC单片机：串口ISP最成功案例，USB-TTL串口直接烧录
- STM32全系列也支持串口ISP

### 各种USB ISP下载器
串口ISP速度慢，大固件需专用USB ISP下载器（AVR、C8051F、MSP430各有配套）

**ISP共同弊端**：
- 依赖专门上位机/下载器，不能统型
- 下载器价格高
- 需要额外操作（STC需重新上电、STM32需设BOOT引脚）

**Bootloader的优势**：统一通信方式（串口）、友好界面（命令行）、一键烧录、固件版本管理

## 2. Bootloader基本形态

BL的4个核心功能：
1. 通过某种途径获取固件数据
2. 将固件数据写入ROM的APP区
3. 跳转到APP区运行
4. 提供必要的人机交互界面

### 2.2 两个设计实例

**1. 带Shell命令行的串口BL**
1. 串口终端输入`program`命令
2. BL等待接收固件数据
3. 传输协议传送固件
4. BL写入APP区
5. 引导APP运行

**2. 插SD卡即烧录的BL**
1. 固件拷贝到SD卡
2. 插入卡槽
3. BL检测SD卡，搜索BIN文件
4. 读取写入APP区
5. 引导APP运行

> Linux的Uboot就是强大的BL；电脑BIOS也是广义的BL

### 2.3 BL实现的要点

#### 要点1：芯片体系架构要支持
- APP必须有自己的中断向量表
- 单片机需支持中断向量表重定向
- **传统51**：中断向量表只能在ROM开头，不支持BL
- **STC51**：增加BOOTROM区，改进架构支持BL
- **AVR**：通过熔丝位控制复位入口和BOOT区
- **STM32**：NVIC提供中断向量表偏移量配置，程序可放ROM任意位置

#### 要点2：ROM要支持IAP
- IAP = In Application Programming，程序运行时可对自身ROM擦写
- STM32固件库已实现ROM擦写接口

#### 要点3：APP程序的配套修改
以STM32为例，ROM 128KB，BL 16KB：
- APP起始地址 = `0x08004000`
- MDK：修改IROM1起始地址
- GCC：修改link.ld的FLASH起始地址
- NVIC配置：`#define VECT_TAB_OFFSET 0x4000`

#### 要点4：BL中的跳转代码
```c
typedef void (*iapfun)(void);
iapfun jump2app;

void MSR_MSP(u32 addr) {
    __ASM volatile("MSR MSP, r0");  // 设置主栈指针
    __ASM volatile("BX r14");
}

void load_app(u32 appxaddr) {
    if((*(vu32*)appxaddr & 0x2FFE0000) == 0x20000000) {  // 检查栈顶合法
        jump2app = (iapfun)*(vu32*)(appxaddr + 4);  // 复位地址
        MSR_MSP(*(vu32*)appxaddr);                   // 设置SP
        jump2app();                                    // 跳转
    }
}
```

## 3. 把Bootloader玩出花

### 3.1 串口传输固件的实现

三个关键问题：
1. 串口通信协议如何实现？
2. 为什么先暂存再写入？→ 传输可能出错，需整体校验后再写
3. 如何校验？→ 上位机补齐固件+追加校验码（CheckSum/CRC 16/32位）

**ROM划分策略**：
- 外扩ROM暂存固件（推荐）
- 片上ROM三等分：BL / APP / 暂存（如128KB → 16K/56K/56K）
- 成本敏感方案：直接边传边写（STC/STM32串口ISP也是这样）

> STM32F103C8T6隐藏ROM：与RBT6同晶元，后64KB可用但不保证质量

### 3.2 隔空烧录OTA
空调外机工况监测IoT项目 → 串口BL+蓝牙 → 手机蓝牙串口助手即可OTA烧录

### 3.3 BL的分散烧录
复杂系统（主MCU+CPLD+协处理器）：
1. 所有固件拼接成大固件，预烧到spiFlash
2. 主MCU烧好BL，SMT焊接
3. 首次上电，BL读取大固件→分离→分别烧录各部件

## 4. 不走寻常路的BL

### 4.1 Bootpatcher
APP必须在0x08000000时：
- BL放在APP后面
- APP运行时跳转到BL → BL烧录新固件到APP区 → 重启
- 风险：APP区烧录失败则变砖

### 4.2 APP反烧BL
逆向思维：APP程序接收固件→暂存校验→烧录到BL区
- 与Bootpatcher同理，有风险但一般可行

## 原文链接

https://mp.weixin.qq.com/s/4NJhxwnA660MXkO7Y4xNtA
