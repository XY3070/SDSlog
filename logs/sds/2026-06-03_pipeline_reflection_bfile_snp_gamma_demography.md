# Pipeline 三问反思：b_file 着丝粒、SNP 保留策略、gamma 差异之谜

Date: 2026-06-03
Topic: sds
Type: summary

## Question

当前 pipeline 虽然已经产出 plausible Manhattan，但仍存在三个根本性的未解决问题需要系统反思：

1. b_file 为什么把着丝粒区域包含进了 q-arm / p-arm，导致 pericentromeric heterochromatin 中的 mapping artifact 被当作正常 SNP 计算？
2. 怎样的 SNP 才值得被纳入 SDS 计算？当前 PASS + biallelic SNP + MAF filter 是否足够？
3. olddefault gamma 原则上也来自 Gravel_CHB demography，为什么 shape 值与 SDS-MODEL-003 的 Gravel_CHB@100k 相差 2-27 倍？怎样的 demographic model 才是对的？

---

## 1. b_file 着丝粒包含问题

### 1.1 事实：b_file 的 arm 边界定义

b_file 格式为两行：第一行是 p-arm 范围（`start end`），第二行是 q-arm 范围（`start end`）。

| 染色体 | p-arm | q-arm start | GRCh38 着丝粒 | q-arm 与着丝粒重叠 |
|---|---|---|---|---|
| chr1 (NCN/SCN) | 1 - 122,026,458 | **124,932,725** - 248,956,422 | 122,026,459 - **125,184,587** | **252 kb 重叠**（q-arm 在着丝粒结束前就开始了） |
| chr6 (NCN) | 1 - 58,553,887 | **59,829,935** - 170,805,979 | 58,830,167 - **62,134,745** | **2.3 Mb 重叠**（q-arm 深入着丝粒 2.3 Mb） |
| chr9 (SCN) | 1 - 43,389,634 | 45,518,559 - 138,394,717 | 43,045,656 - 45,017,247 | 无重叠（q-arm 在着丝粒之后开始） |
| chr16 (SCN) | 1 - 36,311,157 | 38,265,670 - 90,338,345 | 36,311,158 - 38,265,670 | 无重叠（q-arm 恰好从着丝粒结束处开始） |

关键发现：
- **chr1** 的 q-arm 起始位点 124,932,725 **在着丝粒内部**（着丝粒延伸到 125,184,587）
- **chr6** 的 q-arm 起始位点 59,829,935 也**在着丝粒内部**（着丝粒延伸到 62,134,745）
- 这意味着 b_file 的 arm 边界**不是按着丝粒 gap 精确切割的**，而是按重组率或其他标准估算的

### 1.2 后果：pericentromeric heterochromatin 被纳入计算

chr1 的 1q12 pericentromeric heterochromatin（HSATII 卫星 DNA 密集区）从 ~125.1 Mb 一直延伸到 ~143.2 Mb。这恰好是 NCN Manhattan 上两个最强峰的位置：

| 峰 | 位置 | 蛋白编码基因 | 实际内容 |
|---|---|---|---|
| peak125 | chr1:125,148,000-125,178,000 | **0 个** | 着丝粒内部 / HSATII 卫星重复 |
| peak143 | chr1:143,209,000-143,287,000 | **0 个** | 1q12 异染色质远端边界 / HSATII 卫星重复 |

这两个峰在**全基因组显著区域排名中遥遥领先**：
- chr1:143,000,001-144,000,000：211/626 = **33.7%** 的 SNV 显著
- chr1:125,000,001-126,000,000：177/551 = **32.1%** 的 SNV 显著
- 全基因组其他 1 Mb 窗口：通常只有 1-3 个显著 SNV

### 1.3 根本原因

b_file 的 arm 边界生成逻辑没有显式排除着丝粒 gap（UCSC `cytoBand` 中的 `acen` 带）。它可能来自：
- 重组率图谱中的低重组区域自动切割
- 或历史代码中 hard-code 的近似值

无论原因是什么，**b_file 把着丝粒周围 252 kb (chr1) 到 2.3 Mb (chr6) 的区域纳入了 arm 计算范围**，使得大量短读长测序在卫星 DNA 上的 mapping artifact 被当作正常 test SNP 进入 SDS 计算。

### 1.4 修复方向

b_file 生成脚本应该：
1. 显式读取 GRCh38 的着丝粒坐标（UCSC `gap` table 中 `type='centromere'` 或 `cytoBand` 中 `gieStain='acen'`）
2. p-arm end = min(p-arm recombination boundary, centromere start - 1)
3. q-arm start = max(q-arm recombination boundary, centromere end + 1)
4. 可选：进一步排除 pericentromeric heterochromatin 注释（如 `gvar` 带）

