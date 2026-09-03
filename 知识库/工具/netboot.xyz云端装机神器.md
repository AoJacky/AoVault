---
title: "netboot.xyz——云端一键装机神器（120K+ Star）"
date: 
tags: [知识库, 工具, netboot.xyz, 装机, iPXE, 网络启动, Ventoy]
source: "微信公众号 宝藏工具"
---

# netboot.xyz——云端一键装机神器

## 核心概念

- **Ventoy** = 本地系统全家桶（ISO放U盘）
- **netboot.xyz** = 云端系统自助餐（无需下载ISO）

## 技术原理

基于**iPXE网络启动技术**：
- 开机联网后，云端菜单自动呈现所有发行版列表（永远最新版）
- 按回车→从最近官方镜像站流式加载→即插即用
- 无需手动下载ISO

## 3.x新特性

- 支持x86 + **ARM设备**
- 内置微软签名安全证书→兼容**Secure Boot**，不用关安全启动

## 引导方式

1. **路由器/NAS Docker部署**：局域网内开机F12直接进入菜单
2. **小ISO文件**：官方提供极小ISO，放入旧U盘或配合Ventoy使用

## 原文链接

https://mp.weixin.qq.com/s/y1Mq1Q4qg62E_L7biRS9Mg
