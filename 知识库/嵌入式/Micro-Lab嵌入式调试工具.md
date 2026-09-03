---
title: "Micro-Lab嵌入式调试工具"
date: 2026-07-30
tags: [知识库, 嵌入式, 调试工具, 串口, STM32]
source: "智果芯/果果小师弟"
---

# Micro-Lab嵌入式调试工具

> Micro-Lab集成调试工具：串口/网络调试、虚拟示波器、VanGogh组态画布、ModBus计算器等，拖拽式上位机编程。

## 摘要

Micro-Lab嵌入式调试工具集功能介绍及STM32 SDK移植教程，核心亮点是VanGogh组态画布——无需PC编程语言，鼠标拖拽即可创建可视化调试界面。

## 原文要点

### 功能列表
- 串口/网络调试服务及关联模块
- 发送历史永久保存及支持别名
- 串口/网络动态指令编程
- 串口/网络虚拟示波器
- 超级计算器、ModBus RTU计算器
- DTU、串口-网络分裂器
- 应用影子分身
- VanGogh组态画布（事件驱动型上位机编程）

### 组态画布使用
1. 左侧拖拽控件到中间区域
2. 右侧修改控件属性
3. 右键→Run运行界面
4. 通过界面可视化发送指令控制硬件

### STM32 SDK移植
1. 拷贝SDK包到工程
2. MDK添加SDK源文件和头文件
3. 修改工程配置
4. 调用initCanvas(events)初始化
5. 在主循环中检查events数组处理控件事件
6. 示例：SwitchButton控件控制LED灯

## 原文链接

https://mp.weixin.qq.com/s/UH_h-kxdvYz7A6eUMoYbew
