---
aliases: 
tags: 
title: SDS log
date created: 2026-04-11 22:12
date modified: 2026-04-30 22:39
---

# 25 年 5 月  

拿到自己课题  
开始自己读 SDS 的 R 脚本，输入输出  
学习用 bcftools 准备 s file  
4197 人样本 

> - 拿 chr22 验证脚本，因为小，效率高  

# 25 年 6 月  

读 SMC++ 输入输出  
学习 docker 怎么用  
学习 make 命令   
用 SMC++ chr22 跑了第一个趋势图出来，chr22 抽了 20 个样本  
gamma shape 的 ms 软件需要从 SDS github 给的网址装，下源码下来编译  
用 SMC++ chr22 跑了 400 个样本，Ne 是 2,118,014  
跑通了 chr22 的 SDS 计算，拿到了 rSDS，做了 norm  
从 `148` 小服务器转到 hpc2  
SMC++ chr11 400 个样本，Ne 是 276,151，这次趋势虽然跟之前差不多，但是人口爆炸增长的年代提前了不少，740 代左右就开始很大了，时间推算大概是 1~2 万年前，这感觉不大合理（女娲 200 个样本，Ne 是 152,000）  
尝试了 chr11 的 sds，没有信号  

# 25 年 7 月  

SMC++ 改用了 singularity，不需要 sudo 权限    
学了 uv  
登了 `230` 小服务器  

> - MHC region 很重要，高度复杂  
> - 了解了 k-mer association study，node graph-based pangenome  
> - STR 突变率比 SNP 高很多所以不太可能重叠 -> 正交  
> - 

# 25 年 8 月  

WGS 开始做 preprocess  
singularity 提交在不同的 queue 上，有的需要 namespace  
SMC++ 常染色体一起算，Ne 平均 7w7  
EFO 是现在所有的 GWAS 找到的疾病显著位点  
SMC++ 抽样 200 人，抽 10 次取平均，Ne 平均 24w    
下载 Nyuwa 的 sds 数据，学习做 meta  

> - GWAS 发现的大部分信号不在 gene 上，而在 functional element 上。这其实是仔细一想非常 make sense 的，但就是没想过  
> - SNV 频率在 1/1000  
> - 任务交上去 `bjobs -l` 看一下具体用了多少个线程，有可能分配了很多 CPU，但实际只用了一点点  
> - 做 mapping，对不上的这些 reads 提出来做 blast 和 k-mer GWAS  
> - 

# 25 年 9 月  

学习用 plink 把 vcf 都转成 plink 做 QC  

- biallelic，DP5 GQ20
提出可能用 phlash 来代替 SMC++ 计算 Ne  

> - 学会了用 LSF 交 job array  
> - QC 先提高质量位点，做 sample QC，把人去了之后再做 snp QC，不需要特别严格  
> - 

# 25 年 11 月  

王老师说可以算一下 iHS，做 EHH 的图  
并且看一下 SDS 信号是否富集在尼人的基因  

# 25 年 12 月  

做了 meta analysis，有 14k 的点是 overlap 的，看了下用来 meta 的文件大小，我们只有 200MB，他们有 900 多 MB

# 26 年 4 月  

拿女娲的 significant region 的 top SNV，去计算 rSDS 验证  
把 iHS 显著的结果做一下 gene label，并用 $-\log P$ 画图，只展示 $p<0.01$ 的就可以  

`4_99318162_C` 这个位点我们数据里没有，挑选了 `4_99317841_C` 这个位点  

  - Nyuwa AF/SDS: 0.7669, 10.3947  
  - NCN raw VCF AF: 0.774979  
  - NCN current raw rSDS: 1.1458  

