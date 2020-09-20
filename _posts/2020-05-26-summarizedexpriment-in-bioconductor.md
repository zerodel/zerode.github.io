---
layout: post
title:  "bioconductor中SummarizedExpriment"
date:   2020-05-26
categories: [bioconductor]
---
- [bioconductor中SummarizedExpriment](#bioconductor中summarizedexpriment)
  - [一个场景](#一个场景)
  - [`SummarizedExpriment`类](#summarizedexpriment类)
    - [结构](#结构)
    - [SummarizedExperiment对象的建立](#summarizedexperiment对象的建立)
    - [SummarizedExperiment的子集筛选](#summarizedexperiment的子集筛选)


# bioconductor中SummarizedExpriment

[SummarizedExpriment](https://www.bioconductor.org/packages/release/bioc/html/SummarizedExperiment.html) 是 `bioconductor`中对于生物实验数据的抽象.
一个`SummarizedExpriment`对象(object)就可以完整的代表一个实验.
比如转录本的表达数据,而用它来表示的实验数据可用在后续分析中, 比如[DESeq2](https://www.bioconductor.org/packages/release/bioc/html/DESeq2.html)的差异表达分析.  

## 一个场景

从生物的数据处理过程的一些常见场景出发,我们可以更好地理解这个数据结构.

比如对某种疾病的研究中,试验人员往往首先会去采集多个样本.
比如同时采集数十个病人的样本与一些健康人的样本.

样本采集完毕之后,实验人员往往会对这些样本使用多种手段进行检测,
比如先做一个RNAseq, 然后做一个ChIP,再来一个芯片.

这样等到数据分析的时候,我们拿到的是涉及到多个感兴趣的特征(feature)在多个样本中的多来源的数据结果.

在进行后续的数据分析之前, 我们要能完整的描述整个实验. 那么需要:

1. 描述实验检测的结果, 比如各个特征(比如转录本)在各个样本中的表达水平.
2. 我们要描述研究者关心的基因组特征的信息, 比如这些转录本在基因组的坐标.
3. 我们要对这些样本分别描述表型信息, 比如是否健康,性别年龄等等.
4. 实验本身的信息也要一并进行说明.

如果我们不使用bioconductor提供的 `SummarizedExpriment` 数据结构,
而仅使用R内置的数据结构来描述上文的实验结果,也不是不可以.
那么我们需要某种list,这个list中需要有这些内容:

1. 二维矩阵来存储检测结果,多种类型的结果可以存在一个列表(list)中.
2. 来保存感兴趣分子的信息的二维数据框.
3. 保存样本信息的二维数据框.
4. 保存实验信息数据,这些信息可能较为零碎,单个实验只需要一份,所以可以用一个list.

这样的一个list可以基本满足需求, 但接下来的一些操作可能就难以应付了.
比如如果我们要从中选择部分分子的数据, 比如病人中位于y染色体的转录本的rnaseq表达情况,
那么我们就需要对所有的二维矩阵都要做一次筛选.

类似的操作会很常见,并只针对这个实验的数据集.
这种情况下这些操作或者说函数自然是和数据结合在一起比较合理.

而这已经就是一个类(class)了.

## `SummarizedExpriment`类

### 结构

正如刚刚讨论的那样, `SummarizedExpriment`类就是`bioconductor`提供对实验数据的封装.

在这个类中,保存实验结果的部分被称为 `assays`, 它是包括了多个二维数据集的列表.
这些二维数据集的行(row)代表感兴趣的特征(`feature`),比如各种分子. 而它的列(column)表示不同的样本.

保存样本信息的数据框被称为 `colData`,

而特征(feature)信息被保存在`rowData`中.

用于保存实验自身信息的列表被称为元数据 `metadata`.

在上面的情况中, 我们关注的特征往往是特征的名称, 比如转录本使用ENST开头的号码表示.
如果我们的特征使用基因组上的坐标来表示, 比如用`GenomicRanges`表示转录本对应的位置.
那我们的数据结构就要稍稍变化一下, 也是 `RangedSummarizedExperiment`,
它和SummarizedExperiment的区别就是用`rowRanges`替换了`rowData`.

### SummarizedExperiment对象的建立

理解``SummarizedExpriment``的结构之后,那么用于建立它的对象所需的代码也就好理解了.

``` R
nrows <- 200
ncols <- 6
counts <- matrix(runif(nrows * ncols, 1, 1e4), nrows)
rowRanges <- GRanges(rep(c("chr1", "chr2"), c(50, 150)),
                     IRanges(floor(runif(200, 1e5, 1e6)), width=100),
                     strand=sample(c("+", "-"), 200, TRUE),
                     feature_id=sprintf("ID%03d", 1:200))
colData <- DataFrame(Treatment=rep(c("ChIP", "Input"), 3),
                     row.names=LETTERS[1:6])

se <- SummarizedExperiment(assays=list(counts=counts),
                     rowRanges=rowRanges, colData=colData)
```

以上代码中,我们先是分别设定好了表达数据(counts)/特征信息(rowRanges)/样本信息(colData),
并通过构造函数 `SummarizedExperiment(assays=list(counts=counts),rowRanges=rowRanges, colData=colData)` 建立了一个`SummarizedExperiment`的对象 se.
之后就可以使用 `assays(se)`/`rowRanges(se)`/`colData(se)`分别提取相关的信息.

### SummarizedExperiment的子集筛选

上文中我们讨论到数据的筛选会是个很重要的功能, `SumarizedExpriment` 对象提供两种方式

1. 类似矩阵的下标选择
这种使用方式很好理解,形式上类似于二维矩阵的下标选择, 比如 `se[1,2]`, 或者 `se[, se$cell == "N61311"]`

2. 利用Ranges
这种情况适用于`RangedSummarizedExperiment`,
使用 `subsetByOverlaps`来选择与感兴趣的范围存在重叠的数据. 比如:

``` R
roi <- GRanges(seqnames="1", ranges=100000:1100000)
subsetByOverlaps(se, roi)
```