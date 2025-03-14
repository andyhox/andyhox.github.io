---
title: Chap.15  计算小白硬学VASP —— 各类结构“Transformation”使用说明（二）
date: 2024-08-16 00:00:00
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

继续介绍`pymatgen`中`standard_transformations`中各类`Transformation`的使用说明。

### SubstitutionTransformation

`SubstitutionTransformation`类主要用于对结构进行某一元素替换或者部分掺杂。如何控制替换的元素活掺杂比例？则需要我们在使用时传入`species_map`参数，以`Si`为例：

- 完全替换元素

```python
from pymatgen.transformations.standard_transformations import SubstitutionTransformation
from pymatgen.io.cif import CifParser

# 初始结构
ini_Si = Cifparser('./Si.cif').parse_structures()[0]
print(f'初始结构：\n{ini_si}')
print('\n')

# 替换规则，Si全部替换为C
species_map = {'Si':'C'}
sub_trans =  SubstitutionTransformation(species_map)
new_C = sub_trans.apply_transformation(ini_Si)
print(f'全部替换为C：\n{new_C}')
```

运行代码：

```tex
初始结构：
Full Formula (Si8)
Reduced Formula: Si
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  Si    0     0     0
  1  Si    0.25  0.25  0.25
  2  Si    0     0.5   0.5
  3  Si    0.75  0.25  0.75
  4  Si    0.25  0.75  0.75
  5  Si    0.75  0.75  0.25
  6  Si    0.5   0     0.5
  7  Si    0.5   0.5   0


全部替换为C：
Full Formula (C8)
Reduced Formula: C
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  C     0     0     0
  1  C     0.25  0.25  0.25
  2  C     0     0.5   0.5
  3  C     0.75  0.25  0.75
  4  C     0.25  0.75  0.75
  5  C     0.75  0.75  0.25
  6  C     0.5   0     0.5
  7  C     0.5   0.5   0
```

- 部分掺杂

```python
from pymatgen.transformations.standard_transformations import SubstitutionTransformation
from pymatgen.io.cif import CifParser

# 初始结构
ini_Si = Cifparser('./Si.cif').parse_structures()[0]
print(f'初始结构：\n{ini_si}')
print('\n')

# Si位点掺杂50%C
species_map = {'Si':{'Si':0.9, 'C':0.1}}
sub_trans =  SubstitutionTransformation(species_map)
new_Si = sub_trans.apply_transformation(ini_Si)
print(f'掺杂50%C：\n{new_Si}')
```

运行代码：

```tex
初始结构：
Full Formula (Si8)
Reduced Formula: Si
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP       a     b     c
---  ----  ----  ----  ----
  0  Si    0     0     0
  1  Si    0.25  0.25  0.25
  2  Si    0     0.5   0.5
  3  Si    0.75  0.25  0.75
  4  Si    0.25  0.75  0.75
  5  Si    0.75  0.75  0.25
  6  Si    0.5   0     0.5
  7  Si    0.5   0.5   0


掺杂50%C：
Full Formula (Si4 C4)
Reduced Formula: SiC
abc   :   5.443702   5.443702   5.443702
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (8)
  #  SP                a     b     c
---  -------------  ----  ----  ----
  0  Si:0.5, C:0.5  0     0     0
  1  Si:0.5, C:0.5  0.25  0.25  0.25
  2  Si:0.5, C:0.5  0     0.5   0.5
  3  Si:0.5, C:0.5  0.75  0.25  0.75
  4  Si:0.5, C:0.5  0.25  0.75  0.75
  5  Si:0.5, C:0.5  0.75  0.75  0.25
  6  Si:0.5, C:0.5  0.5   0     0.5
  7  Si:0.5, C:0.5  0.5   0.5   0
```

**Note**：`SubstitutionTransformation`替换只区别元素符号，不区分位点。如果没有特殊的区分，`SubstitutionTransformation`考虑的体系中所有的同类原子。

如果只需要替换指定`wyckoff site`位点的元素，能实现吗？

