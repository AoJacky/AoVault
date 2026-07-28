---
title: "FOC无感控制算法"
date: 2026-07-28
tags: [知识库, 电机控制, FOC, 无感控制]
source: "微信整理"
---

# FOC无感控制算法

> FOC无感控制主要方法分类及关键论文与作者梳理。

## 摘要

FOC无感控制的核心方法分为反电势法、电感法和磁链法三大类。本文梳理了各方法的代表性论文、关键国际与国内学者及其贡献方向。

## 原文要点

# 分类
1. 反电势法（EMF）
2. 电感法
3. 磁链法
# 论文
1. AN1078;
2. AN1299;
3. A Globally Exponentially Stable Position Observer for Interior Permanent Magnet Synchronous Motors;
4. Synchronization at Startup and Stable Rotation Reversal of Sensorless Nonsalient PMSM Drives;
5. Sensorless Control of Surface-MountPermanent-Magnet Synchronous MotorsBased on a Nonlinear Observer;
6. Sensorless Control Method for PMSM Based on Frequency-Adaptive Disturbance Observer;
Key Author:

### 国际

1. Romeo Ortega

国际自动控制与电机控制领域的顶尖学者，学术地位极高，是非线性与自适应控制、永磁同步电机（PMSM）无感观测器方向的奠基人之一，也是工程落地与理论严谨性结合的典范。

2. Zi-Qiang Zhu 诸自强（英国谢菲尔德大学）

地位：无感控制全球第一人，英国皇家工程院院士，IEEE Fellow。

贡献：

系统建立PMSM 无感控制理论体系，高频注入、反电动势、全速域方案集大成者。

著有 IEEE/Wiley 经典专著 Sensorless Control of Permanent Magnet Synchronous Machine Drives（2024），行业 "红宝书"。

关键词：高频注入、凸极电机、全速域无感、容错控制。

3. D. G. Luenberger

地位：观测器理论之父，你前面问的 1971 年 IEEE TAC 经典文章作者。

贡献：提出Luenberger 观测器，奠定所有状态观测器的数学基础，无感控制的理论源头。

4. Joachim Holtz（德国亚琛工业大学）

地位：电机控制泰斗，IEEE Millennium Medal。

贡献：无传感器控制早期奠基人，提出基于反电动势、磁链观测的经典方案，影响至今。

5. Robert D. Lorenz（美国威斯康星大学）

贡献：高频注入、零速 / 低速无感权威，EKF、参数鲁棒性、伺服级无感控制标杆。

6. Jang-Hwan Kim / Seung-Ki Sul（韩国首尔大学）

Sul 教授：韩国电机控制学派领袖，扩展反电动势、无感观测器、PLL 结构大量被沿用。

7. Alfio Consoli（意大利）

贡献：早期无传感器控制重要贡献者，模型参考自适应、磁链观测。
### 国内

1.  刘计龙（海军工程大学）

方向：全速域无感、高频注入、启动与切换、舰船电驱。

代表作：《永磁同步电机无位置传感器控制技术研究综述》（电工技术学报高引）。

2. 邹继斌 / 王盼（哈尔滨工业大学）

方向：滑模观测器 SMO、无感 FOC、逆变器非线性补偿，国内工程落地最强团队之一。

3. 原熙博 / 陈硕（中国矿业大学）

方向：新型观测器、有限位置集 PLL、ESO、低算力高性能无感，近年 TIE/TPE 高产出青年领军

4. 刘刚（北京航空航天大学）

方向：航空航天级高精度无感、高速电机、磁悬浮飞轮，国防与高端装备方向权威。

## 个人想法

## 原文链接

微信整理
