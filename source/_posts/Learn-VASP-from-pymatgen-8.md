---
title: Chap.8  计算小白硬学VASP —— 材料性质计算—>能带计算
date: 2024-06-18 00:00:00
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

本章介绍如何计算能带结构，VASP计算流程代码如下：

{% tabs primtive_cell %}

<!-- tab 下载结构 -->

```python
from mp_api.client import MPRester

api_key = 'your key'

with MPRester(api_key) as mpr:
    # 获取结构
    structure = mpr.get_structure_by_material_id('mp-924129')
    # 转化成原胞
    primtive_structure = structure.get_primitive_structure()
    # 保存结构
    primtive_structure.to(filename='ZrNiSn_primitive.cif')
```

<!-- endtab -->

<!-- tab 结构优化 -->

```python
from pymatgen.io.vasp.sets import MPRelaxSet
from pymatgen.io.vasp.inputs import Kpoints
from pymatgen.core.structure import Structure
import os

# 结构路径
primtive_cell = './ZrNiSn_primitive.cif'
#自定义INCAR
incar = {
    "ENCUT":400,
    "EDIFF":1E-4,
    "EDIFG":-0.05,
    "ISPIN":1,
    "LORBIT":12, 
}
# 读取结构
struct = Structure.from_file(primtive_cell)
# 生成relax输入
primtive_relax = MPRelaxSet(struct, user_incar_settings=incar, user_potcar_functional='PBE_54')
# 保存
primtive_relax.write_input('./ZrNiSn_primitive/relax')
```

<!-- endtab -->

<!-- tab 自洽计算 -->

```python
from pymatgen.io.vasp.sets import MPStaticSet
import os

# 结构优化文件路径
relax_dir = './ZrNiSn_primitive/relax'
# 自定义INCAR
static_incar = {
    "EDIFF":1E-6,
    "ISMEAR":0,
    "LORBIT":12,
}
# 生成自洽计算输入
primtive_static = MPStaticSet.from_prev_calc(prev_calc_dir=relax_dir,user_incar_settings=static_incar, user_potcar_functional='PBE_54')
# 保存
prim_static.write_input('./ZrNiSn_primitive/static')
```

<!-- endtab -->

<!-- tab 能带计算 -->

```python
from pymatgen.io.vasp.sets import MPNonSCFSet
from pymatgen.symmetry.bandstructure import HighSymmKpath
from pymatgen.core.structure import Structure
from pymatgen.io.vasp.inputs import Kpoints
import os

# 自洽计算路径
static_dir = './ZrNiSn_primitive/static'
# 读取结构
structure = Structure.from_file(os.path.join(static_dir, 'POSCAR'))
# 生成高对称路径
kpath = HighSymmKpath(structure=structure,path_type='hinuma')
# 自动根据高对称路径生成K点
kpoints = Kpoints.automatic_linemode(divisions=10,ibz=kpath)
# 自定义INCAR
band_incar = {
    "EDIFF":1E-6,
    "ISMEAR":0,
    "LORBIT":12,
    "NBANDS":64,
}
# 生成band输入
band = MPNonSCFSet.from_prev_calc(prev_calc_dir=static_dir,user_incar_settings=band_incar,user_kpoints_settings=kpoints,user_potcar_functional='PBE_54')
# 保存
band.write_input('./ZrNiSn_primitive/band')
```

<!-- endtab -->

{% endtabs %}

INCAR中有几点设置需要注意：

- `ISPIN`开关需要根据自己的体系来决定是否考虑自旋；
- `LORBIT`建议全程设置为12，方便后面分析轨道投影能带/态密度。

能带计算中这里我们采用`HighSymmKpath`方法自动针对结构生成高对称路径，需要注意的是，这里的结构必须是`primtive_cell`。

***¡Muchas gracias!***
