---
title: Chap.3  重生之计算小白硬学VASP —— 学会使用Materials Project
date: 2024-05-30 16:00:00 
author: 炫酷老司机
tags:
  - VASP
  - pymatgen
  - Materials Project
categories:
  - Learn VASP from pymatgen
sticky: 98
copyright: 欢迎个人转载、使用、转贴等，但请保留原作者信息！
# cover: "图片链接"
---

***¡Hola a todos!***

很遗憾的告诉各位，本章还没有开始涉及到VASP的计算。

**Why????**

根据老司机以往的使用经验，做计算最关键的不是了解各种输入参数的意义，不是高超的编程能力，不是各种计算流程信手拈来，而是一个靠谱的结构。往往一个靠谱的结构，你的计算就已经成功了90%，这么说一点也不为过。一个“垃圾”结构，你往往算了几天、一周、一个月，得到的结果都不见得令人满意。

如何得到一个靠谱的结构，或者说到哪里去找一个靠谱的结构，这里不得不介绍一下本期嘉宾——[Materials Project](https://next-gen.materialsproject.org/)网站（下面简称MP）。截至到今天，MP提供了超过15万个无机化合物的结构信息以及性质，其中有实验已经发现的材料，也有纯计算预测的新材料。每一种化合物的计算源文件都可以免费下载本地，特别适合小白们自学VASP。

**当然，最重要的一点是，免费！免费！免费！Free~~~~~~~**

你只需要邮箱登录后就可以尽情使用

![Materials Project_login](/images/Learn_VASP_from_pymatgen/chap3/0_MP_login.png)

登录后进入主页，开始今天的MP学习之旅。

![MP_home](/images/Learn_VASP_from_pymatgen/chap3/1_MP_homepage.png)

## Materials Project网页端

### 材料检索

MP上面材料的检索方式有三种:

![MP_search_type](/images/Learn_VASP_from_pymatgen/chap3/3_MP_search_type.png)

- `Only Elements`: 仅包含选定元素，如输入`Si`检索出来的结构只是单质硅；
- `At least Elements`: 至少包含选定元素，如输入`Si`检索出来的结构既有单质硅，也有含`Si`的化合物：`SiO`、`SiO2`等；
- `Formula`: 输入完整的分子式，如输入`SiO2`，检索出来的结构只有`SiO2`。

这里我们用`Only Elements`检索`Si`，可以得到一系列的`Si`单质结构。

![MP_search_Si](/images/Learn_VASP_from_pymatgen/chap3/4_MP_Si.png)

这么多`Si`单质结构，到底哪个是我们想要的呢？我们可以根据`Crystal System`、`Spacegroup`等信息筛选，这里我们以第一个`Si`结构为例，了解一下MP上面提供了哪些信息。

### 页面信息

`Summary`一栏基本提供了晶体结构的常见基本信息：空间群信息、能带信息、磁性信息、实验上是否得到等。

![MP_Si_info1](/images/Learn_VASP_from_pymatgen/chap3/5_MP_Si_info1.png)

继续往下拉还可以看到具体的晶格常数信息，能带态密度图、模拟光谱信息等。

- 晶格常数信息

![MP_Si_info2](/images/Learn_VASP_from_pymatgen/chap3/6_MP_Si_info2.png)

- 能带态密度图

![MP_Si_info3](/images/Learn_VASP_from_pymatgen/chap3/7_MP_Si_info3.png)

- 模拟光谱

![MP_Si_info4](/images/Learn_VASP_from_pymatgen/chap3/8_MP_Si_info4.png)

对于自学的各位来说，自我检验计算能力的方法就是看自己算的跟别人算的结果是不是一样，可能有人会说直接找一篇有计算的文献复现一下就行了。但是，很遗憾，目前绝大多数的文献`Computational Method`部分的撰写都是比较简略的，对于新手小白来说，只阅读文字很难从里面提炼出一个与作者差不多的INCAR设置，更不用说具体计算的结构模型搭建了。

所以刚刚接触计算的小白们，老司机不建议直接从复现文献开始，而是从简单的材料，简单的计算入手，先培养计算的感觉，然后熟练计算的流程，最后再去尝试复杂的计算或者尝试复现文献。








---------------------------------------
To be continued...