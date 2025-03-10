---
title: Chap.6  计算小白硬学VASP —— 材料性质计算—>态密度分析(1)
date: 2024-06-13 00:00:00
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

本章介绍如何使用`pymatgen`对态密度结果进行分析。

#### 案例1：Si态密度

先从一个简单的案例入手，了解下`pymatgen`怎么从`VASP`的计算结构中分析态密度数据。

下面是生成从结构优化到态密度计算的完整代码：

{% label 参考代码 blue %}

```python
from pymatgen.io.vasp.sets import MPRelaxSet, MPStaticSet, MPNonSCFSet
from mp_api.client import MPRester

api_key = "your key"

with MPRester(api_key) as mpr:
  struct = mpr.get_structure_by_material_id("mp-149")
  struct.to(filename="./SiPOSCAR")
    
# 结构优化
relax_kpoints = {"length":30}
relax_incar = {
    "EDIFFG":-0.01,
    "ISMEAR":0,
    "ENCUT":300,    
}
relax = MPRelaxSet(
    struct, 
    user_kpoints_settings=relax_kpoints,
    user_incar_settings=relax_incar,
    user_potcar_functional='PBE_54',
    )
relax.write_input('./relax')

# 静态自洽计算(等relax算完再运行)
static_kpoints = {"length":60}
static = MPStaticSet.from_prev_calc(
    prev_calc_dir='./relax', 
    user_kpoints_settings=static_kpoints,
    user_potcar_functional='PBE_54',
    )
static.write_input('./static')

# DOS计算(等静态自洽算完再运行)
dos_incar = {
    "EMIN":-10,
    "EMAX":10,
    "NEDOS":2001,
    "LORBIT":10,
}
dos_kpoints = {"length":60}
DOS = MPNonSCFSet.from_prev_calc(
    prev_calc_dir='./static', 
    user_incar_settings=dos_incar,
    user_kpoints_settings=dos_kpoints,
    user_potcar_functional='PBE_54',
    )
DOS.write_input('./dos')
```

分析态密度的结果需要读`vasprun.xml`文件，这里我们需要导入`Vasprun`模块：

```python
from pymatgen.io.vasp import Vasprun

# 读态密度算完的vasprun.xml文件
vasprun = Vasprun('./dos/vasprun.xml')
# 获取Complete dos对象
dos = vasprun.complete_dos
```

导入`Vasprun`模块后，通过该模块`complete_dos`方法获取完整的态密度对象，`dos`里面包含着所有的态密度信息以及对应的能量范围。**但是有一点需要注意，通过`pymatgen`获取的`dos`里面，费米能级是没有拟合到0的，然而为了便于分析，一般的操作是把费米能级拟合到0，因此我们还需要手动把`Energy`范围拟合一下：**

```python
from pymatgen.io.vasp import Vasprun

# 读态密度算完的vasprun.xml文件
vasprun = Vasprun('./dos/vasprun.xml')
# 获取Complete dos对象
dos = vasprun.complete_dos
# 获取费米能级
fermi = dos.efermi
# 整体能量平移至费米能级为0
energy = dos.energy - fermi
```

##### 体系总态密度

体系总态密度，即TDOS；代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi

# 体系总态密度
# 获取全部的Density
total_densities = dos.densities
# 将费米能级拟合至0之后的Energy与Density重新组合
total_dos = pd.DataFrame({'Energy': energy, 'Density': total_densities[Spin.up]})
# 输出TDOS
print(total_dos)
# 保存为csv文件
total_dos.to_csv('total_dos.csv', index=False)
```



上述代码第14行`'Density': total_densities[Spin.up]`只指定了自旋向上的态。是因为体系无磁性，`MPNonSCFSet`读`static`文件夹自动生成输入是把`ISPIN`关掉了，所以这里的`Spin.up`的数值就是总的`Density`的数值。同理，对于有磁性的体系，这一行代码可以写成：

```python
total_dos = pd.DataFrame({'Energy': energy, 'Spin_Up': total_densities[Spin.up], 'Spin_Dn': total_densities[Spin.down]})
```



##### 元素总态密度

代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi

# 元素总态密度
# dos信息按照元素分类
element_dos = dos.get_element_dos()
# 提取指定元素的Density
Si_densities = element_dos[Element('Si')].densities
# 组合
Si_dos = pd.DataFrame({'Energy': energy, 'Density': Si_densities[Spin.up]})
print(Si_dos)
# 保存输出
Si_dos.to_csv('Si_dos.csv', index=False)
```

