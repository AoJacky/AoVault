---
title: "Cortex-M裸机环境下临界区保护的三种实现"
date: 2021-09-07
tags: [知识库, 嵌入式, Cortex-M, 临界区, 中断, PRIMASK]
source: "微信公众号 痞子衡嵌入式 / strongerHuang"
---

# Cortex-M裸机临界区保护的三种实现

> 从入门到终极：利用系统全局中断控制实现裸机临界区保护。

## 两个测试场景

**场景一**：非嵌套任务，enter/exit严格成对
```c
enter_critical(); do_task1(); exit_critical();
enter_critical(); do_task2(); exit_critical();
```

**场景二**：嵌套任务，task2是task1的子任务
```c
// critical_section_test: enter → do_task1 → exit
// do_task1内部: enter → do_task2 → exit, 然后 do_task3
```
task3没有被显式保护，但主任务整体受保护 → task3也应受保护。

## 三种实现

### 2.1 入门做法（不能处理嵌套）
```c
void enter_critical(void) { __disable_irq(); }
void exit_critical(void)  { __enable_irq(); }
```
❌ 嵌套场景下task3不被保护（task2后面的exit直接开了全局中断）

### 2.2 改进做法（全局计数器）
```c
static uint32_t s_lockObject;

void enter_critical(void) {
    __disable_irq();
    ++s_lockObject;
}

void exit_critical(void) {
    if (s_lockObject <= 1) {
        s_lockObject = 0;
        __enable_irq();
    } else {
        --s_lockObject;
    }
}
```
✅ 解决嵌套问题。❌ 需全局变量+初始化函数，有篡改风险。

### 2.3 终极做法（PRIMASK寄存器保存/恢复）

PRIMASK最低位PM：1=关全局中断, 0=开全局中断

```c
uint32_t enter_critical(void) {
    uint32_t regPrimask = __get_PRIMASK();  // 保存当前状态
    __disable_irq();                         // 关中断
    return regPrimask;
}

void exit_critical(uint32_t primask) {
    __set_PRIMASK(primask);                  // 恢复原状态
}
```

使用方式：
```c
uint32_t primask = enter_critical();
do_task();
exit_critical(primask);
```

✅ 无全局变量 ✅ 优雅处理嵌套 ✅ PRIMASK自动记录历史状态

### PRIMASK操作在各IDE的实现

**IAR**（cmsis_iccarm.h）：
```c
#define __set_PRIMASK(VALUE)  (__arm_wsr("PRIMASK", (VALUE)))
#define __get_PRIMASK()       (__arm_rsr("PRIMASK"))
```

**Keil**（cmsis_armclang.h）：
```c
__STATIC_FORCEINLINE void __set_PRIMASK(uint32_t priMask) {
    __ASM volatile("MSR primask, %0" :: "r" (priMask) : "memory");
}

__STATIC_FORCEINLINE uint32_t __get_PRIMASK(void) {
    uint32_t result;
    __ASM volatile("MRS %0, primask" : "=r" (result));
    return result;
}
```

## 原文链接

https://mp.weixin.qq.com/s/w4J5DCJMEnTdZpGQ8QlKUw
