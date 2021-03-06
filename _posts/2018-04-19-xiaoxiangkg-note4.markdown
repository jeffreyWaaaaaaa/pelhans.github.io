---
layout:     post
title:      "知识图谱入门 (四)" 
subtitle:   "知识挖掘"
date:       2018-04-19 00:15:18
author:     "Pelhans"
header-img: "img/post-bg-universe.jpg"
header-mask: 0.3 
catalog:    true
tags:
    - Knowledge Graph
---


<span id="busuanzi_container_page_pv">
  本文总阅读量<span id="busuanzi_value_page_pv"></span>次
</span>

> 本节介绍了知识挖掘的相关技术，包含实体链接与消歧，知识规则挖掘，知识图谱表示学习。

* TOC
{:toc}

#  知识挖掘

知识挖掘是指从数据中获取实体及新的实体链接和新的关联规则等信息。主要的技术包含实体的链接与消歧、知识规则挖掘、知识图谱表示学习等。其中实体链接与消歧为知识的内容挖掘，知识规则挖掘属于结构挖掘，表示学习则是将知识图谱映射到向量空间而后进行挖掘。

## 实体消歧与链接

![](/img/in-post/xiaoxiangkg_note3/xiaoxiangkg_note3_5.png)

实体链接的流程如上图所示，这张图在前一章出现过，那里对流程进行了简要说明。此处对该技术做进一步的说明。

### 示例一: 基于生成模型的 entity-mention 模型

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_1.png)

该模型的流程如上图所示，文字表述为: 我们有两个句子，其中的实体分别为 Jordan(左)和 Michael Jordan(右)，我们称之为Mention。我们想要判断这两个Jordan指的到底是篮球大神还是ML大神？ 这个问题可以用公式表述为:

$$ e = \arg\max\limits_{e} \frac{P(m, e)}{P(m)} $$

等价于:

$$ e = \arg\max\limits_{e} P(m ,e) = \arg\max\limits_{e} P(e) P(s|e)P(c|e) $$

其中P(e)表示该实体的活跃度，P(s|e) 来自前面流程图中的实体引用表,它表示s作为实体的毛文本出现的概率，s表示名字。P(c|e )表示的是翻译概率(?)。

简单来说就是根据mention所处的句子和上下文来判断该mention是某一实体的概率。

### 示例二: 构建实体关联图

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_2.png)

实体关联图由3个部分组成：
* 每个顶点 $$Vi = <mi, ei> $$ 由mention-entity构成。    
* 每个顶点得分：代表实体指称mi的目标实体为ei概率可能性大小。    
* 每条边的权重：代表语义关系计算值，表明顶点Vi和Vj的关联程度。

其示例如上图所示，其流程包括：顶点的得分初始化方法、边权初始化方法和基于图的标签传播算法。

#### 顶点的初始化

* 若顶点V实体不存在歧义，则顶点得分设置为1；    
* 若顶点中mention和entity 满足
$$p(e|m)\le 0.95$$, 则顶点得分也设置为1.    
* 其余顶点的得分设置为 
$$p(e|m)$$;

#### 边的初始化 : 深度语义关系模型

其大体流程如下图所示：

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_3.png)

其中E 表示实体， R表示关系， ET表示实体类型，D表示词。它做的是将这些东西映射到非常稀疏的空间内，而后通过深度学习进行特征提取和标注，最终给出每对实体见的分值。

#### 基于图的标签传播算法

初始时，数据中的标签如左侧表格所示：

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_4.png)

其中标签数据为无歧义的entity-mention，基于此数据，我们采用基于图的标签传播算法，先构造一个相似度矩阵，而后采用图的regulartion，直到最终标签确定。有点类似于协同消歧的作用。

### 示例三：基于知识库

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_5.png)

其流程图如上图所示，    
* 首先我们有一个知识库，我们经由深度学习算法，将RDF三元组转化为实体向量。    
* 有了向量之后，我们就可以计算实体向量间的相似度。    
* 基于相似度构建实体关联图。    
* 基于PageRank算法更新实体关联图。

下面对其中重要的部分做讲解。

#### 基于向量相似度的实体关联图的构建

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_6.png)

上图给出RDF三元组如何生成实体向量并计算实体向量间的相似度。对于相似度的度量可以采用cos函数等方式。即：

$$ SM(e^{i}_{a}, e_{b}^{j} ) = cos (v(e^{i}_{a}), v(e_{b}^{j})) $$

由此我们定义候选实体间的转化概率：

$$ ETP(e^{i}_{a}, e_{b}^{j} ) = \frac{SM(e^{i}_{a}, e_{b}^{j} )}{\Sigma_{k\eta(v, v_{i})} SM(e^{i}_{a},k)} $$

其中分母为该顶点的出度向量相似度求和。

#### 基于PageRank得分

首先根据PageRank算法计算未消歧实体指称实体的得分，取得分最高的未消歧实体。而后删除其他候选实体及相关的边，更新图中的边权值。

其流程如下图所示：

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_7.png)


## 知识图谱表示学习(TranSE)

表示学习即将三元组即各种关系映射成向量进行处理。

![](/img/in-post/xiaoxiangkg_note4/xiaoxiangkg_note4_8.png)

一个典型的系统如上图所示，它将结构知识、文本知识和视觉知识结合进行输入得到一个综合的向量，而后将其与用户的行为向量进行匹配来完成推荐功能。

## PRA 与 TranSE的结合

表示学习无法处理一对多、多对一和多对多问题，同事可解释性不强。PRA难以处理稀疏关系、路径特征提取效率不高。因此两类方法之间存在互补性。因此提出了路径的表示学习等方法。

# Ref
 
[王昊奋知识图谱教程](http://www.chinahadoop.cn/course/1048)
