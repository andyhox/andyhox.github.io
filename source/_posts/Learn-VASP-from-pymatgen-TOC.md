---
title: 计算小白硬学VASP系列——《Learn VASP from pymatgen》章节目录
tags:
  - VASP
  - pymatgen
  - Materials Project
author: 炫酷老司机
categories:
  - Learn VASP from pymatgen
sticky: 999
copyright: 欢迎个人转载、使用、转贴等，但请获得作者同意且注明出处！
comment: true
cover: /images/wallhaven-gpkd77_3840x2160.png
excerpt: Learn VASP from pymatgen
date: 2024-06-20 00:00:00
---


### 《Learn VASP from pymatgen》汇总

### 1、前期准备

{% btn regular :: Click here :: https://andyhox.github.io/2024/05/30/Learn-VASP-from-pymatgen-2/ :: fa-solid fa-play-circle %}

- 编译pymatgen，配置相关环境

### 2、学会使用Materials Project

{% btn regular :: Click here :: https://andyhox.github.io/2024/05/30/Learn-VASP-from-pymatgen-3/ :: fa-solid fa-play-circle %}

- 介绍无机结构网站`Materials Project`下载结构、获取已有性质
- 介绍mp_api接口的用法

### 3、MPRelaxSet用法

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/04/Learn-VASP-from-pymatgen-4/ :: fa-solid fa-play-circle %}

- 了解最常用的结构优化模块`MPRelaxSet()`使用
- 自定义参数设置（INCAR tags，K点个数）

### 4、态密度计算输入自动生成

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/11/Learn-VASP-from-pymatgen-5/ :: fa-solid fa-play-circle %}

- `MPRelaxSet()`、`MPstaticSet()`、`MPNonSCFSet()`配合使用生成dos计算输入

### 5、态密度结果分析

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/13/Learn-VASP-from-pymatgen-6/ :: fa-solid fa-play-circle %}

- 提起总态密度TDOS
- 提取分态密度PDOS

### 6、态密度结果绘图

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/17/Learn-VASP-from-pymatgen-7/ :: fa-solid fa-play-circle %}

- `Matplotlib`库绘图
- `Dosplotter()`模块绘图
- 如何平滑处理态密度图，减少毛刺

### 7、能带计算

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/18/Learn-VASP-from-pymatgen-8/ :: fa-solid fa-play-circle %}

- `HighSymmKpath()`方法自动生成高对称路径

### 8、能带分析

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/27/Learn-VASP-from-pymatgen-9/ :: fa-solid fa-play-circle %}

- `BandStructure()`方法提取能带信息
- `BSPlotter()`方法自动绘图

### 9、能带态密度汇总

{% btn regular :: Click here :: https://andyhox.github.io/2024/06/28/Learn-VASP-from-pymatgen-10/ :: fa-solid fa-play-circle %}

- `BSDOSPlotter()`方法同时绘制能带态密度结果

### 10、构建slab模型

{% btn regular :: Click here :: https://andyhox.github.io/2024/08/05/Learn-VASP-from-pymatgen-11/ :: fa-solid fa-play-circle %}

- `SlabGenerator`模块自动生成`slab`结构

{% note info %}

To be continued

{% endnote %}