---

## 2. SNP 保留策略反思

### 2.1 当前 pipeline 的 SNP 过滤链

当前 `run_sds_input.sh` 的 main contract：
1. PASS filter（VCF FILTER 字段）
2. biallelic SNP（排除 indel / multiallelic）
3. `AC >= 1`（至少有一个 alternate allele 被观察到）
4. `F_MISSING <= 0.005`（缺失率不超过 0.5%）

compute 阶段额外过滤：
5. `skip_boundary_missing_fraction`（边界 singleton 缺失比例）
6. `MAF >= 0.01`（postprocess 展示阈值，不是 compute 过滤）

### 2.2 当前过滤链遗漏了什么

**问题 A：着丝粒 / pericentromeric 区域的 variant 没有被排除**

PASS + biallelic SNP + F_MISSING <= 0.005 不能排除卫星 DNA 上的 variant。短读长 WGS 在 HSATII 区域会产生大量看起来 "PASS" 的假变异：
- 它们通过了 GATK 的 VQSR（因为该区域的系统性偏差是一致的）
- 它们是 biallelic SNP（因为 mis-mapping 模式是系统性的）
- 它们缺失率低（因为 reads 确实 map 到了那里，只是 map 错了）

**问题 B：singleton-only 的极端 rare variant（DAF=0.0004）产生不稳定的 SDS 估计**

chr6 MHC 区域有 ~2600 个 singleton SNV 的 rSDS 在 -13 到 -14 之间。这些是 DAF=0.0004 的 variant（只有 1 个杂合子），SDS 估计极其不稳定。当前 pipeline 在 compute 阶段没有对 DAF 下限做硬切割。

**问题 C：重组率极端区域的 SNP**

着丝粒周围和端粒附近重组率极低或异常。SDS 依赖重组驱动的 singleton 密度差异，在重组率极端区域的解释力存疑。

### 2.3 建议的 SNP 保留策略

| 层级 | 过滤条件 | 目的 | 当前状态 |
|---|---|---|---|
| Input 层 | PASS + biallelic SNP | 基础质量 | 已有 |
| Input 层 | `F_MISSING <= 0.005` | 缺失控制 | 已有 |
| **Input 层** | **排除着丝粒 gap 和 pericentromeric heterochromatin** | **消除 mapping artifact** | **缺失** |
| **Input 层** | **排除端粒 gap（subtelomeric ±N kb）** | **消除端粒 mapping artifact** | **缺失** |
| Compute 层 | `skip_boundary_missing_fraction` | 边界 singleton 缺失控制 | 已有，但阈值争议（见 relaxed035 / cap_to_boundary） |
| Postprocess 层 | `MAF >= 0.01` / `DAF in (0.01, 0.99)` | 展示过滤 | 已有 |
| **Postprocess 层** | **`DAF >= 0.005` 或更高** | **排除 singleton-only 的极 rare variant** | **缺失**（当前 DAF 下限是展示过滤，不影响 compute） |

核心原则：**不是所有能通过 GATK VQSR 的 SNP 都适合做 SDS**。SDS 对 variant calling 质量、mapping 可靠性和 singleton 分布的完整性有比一般 GWAS 更严格的要求。

---

## 3. Gamma file 差异之谜

### 3.1 直接对比：olddefault vs Gravel_CHB@100k

两个 gamma file 的 DAF 网格完全一致（0.05-0.95, step 0.01, 91 行），但 shape 值差异巨大：

| DAF | olddefault shape | Gravel_CHB@100k shape | 比值 (old/gravel) |
|---|---|---|---|
| 0.05 | 0.2468 | 0.7103 | **0.35x** |
| 0.10 | 0.1907 | 0.6721 | **0.28x** |
| 0.20 | 0.1352 | 0.6566 | **0.21x** |
| 0.30 | 0.0974 | 0.6415 | **0.15x** |
| 0.50 | 0.0570 | 0.6366 | **0.09x** |
| 0.70 | 0.0394 | 0.6300 | **0.06x** |
| 0.90 | 0.0280 | 0.6294 | **0.04x** |
| 0.95 | 0.0257 | 0.6295 | **0.04x** |

关键观察：
- **olddefault** 的 shape 从 DAF=0.05 的 0.247 **单调递减**到 DAF=0.95 的 0.026，变化幅度 ~**10 倍**
- **Gravel_CHB@100k** 的 shape 从 DAF=0.05 的 0.710 **基本平坦**到 DAF=0.95 的 0.630，变化幅度仅 ~**1.1 倍**
- 两者的比值从低频的 0.35x 递减到高频的 0.04x

### 3.2 shape 差异对 SDS 的影响

