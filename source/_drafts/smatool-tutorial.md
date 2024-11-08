---
title: 计算工具系列 —— Smatool
tags:
  - VASP
  - Smatool
author: 炫酷老司机
categories:
  - toolkit
sticky: 17
copyright: 欢迎个人转载、使用、转贴等，但请获得作者同意且注明出处！
comment: true
mathjax: true
cover: /images/toolkit.png
excerpt: Be lazy, stay efficient, and enjoy your coffee break!
date: 2024-09-24 00:00:00
---

***¡Hola a todos!***

最近学习了一下`Smatool`的用法，分享一下使用心得。

[官网介绍]([gmp007/smatool: Open-source first-principles computational toolkit for the efficient calculation of the strength of materials in 1D, 2D, and 3D materials at both zero and finite temperatures (github.com)](https://github.com/gmp007/smatool))

> Strength of Materials Analysis Toolkit is a comprehensive computational package developed to evaluate the tensile, shear, biaxial, and indentation strength (Vickers' hardness) of materials at zero temperature (using density functional theory) and finite temperature (using ab-initio molecular dynamics). Integrating advanced algorithms and leveraging the capabilities of well-established electronic structure codes like VASP and QE as the stress calculator, SMATool provides a robust platform for analyzing the strength of 1D, 2D, and 3D materials.

简单来说，`smatool`可用于评估材料的拉伸、剪切、双向和压痕强度，采用密度泛函理论和第一性原理分子动力学。

### 安装流程

推荐在`conda`环境下进行安装，首先新建一个环境，后续专门为`smatool`配置依赖库：

```shell
conda create -n smatool_env
```

进入`smatool_env`环境后，可直接`pip`快速安装：

```shell
conda activate smatool_env
pip install -U smatool
```

如果未出现任何报错就说明安装完成了，后续可以直接使用了。

### 输入文件介绍

使用`smatool`我们只需要提供结构文件即可，`smatool`接受的结构文件格式为`.cif`和`.vasp`格式。确保当前路径下有符合要求的结构文件后，在命令行直接输入`smatool -0`会生成`smatool.in`文件，跟大多数软件一样，`.in`文件的生成大多都是控制计算的参数和流程，下面是`smatool`生成的默认`.in`文件：

```tex
########################################
###  SMATool package input control   ###
########################################
#choose stress calculator: VASP/QE currently supported
code_type = vasp

#choose method: DFT(static) and MD(dynamic)
mode = dft

#structure file name with .cif or .vasp
structure_file = file.cif

#choose dimension: 1D/2D/3D
dimensional = 2D

# strain: start, end, and interval
strains = 0.01 0.8 0.01

#perform compression with tensile 
plusminus = off

#yieldpoint offset, e.g. 0.2%
yieldpoint_offset = 0.002

#stress components: Tensile_x Tensile_y Tensile_z Tensile_biaxial indent_strength Shear (xz for 1D and 3D and xy for 2D)
# you can writ more than one case by just listing them (no comma, just space)  
components = Tensile_x

#supercell for MD simulation
md_supercell = 1, 1, 1

#save structure files at each strain
save_struct = on

#molecular dynamics time step
md_timestep = 500

#apply rotation along strain (plane) and slip directions
rotation = off

#define strain direction, uses Miller indices, e.g., 11, 0-1 for 2D
slip_parameters = 100, 111

#slipping on; must be on/yes/1 once shear modeling is enabled
slipon = off

#explicit potential directory
potential_dir = /potential/

#Use saved data. postprocessing only
use_saved_data = false

#job submission command
job_submit_command = vasp/pw.x > log
```

仅介绍几个关键的词条，其余可看英文注释或查看官网源码：

- `code_type`：`smatool`设有`VASP`和`QE`结构，需要指定后续第一性原理计算用到的软件；
- `mode`：指定计算方式是通过“静态“的弛豫计算还是aiMD，区别在于”静态“的是通过弛豫计算来进行不考虑温度，而aiMD则是在有限温度下进行一定步数的计算来得到结构进行分析；
- `structure_file`：结构文件；
- `strains`：应变区间，三个值对应的是起点，终点，取点步长。如果不知道应变范围该怎么取，可以适当取大一点。该软件自带算法可检测屈服极限后连续五个下降的应力会自动停止计算，无论此时有没有打到应变的取值上限；
- `components`：定义计算内容：抗拉强度，剪切强度，压痕硬度。根据材料维度不同，可选择的计算内容也不同，比如体相的抗拉强度可以计算xyz三个方向的；
- `slip_parameters`：对应材料的滑移系。滑移面，滑移方向；
- `rotation`：是否考虑滑移面和滑移方向的旋转，在塑性变形中涉及；
- `slipon`：与`rotation`合用，在计算剪切时考虑滑移作用。  