以`LiFePO4`为例，首先需要保证`LiFePO4`具有完整的对称性，这样才会识别出不同的位点，如果`cif`结构不包含对称的性，打印出的结构信息只是所有原子的坐标：

```python
from pymatgen.io.cif import CifParser

ini_LFP = CifParser('./LiFePO4.cif').parse_structures()[0]
print(f'初始结构：\n{ini_LFP}')
```

输出：

```tex
初始结构：
Full Formula (Li4 Fe4 P4 O16)
Reduced Formula: LiFePO4
abc   :  10.236196   5.970755   4.654917
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (28)
  #  SP           a         b         c
---  ----  --------  --------  --------
  0  Li+   0         0         0
  1  Li+   0.5       0         0.5
  2  Li+   0.5       0.5       0.5
  3  Li+   0         0.5       0
  4  Fe2+  0.218849  0.25      0.529866
  5  Fe2+  0.781151  0.75      0.470134
  6  Fe2+  0.281151  0.75      0.029866
  7  Fe2+  0.718849  0.25      0.970134
  8  P5+   0.093866  0.75      0.581377
  9  P5+   0.906134  0.25      0.418623
 10  P5+   0.406134  0.25      0.081377
 11  P5+   0.593866  0.75      0.918623
 12  O2-   0.165845  0.545558  0.713735
 13  O2-   0.834155  0.454442  0.286265
 14  O2-   0.334155  0.454442  0.213735
 15  O2-   0.665845  0.545558  0.786265
 16  O2-   0.665845  0.954442  0.786265
 17  O2-   0.334155  0.045558  0.213735
 18  O2-   0.834155  0.045558  0.286265
 19  O2-   0.165845  0.954442  0.713735
 20  O2-   0.044309  0.25      0.290136
 21  O2-   0.955691  0.75      0.709864
 22  O2-   0.455691  0.75      0.790136
 23  O2-   0.544309  0.25      0.209864
 24  O2-   0.094231  0.75      0.255213
 25  O2-   0.905769  0.25      0.744787
 26  O2-   0.405769  0.25      0.755213
 27  O2-   0.594231  0.75      0.244787
```

这样是不利于进行掺杂操作的。虽然在`parse_structures()[0]`中可以添加`symmetrized=True`来寻找对称性，但是如果初始`cif`文件中信息不全，是没办法得到正确的结果的：

```python
from pymatgen.io.cif import CifParser

rough_symm_LFP = CifParser('./LiFePO4.cif').parse_structures(symmetrized=True)[0]
print(f'粗略对称性：\n{rough_symm_LFP}')
```

输出：

```tex
粗略对称性：
SymmetrizedStructure
Full Formula (Li4 Fe4 P4 O16)
Reduced Formula: LiFePO4
Spacegroup: Not Parsed (-1)
abc   :  10.236196   5.970755   4.654917
angles:  90.000000  90.000000  90.000000
Sites (28)
  #  SP           a         b         c  Wyckoff
---  ----  --------  --------  --------  ------------
  0  O2-   0.165845  0.545558  0.713735  16Not Parsed
  1  P5+   0.093866  0.75      0.581377  4Not Parsed
  2  Fe2+  0.218849  0.25      0.529866  4Not Parsed
  3  Li+   0         0         0         4Not Parsed
```

该方法无法正确识别`Wyckoff Site`，可能原因就是`cif`结构中不包含空间群等信息，所以最保险的办法是调用`SpacegroupAnalyzer`来得到包含对称性的结构：

```python
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer
from pymatgen.io.cif import CifParser

ini_LFP = CifParser('./LiFePO4.cif').parse_structures()[0]
symm_LFP = SpacegroupAnalyzer(ini_LFP).get_symmetrized_structure()

print(f'添加对称性后：\n{symm_LFP}')
```

输出：

