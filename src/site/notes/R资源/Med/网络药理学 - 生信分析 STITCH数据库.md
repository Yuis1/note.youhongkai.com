---
{"dg-publish":true,"permalink":"/R资源/Med/网络药理学 - 生信分析 STITCH数据库/","noteIcon":"","created":"2022-10-14T14:46:17.347+08:00","updated":"2024-04-24T00:47:16.769+08:00"}
---


检索 Pubmed 不难发现近 5 年，网络药理学在传统中草药机制研究中是如火如荼如日中天。与传统意义上针对单一靶点的单一药物治疗策略不同，作为天然的 “组合式” 治疗策略，中草药在网略药理学研究中占有天然优势。前面几期给大家伙儿介绍了不少中草药研究神器，今天我们聚焦一个网络药理学研究面临的共性问题，**即如何查找药物的靶基因！**蛋白互作领域大名鼎鼎的 STRING 数据库想必大家不会陌生，那么其兄弟数据库 STITCH 呢？提供化合物与靶基因互作关系信息，一起来康康吧~！  

点击网址 http://stitch.embl.de/ 进入 STITCH 数据库主页，该数据库最早于 2008 年发布，至今更新至 5.0 版本，数据来源有，DurgBank、GLIDA、Matador、TTD 和 CTD 等人工注释数据库，KEGG、PID、Reactome 和 BioCyc 等通路数据库，以及 ChEMBL、PDSPKi 和 PDB 等实验结果数据库。当前版本收录来自 2031 个物种的 43000 个化合物和 9643763 个蛋白之间相互作用信息，并与 STRING 数据库共享蛋白交互数据。STITCH 数据库中的每个化合物与蛋白互作关系都对应一个分值，指示其相互作用概率或结合亲和力，并提供与目标药物相似的化合物及其相似分数。

![](/img/user/Z-attach/f88e28faa5e8a1413f567806aade7a517311a1c7.jpg@942w_438h_progressive.webp)

点击 Help，点击侧边栏 Getting started，可见该数据详细使用指南；点击 Database，展示该数据库各种参数 / 名词含义解释。

![](/img/user/Z-attach/c67edd9baae90f58f89e35ddef25270ec58be06f.jpg@942w_437h_progressive.webp)

![](/img/user/Z-attach/22e1b87ce11368b8864ebe435d2ed05f75abd164.jpg@942w_339h_progressive.webp)

**1、Item by name**

进入数据库，点击 Item by name 进行单个药物或基因检索，检索框可输入化合物或基因名称，物种可选择自动识别，以检索丁酸盐（butyrate）为例，点击 Search，勾选 butyrate，点击 Continue 得到检索结果。首先映入眼帘的是 butyrate 与基因互作关系网络图，下方是各个功能选项。

![](/img/user/Z-attach/3e7145b1a6f2eb60786db0e5fb89d5aef911c8ca.jpg@942w_353h_progressive.webp)

![](/img/user/Z-attach/dd56901c718e88c27f20d117d076ad760c9743da.jpg@942w_285h_progressive.webp)

![](/img/user/Z-attach/b958c15e558dd63c05b3dda2023153e6077c731c.jpg@942w_551h_progressive.webp)

点击 Legend 查看网络图注释：

（1）Notes 目录下展示节点含义：每个节点表示单个编码基因位产生的所有蛋白质；小节点代表蛋白三维结构未知，大节点展示蛋白三维结构或预测的三维结构；彩色节点表示一级交互点，白色节点表示二级交互点。

（2）Edges 目录下展示连线含义：连续粗细代表低、中、高和最高 4 个程度的交互评分。

（3）Your Input 目录下提供输入的 butyrate 相关描述。

（4）Predicted Functional Partners 目录下提供每个蛋白质功能描述，按交互评分降序排列。

（5）Your Current Organism 目录下提供当前数据所属的物种。

![](/img/user/Z-attach/aafed2679bb5d1688672841281663274ee9924de.jpg@942w_819h_progressive.webp)

点击其中某个节点，如 CKD2，查看详细注释信息、结构和功能域信息，Actions 中提供 5 个功能选项，分别是以该因为核心重构网络、添加其为输入基因扩充网络、展示该基因蛋白序列和查找其他物种同源蛋白，以及该基因在 GeneCards 数据库中通路、功能和来源信息。

![](/img/user/Z-attach/b124db45475aacd672fc1c68ef57f2cc469f5721.jpg@942w_570h_progressive.webp)

