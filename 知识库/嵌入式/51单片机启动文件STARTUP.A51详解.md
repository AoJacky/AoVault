---
title: "51单片机启动文件STARTUP.A51详解"
date: 2026-07-30
tags: [知识库, 嵌入式, 51单片机, Keil, STARTUP.A51, 内存模型]
source: "老马识途单片机"
---

# 51单片机启动文件STARTUP.A51详解

> Keil C51创建项目时提示的STARTUP.A51启动文件逐段详解：内存初始化、再入函数堆栈、三种存储模型。

## 摘要

详解51单片机STARTUP.A51启动文件：第一段内存初始化（IDATA/XDATA/PDATA）、第二段再入函数堆栈初始化（SMALL/LARGE/COMPACT三种模型）、$NOMOD51含义及各存储区区别。

## 原文要点

### 是否需要启动文件？
- 一般选择"是"添加STARTUP.A51
- 处理器复位后首先执行启动代码，再跳转到main()

### 第一段：内存初始化
- **IDATALEN**：IDATA区初始化长度，默认80H（128字节）
  - IDATA区(0x00-0xFF)涵盖DATA区(0x00-0x7F)和BIT区
  - 至少保证C51运行库相关存储器进行0初始化
- **XDATASTART**：XDATA起始地址，默认0
- **XDATALEN**：XDATA初始化长度，默认0（不清零）
- **PDATASTART**：PDATA起始地址，默认0
- **PDATALEN**：PDATA初始化长度，默认0

### 51存储区区别
- **data**：0x00-0x7F，128字节内部RAM（直接寻址）
- **idata**：0x00-0xFF，256字节内部RAM（间接寻址，前128与data相同）
- **xdata**：外部扩展RAM，0x0000-0xFFFF
- **pdata**：外部扩展RAM的低256字节

### 第二段：再入函数堆栈初始化
- **SMALL模型**：变量存内部RAM，访问快但空间小（128B/256B）
  - IBPSTACK=1启用，IBPSTACKTOP=0xFF+1
- **LARGE模型**：变量存外部RAM（64KB），16位间接寻址
  - XBPSTACK=1启用，XBPSTACKTOP=0xFFFF+1
- **COMPACT模型**：变量存外部RAM（256B页），8位间接寻址
  - PBPSTACK=1启用，PBPSTACKTOP=0xFF+1
  - 适合少量外部RAM，需程序指定页高位地址

### 存储模型选择建议
- 一般使用SMALL模型（变量优先存内部RAM，不够再存外部）
- COMPACT适合外部RAM≤256字节
- LARGE适合大容量外部RAM

## 原文链接

https://mp.weixin.qq.com/s/ChsjjnUuqhB3b8qA8PHSBg