```tex
添加对称性后：
SymmetrizedStructure
Full Formula (Li4 Fe4 P4 O16)
Reduced Formula: LiFePO4
Spacegroup: Pnma (62)
abc   :  10.236196   5.970755   4.654917
angles:  90.000000  90.000000  90.000000
Sites (28)
  #  SP           a         b         c  Wyckoff
---  ----  --------  --------  --------  ---------
  0  Li+   0         0         0         4a
  1  Fe2+  0.218849  0.25      0.529866  4c
  2  P5+   0.093866  0.75      0.581377  4c
  3  O2-   0.165845  0.545558  0.713735  8d
  4  O2-   0.044309  0.25      0.290136  4c
  5  O2-   0.094231  0.75      0.255213  4c
```

此时可正确显示`O`元素有三种位点，两个`4c`位，一个`8d`位。

在此基础上，如果我们仅考虑在`8d`位`O`位掺杂`S`，能不能实现呢？？

`pymatgen`目前版本中没有能够直接针对`Wyckoff Site`来进行`Partial Substitutions`的类，所以下面是我想出的一个解决办法：

- **找到所有`8d`位的`O`原子的`indices`**

```python
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer

ini_LFP = CifParser('./LiFePO4.cif').parse_structures()[0]
# 获取结构中所有的wyckoff sites
wyckoff_sites = SpacegroupAnalyzer(ini_LFP).get_symmetry_dataset().wyckoffs
# 打印
print(wyckoff_sites)
```

输出：

```tex
wyckoff_sites: ['a', 'a', 'a', 'a', 'c', 'c', 'c', 'c', 'c', 'c', 'c', 'c', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'd', 'c', 'c', 'c', 'c', 'c', 'c', 'c', 'c']
```

**Note**：这里打印出的只有`wyckoff site`名称中的字母部分，并没有数字部分，即没有显示对称位置的数量，但是不妨碍我们去获取`8d`的`indices`

- **获取`8d`位`O`的所有`indices`**

```python
......
oxygen_d_indices = [i for i, site in enumerate(wyckoff_sites) if site == 'd']
print(f'oxygen_d_indices: {oxygen_d_indices}')
```

输出：

```tex
oxygen_d_indices: [12, 13, 14, 15, 16, 17, 18, 19]
```

这里`d`的`wyckoff site`是唯一的，所以我们只需要对上一步打印的结果进行统计即可，但是如果我们想要对`4c`位的`O`进行掺杂，这里就需要增加限定条件了：

```python
......
# 4c O位掺杂
oxygen_c_indices = [i for i, site in enumerate(wyckoff_sites) if site == 'c' and ini_LFP.sites[i].specie.symbol == 'O']
print(f'oxygen_c_indices: {oxygen_c_indices}')
```

输出：

```tex
oxygen_c_indices: [20, 21, 22, 23, 24, 25, 26, 27]
```

同理，如果相对`4c`位的`Fe`或`P`掺杂，只需要修改对应限制条件为```ini_LFP.sites[i].specie.symbol == 'target_element'```即可。

回到正题，筛选出了`indices`，然后进行部分掺杂替换：

- **替换位点**

```python
from pymatgen.transformations.site_transformations import ReplaceSiteSpeciesTransformation

final_LFP = ini_LFP.copy()
for i in oxygen_d_indices:
    indices_species_map = {i:{'O2-':0.9, 'S2-':0.1}}
    trans = ReplaceSiteSpeciesTransformation(indices_species_map)
    final_LFP = trans.apply_transformation(final_LFP)

print(f'最终结构：\n{final_LFP}')
```

这里我们需要导入另外一个类的方法`ReplaceSiteSpeciesTransformation`，然后用一个循环对每一个目标位点进行替换，最终结构为：

