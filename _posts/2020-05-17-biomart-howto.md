---
layout: post
title:  " 如何使用biomaRt查询生物数据库"
date:   2020-05-17
categories: [bioconductor]
---

- [如何使用`biomaRt`查询生物数据库](#如何使用biomart查询生物数据库)
  - [一个比喻](#一个比喻)
  - [biomart的使用逻辑](#biomart的使用逻辑)
    - [简单的查询](#简单的查询)
    - [帮助查询的函数](#帮助查询的函数)
    - [实际使用时的编写过程](#实际使用时的编写过程)
    - [代码实例](#代码实例)


# 如何使用`biomaRt`查询生物数据库

[biomaRt](https://www.bioconductor.org/packages/release/bioc/html/biomaRt.html)是 [bioconductor](https://www.bioconductor.org/)中的一个包,提供了从网上查询基因组注释信息的程序接口. 
它将各个数据库的使用统一起来, 让用户可以从各个数据库中将感兴趣的信息下载为数据框.

## 一个比喻

为了可以更好地理解和掌握`biomaRt`的使用, 我们不妨通过类比, 假象如下的场景:

你管理着多个部门的档案, 每个部门的档案都是用二维表格形式的数据簿保存,
数据簿中每一行代表一个员工,而数据簿的列说明了员工的各种信息,比如卡号,
姓名,考勤等等信息.

你接到上面的一个任务, 要查询某个员工的考勤信息, 为了防止重名,你知道的
是名字和工号,自然你也知道这人所属部门.

查询的步骤就是如下:

1. 你先找到部门对应的考勤数据簿,
2. 接着按照名字和工号找到那个人所在的行,
3. 然后你找到考勤信息所在的列, 把其中信息抄写下来,任务完成.

## biomart的使用逻辑

### 简单的查询
biomaRt也是同样逻辑来组织数据,数据以不同数据集Dataset的形式保存在多
个数据库mart中. 用户从数据集中查询时提供的信息被归类为过滤器Filter,
这些信息的具体内容被称为值value. 而想要得到的信息被称为属性Attribute.

对应到刚才的例子, 部门就是mart,数据簿就是Dataset, 名字和工号就是
Filter, 名字和工号具体叫什么就是value, 考勤信息就是Attribute.

现在来看一下具体的代码怎么写.

首先要载入这个包

```library(biomaRt)```

然后使用`useMart`函数来得到数据库接口.

```mart <- useMart(biomart, dataset)```

用户需要指定数据库`biomart`和其中的数据集`datatset`, 而函数返回的`mart`就是查询接口

接下来的查询步骤在`biomaRt`中被整合成`getBM` 一个函数,对数据库的查询就是围
绕`getBM`函数展开.

``` result <- getBM(attributes, filters,values,mart)```

用户需要指定values, values所属的filters,以及感兴趣的attributes, 从查询
接口mart那边得到数据.

这样一个最简单的查询就完毕了.

### 帮助查询的函数

但是用户可能并不知道刚刚两个函数中需要指定的那些参数具体有哪些值. 因此,
`biomaRt`提供了一系列list* 和 search* 函数, 让用户来查询.

比如 `listAttributes(mart)` 会输出这个mart中所有可用的Attribute. 而
`searchAttributes(mart, pattern)` 则会输出这个mart中所有名称符合pattern
规则的Attribute.

### 实际使用时的编写过程

实际编写相关代码的过程会是这样.

首先使用 `listMart` 或者 `listEnsembl` 来确定使用的数据库mart. 然后用
`listDatasets(mart)` 和 `searchDatasets(mart,pattern)`来确定使用哪个数据集.

接下来,使用`listAttributes`/`searchAttributs`,  `listFilters`/`searchFilters` 来
确定`getBM`函数所需相关参数.

最后使用`getBM`得到相关的数据.

**list****  **search****这类函数往往都是在R的命令行中使用,最终的代码可能不会展
示这些内容.

`biomaRt`包中还有更多其他的功能,可以查询它提供的说明书, 这里不再赘述.

### 代码实例

以下是一些实例代码:

```R

library(biomaRt)

ensembl <- useEnsembl(biomart = "genes") searchDatasets(mart =
ensembl, pattern = "hsapiens") ensembl <- useDataset(dataset =
"hsapiens_gene_ensembl", mart = ensembl)

affyids <- c("202763_at","209310_s_at","207500_at") 

getBM(attributes = c('affy_hg_u133_plus_2', 'entrezgene_id'), 
filters ='affy_hg_u133_plus_2', 
values = affyids, 
mart = ensembl)

```

 