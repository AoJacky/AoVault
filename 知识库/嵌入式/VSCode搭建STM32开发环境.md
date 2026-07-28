---
title: "手把手在VSCode中搭建STM32开发环境"
date: 2026-07-28
tags: [知识库, 嵌入式, STM32, VSCode, OpenOCD, GCC]
source: "微信公众号 嵌入式从入门到放弃"
---

# 手把手在VSCode中搭建STM32开发环境

> STM32CubeMX + VSCode + GCC + OpenOCD，免费开源的STM32开发方案。

## 一、工具下载及环境配置

| 工具 | 用途 |
|------|------|
| STM32CubeMX | ST官方初始化代码生成工具 |
| Visual Studio Code | 代码编辑器 |
| MinGW (gcc) | 提供make工具 |
| GNU Arm Embedded Toolchain | ARM交叉编译工具链 |
| OpenOCD | 烧录调试工具 |

**环境变量**：MinGW、GNU Arm、OpenOCD的bin目录加入PATH

**MinGW配置**：进入`x:\mingw64\bin`，将`mingw32-make.exe`复制一份改名为`make.exe`

**验证**：
```bash
make -v
arm-none-eabi-gcc -v
openocd -v
```

## 二、VSCode插件安装

必须安装：
1. **C/C++ Extension Pack**
2. **Cortex-Debug**

建议安装：
3. One Dark Pro（主题）
4. vscode-icons（文件图标）
5. Rainbow Brackets（彩虹括号）

## 三、STM32CubeMX生成代码

Project Manager → Toolchain/IDE 选择 **Makefile**，配置好时钟外设后 GENERATE CODE。

## 四、VSCode开发配置

### 4.1 解决波浪线错误

打开Makefile查看宏定义和头文件路径，按 `Ctrl+Shift+P` → "C/C++: 编辑配置(JSON)"，生成 `c_cpp_properties.json`：

```json
{
  "configurations": [
    {
      "name": "Win32",
      "includePath": ["${workspaceFolder}/**"],
      "defines": ["_DEBUG", "UNICODE", "_UNICODE", "USE_HAL_DRIVER", "STM32L475xx"],
      "compilerPath": "D:/mingw64/bin/gcc.exe",
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "gcc-arm",
      "configurationProvider": "ms-vscode.makefile-tools"
    }
  ],
  "version": 4
}
```

关键：将Makefile中的宏定义（如`USE_HAL_DRIVER`、`STM32L475xx`）添加到defines，去掉`-D`前缀。

### 4.2 编译

终端输入 `make` 即可编译，生成bin和hex文件。

**静默编译**：修改Makefile，命令前加`@`，用echo输出当前编译文件：
```makefile
$(BUILD_DIR)/%.o: %.c Makefile | $(BUILD_DIR)
	@echo "build $<"
	@$(CC) -c $(CFLAGS) ... $< -o $@
```

### 4.3 Windows兼容clean命令

Windows不支持`rm -rf`，修改Makefile：
```makefile
clean:
	del /Q $(BUILD_DIR)
```

## 五、OpenOCD烧写

### 5.1 配置文件

- 下载器配置：`openocd/scripts/interface/` （如stlink-v2.cfg）
- 芯片配置：`openocd/scripts/target/` （如stm32l4x.cfg）
- 复制到工程目录

### 5.2 烧录命令

```bash
openocd -f interface/stlink-v2.cfg -f target/stlink-v2.cfg -c "program build/firmware.elf verify reset exit"
```

### 5.3 配置为VSCode任务

终端 → 运行任务 → 添加配置任务 → 创建 `tasks.json`：

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Build",
      "type": "shell",
      "command": "make",
      "group": { "kind": "build", "isDefault": true }
    },
    {
      "label": "Flash",
      "type": "shell",
      "command": "openocd -f stlink-v2.cfg -f stm32l4x.cfg -c \"program build/firmware.elf verify reset exit\"",
      "dependsOn": "Build"
    },
    {
      "label": "Clean",
      "type": "shell",
      "command": "make clean"
    }
  ]
}
```

### 5.4 Cortex-Debug调试配置

创建 `launch.json`：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Cortex Debug",
      "cwd": "${workspaceFolder}",
      "executable": "./build/firmware.elf",
      "request": "launch",
      "type": "cortex-debug",
      "servertype": "openocd",
      "configFiles": ["stlink-v2.cfg", "stm32l4x.cfg"],
      "searchDir": ["${workspaceFolder}"]
    }
  ]
}
```

## 原文链接

https://mp.weixin.qq.com/s/VqK2orD8ExX_xdRtTHPbxw
