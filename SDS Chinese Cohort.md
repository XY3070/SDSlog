---
aliases: 
tags: 
title: SDS Chinese Cohort
date created: 2025-11-19 21:07
date modified: 2026-04-30 22:38
---

# Introduction  

## Purpose 

1. 想用 SDS 找到新的、Chinese-specific 的区域  
2. 想找到 polygenic 的 tSDS 信号  

--- 

1. 如何找到选择位点;   
2. 推断选择开始的时间；   
3. 判断潜在的选择压力是什么  

# Results  

## 1. SDS  

### Full  

### Meta  

与 Nyuwa 的 SDS 结果用 SSW 做 meta，然后再用 IVW 做，检查 Heterogeneity  

参考：[Recent positive selection signatures reveal phenotypic evolution in the Han Chinese population - ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2095927323005558?via%3Dihub)  

### GWAS Overlap  

### Archaic Genome Introgression      

看看 SDS 信号好的位点是否富集在尼安德特人的基因  

参考：[Quantifying the contribution of Neanderthal introgression to the heritability of complex traits | Nature Communications](https://www.nature.com/articles/s41467-021-24582-y)  

# Methods  

## 1. Data    

### 1.1 1st QC  

参考：[[QC-11.7]]   

### 1.2 2nd QC  

保留 4025 样本  
剔除 Bankok 人群  
最终剩余 1255+2716=3971 样本

## 2. Singleton Counts  

南方人群的 singleton counts 少于北方人 -> ==为什么？==  

$\implies$ 需要**分人群计算 SDS**  

## 3. Ancestry Sturcture    

### Neural-Admixture  

nadm 有两种模式，一种是 train 一种是 infer。train 可以选择 supervised 或者 unsupervised，unsupervised 更类似于 clustering，需要预先制定超参数 $k$。  

Unsupervised learning 选择了 $k=2\sim 20$，评估表征，最终选择 $k=4$，以 `comp 3` 第四个祖源成分作为分群依据，`0.35` 作为 threshold，`>0.35` 标志北方。  

## 4. Demographic History  

### SMC++  

### Phlash  

## 5. SDS  

### Norm SDS  

### tSDS  

## 6. iHS  

After QC, collected about more than 3.9k WGS samples for downstream analysis.  

Want to compute the Singleton Density Score to see what are the recent positively selected regions, and see if there is any overlap with the SLE or other disease risk alleles. Thus we can tell a story of phylogeny, that why the risk alleles are still preserved because of some phylogenic advantage.  

Before compute the SDS, we should prepare the SDS input files. We need an s file, a t file, an o file, a b file, and a g file.  

The vcf files are all stored at `/share/home/grp-wangyf/xuyuan/sds/plink/smc_vcfs`.  

The inputs are all stored at `/share/home/grp-wangyf/xuyuan/sds/data/processed/sds_input`.

### EHH  

## 7. Relate + CLUES2  
