---
title: Chap.1  重生之计算小白硬学VASP —— 一键生成输入文件
date: 2024-06-04 00:00:00
author: 炫酷老司机
tags:
  - VASP
  - pymatgen
categories:
  - Learn VASP from pymatgen
sticky: 4
copyright: 欢迎个人转载、使用、转贴等，但请保留原作者信息！
comment: true
cover: /images/wallhaven-gpkd77_3840x2160.png
---

***¡Hola a todos!***

本章进入VASP计算实操，首先介绍VASP的计算运行流程是怎样的，`pymatgen`的使用在流程中相当于什么呢：

下图是简单的计算流程介绍：

````mermaid
flowchart LR
	input{{input_file}} 
	vasp(vasp)
	converged(converged)
	raw_data(raw_data)
	
	subgraph prep_input
	1([INCAR])-->input
	2([KPOINTS])-->input
	3([POSCAR])-->input
	4([POTCAR])-->input
	5([sub_job.script])-->input
	end
	input-->vasp
	subgraph vasp_black_box
	vasp---relax([relax])-->converged
	vasp---scf_calc([scf_calc])-->converged
	vasp---nonscf_calc([nonscf_calc])-->converged
	vasp---ect([......])-->converged
	end
	converged-->raw_data
	subgraph post_processing
	raw_data-->table([table])
	raw_data-->gra([graph])
	raw_data-->animation([animation])
	
	end
````

- 第一步为输入文件的准备，除了准备`INCAR KPOINTS POSCAR POTCAR`这四个VASP必需的输入文件之外，通常使用超算集群的用户还需要准备任务提交脚本`sub_job.script`；



- 第二步为提交运行VASP进行计算，这里可以把软件运算过程当成一个黑匣子，作为使用者的角度，我们无需去了解它是如何迭代计算，只需要更加不同的计算内容，我们能够根据不同的依据来判断计算是否完成，也就是是否收敛；



- 第三步为数据后处理过程，VASP输出的输出一般都不能直接转化成为可用的结果，需要我们进一步去分析电荷文件、波函数文件、`OUTCAR`文件等，从其中得到我们最终可以绘图`or`制表的数据。

从上述流程来看，一个完整的计算流程到最后出图，如果不使用辅助工具，我们既要保证输入文件的正确性，又要哼哧哼哧的来进行数据后处理画图。如果计算的体系不多，只算几个结构，可能也就花费几小时半天的时间坐在电脑面前也还可以接受，但是如果计算量倍增，尤其是做高通量计算，甚至要算几十个结构的时候，完全靠传统的方式人肯定很难吃得消，而且很可能由于精神力消耗导致出错。

所以，这就是为什么介绍`pymatgen`的初衷了，学会使用之后，可以在第一步的流程中完全解放出来人力时间；以及在数据后处理的时候批量生成可供绘图的源数据，配合上`matplotlib`也可以实现自动出图。



废话少说，正式开始！！



### 自动生成结构弛豫（优化）输入文件



