---
title: Chap.14  计算小白硬学VASP —— 各类结构“Transformation”使用说明（一）
date: 2024-08-09 00:00:00
tags:
  - VASP
  - pymatgen
copyright_author: 炫酷老司机
categories:
  - Learn VASP from pymatgen
cover: /images/wallhaven-gpkd77_3840x2160.png
description: Learn VASP from pymatgen系列教程
---


***¡Hola a todos!***

本章开始，主要介绍`pymatgen`中`standard_transformations`中各类`Transformation`的使用说明。

### AutoOxiStateDecorationTransformation

该类主要是自动平衡结构中的价态信息，以`LiFePO4`结构为例，初始结构信息如下：

```python
from pymatgen.core.structure import Structure

LFP = Structure.from_file("./LiFePO4.vasp")
print(LFP)
```

运行代码：

```tex
Full Formula (Li4 Fe4 P4 O16)
Reduced Formula: LiFePO4
abc   :   4.654917   5.970755  10.236196
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (28)
  #  SP           a         b         c
---  ----  --------  --------  --------
  0  Li    0         0         0
  1  Li    0.5       0.5       0.5
  2  Li    0.5       0         0.5
  3  Li    0         0.5       0
  4  Fe    0.529866  0.25      0.781151
  5  Fe    0.029866  0.75      0.718849
  6  Fe    0.970134  0.25      0.281151
  7  Fe    0.470134  0.75      0.218849
  8  P     0.418623  0.25      0.093866
  9  P     0.918623  0.75      0.406134
 10  P     0.081377  0.25      0.593866
 11  P     0.581377  0.75      0.906134
 12  O     0.744787  0.25      0.094231
 13  O     0.713735  0.545558  0.834155
 14  O     0.713735  0.954442  0.834155
 15  O     0.255213  0.75      0.905769
 16  O     0.709864  0.75      0.044309
 17  O     0.244787  0.75      0.405769
 18  O     0.286265  0.045558  0.165845
 19  O     0.213735  0.454442  0.665845
 20  O     0.786265  0.545558  0.334155
 21  O     0.786265  0.954442  0.334155
 22  O     0.290136  0.25      0.955691
 23  O     0.209864  0.25      0.455691
 24  O     0.790136  0.75      0.544309
 25  O     0.755213  0.25      0.594231
 26  O     0.213735  0.045558  0.665845
 27  O     0.286265  0.454442  0.165845
```

此时结构信息中只包含元素种类及原子坐标。

调用`AutoOxiStateDecorationTransformation`：

```python
from pymatgen.transformations.standard_transformations import AutoOxiStateDecorationTransformation
from pymatgen.core.structure import Structure

LFP = Structure.from_file("./LiFePO4.vasp")

# 实例化
transformation = AutoOxiStateDecorationTransformation()
new_LFP = transformation.apply_transformation(LFP)
# 打印结果
print(new_LFP)
```

运行代码：

```tex
Full Formula (Li4 Fe4 P4 O16)
Reduced Formula: LiFePO4
abc   :   4.654917   5.970755  10.236196
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (28)
  #  SP           a         b         c
---  ----  --------  --------  --------
  0  Li+   0         0         0
  1  Li+   0.5       0.5       0.5
  2  Li+   0.5       0         0.5
  3  Li+   0         0.5       0
  4  Fe2+  0.529866  0.25      0.781151
  5  Fe2+  0.029866  0.75      0.718849
  6  Fe2+  0.970134  0.25      0.281151
  7  Fe2+  0.470134  0.75      0.218849
  8  P5+   0.418623  0.25      0.093866
  9  P5+   0.918623  0.75      0.406134
 10  P5+   0.081377  0.25      0.593866
 11  P5+   0.581377  0.75      0.906134
 12  O2-   0.744787  0.25      0.094231
 13  O2-   0.713735  0.545558  0.834155
 14  O2-   0.713735  0.954442  0.834155
 15  O2-   0.255213  0.75      0.905769
 16  O2-   0.709864  0.75      0.044309
 17  O2-   0.244787  0.75      0.405769
 18  O2-   0.286265  0.045558  0.165845
 19  O2-   0.213735  0.454442  0.665845
 20  O2-   0.786265  0.545558  0.334155
 21  O2-   0.786265  0.954442  0.334155
 22  O2-   0.290136  0.25      0.955691
 23  O2-   0.209864  0.25      0.455691
 24  O2-   0.790136  0.75      0.544309
 25  O2-   0.755213  0.25      0.594231
 26  O2-   0.213735  0.045558  0.665845
 27  O2-   0.286265  0.454442  0.165845
```

