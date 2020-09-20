---
layout: post
title:  "如何使用BSgenome来forge一个基因组"
date:   2020-09-19
categories: bioconductor
---

- [如何使用BSgenome来forge一个基因组](#如何使用bsgenome来forge一个基因组)
  - [用来存储基因组序列的BSgenome](#用来存储基因组序列的bsgenome)
  - [如何forge一个BSgenome包](#如何forge一个bsgenome包)
    - [准备基因组序列文件](#准备基因组序列文件)
    - [写seed文件](#写seed文件)
    - [生成代码包并编译安装](#生成代码包并编译安装)
      - [在R中生成代码包](#在r中生成代码包)
      - [命令行中编译生成的R代码包](#命令行中编译生成的r代码包)

# 如何使用BSgenome来forge一个基因组

本文将简要说明R中用来保存基因组序列的`BSgenome`包,
并介绍如何使用它的`forge`功能来把非模式生物的基因组打包.以方便他人使用.

## 用来存储基因组序列的BSgenome

使用R语言来处理生物数据的一个必要步骤,
就是把实验得到的生物数据转换成R语言能够理解的形式.

为此,来自于`Bioconductor`项目的`Biostrings`包提供了处理基因序列的功能.
它可以用来读写以fasta格式存储的DNA/蛋白质的序列.

用户通过它的一系列方便实用的函数,可以和处理普通字符串一样查询或者修改序列信息,
并能直接在R中完成序列的比对之类的生物信息学操作.

`BSgenome`包则是`Biostrings`包的进一步扩展,是对一个物种完整基因组的包装.
比如 `BSgenome.Hsapiens.UCSC.hg38` 就是hg38版本的人类(Homo Sapiens)的基因组序列.

`Bioconductor`已经提供常见模式生物的BSgenome包, 比如人/小鼠/酵母.用户可以直接下载使用.

比如以下代码就可以把上述的人类基因组序列包安装好.

```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install("BSgenome.Hsapiens.UCSC.hg38")
```

那么对于没有现成的`BSgenome`包可用的物种,
`BSgenome`提供了`forgeBSgenomeDataPkg`函数用来直接从序列信息开始建立相应的R包.

## 如何forge一个BSgenome包

从头建立BSgenome包的过程大致如下:

1. 准备好基因组序列文件
2. 写一个给`forgeBSgenomeDataPkg`指定参数的配置文件(称为seed)
3. 调用 `forgeBSgenomeDataPkg` 函数生成R代码包, 该代码包可以在R命令行中编译(并安装)

如果序列文件是masked, 那么使用`forgeMaskedBSgenomeDataPkg`函数, 步骤稍有区别.

### 准备基因组序列文件

`BSgenome`需要的基因组序列文件是保存在同一文件夹下的多个fasta文件,

其中每个fasta文件只能有一个序列,并且fasta文件名要对应序列名.

比如在 [ftp://ftp.ensembl.org/pub/release-101/fasta/homo_sapiens/dna/](ftp://ftp.ensembl.org/pub/release-101/fasta/homo_sapiens/dna/) 可以下载人类基因组每个染色体对应的序列文件.

记住这个文件夹的路径, seed文件中会用到.


也可以使用单个2bit文件.但这里不再赘述.

### 写seed文件

这个seed文件包括了生成BSgenome包过程中所需的所有参数和说明.

它是普通R包的DESCRIPION文件的扩展版本.

其中除了R包相关的说明信息外, 还要指明基因组的相关信息.

尤其是要指出刚刚准备好的基因组参考序列文件所在的文件夹路径.

以下就是`BSgenome.Hsapiens.UCSC.hg19`的seed文件.

```ini
Package: BSgenome.Hsapiens.UCSC.hg19
Title: Full genome sequences for Homo sapiens (UCSC version hg19, based on GRCh37.p13)
Description: Full genome sequences for Homo sapiens (Human) as provided by UCSC (hg19, based on GRCh37.p13) and stored in Biostrings objects.
Version: 1.4.3
organism: Homo sapiens
common_name: Human
provider: UCSC
provider_version: hg19
release_date: June 2013
release_name: Genome Reference Consortium GRCh37.p13
source_url: https://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/latest/
organism_biocview: Homo_sapiens
BSgenomeObjname: Hsapiens
SrcDataFiles: hg19.2bit, downloaded from https://hgdownload.cse.ucsc.edu/goldenPath/hg19/bigZips/latest/ on March 24, 2020
PkgExamples: genome$chr1  # same as genome[["chr1"]]
seqs_srcdir: /home/hpages/BSgenomeForge/srcdata/BSgenome.Hsapiens.UCSC.hg19/seqs
seqfile_name: hg19.2bit
```

其中 seqs_srcdir 就是参考序列文件所在的文件夹路径. 

具体每一项的功能, 可以参考[官方的说明](https://www.bioconductor.org/packages/release/bioc/vignettes/BSgenome/inst/doc/BSgenomeForge.pdf).

### 生成代码包并编译安装

#### 在R中生成代码包

在序列文件和seed文件都准备好之后, 就可以在R中调用 **`BSgenome::forgeBSgenomeDataPkg(path/to/seed/file)`** 来生成一个代码包了.
这个函数的参数就是seed文件的路径. 


默认情况下这个代码包会出现在当前工作目录.(使用getwd函数可以查询R的当前工作目录).

生成代码包的耗时根据基因组的规模的不同从几分钟到数小时不等.

#### 命令行中编译生成的R代码包

在运行完毕之后, 需要对代码包进行编译和安装.

这个步骤**不在R环境中进行**, 而是在系统的命令行中完成, 比如 mac/linux 的shell, 或者windows的cmd命令行中.

如果是windows用户的话, 需要修改自己PATH变量,把Rcmd.exe所在的文件夹添加进去.

编译的命令如下:

```shell
R CMD build /path/to/package
```

其中/path/to/package 就是刚刚生成的代码包的路径

编译之后,会生成一个 tar包

接着使用

```shell
R CMD check /path/to/tar
```

来检查这个tar包.

如果一切ok,这个tar包就可以安装到R中了,

命令行中可以使用如下命令.

```shell
R CMD INSTALL /path/to/tar
```

安装完毕后, 包的名称就是seed文件中`Package`项对应的内容.
