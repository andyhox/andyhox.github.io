---
title: 计算工作流——Atomate2自定义工作流
tags:
  - VASP
  - Atomate2
  - workflow
author: 炫酷老司机
categories:
  - Atomate2
sticky: 19
copyright: 欢迎个人转载、使用、转贴等，但请获得作者同意且注明出处！
comment: true
cover: /images/toolkit.png
excerpt: Be lazy, stay efficient, and enjoy your coffee break!
date: 2024-11-08 00:00:00
---

`Atomate2`​除了直接调用现成的工作流，我们还可以根据个人需求自定义任务个数以及工作流走向。具体操作也非常简单，只需要在`Flow`​中按照计算顺序传入任务即可，如果我们需要优化一个结构并进行静态计算：

```python
from jobflow import Flow
from atomate2.vasp.jobs.core import RelaxMaker, StaticMaker
from pymatgen.core.structure import Structure

si_structure = Structure.from_file("Si.cif")

# create a relax job
relax_job = RelaxMaker().make(structure=si_structure)

# create a static job that will use only the structure from the relaxation
static_job = StaticMaker().make(structure=relax_job.output.structure)

# create a static job that will use additional outputs from the relaxation
static_job = StaticMaker().make(
    structure=relax_job.output.structure, prev_dir=relax_job.output.dir_name
)

# create a flow including the two jobs and set the output to be that of the static
my_flow = Flow([relax_job, static_job], output=static_job.output)
```

上面是官网的例子，通过`Flow`​方法把两个任务按照顺序组合起来，既可以实现结构优化完自动开始静态计算。

此外，根据实际计算中，我们是否需要续算，或者读取上一步的计算结果，可以在`make()`​方法中定义：

* `structure`​：读取上一步计算的优化结构的语法为`prev_job.output.structure`​
* `prev_dir`​：读取上一步计算的文件夹，主要是可用于继承`INCAR`​设置，或复制读取`CHGCAR`​和`WAVECAR`​文件。

适用的用法即静态计算中读取弛豫计算的结构，非自洽计算中读取静态计算的`CHGCAR`​等。

因此，上述的`my_flow`​根据需求可以继续增加任务，如继续计算态密度：

```python
......
dos_job = NonSCFMaker().make(structure=static_job.output.structure, prev_dir=static_job.output.dir_name)

my_flow = Flow([relax_job, static_job, dos_job], output=dos_job.output)
```

至此，`Atomate2`​的基本用法就大致介绍完了~~~~~~~~
