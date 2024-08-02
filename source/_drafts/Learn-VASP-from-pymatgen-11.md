---
title: Chap.11  计算小白硬学VASP —— 构建slab模型
tags:
  - VASP
  - pymatgen
  - Materials Project
author: 炫酷老司机
categories:
  - Learn VASP from pymatgen
sticky: 11
copyright: 欢迎个人转载、使用、转贴等，但请获得作者同意且注明出处！
comment: true
cover: /images/wallhaven-gpkd77_3840x2160.png
excerpt: Learn VASP from pymatgen
date: 2024-07-01 00:00:00
---

***¡Hola a todos!***

本章介绍如何构建`slab`模型。

首先切面用到的`miller index`针对的是`bulk`模型的惯用胞（`conventional cell`）而不是原胞（`primtive cell`），这与之前计算能带是刚好相反的。

所以，在进行切面操作时，要确保你的`bulk`模型是惯用胞，可以用`pymatgen`的功能来帮助实现。

- 直接从`Materials project`下载结构

如果是从`MP`上下载结构，我们需要把`conventional_cell = True`打开：

```python
from mp_api.client import MPRester

api_key = "your key"

with MPRester(api_key) as mpr:
    Si = mpr.get_structure_by_material_id("mp-149",conventional_unit_cell=True)
    Si.to(filename="./Si_conventional.cif")
```

这样确保你下载的结构是惯用胞。

- 原胞转化为惯用胞

也可以通过转化来确保使用的是惯用胞：

```python
from pymatgen.core.structure import Structure
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer

structure = Structure.from_file("./Si_primtive.cif")
analyzer = SpacegroupAnalyzer(structure)
print(f'old structure: \n {structure}')
print('\n')
new_structure = analyzer.get_conventional_standard_structure()
print(f'new structure: \n {new_structure}')
```

`new_structure`对应的就是惯用胞结构。运行代码：

```text
old structure: 
 Full Formula (Si2)
Reduced Formula: Si
abc   :   3.849278   3.849279   3.849278
angles:  60.000012  60.000003  60.000011
pbc   :       True       True       True
Sites (2)
  #  SP        a      b      c
---  ----  -----  -----  -----
  0  Si    0.875  0.875  0.875
  1  Si    0.125  0.125  0.125


new structure: 
 Full Formula (Si8)
Reduced Formula: Si
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  Si    0.75  0.75  0.25
  1  Si    0     0.5   0.5
  2  Si    0.75  0.25  0.75
  3  Si    0     0     0
  4  Si    0.25  0.75  0.75
  5  Si    0.5   0.5   0
  6  Si    0.25  0.25  0.25
  7  Si    0.5   0     0.5
```

或者你不知道自己的结构到底是原胞还是惯用胞，可以无脑直接转换，因为对于本身就是惯用胞的结构，`get_conventional_standard_structure()`方法不会修改本身的信息：

```python
from pymatgen.core.structure import Structure
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer

# 读取本身是惯用胞的结构
structure = Structure.from_file("./Si_conventional.cif")
analyzer = SpacegroupAnalyzer(structure)
print(f'old structure: \n {structure}')
print('\n')
new_structure = analyzer.get_conventional_standard_structure()
print(f'new structure: \n {new_structure}')
```

结果不变，运行代码：

```txt
old structure: 
 Full Formula (Si8)
Reduced Formula: Si
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  Si    0.75  0.75  0.25
  1  Si    0     0.5   0.5
  2  Si    0.75  0.25  0.75
  3  Si    0     0     0
  4  Si    0.25  0.75  0.75
  5  Si    0.5   0.5   0
  6  Si    0.25  0.25  0.25
  7  Si    0.5   0     0.5


new structure: 
 Full Formula (Si8)
Reduced Formula: Si
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  Si    0.25  0.75  0.75
  1  Si    0     0     0
  2  Si    0.25  0.25  0.25
  3  Si    0     0.5   0.5
  4  Si    0.75  0.75  0.25
  5  Si    0.5   0     0.5
  6  Si    0.75  0.25  0.75
  7  Si    0.5   0.5   0
```

**Note**：上述方法对于缺陷结构和掺杂结构，由于对称性无法识别可能无法转化。

### SlabGenerator模块

进入正题，切面主要用到`pyamtgen`的`SlabGenerator`模块，首先导入它：

```python
from pymatgen.core.surface import SlabGenerator
```

然后需要先实例化`SlabGenetator`，才能调用里面的切面方法：

```python
from pymatgen.core.surface import SlabGenerator
from pymatgen.core.structure import Structure

Si = Structure.from_file("./Si_conventional.cif")

# 实例化
slabgen = SlabGenerator(
    initial_structure=Si,		# 初始结构
    miller_index=(1, 1, 1),		# 指定晶面
    min_slab_size=15,			# slab厚度
    min_vacuum_size=20,			# 真空层厚度
    in_unit_planes=False,		# 切换 min_slab_size & min_vacuum_size
    center_slab=True,			# slab模型居中
    primitive=True,				# 使用最小切面结构
    reorient_lattice=True,		# 重定向晶格
)
```

这里着重介绍`in_unit_planes`和`primitive`参数。

- `in_unit_planes`：影响`min_slab_size`和`min_vacuum_size`的数值，默认为`False`。

  - 当为`False`时，`min_slab_size`和`min_vacuum_size`对应的单位为`Angstrom `，即上述分别表示`slab`厚度为10埃，真空层厚度为15埃；

  - 当为`True`时，数值为切面的层数，如此时`min_slab_size`和`min_vacuum_size`都设置为3，表示slab厚度和真空层厚度为（1，1，1）面最小重复单元乘以3。需要注意的是，此时数字`3`并不等于原子层数，且不同晶面的最小重复单元的厚度不一样，如`Si`的（1，1，1）面单层厚度是小于（1，0，0）面的。

- `primitive`：默认为True。该参数跟`primitive cell`没有关系，在这里是决定`slab`结构是否选取最小单元。当设置为`True`时，如1x1x1的`Si`惯用胞结构和3x3x3超胞的`Si`惯用胞结构，最后得到的`slab`结构是一样。

`SlabGenerator`下面有许多切面的方法，下面一一介绍。

