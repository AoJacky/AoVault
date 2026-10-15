---
title: "EnDat3 资源占用报告（UC 版 vs Basic/MinIP 版）"
type: Feature
date: 2026-09-28
tags: [工作, EnDat3, FPGA, 资源占用]
source: "工作笔记"
---

# EnDat3 资源占用报告

> [!info] 测试器件
> **Cyclone 10 LP 10CL025YU256I7G**
> 逻辑单元 24,624 / M9K 内存 608,256 bit / DSP 9-bit 132

> [!info] 数据来源
> 采集自两次 Quartus Fitter 成功编译（`route_Mars/output_files/PmsmCtrlTop_Mars.fit.summary`），顶层均为 `PmsmCtrlTop_Mars`：
> - **UC 版**：`dev_fpgaEnDat3Full` 分支，`e3_master_memif_uc_top`（含 UC 桥接），编译时间 16:29:52
> - **Basic/MinIP 版**：`dev_fpgaEnDat3MinIP` 分支，`e3_master_basic_top`（最小 IP），编译时间 17:02:54

> [!tip] 已应用优化
> UC 版响应内存 `dpram64` 已由寄存器式 RAM 换成 Intel block RAM（`dpram64_intel.vhd`）。

---

## 1. 两版整体资源对比（实测）

| 指标 | UC 版 (Full) | Basic 版 (MinIP) | Basic 相对节省 |
|---|---:|---:|---:|
| **逻辑单元 LE** | 20,419 / 24,624 (83%) | 19,943 / 24,624 (81%) | **−476 (−2%)** |
| 组合逻辑 | 15,932 | 15,298 | −634 |
| 寄存器 | 15,139 | 15,142 | +3 |
| **块内存 Memory Bits** | 489,928 / 608,256 (81%) | 480,648 / 608,256 (79%) | **−9,280** |
| DSP 9-bit | 102 / 132 (77%) | 102 / 132 (77%) | 0 |
| PLL | 2 / 4 | 2 / 4 | 0 |

**要点：**
- Basic 版逻辑单元仅少 **476 LE（约 2%）**，寄存器几乎相等，但**少用 9,280 bit 块内存**（正好等于 UC 版多出的响应内存）。
- 两版共享的 `e3_master` 协议核占绝大多数 LE，差异主要来自外围接口。

---

## 2. UC 版资源占用明细（LE 分解）

UC 版整体（`EncoderTop` 实例）≈ **2,021 LE / 1,012 REG / 9,280 bit 块内存 / 4 DSP**。

| 层级 / 模块 | LE | REG | 可裁剪性 |
|---|---:|---:|---|
| **`e3_master:i_core`（EnDat3 串行协议核）** | **1,046** | 571 | 部分可裁（见 §4） |
| `registerblock:i_rb`（UC 寄存器文件） | 398 | 208 | 不可裁（必需） |
| `response_mem:i_rsp_mem`（响应内存控制，含 block RAM） | 218 | 2 | 内存已最优；glue 小 |
| `adapter_uc:i_adapt`（UC 总线适配器） | 166 | 70 | 不可裁（必需） |
| `memory_mux:i_mem`（寄存器/RAM 路由） | 163 | 66 | 基本不可裁 |
| `bg_handler:i_bg`（背景请求处理） | 58 | 54 | 不可裁 |
| `irq_manager:i_irq`（中断管理） | 34 | 8 | 不可裁 |
| `fg_handler:i_fg`（前台循环处理） | 27 | 19 | 不可裁 |
| `start_logic:i_start`（启动逻辑） | 5 | 4 | 不可裁 |
| `clock_reset:i_clock_reset`（含 PLL） | 9 | 8 | 不可裁 |

协议核内部再细分：

| 子模块 | LE | REG |
|---|---:|---:|
| `e3_master_core`（总核） | 745 | 369 |
| `e3_bg_handler`（背景核心/请求） | 190 | 98 |
| `e3_runtime`（运行时间统计） | 76 | 70 |
| `e3_watchdog`（看门狗） | 26 | 19 |
| `e3_rs485_phy` | 372 | 167 |
| `e3_byte2frame` | 254 | 145 |
| `e3_phy_tx` / `e3_phy_tx_fsm` | 174 / 137 | 55 / 36 |
| `e3_master_fsm` | 144 | 54 |
| `e3_phy_rx` | 145 | 64 |