需要注意的是第11行，12行，这里如果我们打印出11行的内容可以得到：

> *{Element Si: <pymatgen.electronic_structure.dos.Dos object at 0x000001918E2EE6D0>}*

说明`get_element_dos()`的输出实际是一个字典，对应着元素的种类已经对应的态密度信息对象。因此为了得到具体的态密度值，就需要第12行代码的内容来进行提取，打印第12行的内容，可以得到以下结果：

> *{<Spin.up: 1>: array([0.    , 0.    , 0.    , ..., 0.5654, 0.3542, 0.145 ])}*

此时的数据还是字典形式，提取`Spin.up`的值就是下一步中的`Si_densities[Spin.up]`；同理，对于磁性体系，`spin.down`值的提取也类似。

##### 体系分态密度

代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi

# 体系分态密度
spd_dos = dos.get_spd_dos()
# 体系s轨道Density
total_s = spd_dos[OrbitalType.s].densities
# 体系s轨道Density
total_p = spd_dos[OrbitalType.p].densities
# 重组
total_sp_dos = pd.DataFrame({'Energy': energy, 'Total_s': total_s[Spin.up], 'Total_p': total_p[Spin.up]})
print(total_sp_dos)
# 保存输出
total_sp_dos.to_csv('total_sp_dos.csv', index=False)
```

##### 元素分态密度

代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi


# 元素分态密度
# 获取元素所有轨道
Si_spd_dos = dos.get_element_spd_dos('Si')
# 提取s轨道
Si_s = Si_spd_dos[OrbitalType.s].densities
# 提取p轨道
Si_p = Si_spd_dos[OrbitalType.p].densities
# 重组
Si_sp_dos = pd.DataFrame({'Energy': energy, 'Si_s': Si_s[Spin.up], 'Si_p': Si_p[Spin.up]})
print(Si_sp_dos)
# 保存输出
Si_sp_dos.to_csv('Si_sp_dos.csv', index=False)
```

##### 原子总态密度

代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi

# 元素位点总态密度
# 提取指定原子Density，structure[0]表示1号原子
site_densities = dos.get_site_dos(dos.structure[0]).densities
# 重组
site_dos = pd.DataFrame({'Energy': energy, 'Density': site_densities[Spin.up]})
print(site_dos)
# 保存输出
site_dos.to_csv('site_dos.csv', index=False)
```

##### 原子分态密度

代码如下：

```python
from pymatgen.io.vasp import Vasprun
from pymatgen.core.periodic_table import Element
from pymatgen.electronic_structure.core import OrbitalType, Spin
import pandas as pd

vasprun = Vasprun('./dos/vasprun.xml')
dos = vasprun.complete_dos
energy = dos.energies - dos.efermi

# 元素位点分态密度
# 获取1号原子所有轨道态密度
site_spd_dos = dos.get_site_spd_dos(dos.structure[0])
# 提取s轨道
site_s_densities = site_spd_dos[OrbitalType.s].densities
# 提取p轨道
site_p_densities = site_spd_dos[OrbitalType.p].densities
# 重组
site_sp_dos = pd.DataFrame({'Energy': energy, 's': site_s_densities[Spin.up], 'p': site_p_densities[Spin.up]})
print(site_sp_dos)
# 保存输出
site_sp_dos.to_csv('site_sp_dos.csv', index=False)
```

##### 总结

上述介绍了从体系整体，元素种类到具体单个原子的总态密度和分态密度的处理方法，每种方法的使用都比较类似，跟着老司机的代码实操一遍应该问题不大。总的来说，就是根据不同的需求，使用不同的方法：

- **体系总态密度**：使用 `dos.densities`

- **元素总态密度**：使用 `dos.get_element_dos()`

- **体系分态密度**：使用 `dos.get_spd_dos()`

- **元素分态密度**：使用 `dos.get_element_spd_dos()`

- **原子总态密度**：使用 `dos.get_site_dos()`

- **原子分态密度**：使用 `dos.get_site_spd_dos()`

***¡Muchas gracias!***