```tex
最终结构：
Full Formula (Li4 Fe4 P4 S0.8 O15.2)
Reduced Formula: Li4Fe4P4S0.8O15.2
abc   :  10.236196   5.970755   4.654917
angles:  90.000000  90.000000  90.000000
pbc   :       True       True       True
Sites (28)
  #  SP                       a         b         c
---  ----------------  --------  --------  --------
  0  Li+               0         0         0
  1  Li+               0.5       0         0.5
  2  Li+               0.5       0.5       0.5
  3  Li+               0         0.5       0
  4  Fe2+              0.218849  0.25      0.529866
  5  Fe2+              0.781151  0.75      0.470134
  6  Fe2+              0.281151  0.75      0.029866
  7  Fe2+              0.718849  0.25      0.970134
  8  P5+               0.093866  0.75      0.581377
  9  P5+               0.906134  0.25      0.418623
 10  P5+               0.406134  0.25      0.081377
 11  P5+               0.593866  0.75      0.918623
 12  S2-:0.1, O2-:0.9  0.165845  0.545558  0.713735
 13  S2-:0.1, O2-:0.9  0.834155  0.454442  0.286265
 14  S2-:0.1, O2-:0.9  0.334155  0.454442  0.213735
 15  S2-:0.1, O2-:0.9  0.665845  0.545558  0.786265
 16  S2-:0.1, O2-:0.9  0.665845  0.954442  0.786265
 17  S2-:0.1, O2-:0.9  0.334155  0.045558  0.213735
 18  S2-:0.1, O2-:0.9  0.834155  0.045558  0.286265
 19  S2-:0.1, O2-:0.9  0.165845  0.954442  0.713735
 20  O2-               0.044309  0.25      0.290136
 21  O2-               0.955691  0.75      0.709864
 22  O2-               0.455691  0.75      0.790136
 23  O2-               0.544309  0.25      0.209864
 24  O2-               0.094231  0.75      0.255213
 25  O2-               0.905769  0.25      0.744787
 26  O2-               0.405769  0.25      0.755213
 27  O2-               0.594231  0.75      0.244787
```

这样我们就实现了只对特定的`wyckoff site`进行替换操作。

完整代码如下：

```python
# 导入所需的pymatgen模块
from pymatgen.transformations.site_transformations import ReplaceSiteSpeciesTransformation
from pymatgen.symmetry.analyzer import SpacegroupAnalyzer
from pymatgen.io.cif import CifParser

# 从CIF文件中解析初始LiFePO4结构
ini_LFP = CifParser('./LiFePO4.cif').parse_structures()[0]
rough_symm_LFP = CifParser('./LiFePO4.cif').parse_structures(symmetrized=True)[0]
# 获取初始结构的对称化信息
symm_LFP = SpacegroupAnalyzer(ini_LFP).get_symmetrized_structure()
print(f'初始结构：\n{ini_LFP}\n')
print(f'粗略对称性：\n{rough_symm_LFP}\n')
print(f'添加对称性后：\n{symm_LFP}\n')

# 获取wyckoff位点信息
wyckoff_sites = SpacegroupAnalyzer(ini_LFP).get_symmetry_dataset().wyckoffs
print(f'wyckoff_sites: {wyckoff_sites}')

# 获取4d位O原子索引
oxygen_d_indices = [i for i, site in enumerate(wyckoff_sites) if site == 'd']
print(f'oxygen_d_indices: {oxygen_d_indices}')

# 进行替换
final_LFP = ini_LFP.copy()
for i in oxygen_d_indices:
    indices_species_map = {i:{'O2-':0.9, 'S2-':0.1}}
    trans = ReplaceSiteSpeciesTransformation(indices_species_map)
    final_LFP = trans.apply_transformation(final_LFP)

print(f'最终结构：\n{final_LFP}')
```

当然，这里我们是直接用代码来给结构增加`partial substitution`信息；当然你也可以直接在`cif`结构里直接修改`occupation`占据也是可以的。

本章花这么大篇幅介绍`partial substitution`，是为了解决材料研究中掺杂结构的建模问题。第一步是构建出符合预期，符合实验条件的`partial substitution`的结构，这里也叫做分数占据结构；下一步就是如何确切的构建出不含分数占据的结构，即`ordered structure`，可以直接用于`VASP`直接计算。

***¡Muchas gracias!***

