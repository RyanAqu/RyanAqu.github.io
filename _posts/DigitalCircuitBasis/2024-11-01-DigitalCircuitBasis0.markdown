---
layout:     post
title:      "数电术语和常用电平标准"
subtitle:   " \"learning……\""
date:       2024-11-01 20:36:00
author:     "Ryan"
header-img: "img/post-bg-dcb.jpg"
catalog: true
tags:
    - DigitalCircuitBasis
    - 常用电平标准
---

> He tosses and he turns again throughout the night to ease the pain. The laughter dies inside his head,only tears remain.

决定准备一下逻辑/FPGA/数字IC验证，就从复习数电开始吧

![CATALOG](https://ryanaqu.github.io/img/inpost_dcb/dcb_catalog.png  "catalog")

# 一些数电术语
## 集成电路规模
集成电路按照 <u>集成度</u>（每一芯片包含的门电路的个数）对电路规模进行区分
```
SSI - 小规模集成电路（Small-Scale Integration）
MSI - 中规模集成电路（Medium-Scale Integration）
LSI - 大规模集成电路（Large-Scale Integration）
VLSI - 超大规模集成电路（Very-Large-Scale Integration）
ULSI - 极大规模集成电路（Ultra-Large-Scale Integration）
```
## ASIC与SOC
```
ASIC - 应用特定集成电路（Application-Specific Integrated Circuit）  
SOC - 片上系统（System on Chip）
```
ASIC 是一种专为特定应用或功能设计的集成电路。与通用芯片（如微处理器）不同，ASIC 不是为了处理多种不同任务而设计，而是针对特定应用的需求进行优化。应用譬如比特币矿机的ASIC芯片Antminer，专用于网络路由的ASIC，图像处理中的某些专用芯片（如视频解码/编码芯片）。


SoC是一种集成电路技术，它将一个完整的计算机系统集成到一个单独的芯片上，通常包括中央处理器（CPU）、图形处理单元（GPU）、内存（RAM）、输入输出接口（USB、HDMI、SPI、I2C、UART）等、存储控制器、无线通信模块（Wi-Fi、蓝牙、GPS、4G/5G）、电源管理（PMU）等多个子系统和功能模块。SoC具有高度集成、低功耗、高度定制化、小尺寸等优点。应用于智能手机、物联网、嵌入式系统、消费电子产品、智能家居设备等。





