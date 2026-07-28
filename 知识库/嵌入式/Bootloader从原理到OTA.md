---
title: "一文读懂Bootloader：从原理到OTA应用"
date: 2026-07-28
tags: [知识库, 嵌入式, Bootloader, IAP, OTA, 单片机]
source: "微信公众号 无际单片机编程"
---

# 一文读懂Bootloader：从原理到OTA应用

> Bootloader是单片机上电后先于应用程序运行的特殊程序，核心使命是实现固件远程更新（IAP/OTA）。

## 一、Bootloader是什么

单片机上电/复位后，在主应用程序之前先运行的一小段特殊程序。类似电脑的BIOS/UEFI：
- 初始化必要硬件
- 决定启动主应用还是进入升级模式
- 通常放在Flash特定区域，带写保护

## 二、为什么需要Bootloader

直接跑应用可以，但产品卖出后：
- 发现bug→无法让用户用J-Link重新烧录
- 需要加功能→无法物理接触设备

**Bootloader核心优势**：
1. **固件更新能力**：无需物理接触和烧录工具
2. **灵活性**：先发布，后续通过软件升级迭代
3. **可靠性**：应用损坏时可通过Bootloader恢复

## 三、工作原理

### 1. 启动流程
- CPU从Flash起始地址（复位向量表）取指令
- Bootloader放在起始地址→CPU先执行Bootloader

### 2. 硬件初始化
只需初始化最基本的硬件：
- 时钟系统
- GPIO（检测升级模式引脚、LED指示）
- 通信接口（UART/SPI/I2C/CAN/USB）

### 3. 决策逻辑：跑应用还是升级？
- **检测GPIO引脚**：上电时长按按钮→进入升级模式
- **检查共享内存标志位**：APP写入升级请求标志
- **通信接口等待**：超时内收到升级指令→升级，否则启动APP
- **检查APP有效性**：CRC校验失败→强制进入升级模式

### 4. 跳转到APP（ARM Cortex-M）
```c
typedef void (*pFunction)(void);
void JumpToApplication(uint32_t AppAddr) {
    uint32_t appStack = *(__IO uint32_t*)AppAddr;
    uint32_t appEntry = *(__IO uint32_t*)(AppAddr + 4);
    
    __set_MSP(appStack);              // 设置主堆栈指针
    pFunction JumpToApp = (pFunction)appEntry;
    JumpToApp();                       // 跳转到Reset_Handler
}
```
**注意**：需设置NVIC的VTOR寄存器重定向中断向量表

### 5. 升级模式（IAP过程）
1. 初始化通信接口
2. 握手→发数据块→应答→结束
3. 按块接收固件数据
4. CRC校验每块数据
5. 擦除Flash扇区（写前必擦）
6. 写入Flash
7. 状态反馈
8. 整体校验→复位→启动新APP

## 四、Bootloader vs IAP vs OTA

| 概念 | 定义 |
|------|------|
| **Bootloader** | 程序本身，提供加载APP和固件更新的基础能力 |
| **IAP** | In-Application Programming，在设备运行状态下对自身Flash擦写的技术 |
| **OTA** | Over-The-Air，通过无线通信远程交付固件的方式 |

关系：**OTA → 依赖IAP能力 → 依赖Bootloader**

## 五、动手写Bootloader关键点

### 1. 内存规划
- Bootloader：0x8000000起
- APP：0x800C800起（举例）
- 通过修改链接脚本（Keil中设置IROM1起始地址）

### 2. 通信协议设计
定义命令如：
- 0x21/0x22：查询服务器是否有新固件
- 0x24/0x25：MCU申请获取OTA固件内容

### 3. APP注意事项
- 链接脚本修改APP起始地址
- 初始化时设置VTOR指向自己的向量表

## 六、从IAP走向OTA

OTA额外需要：
1. **无线通信栈**：在Bootloader中集成Wi-Fi/BLE协议栈，或用两阶段Bootloader
2. **安全性**：
   - 固件签名与验签（私钥签名、公钥验证）
   - 传输加密（TLS/DTLS）
   - 密钥安全存储
3. **健壮性**：
   - 断点续传
   - 双备份区（Dual-Bank）防断电变砖
   - 版本管理与回滚
4. **后端支持**：服务器存储固件、管理设备、推送更新

## 七、开发"坑"与建议

- **大小限制**：Bootloader越精简越好，避免引入不必要的库
- **稳定性压倒一切**：充分测试边界条件和异常情况
- **别忘了看门狗**：擦写Flash耗时操作要"喂狗"
- **日志与调试**：用UART输出关键日志，或JTAG/SWD单步跟踪
- **保护机制**：Flash写保护保护Bootloader自身
- **版本意识**：Bootloader和APP都需要版本号，升级时做兼容性检查

## 原文链接

https://mp.weixin.qq.com/s/2W71UB_hcQKBcIwGuGC8uQ
