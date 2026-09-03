---
title: "写了十年C，这些结构体和联合体的用法我后悔没早知道"
date: 2026-07-28
tags: [知识库, 嵌入式, C语言, 结构体, 联合体]
source: "微信公众号 一枚嵌入式码农"
---

# 写了十年C，这些结构体和联合体的用法我后悔没早知道

> 不讲基础语法，直接上干货。结构体和联合体在嵌入式项目中的7个实用技巧。

## 摘要

本文介绍C语言结构体和联合体在嵌入式项目中的7个高级用法，包括柔性数组、位域、大小端判断、寄存器映射、Tagged Union、container_of宏和内存对齐优化。这些技巧能显著提升代码的可读性、可维护性和运行效率，是嵌入式开发者的必备知识。

## 一、柔性数组——一次malloc搞定变长数据

消息头固定、消息体长度不定的场景，C99柔性数组成员（Flexible Array Member）：

```c
typedef struct {
    uint16_t msg_id;
    uint16_t length;
    uint8_t payload[];  // 柔性数组，不占结构体大小
} Message;

// 一次分配搞定
Message *msg = malloc(sizeof(Message) + data_len);
msg->msg_id = 0x01;
msg->length = data_len;
memcpy(msg->payload, raw_data, data_len);
```

**好处**：头和数据在一块连续内存里，缓存友好，分配释放只需一次。RTOS内存池对碎片化敏感，少一次分配就少一份风险。

> **注意**：柔性数组必须是结构体最后一个成员，且结构体至少还有一个其他成员。

## 二、位域——用1个字节管8个开关

```c
// 传统写法：看三遍才知道干什么
reg |= (1 << 3);        // 设置bit3
val = (reg >> 5) & 0x03; // 取bit6:5

// 位域写法：见名知意
typedef struct {
    uint8_t led_on    : 1;  // bit0: LED开关
    uint8_t buzzer_on : 1;  // bit1: 蜂鸣器
    uint8_t mode      : 3;  // bit4:2: 工作模式(0~7)
    uint8_t reserved  : 3;  // bit7:5: 保留
} DeviceStatus;

DeviceStatus status = {0};
status.led_on = 1;
status.mode = 3;
```

**注意**：C标准没有规定位域排列顺序，跨平台传输时需确认编译器行为。单一平台（如单片机项目）放心用。

## 三、联合体判断大小端——四行代码经典面试题

```c
int is_little_endian(void) {
    union {
        uint16_t value;
        uint8_t bytes[2];
    } test = { .value = 0x0001 };

    return test.bytes[0] == 0x01;  // 1=小端, 0=大端
}
```

## 四、联合体+结构体组合拳——寄存器/协议字段拆解利器

```c
typedef union {
    uint32_t all;      // 整体读写
    struct {
        uint32_t enable : 1;   // bit0
        uint32_t mode   : 2;   // bit2:1
        uint32_t speed  : 3;   // bit5:3
        uint32_t        : 2;   // bit7:6 保留
        uint32_t addr   : 8;   // bit15:8
        uint32_t        : 16;  // bit31:16 保留
    } bits;
} SPI_Config;

// 用法
SPI_Config cfg;
cfg.all = read_register(SPI_CFG_REG);  // 整体读出
cfg.bits.enable = 1;                     // 按字段修改
cfg.bits.speed = 5;
write_register(SPI_CFG_REG, cfg.all);   // 整体写回
```

STM32 HAL库、Linux内核驱动里随处可见此模式。

## 五、Tagged Union——C语言里的"多态"

```c
typedef enum { SHAPE_CIRCLE, SHAPE_RECT, SHAPE_TRIANGLE } ShapeType;

typedef struct {
    ShapeType type;   // 类型标签
    union {
        struct { float radius; }              circle;
        struct { float width, height; }       rect;
        struct { float base, height; }        triangle;
    } data;
} Shape;

float shape_area(const Shape *s) {
    switch (s->type) {
    case SHAPE_CIRCLE:   return 3.14159f * s->data.circle.radius * s->data.circle.radius;
    case SHAPE_RECT:     return s->data.rect.width * s->data.rect.height;
    case SHAPE_TRIANGLE: return 0.5f * s->data.triangle.base * s->data.triangle.height;
    default:             return 0;
    }
}
```

Discriminated Union模式，cJSON等C项目广泛使用。

## 六、container_of——从成员反推宿主结构体

```c
#define container_of(ptr, type, member) \
    ((type *)((char *)(ptr) - offsetof(type, member)))

typedef struct list_node {
    struct list_node *next;
    struct list_node *prev;
} ListNode;

typedef struct {
    int id;
    char name[32];
    ListNode node;  // 嵌入的链表节点
} Student;

// 遍历时从ListNode反推Student
ListNode *cur = head->next;
Student *stu = container_of(cur, Student, node);
```

侵入式链表核心思想，链表代码无需知道外层结构体类型。Linux内核最精妙的宏之一。

## 七、结构体内存对齐——省内存的实战技巧

```c
// 写法A：12字节（padding浪费4字节）
struct BadLayout {
    char a;   // 1 byte + 3 padding
    int  b;   // 4 bytes
    char c;   // 1 byte + 3 padding
};

// 写法B：8字节
struct GoodLayout {
    int  b;   // 4 bytes
    char a;   // 1 byte
    char c;   // 1 byte + 2 padding
};
```

**经验法则**：按成员大小从大到小排列，自然减少padding浪费。

强制取消对齐（按通信协议帧格式对齐）：
```c
#pragma pack(push, 1)
typedef struct { ... } PackedFrame;
#pragma pack(pop)
```

## 个人想法

这些技巧在实际项目中非常实用，特别是container_of宏和Tagged Union模式。container_of是Linux内核链表的核心，理解它能帮助读懂更多底层代码；Tagged Union则是C语言实现多态的优雅方式，在协议解析和状态机中广泛应用。

## 原文链接

https://mp.weixin.qq.com/s/qLZxarsvo25UF7NYPjYn_w
