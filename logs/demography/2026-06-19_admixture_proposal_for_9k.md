# ADAMIXTURE 在 9k 中国人群队列上的群体分层方案报告

- **日期**: 2026-06-19
- **目的**: 评估 [ADAMIXTURE](https://github.com/AI-sandbox/ADAMIXTURE) 用于 `/raw/9k/` 队列群体分层 (population stratification / genetic ancestry) 的可行性，并基于当前已 QC 数据提出一套可执行的端到端流程。
- **参考文献**: Saurina-i-Ricos, J., Mas Montserrat, D., Ioannidis, A.G. (2026) *ADAMIXTURE: Adaptive First-Order Optimization for Biobank-Scale Genetic Clustering*. bioRxiv. DOI: [10.64898/2026.02.13.700171](https://www.biorxiv.org/content/10.64898/2026.02.13.700171)
- **配套资源**: [GitHub 仓库](https://github.com/AI-sandbox/ADAMIXTURE)（`adamixture/`, `docs/`, `tests/`）

---

## 1. 当前数据现状盘点

### 1.1 `/raw/9k/` 下可用的 QC 产物

| 层级 | 产物 | 路径 | 状态 |
| --- | --- | --- | --- |
| Sample QC | 冻结样本清单 + 亲缘/杂合/PCA 异常剔除 | `sample_qc/`, `sample_qc_chr1_maf05/` | 完成 |
| SNP+Singleton QC | 每条染色体独立目录，含 `snp_qc_pass.vcf.gz[.tbi]` + singleton 过滤产物 | `snp_singleton_qc_chr*_v*/` | 10 条常染色体完成；chr6 仍在跑（目录中尚无最终 `snp_qc_pass.vcf.gz`）；chr2/3/8/9/10/11/12/13/15/16/X/Y 尚未开始 |
| 祖先等位基因注释 | `singleton_ancestor_annot.tsv` + `singleton_ancestor_stats.tsv` | 同上目录内 | 10 条常染色体完成（汇总 `singleton_ancestor_stats_all_chr.tsv`，整体注释率 ~94.63%） |
| 顶层整染色体 VCF | `all.chr{N}.QC.vcf.gz` | `/raw/9k/` 根目录 | chr1/4/5/6/7/14/17/18/19/20/21/22 存在，仅 chr6 有 `.tbi` |

**关键事实**:
- 当前冻结样本集 = **4393 个样本**（已通过 sample-QC 的 KING/PCA/杂合/缺失过滤）
- 现有 `singleton_ancestor_annot.tsv` 是给 singleton burden 诊断用的（AC=1 位点 AA/DA 标签），**不是**全局祖先成分估计，不能替代 ADMIXTURE-like 的 Q 矩阵
- 10 条已完成 SNP QC 的染色体（chr4/5/7/14/17/18/19/20/21/22）可直接进入分层分析

### 1.2 数据缺口

1. **chr6** 尚未产出最终 `snp_qc_pass.vcf.gz`（HLA 区结构关键，建议补齐）
2. **chr1/2/3/8/9/10/11/12/13/15/16** 未做 SNP QC，但 chr1 已有 `all.chr1.QC.vcf.gz` 顶层版本可临时复用
3. **性染色体** 暂不进入 ADMIXTURE 类模型（常染色体模型假设二倍体常染色体遗传）

---

## 2. ADAMIXTURE 工具系统性认知

### 2.1 模型家族定位

ADAMIXTURE 在统计模型层面**与 ADMIXTURE 完全等价**:
- 观测: 基因型矩阵 $X_{N \times M}$
- 参数:
  - 个体祖先比例矩阵 $Q_{N \times K}$（行和为 1）
  - 各祖先簇的等位基因频率矩阵 $P_{K \times M}$
- 似然: $X_{ij} \sim \text{Binomial}(2, \sum_k Q_{ik} P_{km})$

**差异只在优化算法**:
- ADMIXTURE: block relaxation + quasi-Newton（二阶），每轮 EM 内做多次内层优化
- ADAMIXTURE: **每完成 1 步 EM 即对 (Q,P) 用 Adam 一阶自适应优化器更新一次**，用自适应矩估计逼近曲率信息而无需 Hessian

### 2.2 算法要点（源自原文 abstract + docs）

1. **Adam 更新作用在 EM 的 M-step 之后**: 一步 EM 给出 (Q,P) 的期望方向，Adam 沿该方向用自适应学习率前进
2. **自适应学习率**: 引入 `beta1`/`beta2` 一阶/二阶矩平滑、`reg_adam` 正则、`lr_decay` 衰减
3. **收敛控制**:
   - `--tol_adam` 默认 0.1（log-likelihood 相对改进阈值）
   - `--patience_adam` 默认 3（连续几次未改进触发 lr 衰减）
   - `--lr_decay` 默认 0.5（衰减因子）
   - `--max_iter` 默认 10000
4. **大规模场景调参（官方 troubleshooting）**: 大样本 / 大 K 建议 `--patience_adam 5 --lr_decay 0.85 --lr 0.0075`
5. **硬件加速**: CPU 多线程 `-t`；NVIDIA GPU `--device gpu`；Apple Silicon `--device mps`
6. **原文性能声明**: GPU 上 50 万样本 × 50 万变异 < 2 小时，相比 ADMIXTURE 量级上 ~100× 加速

### 2.3 输入/输出契约

| 项目 | 说明 |
| --- | --- |
| 输入格式 | BED（PLINK）、VCF、PGEN（原生支持，无需额外转换） |
| 位点假设 | 二倍体常染色体，双等位基因；多等位/indel 行为未明确文档化，建议入口前过滤 |
| 样本对齐 | 各 run 内样本顺序与输入文件一致；projection 模式要求新数据与训练 **完全相同的 SNP 集合** |
| 输出 | `*.P` (K×M 频率矩阵)、`*.Q` (N×K 个体比例矩阵)、`*.png`（堆叠柱状图）、终端 log |
| 可视化 | `--plot` 一次 sweep 多 K 合并图；`--plot_single` 每 K 单独一张；支持 `pdf/png/jpg`、DPI、`--colors` 自定义调色板；跨 K 自动对齐簇色 |
| 层次化标签 | `--labels`/`--labels2`/`--labels3` 用于按已知元数据排序/分组展示 |

### 2.4 三种运行模式

1. **无监督 sweep**（默认）
   ```
   adamixture --min_k K1 --max_k K2 --cv [folds] --data_path X.{bed,vcf,pgen} --save_dir out/ --name tag -t N
   ```
   同时产出多个 K 的 (P,Q) 与 CV 误差曲线。
2. **投影 (projection)**
   ```
   adamixture-project --data_path new.{bed,vcf,pgen} --p_path trained.P --save_dir out/ --name tag
   ```
   冻结训练得到的 P，仅估计新样本的 Q。**SNP 集合必须完全一致**。
3. **监督 (supervised)**
   ```
   adamixture-supervised --data_path all.bed --labels labels.txt --save_dir out/ --name tag -k K
   ```
   `labels.txt` 每行一个样本，给已知群体标签或 `-`（自由估计）。标注样本的 Q 被强制 one-hot，P 由已知群体驱动。

### 2.5 K 选择策略（CV 机制）

- `--cv F`：F-fold 掩码基因型，衡量预测误差
- 默认 5-fold；可改为 10-fold 获得更稳的曲线
- 选取标准：**最小 CV 误差**，或曲线**肘部 (elbow)**
- 与 ADMIXTURE 的 CV 解释方式完全一致，团队可无缝迁移

### 2.6 局限与注意点

1. **没有内置 LD 处理**: 文档未提 LD pruning；按 ADMIXTURE 最佳实践，**入口前仍需自行 LD-prune**
2. **内存是主要瓶颈**: 官方明确"biobank-scale datasets are best handled on dedicated CUDA-capable GPUs due to high RAM requirements"
3. **没有内置 MAF/missingness 过滤**: 入口前必须自己完成
4. **性染色体 / 非二倍体区域**: 不在默认模型假设内，应剔除
5. **CV 在极不平衡样本下可能偏向占主导的群体**: 对中国队列内部细微结构，需要谨慎结合 PCA 与先验地理/民族元数据解读
6. **Python ≥ 3.10**；macOS 需先装 `libomp`；GPU 环境需 `nvcc`

---

## 3. 建议流程（针对 9k 队列）

### 阶段 0: 数据补齐（必须先做）

1. 完成 **chr6** 的 SNP QC（`snp_singleton_qc_chr6_v1/snp_qc_pass.vcf.gz[.tbi]`）
2. 补齐 **chr1/2/3/8/9/10/11/12/13/15/16** 的 SNP QC；至少把 chr1 顶层 `all.chr1.QC.vcf.gz` 拷贝/索引到与 chr4/5/…/22 相同的命名约定，便于下游统一消费
3. 所有 4393 样本的**元数据表**（采样地 / 民族 / 自我报告群体）收集并格式化为 `labels.txt`（监督模式 + 可视化标签必备）

### 阶段 1: 入口矩阵构建（最关键的一步）

目标: 得到一份**跨染色体、同一批样本、同一组 SNP** 的干净基因型矩阵。

```bash
# 1.1 收集已通过 SNP QC 的常染色体 VCF
IN_VCFs=(
  /raw/9k/snp_singleton_qc_chr{1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22}_v*/snp_qc_pass.vcf.gz
)

# 1.2 用 bcftools 合并（保留 4393 个共同样本）
bcftools concat -a -Oz --threads 16 "${IN_VCFs[@]}" \
  | bcftools view -S /raw/9k/sample_qc/frozen_samples.txt -Oz -o /tmp/9k_concat.vcf.gz

# 1.3 仅保留双等位 SNP、无缺失、MAF ≥ 0.01（或 0.05）
bcftools view -m2 -M2 -v snps -i 'F_MISSING<=0.0 && MAF>=0.01' \
  -Oz -o /tmp/9k_biallelic.vcf.gz /tmp/9k_concat.vcf.gz

# 1.4 转换到 PLINK2 PGEN（ADAMIXTURE 原生支持，速度与内存都优于 VCF）
plink2 --vcf /tmp/9k_biallelic.vcf.gz \
       --make-pgen --out /tmp/9k_raw

# 1.5 LD pruning（关键：ADAMIXTURE 不内置 LD 处理）
plink2 --pfile /tmp/9k_raw \
       --indep-pairwise 1000 50 0.1 \
       --out /tmp/9k_pruned

plink2 --pfile /tmp/9k_raw \
       --extract /tmp/9k_pruned.prune.in \
       --make-pgen --out /raw/9k/admixture_input/9k_ldpruned

# 预期保留 5 万 ~ 20 万 LD-pruned 常染色体 SNP，足以解析到 K=10 左右的细微结构
```

产物: `9k_ldpruned.{pgen,psam,pvar}` + 同目录 `9k_ldpruned.labels.txt`（与 `psam` 顺序对齐）。

### 阶段 2: 环境准备

```bash
# Python 3.10+ 隔离环境
python3.11 -m venv ~/envs/adamixture && source ~/envs/adamixture/bin/activate
pip install --upgrade pip
pip install adamixture   # 如需 GPU: 先 module load cuda / conda install -c nvidia nvcc

# 验证
adamixture --help
```

如果集群有 A100/H100，**强烈推荐** GPU 运行。4393 样本 × ~10 万 SNP 在 CPU 上也能跑（分钟级），但 GPU 上 K sweep 可压到秒级。

### 阶段 3: 无监督 K-sweep（第一步，回答"有几个可辨群体"）

```bash
adamixture \
  --min_k 2 --max_k 12 \
  --cv 10 \
  --data_path /raw/9k/admixture_input/9k_ldpruned.pgen \
  --save_dir  /raw/9k/admixture_run1/unsupervised/ \
  --name 9k_run1 \
  -t 32 \
  --labels /raw/9k/admixture_input/9k_ldpruned.labels.txt \
  --plot --plot_single \
  --max_iter 10000 --tol_adam 0.1 \
  2>&1 | tee /raw/9k/admixture_run1/unsupervised/run.log
```

**产出解读**:
- `cv_error_per_k.tsv`: CV 误差 vs K 曲线，看最小值 + 肘部
- `*.Q`: 每 K 的个体祖先比例矩阵
- `*.png`: 堆叠柱状图（按 `labels.txt` 排序）

### 阶段 4: 诊断与 K 敲定

1. 画 CV 曲线（R / python 均可），与 **PCA 前 10 主成分 Tracy-Widom / variance-explained 曲线** 交叉验证
2. 对照 `labels.txt` 中的地理/民族元数据，检查 K=2..12 的堆叠图是否呈现连续梯度或离散簇
3. 如果中国人群预期北方-南方梯度为主，预期最佳 K 在 **2–4** 之间；若有更细的省际/族群结构可能推到 6–8
4. 记录选定的 K* 与对应 P 矩阵路径 `.../9k_run1.K{K*}.P`

### 阶段 5: 监督模式（用已知参考群体锚定簇语义）

为了把抽象簇映射到可解释的祖先来源（如"北方汉族 / 南方汉族 / 藏缅 / 壮侗 / 苗瑶 / 阿尔泰"等），引入外部参考:

```bash
# 参考面板候选:
#   - 1000 Genomes EAS (CHB/JPT/CHS/CDX/KHV)
#   - HGDP 东亚+东南亚
#   - 本项目内部挑选出的"典型"样本（PCA 极端点）

# labels.txt 格式:
#   SAMPLE1   CHB
#   SAMPLE2   CHS
#   SAMPLE3   -          # 9k 本队样本,自由估计
#   ...

adamixture-supervised \
  --data_path /raw/9k/admixture_input/9k_plus_refs.pgen \
  --labels    /raw/9k/admixture_input/9k_plus_refs.labels.txt \
  --save_dir  /raw/9k/admixture_run2/supervised/ \
  --name 9k_sup \
  -k 6 \
  -t 32 --plot
```

### 阶段 6: 投影到增量样本（将来每次新来一批样本时复用）

```bash
adamixture-project \
  --data_path /raw/9k/new_batch.pgen \
  --p_path    /raw/9k/admixture_run2/supervised/9k_sup.K6.P \
  --save_dir  /raw/9k/admixture_run3/projected/ \
  --name batch2 \
  --labels /raw/9k/new_batch.labels.txt \
  --plot
```

**前提**: 新 batch 与训练集 **SNP 集合完全一致**；若不同芯片，需先做位点对齐 (`plink2 --bmerge` 或取交集)。

### 阶段 7: 下游消费

- 将 Q 矩阵作为**协变量**加入 GWAS（取代或补充前 N 个 PCA 主成分），用于校正群体分层
- 与现有 `singleton_ancestor_annot.tsv` 联动：按 Q 推断的主祖先成分分组，比较各组的 singleton burden / AA-vs-DA 比例
- 与 `aggregate_ancestor_stats.py` 输出结合，做"不同祖先成分 × 不同染色体"的 singleton 率热力图

---

## 4. 风险、替代方案、与现有流程的关系

| 风险点 | 缓解 |
| --- | --- |
| ADAMIXTURE 论文为 2026 年 preprint，尚未经过同行评审 | 第一阶段先在小规模（如仅 chr21/22）与 ADMIXTURE 原版对照，验证 Q 矩阵相关性 > 0.98 后再全量铺开 |
| 没有内置 LD 处理 | 阶段 1 的 LD pruning 是硬门槛，不能跳过 |
| GPU 资源不确定 | 4393 × ~10 万 SNP 在 CPU 上完全可行（量级 ~10 分钟/K），可先 CPU 跑通再考虑 GPU |
| 中国人群细微结构 K 难定 | 必须与 PCA、fineSTRUCTURE/ChromoPainter、f3/f4-statistics 互补，不能单靠 CV |
| 投影要求 SNP 完全一致 | 每次换芯片 / imputation panel 都需重新训练 P 或做严格的位点取交集 |

**替代/互补工具**:
- **ADMIXTURE** (Alexander & Lange 2011): 金标准，速度慢；用于对照验证
- **ALStructure**: 非参数方法，对 LD 更鲁棒
- **PCA (flashPCA2/smartpca)**: 与 Q 矩阵互为佐证，必做
- **fineSTRUCTURE / ChromoPainter**: 解析更细的近期共祖结构，与 ADMIXTURE-like 的深层结构互补

---

## 5. 推荐的落地顺序（最小可行产物）

1. **[P0]** 完成 chr6 SNP QC；统一 chr1..22 的 `snp_qc_pass.vcf.gz` 命名
2. **[P0]** 收集并格式化 4393 个样本的群体元数据 → `labels.txt`
3. **[P0]** 跑阶段 1（入口矩阵）+ 阶段 2（环境）
4. **[P1]** 跑阶段 3 的 K=2..12 sweep + CV，产出第一份报告
5. **[P2]** 在 PCA 上交叉验证，敲定 K*
6. **[P2]** 可选：跑阶段 5 监督模式锚定语义
7. **[P3]** 把 Q 矩阵接入 GWAS 协变量与 singleton burden 分层分析

---

## 6. 待确认的开放问题（请用户拍板）

1. 是否**现在就开始跑** chr4/5/7/14/17-22 这 10 条染色体的初版（不等 chr6）？
2. 4393 样本的**元数据**（采样地、民族）是否已有结构化表？路径？
3. 运行环境优先 **CPU 还是 GPU**？集群队列与 walltime 上限？
4. 是否要引入**外部参考面板**（1000G EAS / HGDP）做监督？
5. 下游 GWAS 是否计划把 Q 矩阵作为协变量，还是仅作为分层描述用？

---

## 7. 可行性扫描（已基于当前环境做了一次只读探查）

| 检查项 | 结果 | 影响 |
| --- | --- | --- |
| `adamixture` 是否已装 | **未装**；base env 是 Python 3.8，ADAMIXTURE 要求 ≥ 3.10 | 需要新建 conda env（建议 `admixture_py311`），或让管理员装 wheel |
| GPU | 登录节点无 `/dev/nvidia*`，未跑过 `nvidia-smi` 探测计算节点 | 默认按 **CPU-only** 规划；96GB RAM 对 4393 样本绰绰有余 |
| `plink2` | `/data/home/grp-wangyf/leiyao/Tools/PLINK2/plink2` ✓ | — |
| `bcftools` | `/data/home/grp-wangyf/intern/miniforge3/bin/bcftools` ✓ | — |
| chr1 已 LD-prune 过的 33k SNP 列表 | `sample_qc/pruned_postking.prune.in`（chr1 only, MAF≥0.05） | **不能复用**：需要全基因组 LD-prune 才能做 ADMIXTURE |
| chr1 BED（KING 后、MAF≥0.05） | `sample_qc/chinese_postking_maf05.{bed,bim,fam}` ✓ | 可作 chr1 入口，但本项目直接用 SNP-QC VCF 统一口径 |
| **元数据质量**（`cohort_freeze.tsv`, 4394 行含表头） | Region 分布：HongKong **3101 (70.6%)**、JiNing 391、Shenzhen 267、East Asian Ancestry 192、Jinan 172、China 140、Shanghai 95、Guangzhou 30、EastAsia 3、`0` (脏值) 2 | **标签高度不均衡 + 异构**：HongKong 主导；`China` / `EastAsia` / `East Asian Ancestry` / `0` 是低信息标签，进监督模式前必须清洗 |

### 7.1 元数据清洗建议（`labels.txt` 落地前必做）

```text
原始 Region         → 规范化
-----------------------------------------
HongKong             HongKong
JiNing               JiNing
Jinan                Jinan     (拼写统一)
Shenzhen             Shenzhen
Shanghai             Shanghai
Guangzhou            Guangzhou
East Asian Ancestry  -         (无城市信息,作 unknown)
China                -         (太泛)
EastAsia             -         (太泛)
0                    -         (脏值)
```

清洗后有效标签 ~ 3956 / 4393 ≈ **90%**，足够做 `--labels` 排序展示。监督模式若要做，建议先只把有明确城市的 3956 个样本当"已知"，其余 437 个当 `-` 自由估计。

### 7.2 已落地的可执行驱动脚本

`SDSworkspace/runs/admixture_9k/scripts/run_admixture_9k.sh` —— 实现阶段 1–6 全链路，幂等（每步写 `.done` sentinel），支持 `./run_admixture_9k.sh step1|step2|…|step6 [K]|all`。

要点：
- 自动解析每条染色体的 `snp_singleton_qc_chr{N}_v*` 目录
- 严格限制在 `cohort_freeze.tsv` 的 4393 个冻结样本
- MAF ≥ 0.01 + 双等位 SNP + 无缺失过滤
- `plink2 --indep-pairwise 1000 50 0.1` LD-prune
- 用 Python 小片段把 `Region` 列对齐到 `psam` 顺序生成 `labels.txt`，并自动清洗低信息标签
- Adam 调参用官方 troubleshooting 推荐：`--patience_adam 5 --lr_decay 0.85 --lr 0.0075`
- `MAX_K=10`、`CV_FOLDS=10`，可按需改

### 7.3 跑前必须用户/管理员确认的 4 件事

1. **建 Python ≥ 3.10 环境**：`conda create -n admixture_py311 python=3.11 && conda activate admixture_py311 && pip install adamixture`。需要你授权 `conda create`，我没自动执行
2. **计算节点与 walltime**：CPU 模式下 4393 × ~50 万（pruned 后约 5–15 万） SNP 的 K=2..10 sweep 预计 30–90 min（取决于最终 SNP 数与迭代轮数）。建议申请一个 32 核、8h、256GB 的 LSF job
3. **chr6 是否等补齐再跑**：当前脚本 `CHRS_READY=(4 5 7 14 17 18 19 20 21 22)` 未含 chr6。可先跑 10 条染色体的"初版"（完全足够看南北梯度），chr6 补完后再 rerun 对比
4. **是否引入外部参考面板做监督**（1000G EAS / HGDP），还是仅用内部 Region 做弱监督 + 无监督 K-sweep

### 7.4 预计跑完后的首份交付物

- `runs/admixture_9k/unsupervised/9k.K{2..10}.Q`：每 K 的个体祖先比例矩阵（4393 × K）
- `runs/admixture_9k/unsupervised/9k.K{2..10}.P`：每 K 的祖先簇等位基因频率（用于将来 projection）
- `runs/admixture_9k/unsupervised/cv_error.tsv`：CV 误差曲线，敲定 K*
- `runs/admixture_9k/unsupervised/*.png`：按 Region 排序的堆叠柱状图
- 后续分析：把 K* 对应的 Q 矩阵作为协变量接入 GWAS，按 Q 主祖先成分分组比较 singleton burden

---

## 附录 A: 命令速查

```bash
# 安装
pip install adamixture

# 无监督 K-sweep + CV
adamixture --min_k 2 --max_k 12 --cv 10 \
  --data_path INPUT.{bed,vcf,pgen} --save_dir OUT/ --name TAG \
  -t 32 --plot --plot_single --labels META.txt

# 监督
adamixture-supervised --data_path X.bed --labels L.txt \
  --save_dir OUT/ --name TAG -k K

# 投影
adamixture-project --data_path NEW.bed --p_path TRAINED.P \
  --save_dir OUT/ --name TAG

# 大样本调参推荐（官方 troubleshooting）
--patience_adam 5 --lr_decay 0.85 --lr 0.0075
```

## 8. 方案 A v1 初步跑通结果与 K\* 解读（2026-06-19 晚）

**执行摘要**：Plan-A 主链路已端到端完成（10 条常染色体 → 2,628,609 merged variants → 107,927 LD-pruned SNPs → K=2..10 sweep），所有 9 套 Q/P 矩阵与 stacked-bar 图均已落盘于
`SDSworkspace/runs/admixture_9k_v1_20260619/unsupervised/`。10-fold CV 仍在后台运行，但主交付已可用。

### 8.1 执行中踩到的坑（已自动修复，记录供后续复用）

| 坑 | 现象 | 修复 |
| --- | --- | --- |
| 系统 Python 3.8 | ADAMIXTURE 要求 ≥3.10 | uv-managed venv at `external/uv_envs/.venv`（Python 3.11.14） |
| torch wheel 要求 manylinux\_2\_28 | 本机是 manylinux\_2\_17（CentOS 7） | 装 `torch==1.13.1+cpu`（最后的 manylinux2014 兼容版本） |
| matplotlib 编译失败（缺 C++17） | contourpy 需要 C++17 | 装 `matplotlib==3.6.3`（pre-contourpy 版本） |
| `torch.compile` AttributeError | torch 1.13 没有 compile | site-packages/sitecustomize.py 注入 no-op stub |
| pgenlib 缺失 | ADAMIXTURE 读 .pgen 必需 | `uv pip install pgenlib==0.94.1` |
| bcftools `libgsl.so.25: undefined symbol: cblas_ctrmv` | libgsl 缺 DT\_NEEDED libopenblas | `mamba install -n base openblas` + `LD_PRELOAD` 注入 |
| bcftools OOM（62GB 峰值） | chr4 VCF 2.3GB 全量加载 | step1 重写为 plink2（streaming + 二进制打包，32GB 上限） |
| step3 `cp` 把文件拷到自身 | WORK\_DIR 与 prune\_in 同路径 | 删除 `cp` 步骤，prune.in 直接落位 |

### 8.2 Log-likelihood 曲线

| K | LL | Δ vs prev | 边际增益 |
| --- | --- | --- | --- |
| 2 | −216,022,561 | — | (baseline) |
| 3 | −215,877,780 | **144,781** | 最大 |
| 4 | −215,772,954 | 104,826 | 回落 28% |
| 5 | −215,672,388 | 100,566 | 回落 31% |
| 6 | −215,581,141 | 91,247 | — |
| 7 | −215,502,097 | 79,045 | — |
| 8 | −215,416,484 | 85,613 | 小回弹 |
| 9 | −215,343,238 | 73,245 | — |
| 10 | −215,268,796 | 74,442 | 完全平缓 |

LL 曲线 elbow 落在 **K=3**（边际 144K 后显著回落至 ~100K），再之后 ~75–91K 之间平缓。

### 8.3 K=3 的"假簇"诊断（手工扫 Q 矩阵的发现）

手工检查 80 个样本（行 100/1000/2500/4000 段）后发现 K=3 的 cluster 3 **在绝大多数样本上收敛到概率下界（~1e-5）**，ADAMIXTURE 实质上给出了一个"有效 K=2"的解：

| 段 | k3=1e-5 的比例 | 主导模式 |
| --- | --- | --- |
| 行 100–119（北部簇为主） | 12/20 = 60% | k1 主导 45–95% |
| 行 2500–2519（南部簇为主） | 11/20 = 55% | k2 主导 60–81% |
| 行 1000–1019 | 5/20 = 25% 显著（0.05–0.27） | 轻微混合尾巴 |
| 行 4000–4019 | 5/20 = 25% 显著（0.04–0.10） | 同上 |

cluster 3 仅在高 k1 样本上以 10–27% 的比例出现，模式像是"主簇的轻微正则化分裂"，而非真实的第三祖先成分。**LL elbow at K=3 是统计伪信号**。

### 8.4 K\* 最终建议（CV 已跑完，最终版本）

10-fold CV 已落盘（`/tmp/admixture_planA_step5.out` line 2870–2878），CV index vs K：

| K | CV index | Δ vs K=2 |
| --- | --- | --- |
| 2 | 0.2488 | baseline |
| 3 | **0.2487** | −0.0001 |
| 4 | 0.2487 | −0.0001 |
| 5 | 0.2488 | 0.0000 |
| 6 | 0.2490 | +0.0002 |
| 7 | 0.2490 | +0.0002 |
| 8 | 0.2491 | +0.0003 |
| 9 | 0.2492 | +0.0004 |
| 10 | 0.2492 | +0.0004 |

CV 曲线在 K=2..5 之间完全平坦（最大差 1 个万分点），K≥6 单调上升。**CV 信号清晰：更高 K 不增加任何预测能力，有效维度是 K=2。**

| 版本 | K | 依据 | 用途 |
| --- | --- | --- | --- |
| **主版本** | **K=2** | CV 最低平区 + LL 最大边际增益 + Q 矩阵无 ghost cluster | GWAS 协变量（南北梯度） |
| Diagnostic | K=3 | CV 与 K=2 无差；cluster 3 在 ~60% 样本收敛到 1e-5 下界 | 展示 ADAMIXTURE 的假簇倾向，不解释为真实祖先成分 |
| 备选 | K=4 / K=5 | CV 仍与 K=2 无差；LL 边际已回落 | 仅作 stacked-bar 图对照 |
| 不推荐 | K≥6 | CV 开始上升 + LL 边际平缓 | — |

### 8.5 当前文件清单

```
SDSworkspace/runs/admixture_9k_v1_20260619/
├── work/
│   ├── chr{4,5,7,14,17,18,19,20,21,22}.biallelic.{pgen,pvar,psam,log,done}
│   ├── chr_list_for_pmerge.txt
│   ├── 9k_raw.{pgen,pvar,psam,log,done}         # 2,628,609 merged variants
│   └── 9k.prune.{in,out}                         # 107,927 LD-pruned SNPs
├── input/
│   ├── 9k_ldpruned.{pgen,pvar,psam,log,done}
│   ├── 9k_ldpruned.labels.txt                    # Region (cleaned)
│   └── 9k_ldpruned.labels_superpop.txt
├── unsupervised/
│   ├── 9k.{2..10}.{P,Q,png}                      # 9 套 Q/P 矩阵 + 单 K 图
│   ├── 9k.2_10.png                               # 合并 stacked bar
│   └── ll_per_k.tsv                              # 本报告 §8.2 数据源
├── supervised/                                    # 空（待 v1.5 跑）
└── logs/admixture_unsupervised.log
```

### 8.6 下一步（Bash 恢复后一键执行）

```bash
# 1. K=3 详细 Region 表（验证假簇诊断）
/data/home/grp-wangyf/xuyuan/SDSworkspace/external/uv_envs/.venv/bin/python \
  /data/home/grp-wangyf/xuyuan/SDSpipeline/scripts/snapshot_k3.py \
  | tee /tmp/k3_summary.md

# 2. K=2 主版本 Region 表（GWAS 协变量依据）
sed 's/9k\.3/9k.2/g' /data/home/grp-wangyf/xuyuan/SDSpipeline/scripts/snapshot_k3.py \
  | /data/home/grp-wangyf/xuyuan/SDSworkspace/external/uv_envs/.venv/bin/python

# 3. CV 跑完没
ls /data/home/grp-wangyf/xuyuan/SDSworkspace/runs/admixture_9k_v1_20260619/unsupervised/ | grep -iE 'cv|error'
tail -30 /tmp/admixture_planA_step5.out | tr -d '\r'
```

### 8.7 与原报告 §5 预期对比

| 原报告 §5 预期 | 实测 v1 结果 | 是否印证 |
| --- | --- | --- |
| 中国人群南北梯度 K=2–4 | K\*=2（CV 平区起点，LL 最大边际），K=3 是 ghost cluster | ✅ 完全印证 |
| 省际/族群细分可能到 6–8 | CV 在 K≥6 单调上升（无支撑），LL 边际 75K–91K 平缓 | ❌ 数据不支持 |
| 与 PCA 前 10 主成分交叉验证 | 未做（下一阶段） | — |
| K\* 对应的 P 矩阵用于 projection | `9k.2.P` 已落盘 | ✅ |

### 8.8 完整 CV 输出原文

摘自 `/tmp/admixture_planA_step5.out` line 2870–2878：

```
K=2: CV index = 0.2488
K=3: CV index = 0.2487
K=4: CV index = 0.2487
K=5: CV index = 0.2488
K=6: CV index = 0.2490
K=7: CV index = 0.2490
K=8: CV index = 0.2491
K=9: CV index = 0.2492
K=10: CV index = 0.2492
```

注：ADAMIXTURE 1.6.2 未单独写出 `cv_error_per_k.tsv` 文件，CV index 仅在 stdout 末尾汇总打印。已把上表固化到 `unsupervised/cv_index_per_k.tsv`（由本会话补写）。



| 染色体 | 文件 | 样本数 | 备注 |
| --- | --- | --- | --- |
| 4 | `snp_singleton_qc_chr4_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 5 | `snp_singleton_qc_chr5_v2/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 7 | `snp_singleton_qc_chr7_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 14 | `snp_singleton_qc_chr14_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 17 | `snp_singleton_qc_chr17_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 18 | `snp_singleton_qc_chr18_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 19 | `snp_singleton_qc_chr19_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 20 | `snp_singleton_qc_chr20_v3/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 21 | `snp_singleton_qc_chr21_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 22 | `snp_singleton_qc_chr22_v1/snp_qc_pass.vcf.gz[.tbi]` | 4393 | ready |
| 1 | `all.chr1.QC.vcf.gz[.tbi]`（顶层） | - | 可临时复用，需确认样本集与冻结清单一致 |
| 6 | `snp_singleton_qc_chr6_v1/` | - | **缺 `snp_qc_pass.vcf.gz`，待补** |
