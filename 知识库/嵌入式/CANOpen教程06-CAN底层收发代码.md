---
title: "CANOpen教程06-CAN底层收发代码"
date: 2026-07-30
tags: [知识库, 嵌入式, CAN, CANOpen, STM32, 位时序]
source: "strongerHuang"
---

# CANOpen教程06-CAN底层收发代码

> 结合STM32F103标准外设库例程讲解CAN数据帧参数、发送接收代码、位时序与波特率配置。

## 摘要

strongerHuang原创CANOpen系列教程第06篇，结合STM32代码讲解：CAN数据帧编程控制参数（ID/IDE/RTR/DLC/Data）、发送接收数据结构、位时序与波特率计算公式及配置。

## 原文要点

### 传输数据相关参数
- CAN总线传输核心就是发送和接收
- 数据帧中部分字段由控制器自动完成（帧起始、CRC校验）
- 由编程控制的参数：ID、IDE、RTR、DLC、Data

### CAN发送代码
- **发送数据结构**：ID、IDE（扩展帧标识）、RTR（远程帧标识）、DLC（数据长度）、Data
- 发送配置参数对应数据帧格式中的可编程字段

### CAN接收代码
- 接收参数与发送参数对应
- STM32内置CAN接收多一个FMI参数（接收消息邮箱筛选器索引）
- 示例使用中断接收，中断函数中打印用于测试

### 位时序及传输波特率
- 位时序间接决定传输波特率
- **波特率计算公式**：BaudRate = CAN_CLK / (Prescaler × (1 + BS1 + BS2))
- 波特率1M时参数配置示例：Prescaler、BS1、BS2具体取值
- CAN时钟36M（具体看时钟配置）
- 波特率固定时位时序参数可以不同，只要遵循计算公式

### 例程下载
- 百度网盘：https://pan.baidu.com/s/1LzD0Epc-Z8vlHsb-sD3WVw
- 提取码：l2dc

## 原文链接

https://mp.weixin.qq.com/s/3PPZxbDu_ff71hreP73b2g