gamma shape 参数控制 SDS null distribution 的尾部厚度：
- **shape 大**（Gravel_CHB@100k: ~0.63-0.71）→ null 分布更宽 → 更多 SNP 落入极端尾 → diffuse tail Manhattan
- **shape 小**（olddefault: ~0.026-0.25）→ null 分布更紧 → 只有真正极端的 SNP 才显著 → 更 sparse 的 Manhattan

这就是为什么：
- Gravel_CHB@100k 产生多染色体弥漫性 extreme tail（被否决）
- olddefault 产生 sparse、localized 的 peak（被保留）

### 3.3 为什么 olddefault 和 Gravel_CHB@100k 差这么大？

这是一个**关键的未解之谜**。已知的事实：

1. olddefault 的 `g_file.txt` 是项目早期从 SDS 原作者（或相关 pipeline）获得的预计算 gamma table
2. 它的 provenance 记录为 `implicit / unknown from historical file provenance`
3. 它**可能**来自 Gravel et al. (2016) 的 CHB demography，但从未被明确确认
4. 如果 olddefault 确实来自 Gravel_CHB，那 shape 差异只能来自：
   - **不同的 Ne(0)**：olddefault 的 implicit Ne(0) 可能远小于 100,000（Ne(0) 越小，gamma shape 越小）
   - **不同的 gamma generation 方法**：可能是不同的 coalescent simulator、不同的 grid 分辨率、或不同的拟合策略
   - **可能根本不是 Gravel_CHB**：可能是 constant Ne 或其他简单 demography

5. 直接证据——2026-05-26 的 olddefault 复现 smoke 发现：
   - 当前 Makefile 的 gamma generation（基于 Gravel_CHB）在 DAF=0.05/0.07/0.10 给出 ~0.67-0.71 的 shape
   - 而 olddefault g_file 在同样 DAF 给出 ~0.19-0.25 的 shape
   - 两者无法 reconcile

### 3.4 怎样的 demographic model 才是对的？

这是一个需要重新审视的根本问题。当前状态：

| 考量 | olddefault (SDS-MODEL-001) | Gravel_CHB@100k (SDS-MODEL-003) | phlash q50@100k (SDS-MODEL-002) |
|---|---|---|---|
| Shape 值 | 0.026-0.25（递减） | 0.63-0.71（平坦） | 与 Gravel 类似的量级 |
| Manhattan 表现 | sparse peaks，局部化 | diffuse tail，多染色体 | diffuse tail，多染色体 |
| 是否符合 known biology | 部分符合（但 chr1 是 artifact） | 不符合（过多假阳性） | 不符合 |
| Provenance | 不明确 | 明确（Gravel_CHB + Ne(0)=100k） | 明确（phlash fit + Ne(0)=100k） |
| 可复现性 | 不可复现（来源不明） | 可复现 | 可复现 |

核心矛盾：
- **olddefault 产出最合理的 Manhattan，但 provenance 不明且不可复现**
- **Gravel_CHB@100k 和 phlash q50@100k provenance 明确且可复现，但产出 diffuse tail**

这意味着以下可能性之一：
1. **Ne(0) 不应该设为 100,000**：中国北方人群的有效群体大小可能远小于 100k，需要更小的 Ne(0) 来压低 shape
2. **Gravel_CHB 的 demography shape 本身有问题**：也许 Gravel 的 CHB 模型不适用于 WGS 数据或不适用于当前样本
3. **gamma generation pipeline 有 bug 或参数差异**：Makefile 的 gamma 生成过程与 olddefault 的原始生成过程有未知差异
4. **olddefault 根本不是 Gravel_CHB**：它来自一个完全不同的 demography（比如 constant Ne 或简单 bottleneck）

### 3.5 建议的验证步骤

1. **确定 olddefault 的来源**：
   - 检查 olddefault g_file.txt 是否能在任何已发表文献的 supplementary material 中找到
   - 检查 SDS 原作者提供的 R 脚本或文档中是否记录了 demography 参数
   - 对比 Gravel et al. (2016) 的 Supplementary 中公布的 gamma table

2. **Ne(0) 敏感性扫描**：
   - 用 Gravel_CHB shape 但 varying Ne(0) 从 10k 到 100k，找到哪个 Ne(0) 能复现 olddefault 的 shape range
   - 如果 Ne(0) ~ 10k-30k 能复现 olddefault，说明问题在 Ne(0) 而非 shape

3. **phlash / SMC++ 独立验证**：
   - phlash 拟合的 NCN demography 给出的 recent Ne 是多少？
   - SMC++ fine run 给出的 recent Ne 是多少？
   - 这些独立估计是否与 olddefault 的 implicit Ne(0) 一致？