---

## 3. 内存（Memory Bits）分解

UC 版的块内存完全来自 **`response_mem` → `dpram64`（block RAM）**：9,280 bit = FID(128×64)+HPF/LPF(16×64)+LPH(1×64) 响应缓冲。其余模块均无块内存。

---

## 4. 可裁剪项与不可裁剪项

### 🔴 不可裁剪（功能性必需）

| 项 | LE | 原因 |
| :--- | ---: | :--- |
| 串行协议核 PHY/编解码主体 | ~1,500+ | EnDat3 协议实现，IP 内核，两版几乎不变 |
| `registerblock`（UC 寄存器文件 64-bit） | 398 | MCU 经 UC 总线访问配置/状态必需 |
| `adapter_uc`（UC 总线适配） | 166 | MCU↔内部内存接口桥 |
| `memory_mux` | 163 | 寄存器/RAM 地址路由 |
| `fg_handler` / `bg_handler` / `irq_manager` / `start_logic` | ~124 | 主控循环/中断/启动必需 |
| 响应内存（FID/OEM）block RAM | 9,280 bit | **FID/OEM 数据缓冲，已确认不能关** |

### 🟡 可裁剪（省资源但需权衡）

| 项 | LE 上限 | 可行性 / 权衡 |
| :--- | ---: | :--- |
| `G_IMPL_RUNTIME=1`→0（`e3_runtime`） | ~70-90 | 去掉运行时间计数器；需确认不用 `runtime` 寄存器 |
| `G_IMPL_WATCHDOG=1`→0（`e3_watchdog`） | ~26-30 | 去掉看门狗；需确认不用 `cfg_wd_timeout` / `wd_error` |
| `G_IMPL_REQ_HANDLER=1`→0（`e3_bg_handler`） | ~190 | 去掉背景请求处理器；需确认不用背景 FID 请求 |
| `G_RSP_WIDTH`=3（64-bit rsp）→2/1 | 少量 | 压缩 rsp 通路；会影响 LPF/HPF 数据宽度 |

> 以上改动均需修改 `e3_master_memif.vhd` 中 `i_core` 的 generic 映射（第 92-102 行）。该文件不在用户明确"保持不变"清单内，但触及 IP 源，需谨慎评估功能影响。

### 🔵 已确认不可行 / 不建议

| 项 | 原因 |
| :--- | :--- |
| 关闭 FID（`G_FID_BASED_MEMORY_ENABLE=0`） | ❌ 破坏 OEM1/2/3（FID 帧）访问 |
| 改 8-bit 数据（`G_D_WIDTH_16=0`） | ❌ 内部地址恒定 13-bit、内存不变，仅省 `adapter_uc` 内少量字节选择逻辑，且需连带改桥接与 MCU 端 |

---

## 5. UC 版 vs Basic/MinIP 版的选择

- **Basic 版净省 ~476 LE + 9,280 bit 内存**，但代价是：
  - 失去 UC 寄存器/内存接口（MCU 不能经 UC 总线配置寄存器或读响应内存）；
  - **失去 FID/OEM 内存镜像**：OEM1/2/3 只能靠 MCU 逐帧发 `req` 请求、从 `rsp` 解码字段拿（MinIP 版的 `o_lpf_fid`/`o_lpf_bgrsp` 即逐帧输出），无缓冲、无内存寻址。
- 若需要 OEM 且已接 UC 桥接：**UC 版多花 ~476 LE + 9,280 bit，换取完整的寄存器/内存/OEM 访问，明显划算**（LE 占用 83% vs 81% 差距很小）。

---

## 6. 结论与建议

1. UC 版 2,021 LE 是"接入 UC 总线 + 保留 FID/OEM"下的合理基线；其中 ~1,046 LE 是协议核（IP，不能动）。
2. 若仍需降 LE，可选路径为裁剪协议核内的 `runtime`/`watchdog`/`bg_handler`（改 `e3_master_memif.vhd` generic，收益 ≤300 LE），需先确认应用不用这些功能。
3. 全片 LE 83%（UC）/81%（Basic）、内存均 <81%，可 fit。若无容量压力，**建议维持 UC 版现状**，避免触碰 IP 引入风险。
