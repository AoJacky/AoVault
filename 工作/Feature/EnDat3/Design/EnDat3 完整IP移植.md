---
title: "EnDat3 完整 IP 移植"
type: Feature
date: 2026-09-03
tags: [工作, EnDat3, FPGA]
source: "工作笔记"
---

### Pin 脚修改

![[Pasted image 20260902170104.png]]

```
A14——ENDAT3_RX
A15——ENDAT3_TX
B14——ENDAT3_DIR
```

### Clock 时钟配置
	1. 输入时钟：120MHz
	2. clk_sys: 100MHz
	3. clk_rxtx: 200MHz
```
	clk_rxtx:clk_sys = 2:1,目的是要保持过采样
```
	4. 波特率配置：

| 参数        | 旧值  | 新值  |
| --------- | --- | --- |
| mc_clkdiv | 8   | 16  |
| mc_dec_ 0 | 1   | 1   |
| mc_dec_ 1 | 8   | 16  |
| mc_dec_ 2 | 9   | 17  |
| mc_dec_ 3 | 15  | 27  |



```
12.5Mbps = 200MHz rxtx / 16 （SHORT=8 周期=40ns，LONG=16 周期=80ns，位周期 80ns）
```