4. **constant Ne baseline**：
   - 生成一组 constant Ne 的 gamma table（Ne = 5k, 10k, 20k, 50k）
   - 如果 constant Ne ~ 10k-20k 也能给出类似 olddefault 的 sparse Manhattan，说明当前数据对 demography shape 的分辨力不足，olddefault 的 success 可能只是因为它的 shape 恰好保守

---

## 4. 补充发现：chr6 MHC 区域缺乏信号

### 4.1 事实

在所有 baseline 中，chr6 MHC (28.5M-33.5M) **都没有 Bonferroni 显著的 positive selection 信号**：

| Baseline | chr6 MHC 显著区域 | chr6 整体 |
|---|---|---|
| NCN olddefault + rebuilt input | 0 个 | 5 个，全在 MHC 外 |
| NCN olddefault mainline (historical) | 0 个 | 4 个，全在 MHC 外 |
| SCN olddefault repaired | 0 个 | 0 个 |

MHC 正尾几乎空白（NCN 只有 1 个 rSDS>5 的 artifact SNP），负尾有大量 singleton（~2600 个，rSDS -13 到 -14，全是 DAF=0.0004 的不稳定估计）。

### 4.2 这与预期矛盾

MHC 是人类基因组中**已知正选择最强的区域之一**（HLA 基因的 balancing selection 和 directional selection 有大量文献支持）。SDS 原作者（Field et al. 2016）在英国人群中确实观察到了 MHC 信号。

### 4.3 可能原因

1. **olddefault gamma 的 shape 在 MHC 频率范围内过于保守**：如果 MHC 的选择信号主要体现在 intermediate frequency 的 allele 上，而 olddefault 的高 DAF shape 值极小（0.026-0.06），对应的 null 分布极窄，反而可能使得 intermediate rSDS 不够显著
2. **MHC 区域高重组率稀释了 singleton 模式**：SDS 依赖 singleton 密度差异，极高重组区域的 singleton 分布可能更均匀
3. **样本量不足**：NCN 1255 个样本对 MHC 这种高度多态但等位基因频率分布复杂的区域 power 可能不足
4. **variant calling 质量问题**：MHC 区域的 HLA 基因高度多态，短读长 mapping 在该区域的质量本身就不好

### 4.4 与 chr1 artifact 的对比

chr1 的两个 pericentromeric artifact 峰有数百个显著 SNV，而 chr6 MHC 有零个。这说明当前的 pipeline **对假信号敏感（着丝粒 artifact 被放大），对真信号不敏感（MHC 选择信号缺失）**。这进一步支持需要重新审视整个 input 层和 gamma 层的合理性。

---

## Key files

- olddefault gamma: `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- Gravel_CHB@100k gamma: `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- NCN chr1 b_file: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/NCN/chr1_b_file.txt`
- NCN chr6 b_file: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/NCN/chr6_b_file.txt`
- SCN chr9 b_file: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr9_b_file.txt`
- SCN chr16 b_file: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr16_b_file.txt`

## Conclusion

- **b_file 问题**：chr1 和 chr6 的 b_file 把着丝粒区域纳入了 arm 范围，导致 pericentromeric satellite DNA 上的 mapping artifact 进入 SDS 计算。chr1 的两个"最强峰"（125.1M / 143.2M）位于无基因的 1q12 异染色质，极大概率是 artifact。
- **SNP 策略**：当前 PASS + biallelic + F_MISSING 过滤不足以排除卫星 DNA 和极端 rare variant 的干扰。需要在 input 层增加基因组区域 mask（centromere, pericentromeric heterochromatin, subtelomeric）和 DAF 下限。
- **gamma 之谜**：olddefault shape (0.026-0.25) 与 Gravel_CHB@100k shape (0.63-0.71) 相差 3-27 倍。olddefault provenance 不明，可能是不同 Ne(0)、不同 demography 或不同 gamma generation 方法的结果。这是一个必须先解决的根本问题。
- **MHC 缺失**：chr6 MHC 在所有 baseline 中都无显著信号，与已知的人类正选择证据矛盾。说明 pipeline 对真信号的 power 或对 null model 的校准存在问题。

## Next step

1. 暂停新 compute run，优先回答 gamma provenance 问题
2. 追溯 olddefault g_file.txt 的确切来源（检查 SDS 原作者的 R 脚本、supplementary、邮件记录）
3. 做 Ne(0) 敏感性扫描（constant Ne + Gravel_CHB shape @ varying Ne(0)）
4. 修正 b_file 生成逻辑，显式排除着丝粒和 pericentromeric heterochromatin
5. 重新定义 SNP input contract：增加基因组区域 mask 和 DAF 下限
6. 用 iHS/EHH 对 chr6 MHC 做正交验证