此时可以发现对应的元素符号之后多了化合价信息。

`VASP`计算磁性材料时，初始磁矩的设置直接影响到计算过程的收敛快慢以及最终的能量。该类的用法在实际计算中可以为我们计算磁矩信息提供一个合理的初猜值，便于更好的自定义`MAGMOM`参数。

### ChargedCellTransformation

该类主要给结构施加额外电荷，使体系整体带正电或者负电。

```python
from pymatgen.transformations.standard_transformations import ChargedCellTransformation
from pymatgen.core.structure import Structure

LFP = Structure.from_file("./LiFePO4.vasp")
print(LFP)
# 实例化
transformation = ChargedCellTransformation(charge=2)
new_LFP = transformation.apply_transformation(LFP)
print(new_LFP)
```

`ChargedCellTransformation`实例化时传入`charge`参数来确定体系带电情况。

部分输出：

```tex
LFP：
Full Formula (Li4 Mn1 Fe3 P4 O16)
Reduced Formula: Li4MnFe3(PO4)4
abc   :   4.654917   5.970755  10.236196
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (28)

new_LFP:
Full Formula (Li4 Mn1 Fe3 P4 O16)
Reduced Formula: Li4MnFe3(PO4)4
abc   :   4.654917   5.970755  10.236196
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Overall Charge: +2	# 体系带正电
Sites (28)
```

`Overall Charge: +2`说明给体系加了两个正电荷。

怎么在计算中体现带电情况呢？在之前直接调用`MPRelaxSet`的基础上有一点点变化：

```python
from pymatgen.transformations.standard_transformations import ChargedCellTransformation
from pymatgen.core.structure import Structure
from pymatgen.io.vasp.sets import MPRelaxSet

LFP = Structure.from_file("./LiFePO4.vasp")

transformation = ChargedCellTransformation(charge=2)
new_LFP = transformation.apply_transformation(LFP)


old_set = MPRelaxSet(LFP, use_structure_charge=True)
print(f'不带电体系INCAR：\n{old_set.incar}')

new_set = MPRelaxSet(new_LFP, use_structure_charge=True)
print(f'带电体系INCAR：\n{new_set.incar}')
```

首先，在调用`MPRelaxSet`的时候需要设置`use_structure_charge=True`，这样才读取的是结构中的电荷信息，反之则采用默认的设置。

代码部分输出：

```tex
不带电体系INCAR：
......
MAGMOM = 4*0.6 4*5.0 20*0.6
NELECT = 183.0
NELM = 100
NSW = 99
PREC = Accurate
SIGMA = 0.05

带电体系INCAR：
......
MAGMOM = 4*0.6 4*5.0 20*0.6
NELECT = 181.0
NELM = 100
NSW = 99
PREC = Accurate
SIGMA = 0.05
```

对比可发现，两种`INCAR`的`NELECT`的参数不同。`NELECT`参数描述的是体系的电荷数，带电体系中`NELECT`数目变少，则说明此时体系带正电；反之，`NELECT`数目增多说明体系带负电。

该类可用于在半导体材料中由于缺陷和掺杂使得体系带电，进而研究这些带电缺陷的形成能、迁移机制等对材料的影响；也可以用于模拟电池界面上额外电荷的分布情况来研究枝晶的生长情况。

***¡Muchas gracias!***
