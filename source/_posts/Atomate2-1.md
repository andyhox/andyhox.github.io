---
title: 计算工作流——初识Atomate2
date: 2024-11-08 00:00:00
tags:
  - VASP
  - Atomate2
copyright_author: 炫酷老司机
categories:
  - Atomate2
cover: /images/toolkit.png
description: Atomte2系列教程

---

`Atomate2` ​是一款开源可用来处理复杂庞大计算任务的软件包，开发团队也是大名鼎鼎的劳伦斯伯克利国家实验室的研究人员，耳熟能详的 `pymatgen`​、`Materials Project`​、`FireWorks` ​等也是出自该团队之手。

`Atomate2` ​环境配置可参考[官网](https://materialsproject.github.io/atomate2/user/install.html)进行。

在此之前，强烈建议先了解学习一下 `pymatgen` ​的用法！老司机之前也出过相关介绍。如果在此之前有了解过或使用过 `pymatgen` ​的朋友，那么 `Atomate2` ​使用起来应该会更加容易上手，而且用习惯之后，发现 `Atomate2` ​用起来更加顺手。下面先贴一个官网上的例子：

```python
from atomate2.vasp.jobs.core import RelaxMaker
from jobflow.managers.local import run_locally
from pymatgen.core.structure import Structure

si_structure = Structure(
    lattice=[[0, 2.73, 2.73], [2.73, 0, 2.73], [2.73, 2.73, 0]],
    species=["Si", "Si"],
    coords=[[0, 0, 0], [0.25, 0.25, 0.25]],
)


relax_job = RelaxMaker().make(si_structure)


run_locally(relax_job, create_folders=True)
```

这段代码很简单，意思是优化 `Si` ​单质结构，整段代码又三部分组成：

* 定义结构

```python
si_structure = Structure(
    lattice=[[0, 2.73, 2.73], [2.73, 0, 2.73], [2.73, 2.73, 0]],
    species=["Si", "Si"],
    coords=[[0, 0, 0], [0.25, 0.25, 0.25]],
)
```

* 指定任务/工作流

```python
relax_job = RelaxMaker().make(si_structure)
```

* 运行

```python
run_locally(relax_job, create_folders=True)
```

有人会问，如果只用 `pymatgen` ​能不能实现同样的功能，答案是肯定的：

```python
from pymatgen.io.vasp.sets import MPRelaxSet
from pymatgen.core.structure import Structure

# 定义结构
si_structure = Structure(
    lattice=[[0, 2.73, 2.73], [2.73, 0, 2.73], [2.73, 2.73, 0]],
    species=["Si", "Si"],
    coords=[[0, 0, 0], [0.25, 0.25, 0.25]],
)

#生成输入文件
relax_set = MPRelaxSet(si_structure)
relax_set.write_input('./')
```

相比于 `Atomate2` ​的代码，如果仅仅从单个的计算任务来说，`pymatgen` ​差别不大，都是读结构，进行单次计算。

但是对于多步计算，如多步优化，能带态密度等计算，`pymatgen` ​该如何实现，一般来说，我们需要把每一步的计算输入相关的代码都写好，比如计算 `Si` ​的能带：

* 结构优化

```python
relax_set = MPRelaxSet(si_structure)
relax_set.write_input('relax')
```

运行 `vasp` ​计算

* 静态计算

```python
# 读取优化的结果
static_set = MPStaticSet.from_prev_calc(prev_dir='relax')
static_set.write_input('static')
```

运行 `vasp` ​计算

* 能带计算

```python
# 读取静态计算的结果
nonscf_set = MPNonScfSet.from_prev_calc(prev_dir='static')
nonscf_set.write_input('band')
```

运行 `vasp` ​计算

只使用 `pymatgen`​，从提交任务的次数来说，需要提交三次 `vasp` ​任务，而且整个计算过程是中断的，每一次计算都需要人力去手动提交，比较麻烦。而且如果你使用的是公用集群，且恰好集群用的人比较多，计算资源比较紧张，那么你很难保证上一步计算完成后，手动提交的下一步任务能马上开始计算，可能要排很久的队，这样一来，计算周期就不可控的变长了。

如果要使用 `Atomate2` ​来完成上述计算任务的话，代码会非常简单明了：

```python
from atomate2.vasp.flows.core import RelaxBandStructureMaker
from jobflow.managers.local import run_locally
from pymatgen.core import Structure

si_structure = Structure(
    lattice=[[0, 2.73, 2.73], [2.73, 0, 2.73], [2.73, 2.73, 0]],
    species=["Si", "Si"],
    coords=[[0, 0, 0], [0.25, 0.25, 0.25]],
)

bandstructure_flow = RelaxBandStructureMaker().make(si_structure)

run_locally(bandstructure_flow, create_folders=True)
```

上述代码省略了可自定义的 `INCAR`​、`KPOINTS` ​等参数设置。

有 `pymatgen` ​代码对比，相当于 `bandstructure_flow = RelaxBandStructureMaker().make(si_structure)` ​这一行代码包含了 `pymatgen` ​三个分步计算过程，从代码的结构上看更加的简洁易懂，而且最关键的是自动控制任务的进行，只需要提交一次任务即可。相当于把单个独立的子任务串联了起来，这便是“工作流”了。

后面会逐步来介绍如何使用 `Atomate2` ​来调用/自定义工作流来高效进行计算。

‍
