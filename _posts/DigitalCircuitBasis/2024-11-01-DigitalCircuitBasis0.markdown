---
layout:     post
title:      "数电术语"
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
> SSI - 小规模集成电路（Small-Scale Integration）  
> MSI - 中规模集成电路（Medium-Scale Integration）  
> LSI - 大规模集成电路（Large-Scale Integration）  
> VLSI - 超大规模集成电路（Very-Large-Scale Integration）  
> ULSI - 极大规模集成电路（Ultra-Large-Scale Integration）  
## ASIC与SOC
> ASIC - 应用特定集成电路（Application-Specific Integrated Circuit）    
> SOC - 片上系统（System on Chip）  

ASIC 是一种专为特定应用或功能设计的集成电路。与通用芯片（如微处理器）不同，ASIC 不是为了处理多种不同任务而设计，而是针对特定应用的需求进行优化。应用譬如比特币矿机的ASIC芯片Antminer，专用于网络路由的ASIC，图像处理中的某些专用芯片（如视频解码/编码芯片）。


SoC是一种集成电路技术，它将一个完整的计算机系统集成到一个单独的芯片上，通常包括中央处理器（CPU）、图形处理单元（GPU）、内存（RAM）、输入输出接口（USB、HDMI、SPI、I2C、UART）等、存储控制器、无线通信模块（Wi-Fi、蓝牙、GPS、4G/5G）、电源管理（PMU）等多个子系统和功能模块。SoC具有高度集成、低功耗、高度定制化、小尺寸等优点。应用于智能手机、物联网、嵌入式系统、消费电子产品、智能家居设备等。
## 常见芯片封装
> **双列直插封装（DIP）**
> * **全名：** Dual In-line Package（双列直插封装）
> * **特点：** DIP 封装有两排引脚，适合通过插座插入电路板，适用于手工焊接。它的引脚位于两侧，尺寸较大。
> * **应用：** 早期的计算机、家电和一些原型设计中仍然使用。

> **表面贴装封装（SMD）**
> * **全名：** Surface Mount Device（表面贴装设备）
> * **特点：** SMD 封装通过直接焊接到电路板表面，适合自动化生产，体积较小，安装密度高，适合现代的高集成度电路。
> * **应用：** 现代电子设备中广泛应用，几乎所有的消费电子产品（手机、电脑、电视等）都使用 SMD 封装。

> **四方扁平封装（QFP）**
> * **全名：** Quad Flat Package（四方扁平封装）
> * **特点：** QFP 封装的引脚在芯片的四周，而不是两侧。通常适用于引脚较多、需要更高密度的封装。引脚通过表面贴装技术焊接到电路板。
> * **应用：** 用于较高密度集成电路的封装，例如微处理器、通信芯片等。

> **球栅阵列封装（BGA）**
> * **全名：** Ball Grid Array（球栅阵列封装）
> * **特点：** BGA 使用球形焊接点（锡球）代替传统引脚，焊接点分布在封装的底部。BGA 提供了更高的引脚密度、更好的热性能和电气性能。
> * **应用：** 广泛应用于高性能微处理器、图形处理器（GPU）、内存芯片等，特别适合需要高引脚密度和良好散热的应用。

## TTL与CMOS
**TTL电路（晶体管-晶体管逻辑电路）**  
TTL（Transistor-Transistor Logic）是一种基于双极型晶体管（BJT，Bipolar Junction Transistor）和电阻的逻辑电路。TTL电路使用的是NPN型晶体管，并采用电流驱动的工作原理，因此主要依赖于电流的开关来实现逻辑运算。

**CMOS电路（互补金属氧化物半导体电路）**
CMOS（Complementary Metal-Oxide-Semiconductor）电路使用N型MOSFET和P型MOSFET晶体管，这些晶体管工作在电压驱动模式下。CMOS电路的核心特点是使用了互补的N型和P型晶体管，它们交替工作以实现逻辑运算。

## 常用电平标准