点击 Setting 进行个性化设置：

（1）Basic Setting 目录下，可选择显示的连接方式、连线代表的含义、连线数据来源、展示的最小交互评分和最大交互数，设置完成后点击 UPDATE 刷新。

（2）Advanced Setting 目录下，可选择是否显示组织特异性交互，是否显示蛋白三维结构。

![](/img/user/Z-attach/486dbd2eb42783361a602f04857590f1b847b296.jpg@942w_623h_progressive.webp)

![](/img/user/Z-attach/4965dd554d1c11663b514f48be25ee909a9708be.jpg@942w_534h_progressive.webp)

点击 Analysis 展示 GO/KEGG 富集分析结果，按 FDR 值从小到大排列，页面底部提供下载链接。

![](/img/user/Z-attach/3afe1e36751c564afe41c86fdd62cf47c08d0187.jpg@942w_809h_progressive.webp)

点击 Tables/Exports 查看上述交互网络结果表格形式，并提供位图、矢量图和表格数据下载，以及蛋白序列和注释信息下载。

![](/img/user/Z-attach/d834dfe3a0b04bfd0eac63e41570280661850f95.jpg@942w_639h_progressive.webp)

点击 More 在当前基础上显示更多节点，点击 Less 在当前基础上显示更少节点。

![](/img/user/Z-attach/2927430fe2b68408799f7c282e4e797442d916d5.jpg@942w_537h_progressive.webp)

上述信息为 Viewers 目录下 Network 结果。点击 Experiments，展示实验支持的 butyrate 与基因交互信息；点击 Databases，展示各种数据库来源的 butyrate 与基因交互信息；点击 Textmining，展示文献挖掘中 butyrate 与基因交互信息；点击 Coexpression，展示蛋白共表达信息。上述结果的下方功能选项基本同前。

![](/img/user/Z-attach/65fa2b660c7ac40dd7fbfc933dd46ed0445f6fa1.jpg@942w_281h_progressive.webp)

![](/img/user/Z-attach/b6f46cdb8bb6a0e93f356ca43e8cf0d3178631fc.jpg@942w_426h_progressive.webp)

![](/img/user/Z-attach/dd4ec6e432fa9cbb80030feaaaa3ed7c8d65438a.jpg@942w_452h_progressive.webp)

![](/img/user/Z-attach/1c93a3c08d03921c6e445ea0be4a44f6d4091fc8.jpg@942w_551h_progressive.webp)

![](/img/user/Z-attach/54e553fc6566a59fb111bc9ef9d0bbeebb072e16.jpg@942w_485h_progressive.webp)

接下来以检索 SOX2 基因为例，点击 Search，勾选 Homo sapiens，点击 Continue 得到检索结果，展示 SOX2 基因相关蛋白交互网络，该结果与 STRING 数据库类似，不再赘述。同样地，Protein sequence(s) 检索功能与 STRING 数据库结果类似，不再赘述。

![](/img/user/Z-attach/4e6e6c905b0344155e8e3b96bd584b7b00f3bbfc.jpg@942w_314h_progressive.webp)

**2、Multiple names**

进入数据库，点击 Multiple names 进行药物或基因列表检索，检索框可输入化合物或基因名称列表，物种选择自动识别，以检索乙酸盐、丙酸盐和丁酸盐为例，点击 Search，勾选 Homo sapiens，点击 Continue 得到检索结果。映入眼帘的是 3 种化合物与基因互作关系网络图，下方各个功能选项与前文类似。

**3、Chemical structure(s)**

进入数据库，点击 Chemical structure(s) 以药物化学结构式检索，检索框输入化合物 SMILES 结构式，以丁酸的 CCCC(O)=O 为例，自动识别物种，点击 Search，勾选 butyric acid sodium salt，点击 Continue 得到检索结果，与前文类似。

Tips：SMILES（Simplified Molecular Input Line Entry System）以单行文本表达化合物结构的线性符号，由 David Weininger 于 1986 年开始采用及 Daylight Chemical Information Systems 开发并共同创建。那么如何获得丁酸的 SMILES 结构式？

根据 SMILES 规则自行书写往往容易出错，可以借助网站完成。点击（**http://www.swissadme.ch/index.php**）进入 SwissADME 网站，以丁酸为例，在绘图面板绘制丁酸结构式，点击双箭头，右侧可显示其 SMILES 结构式。该网站还提供丁酸 ADME 信息，即药代动力学数据，机体对外源化学物吸收（absorption）、分布 (distribution)、代谢(metabolism) 和排泄 (excretion) 相关信息。

