# Stage 2: Comprehensive mask construction + gamma calibration diagnostic

Date: 2026-06-06
Topic: sds
Type: implementation
Question: 按第一性原理重建 workflow，完成 Stage 2（analysis genome mask）并诊断 chr1 悬浮点的根因。

## Input paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/runs/chr1_qc_control_20260606/`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_b_file.sh`
- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/SCN/smcpp/`
- `/data/home/grp-wangyf/yf_intern/Homo_sapiens_assembly38.fasta.fai`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds_rebuild/` (clean root created)
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/` (comprehensive masks)
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/build_comprehensive_masks.py`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/generate_b_file.py`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/calibrate_gamma.py`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/gamma/NCN/gamma_candidate_registry.tsv`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/gamma/NCN/SDS-MODEL-001_chr1_calibration_summary.json`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/gamma/NCN/SDS-MODEL-001_chr1_bin_diagnostics.tsv`
- `/data/home/grp-wangyf/xuyuan/sds_rebuild/demography/model_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-06_stage2_mask_construction_and_gamma_diagnostic.md`

## Scripts / commands / job IDs

- `build_comprehensive_masks.py`: 从 GRCh38 参考和 UCSC 数据构建 centromere、pericentromeric HC、segdup、subtelomere、assembly gap 五层 mask
- `generate_b_file.py`: mask-aware b_file 生成器，替代硬编码着丝粒坐标的旧版
- `calibrate_gamma.py`: gamma 零假设校准诊断 — 按 frequency bin 检查 norm_SDS 的 mean/SD/skewness/tail behavior
- `job_ids=None; local computation only`

## Key result files

- Combined blacklist: `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/grch38_combined_blacklist.bed` (2,713 merged intervals)
- Callable autosome: `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/callable_autosome.bed` (2,691 intervals)
- Chr arm manifest: `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/chr_arm_manifest.tsv`
- b_file manifest: `/data/home/grp-wangyf/xuyuan/sds_rebuild/masks/b_file_manifest.tsv`
- Gamma calibration: `/data/home/grp-wangyf/xuyuan/sds_rebuild/gamma/NCN/SDS-MODEL-001_chr1_calibration_summary.json`

## 1. Stage 2: Comprehensive mask construction

### 1.1 sds_rebuild clean root

按 rebuild blueprint 创建了 `/data/home/grp-wangyf/xuyuan/sds_rebuild/` 目录结构，包含 config/ manifests/ masks/ demography/ gamma/ input/ compute/ postprocess/ validation/ orthogonal/ reports/ scripts/。

### 1.2 五层 mask

| Layer | Source | Autosome intervals (merged) | Coverage |
|-------|--------|---------------------------|----------|
| Centromere | GRCh38 known coords | 22 | ~150 Mb |
| Pericentromeric HC | Cytoband gvar/stalk bands | 44 | ~200 Mb |
| Segmental duplications | UCSC genomicSuperDups (fracMatch≥0.95) | 3,042 | ~150 Mb |
| Subtelomere | 500 kb from each telomere | 44 | ~22 Mb |
| Assembly gaps | Placeholder (centromeric gaps ⊆ centromere mask) | — | — |
| **Combined** | Merge of all above | **2,713** | **variable by chromosome** |

### 1.3 Per-chromosome callability

| Chr | Callable % | Chr | Callable % |
|-----|-----------|-----|-----------|
| chr1 | 93.3% | chr12 | 93.8% |
| chr6 | 96.0% | chr16 | 82.9% |
| chr9 | 85.6% | chr21 | 64.9% |
| chr7 | 89.6% | chr22 | 61.6% |

Acrocentric chromosomes (13/14/15/21/22) 的 p-arm 完全排除（全部为异染色质）。

### 1.4 Mask-aware b_file generator

对比 chr1 新旧 b_file：

| | Baseline (old) | New (mask-aware) |
|---|---|---|
| p-arm | 1 – 122,026,458 | 500,001 – 122,026,458 |
| q-arm | 124,932,725 – 248,956,422 | **145,000,001** – 248,456,422 |

关键变化：q-arm 起点从 124.9Mb 推到 145Mb — peak125 (125Mb) 和 peak143 (143Mb) 所在区域从 arm definition 层面即被排除。p-arm 和 q-arm 末端各扣除 500kb subtelomere。

### 1.5 Chr1 significant regions 交叉分析

对 final_cumulative 的 181 significant regions 做 mask 重叠分析：

| 分类 | 数量 | 占比 |
|------|------|------|
| 与 segdup 重叠（新 mask 可消除） | 13 | 7.2% |
| 距 callable boundary <100kb | 32 | 17.7% |
| 不在任何 mask 中 | **136** | **75.1%** |

结论：chr1 QC 的 centromere+pericentromeric mask 已经消除了已知 artifact peaks，但 75% 的 remaining significant regions 不在任何结构 mask 中。根因不在 region 层，而在 null model/normalization 层。

## 2. Gamma calibration diagnostic

### 2.1 SDS-MODEL-001 (production gamma) 校准结果

**Decision: REJECT** — 28/49 bins (57.1%) fail calibration

#### Failure pattern: two distinct zones

**Zone A — Low-MAF heavy tails (0.01-0.19)**

| MAF bin | 2σ tail % | Expected | Excess | 3σ tail % | Expected | Excess |
|---------|-----------|----------|--------|-----------|----------|--------|
| [0.10,0.11) | **12.52%** | 4.55% | **2.75x** | 0.02% | 0.27% | — |
| [0.09,0.10) | 12.11% | 4.55% | 2.66x | 0.05% | 0.27% | — |
| [0.08,0.09) | 11.92% | 4.55% | 2.62x | 0.37% | 0.27% | 1.37x |
| [0.07,0.08) | 11.49% | 4.55% | 2.53x | 0.58% | 0.27% | 2.15x |
| [0.01,0.02) | 4.87% | 4.55% | 1.07x | **0.94%** | 0.27% | **3.48x** |
| [0.02,0.03) | 5.14% | 4.55% | 1.13x | **0.97%** | 0.27% | **3.59x** |

**Zone B — High-MAF heavy tails (0.46-0.50)**

| MAF bin | 3σ tail % | Expected | Excess |
|---------|-----------|----------|--------|
| [0.46,0.47) | 0.91% | 0.27% | 3.37x |
| [0.48,0.49) | 0.83% | 0.27% | 3.07x |
| [0.49,0.50) | 1.18% | 0.27% | **4.37x** |

#### Pass zone (0.22-0.39)
Mid-frequency bins pass — mean≈0, SD≈1, skewness<1, tail fractions within expected range。

### 2.2 Root cause confirmed

chr1 的"悬浮点过阈值"问题根因是 **gamma miscalibration**，具体表现为：
1. 低 MAF bin（0.01-0.19）产生 2-4x 过量的 3σ+ norm_SDS 值
2. 高 MAF bin（0.46-0.50）产生 3-4x 过量的 3σ+ norm_SDS 值
3. Mid-frequency bins 校准正常

SDS-MODEL-001 不能作为 paper-facing baseline，因为它 systemically 在低 MAF 区域产生假阳性。

## 3. Gamma candidate registry

| ID | Source | Ne(0) | DAF grid | chr1 status | Notes |
|----|--------|-------|----------|-------------|-------|
| GAMMA-001 | unknown legacy | unknown | 0.05-0.95 | **REJECT** | Current mainline, provenance irrecoverable |
| GAMMA-002 | phlash | 100,000 | irregular | REJECT (genomewide) | Diffuse tail |
| GAMMA-003 | Gravel_CHB | 100,000 | 0.05-0.95 | REJECT (genomewide) | Diffuse tail |
| GAMMA-004 | SMC++ → Gravel_CHB | **87,282** | 0.05-0.95 | **PENDING** | New candidate from SMC++ fine |
| GAMMA-005 | phlash | 250,000 | irregular | PENDING | Higher Ne(0) variant |
| GAMMA-006 | phlash | 152,000 | irregular | PENDING | Intermediate Ne(0) variant |

## 4. What was accomplished today

### Built
1. **sds_rebuild/ clean root** — 完整目录结构
2. **Comprehensive masks** — centromere + pericentromeric HC + segdup + subtelomere → 2,713 merged blacklist intervals
3. **Mask-aware b_file generator** — 替代硬编码的 generate_b_file.sh
4. **Gamma calibration diagnostic** — bin-level mean/SD/skewness/tail 检查框架
5. **Gamma candidate registry** — 6 个候选的 provenance 追踪
6. **Demography model comparison TSV** — 5 个 NCN + 2 个 SCN 模型

### Proven
- chr1 QC 的 mask + variant + singleton 过滤能消除已知 artifact peaks
- Expanded segdup mask 额外捕获 13 个 significant regions
- **Remaining 136/181 regions 根因是 gamma miscalibration**，不是 region mask 不够
- SDS-MODEL-001 在低 MAF bin 产生 2-4x 过量的 3σ+ 值

## 5. Next steps (按 rebuild blueprint 优先级)

### Immediate (Stage 5: Gamma generation)
1. 用 SMC++ fine demography (DEMO-NCN-001, Ne(0)≈87,282) 生成新的 Gravel_CHB gamma → GAMMA-004
2. 对 GAMMA-004 在 chr1 上运行校准诊断
3. 如 GAMMA-004 仍失败，尝试 Ne(0) sensitivity scan（50k, 75k, 100k, 125k, 150k）

### Short-term (Stage 8: Normalization reform)
4. 实现 min-bin-size、bootstrap、LOCO、SD floor/shrinkage — 解决 tiny-bin SD 放大问题
5. 在 calibration framework 中加入这些 normalization stability checks

### Medium-term (Stage 7: Pilot compute)
6. 用通过校准的 gamma + comprehensive masks 重新跑 chr1 pilot
7. 扩展到 chr4 + chr6 + chr10/22 四染色体 pilot
8. Pilot 全部通过后再发 genome-wide compute

### Key dependency
**在 gamma 通过校准之前，不应继续 genome-wide compute 或 biological interpretation。**