| Scenario      | Status     | Percentile |   rSDS | Delta vs control | Delta vs q50 | gamma(1-DAF) | gamma(DAF) | Projected norm_SDS | Projected -log10(p) |
| ------------- | ---------- | ---------: | -----: | ---------------: | -----------: | -----------: | ---------: | -----------------: | ------------------: |
| current_gfile | ok         |            | 1.1458 |           0.0000 |       0.0231 |     0.002265 |   0.000528 |             3.3155 |              3.0387 |
| q2.5          | ok         |        2.5 | 0.5501 |          -0.5957 |      -0.5726 |     2.547937 |   0.799535 |             1.8533 |              1.1949 |
| q25           | ok         |       25.0 | 1.2539 |           0.1081 |       0.1312 |     0.224921 |   0.001227 |             3.5808 |              3.4653 |
| q50           | ok         |       50.0 | 1.1227 |          -0.0231 |       0.0000 |     0.002284 |   0.000576 |             3.2588 |              2.9512 |
| q75           | ok         |       75.0 | 1.0440 |          -0.1018 |      -0.0787 |     0.001629 |   0.000528 |             3.0656 |              2.6631 |
| q97.5         | incomplete |       97.5 |     NA |               NA |           NA |           NA |         NA |                 NA |                  NA |
| constNe152000 | ok         |            | 0.6868 |          -0.4590 |      -0.4359 |     0.230397 |   0.137604 |             2.1889 |              1.5435 |