**1、CancerHSP**

点击 Database 下 CancerHSP 进入功能页面，该子数据库包含 2439 种抗癌中草药的 3575 种抗癌化合物成分的 832 个靶基因信息，提供每种化合物的分子结构和 ADME 参数，以及基于 492 种不同肿瘤细胞系的化合物抗癌活性，有助于揭示天然抗癌药物的分子机制和抗癌药物的开发。该模块使用方法比较简单，可参考侧边栏功能导航自行探索。

**2、CVDSP**

点击 Database 下 CVDSP 进入功能页面，该子数据库提供心血管疾病相关中草药及其所含化合物成份药理学信息，包含已知的 254 种心血管药物的 206 种治疗靶基因，以及 98 种心血管疾病及 268 种心血管疾病基因，有助于心血管疾病相关药物机制研究和药物研发。

**3、PreDC**

点击 Database 下 PreDC 进入功能页面，即 Predict Drug Combination，提供预测和实验验证的药物组合疗效信息。其中 951 种药物的 1,571 种已知组合信息来源于 DCDB、TTD 和文献报道数据。另有 8,686 种组合的预测结果，具有两个 P 值，P 值 1 表示成为有效组合的概率，P 值 2 表示对该组合产生不利影响的概率。该模块目前在维护中。

该数据库自 2014 年发布以来，目前引用 533 次，其中不乏高分文献。

**文献案例：PMID: 33963245，IF=3.996 分**

本文拟探讨穿心莲抗炎治疗的潜在分子机制。作者基于 TCMSP 和 ECTCM 数据库获得穿心莲 23 种生物活性化合物，以口服生物利用度（OB）≥30% 和药物相似性（DL）≥0.18 为筛选标准获得 13 中化合物，展示在 Table1。而后，从 TCMSP 和 BATMAN-TCM 数据库获得与上述 13 中生物活性化合物相关的 283 个靶基因，并基于此使用 Cytoscape3.7.2 构建 PPI 网络。之后，从 OMIM 数据库共获得 742 个炎症相关基因，与上述 283 个穿心莲潜在靶基因取交集，并基于 String 数据库 PPI 网络分析获得 5 个 Hub 基因，包括 IL-6、TNF-α、MMP-9、PTGS2 和 VEGFA。GO/KEGG 富集分析表明，上述基因与肿瘤（胰腺癌）、免疫学（T 细胞受体信号通路、B 细胞受体信号通路、Toll 样受体信号通路和 NF-κB 信号通路）和炎症过程（TNF 信号通路和 TRP 炎性介质调节）相关。此外，分子对接表明大多数穿心莲生物活性成分对预测的 Hub 基因表现出很强的结合能力。而后细胞实验证实，川新莲提取物抑制炎性介质的产生，质谱、PCR 和 Western 检测结果与预测 Hub 基因结果类似。

**Table1 复现如下**：进入 TCMSP 数据库主页，下拉菜单选择 Herb name，检索框输入穿心莲或汉语拼音，点击 Search，结果中点击拉丁文名称 Isatidis Radix 进入详情页面。

![](/img/user/Z-attach/90e4b638483393b43164c7605c835bff8d1296b0.png@942w_135h_progressive.webp)

Ingredients 部分展示穿心莲化合物成分信息，设置口服生物利用度（OB）≥30% 和药物相似性（DL）≥0.18，结果获得 24 种化合物成分。

![](/img/user/Z-attach/27a9106b4a636683350d329ac880fa7feaefd267.jpg@942w_303h_progressive.webp)

![](/img/user/Z-attach/6f831b98912613b4ace7ebb3eafb5cdd5e284f37.jpg@942w_305h_progressive.webp)

![](/img/user/Z-attach/b11036647924913dd03954769c459bfd9e789fe9.jpg@942w_462h_progressive.webp)

分别摘录 Molecular name、OB (%)、DL 和 Degree of node in network，点击化合物名称下载 Structure，整理即可得到本文 Table1。

![](/img/user/Z-attach/bc74417da26e8ed85e5b579f5a6cffc5f2a9cce9.jpg@942w_189h_progressive.webp)

![](/img/user/Z-attach/8747352c622e1e549c1cc62e12d44341091611af.jpg@942w_230h_progressive.webp)

更多文献案例可以参考：PMID 34040346、33995004 和 33986915，小伙伴么可以自行安排学习和复哈~！