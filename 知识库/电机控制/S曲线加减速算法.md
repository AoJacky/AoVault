---
title: "电机运动控制S曲线算法"
date: 2026-07-28
tags: [知识库, 电机控制, S曲线, 运动控制, 加减速]
source: "微信公众号 strongerHuang / 小麦大叔"
---

# 电机运动控制S曲线算法

> S形加减速速度曲线平滑，减少控制过程中的冲击，相比T形曲线在加速/匀速切换时过冲更小。

## 1 前言

S形加减速的核心特征：速度曲线形状如字母S。相比T形曲线（梯形加减速），S曲线在加速到匀速的切换过程中过冲更小，运动更平滑。

**T形 vs S形对比**：
- T形：加速度突变 → 冲击大
- S形：加速度渐变 → 冲击小，平滑过渡

## 2 理论分析

S曲线在加减速过程中加速度是变化的，引入新变量**加加速度J（Jerk）** = da/dt。

### 三个基本系统参数

- **v_max**：系统最大速度
- **a_max**：最大加速度
- **J**：加加速度

确定这三个参数即可确定整个运行过程。

### S曲线7段划分

完整的S曲线分为7个阶段：

| 阶段 | 时间 | 加速度 | 说明 |
|------|------|--------|------|
| T1 | a_max/J | 0→a_max | 加加速段 |
| T2 | v_max/a_max - a_max/J | a_max | 匀加速段 |
| T3 | a_max/J | a_max→0 | 减加速段 |
| T4 | Tf - 2a_max/J - 2v_max/a_max | 0 | 匀速段 |
| T5 | T3 | 0→-a_max | 加减速段 |
| T6 | T2 | -a_max | 匀减速段 |
| T7 | T1 | -a_max→0 | 减减速段 |

### 2.1 加速度-时间关系

在区间[t_i, t_i+1]内，以t_i为初始点，τ为相对时间：

a(t) = a_i + J_i · τ

### 2.2 速度-时间关系

v = ∫a dt，结合加速度方程积分得到各段速度曲线。

### 2.3 位移-时间关系

s = ∫∫J dt，三重积分得到各段位移方程。

## 3 程序实现思路

关键推导公式：

**加加速度J**：
```
J = (a_max² · v_max) / (Tf · v_max · a_max - v_max² - a_max)
```

**各段时间**：
- T1 = T3 = T5 = T7 = a_max / J
- T2 = T6 = v_max / a_max - a_max / J
- T4 = Tf - 2·a_max/J - 2·v_max/a_max

**需判断S曲线类型**：
- T2 < 0 → 无匀加速段，需调整a_max
- T4 < 0 → 无匀速段，需调整v_max
- J < 0 → 参数不合理，需调整Tf

**总位移**：
```
S = 2·(a_max·T2² + a_max²·T1/J) + v_max·T4
```

## 4 MATLAB程序

参数计算函数：
```matlab
function [Tf1,V,A,J,T] = SCurvePara(Tf, v, a)
T = zeros(1,7);
for i = 1:1000
    J = (a^2 * v) / (Tf*v*a - v^2 - a);
    T(1) = a/J;
    T(2) = v/a - a/J;
    T(3) = T(1);
    T(4) = Tf - 2*a/J - 2*v/a;
    T(5) = T(3);
    T(6) = T(2);
    T(7) = T(1);
    if T(2) < -1e-6
        a = sqrt(v*J);
    elseif T(4) < -1e-6
        v = Tf*a/2 - a*a/J;
    elseif J < -1e-6
        Tf = (v^2 + a) / (v*a) + 1e-1;
    else
        break;
    end
end
A = a; V = v; Tf1 = Tf;
end
```

位移计算函数（7段分段计算）：
```matlab
function s = SCurveScaling(t, V, A, J, T, Tf)
if (t >= 0 && t <= T(1))
    s = 1/6 * J * t^3;
elseif (t > T(1) && t <= T(1)+T(2))
    dt = t - T(1);
    s = 1/2*A*dt^2 + A^2/(2*J)*dt + A^3/(6*J^2);
elseif (t > T(1)+T(2) && t <= T(1)+T(2)+T(3))
    % ... 减加速段
elseif (t > T(1)+T(2)+T(3) && t <= T(1)+T(2)+T(3)+T(4))
    % ... 匀速段
    dt = t - T(1) - T(2) - T(3);
    s = V*dt + ...;
% 减速段利用对称性：t_temp = Tf - t，计算后 s = 1 - s
end
end
```

## 参考文献

[^1]: S形加减速速度曲线平滑
[^2]: 三个参数确定整个运行过程

## 原文链接

https://mp.weixin.qq.com/s/p-V316OKJg6jayv-Buo3jw
