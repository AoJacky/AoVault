---
title: "void型指针的高阶用法——从底层存储到驱动抽象"
date: 2021-08-03
tags: [知识库, 嵌入式, C语言, void指针, NV管理, 驱动抽象]
source: "微信公众号 嵌入式客栈 / strongerHuang"
---

# void型指针的高阶用法

> void指针=通用指针，巧妙利用可实现业务数据与底层存储的解耦、驱动分层抽象。

## void指针本质

- **通用指针/泛指针**：只表示纯地址，不属于任何类型
- 不可算术运算（编译器不知道自增偏移量）
- **任意时刻**可用其他类型指针代替void指针，反之亦然
- 解引用必须**类型转换**

```c
void *p;
int a = 10;
p = &a;
// *p = 20;           // ❌ 编译不过
*(int *)p = 20;       // ✅ 须类型转换
```

> 注意：不同编译器行为不同（IAR/ANSI C/VC会报错，GNU将void当char处理）

## 高阶应用：非易失存储（NV）管理

**需求**：不同模块有不同结构体需要掉电保存到EEPROM/FLASH。

```c
// 模块A
typedef struct { int language; char SN[20]; } T_PARAS;
T_PARAS sysParas;

// 模块B
typedef struct { float kp, ki, kd, T; } T_PID;
T_PID pidParas;
```

**设计统一NV管理子系统**（利用void *）：

```c
typedef struct {
    void *pElement;   // 参数地址（void* 忽略类型）
    int   length;     // 参数长度
} T_NV_LAYOUT;

// 参数映射表
T_NV_LAYOUT nvLayout[] = {
    {&sysParas, sizeof(T_PARAS)},
    {&pidParas, sizeof(T_PID)},
    ...
};

void nv_load(T_NV_LAYOUT *pLayout, int nvAddr, int number);
void nv_store(T_NV_LAYOUT *pLayout, int nvAddr, int number);
```

**优势**：
- 忽略业务数据的具体类型，仅作为"数据块"处理
- 业务逻辑与存储后台隔离解耦
- 通用性强，新参数只需在映射表加一条记录

### 进一步延伸——驱动分层抽象

- 抽象I2C/SPI EEPROM接口统一
- 用**函数指针**定义统一接口 → 类似C++虚函数思想
- 实现驱动模型简易雏形

## 总结

| 技巧 | 作用 |
|------|------|
| void *指针 | 业务数据与底层存储抽象解耦 |
| 分层抽象 | 代码良好可移植性 |
| 函数指针 | 实现虚函数定义接口 |
| 统一接口+底层抽象 | 驱动分层思想 |

## 原文链接

https://mp.weixin.qq.com/s/0rF3lHXdMDO-Qi0vL6eqkA
