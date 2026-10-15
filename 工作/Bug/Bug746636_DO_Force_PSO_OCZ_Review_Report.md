---
title: "Bug746636 - DO Force 与 PSO/OCZ 交互缺陷修复 Review 报告"
type: Bug
date: 2026-09-28
tags: [工作, Bug, FPGA, DO]
source: "工作笔记"
---

# 修复 Review 报告：DO Force 与 PSO/OCZ 交互缺陷

- **缺陷编号**：Bug746636
- **标题**：DO force 在 DO:OCZ/PSO 上不生效
- **评审主题**：缺陷（1）根因分析、修复方案 A
- **适用机型**：E530_EC（Mercury，EC_200V）、E530_EC_400（Jupiter，EC_400V）
- **关联仓库**：SW：Venus_Servo_SW（分支 `fix_dualDO`，HEAD `146dd46b4`）；FPGA：Venus_Servo_FPGA（分支 `fix_dualDOsame`，HEAD `55e7ae44`）

---

## 1. 缺陷现象

缺陷：“多个通道同时配置 PSO/OCZ 时仅最后一个生效”。

---

## 2. 根因分析

旧设计采用“全局唯一索引”（PSO index、OCZ index）模型。FPGA 侧通过这两个全局索引字段决定“哪个物理 DO 通道使用 PSO”或“哪个物理 DO 通道使用 OCZ”。SW 在遍历各 DO 通道（HandleChannel）时，每当发现某个通道被配置为 `DO_FN_PSO` 或 `DO_FN_OCZ`，就会将该通道号写入对应的全局索引寄存器字段。后一次配置覆盖前一次配置，导致全局索引最终只指向最后一次配置的通道，只有该通道才有 PSO/OCZ 输出。

## 2.1 映射关系分析（缺陷点）

旧设计中，SW 将“每通道独立的 PSO/OCZ 配置”错误地映射到“全局唯一”的索引。SW 根据通道配置逐通道更新寄存器：当检测到某通道 fn 为 PSO 或 OCZ 时，向全局索引寄存器写入该通道号。该机制导致同一功能类型（PSO/OCZ）只能最终指向一个通道，无法支持多个通道同时配置 PSO/OCZ。