| Paper                                                                                                                                                                                  | Description                                                                                                         |                                  Cohort                                   |                                Method                                 |                                                    Ne | Remark                                                                                                                                                                    |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------ | :-----------------------------------------------------------------------: | :-------------------------------------------------------------------: | ----------------------------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Recent positive selection signatures reveal phenotypic evolution in the Han Chinese population](https://www.sciencedirect.com/science/article/pii/S2095927323005558?via%3Dihub#s0125) | **重复随机选取 200 个样本 9 次**，估计取平均值。准备了 10 个备选输入文件，每个文件包含不同的个体作为 distinguished lineage，所有这些文件作为输入，得到复合似然。使用 Nyuwa 自己的人口模型 |                 3,946 (27.8x) WGS, Chinese (Nyuwa cohort)                 |                                 SMC++                                 |                                               152,000 | 对基因组（从 1KGP 下载）的低复杂度区域进行了 mask                                                                                                                                            |
| [Deep whole-genome sequencing reveals recent selection signatures linked to evolution and disease risk of Japanese](https://www.nature.com/articles/s41467-018-03274-0)                | `Gravel_CHB` 默认的人口模型                                                                                                |                        2,234 WGS (25.9x), Japanese                        |                        SMC++ (*结果没有用于 SDS 计算*)                        |                     100,000 (SMC++ 结果最近代达到 $10^7$ 量级) | **Variant calling** 用了 1) `DP<5` 2) `GQ<20` 3) `DP>60 OR GQ<95`. **排除了** genotype call rate < 0.90% 的位点，<0.99% 的样本。**排除了**深度 >2xAVG/<0.5xAVG 的位点，`QUAL<56`。**H-W** `1e-6` |
| [Prioritizing autoimmunity risk variants for functional analyses by fine-mapping mutations under natural selection](https://www.nature.com/articles/s41467-022-34461-9)                | 随机**抽 100 个 diploids** 估计。使用 discrete time Wright-Fisher model，1,000 代以后换成 Hudson model                             |                  1,800 WGS, European (Estonian Biobank)                   |                                Relate                                 | 15,000 (30k haplotypes, supplemetary 中 Ne(0)=331,373) | SNP 基于 GRCh37 人类祖先基因组做了 polarization，移除祖先状态未知的 SNP。                                                                                                                       |
| [Inferring past demography and genetic adaptation in Spain using the GCAT cohort](https://www.nature.com/articles/s41598-025-98272-w)                                                  | `Tennesse` 默认的人口模型                                                                                                  |                   704 (30x) WGS, Spanish (GCAT cohort)                    |                                 SMC++                                 |                                           Unmentioned | 高选择峰代表 SDS > 99.99% SNP，且在 1Mb 区域内至少有 10 个 SDS > 99.995% SNP                                                                                                              |
| [Using singleton densities to detect recent selection in Bos taurus](https://academic.oup.com/evlett/article/5/6/595/6697594)                                                          | Boitard et al. 推断的**荷斯坦牛** demographic model，有两个，Low $N_0$ 和 High $N_0$，                                            |                              102 WGS, **家牛**                              | Unmentioned (Boitard et al. 用了 Approximate Bayesian Computation, ABC) |               793 (High $N_0$) / 49 (Low $N_0$, 系谱数据) | 排除过度组装区域，singletons 需两端等位基因均有足够 coverage。SNP 做了 polarization。用 $N_e = 793$ 和 $N_e = 49$ 计算的 SDS 结果大致相近，但后者检测到更多基因。*样本量和检测时间在对数上成反比*，但太少会限制 statistical power              |
| [Accelerated Bayesian inference of population size history from recombining sequence data](https://www.nature.com/articles/s41588-025-02323-x)                                         | 通过平均多个 piecewise constant 历史得到平滑估计                                                                                  | 3,601 现代 + 古 DNA, 159 unique populations, 6 geographical superpopulations |                             Phlash (原始论文)                             |        各人群 $N_{e} \in [ 10^{3}, 10^{5} ]$, 后验分布，非单点估计 | 极近期不确定性大（近期 coalescence 少，信号弱），检测古代瓶颈但统计检验不支持极端瓶颈假说                                                                                                                       |

一些文章似乎对 SNP 做了 polarization，就是用外部序列去为 SNP 标注 ancestral 和 derived，这件事情是否有必要？我觉得没有必要的原因是 tSDS 可以聚合 statistics，借助 GWAS sum stat 把方向矫正；gene-wise SDS 应该也可以做类似的事情。  

| 组       | 用哪条 Ne(t)                  | 正式算 SDS 吗 | 参与最后 Ne(0) 区间判断吗 | 备注         |
| ------- | -------------------------- | --------- | ---------------- | ---------- |
| q2.5    | phlash 的 q2.5 整条曲线         | 是         | 否                | 参照组        |
| q25     | phlash 的 q25 整条曲线          | 是         | 否                | 参照组        |
| q50     | phlash 的 q50 整条曲线          | 是         | 否                | 参照组，也是缩放模板 |
| 100k    | q50 整条曲线整体缩放到 Ne(0)=100000 | 是         | 是                | 主扫描        |
| 152k    | 同上，缩到 152000               | 是         | 是                | 主扫描        |
| 250k    | 同上，缩到 250000               | 是         | 是                | 主扫描        |
| 500k    | 同上，缩到 500000               | 是         | 是                | 主扫描        |
| 750k    | 同上，缩到 750000               | 是         | 是                | 主扫描        |
| 1e6     | 同上，缩到 1000000              | 是         | 是                | 主扫描        |
| 1743549 | 同上，缩到 1743549              | 本质上等于 q50 | 是，但不该重复模拟        | 应和 q50 复用  |
| 2.5e6   | q50 曲线整体放大到 2500000        | 是         | 否                | 高侧诊断       |
| 5e6     | q50 曲线整体放大到 5000000        | 是         | 否                | 高侧诊断       |
| q97.5   | phlash 的 q97.5 整条曲线        | 否，主要看能否跑通 | 否                | 只做冒烟检查     |

SMC 更可信，因为 bottleneck 的 peak 来自于 African，大概在 $10^3$ 代以前，phlash 的 bottleneck 太近了。想比较 demographic history 是否合理，可以去找 MSMC 和 SMC++ 的 plots，近期的历史虽然不一样，但 bottleneck 应该是一样的。检查 timepoint

但是现有的方法对 Ne(0) 的估计都很差，Relate 并不能更好解决这个问题，或许 IBD method 会好一些

但是建议是没有必要自己跑模拟，找同样是 Chinese cohort 的 paper，直接用他们的 Ne(0) 就好了。毕音 SDS 很久以前就发表了。至今也没有很好的方法能够 estimate 近期的 Ne，大家都是这么做的

可以从 10Ok 开始，上下取一些值算 SDS，不应该太关注 rSDS，而是 norm SDS，或者说 gene rank。Ne 不应该对 rank 有大的影响。rSDS 有上下浮动是正常的，阳性检验的符号没有变，这一点是合理的，对于 true signal 符号不应该变动，数值变化是 noise，正常的

看目的，如果想要和 GWAS hits 做 overlap，那么需要用强选择的信号。要定义 Chinese-specific 信号就必须我们的 peak 过了过往文章设定的 threshold

但对于那些 pvalue 不显著的信号，事情就不一样，selection scan 和 GWAS 是不一样的，

selection 很难 justify 这是 weak selection 还是没有到达 stat significance。因此对于这些点不能去 call 一个 threshold。这一定是一个问题，tSDS 是可能的说明的方法，用 stat sum 对微小信号做矫正来放大，来证明这是真的广泛的选择信号 (类比于 GeneBayes 利用 constraint 来矫正 burden test 的简单 gene shrinkage)

要看跑出来的 SDS 结果是否 reasonable，看几个特征峰是否一致。但是如果有大量的散落点，那肯定是出问题了。chr1 那样奇怪的高峰也要看是不是 manifestation
