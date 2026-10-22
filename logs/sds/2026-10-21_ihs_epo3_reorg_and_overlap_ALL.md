# iHS epo3 全群体重算、出图重绘与目录整理

Date: 2026-10-21  
Topic: sds / orthogonal / iHS  
Status: complete

## Question

按新的 iHS 计算规则（epo3 三层 EPO 规范化）重算全部 iHS 结果，重绘所有出图，删除旧规则产物，整理目录结构。

## Decision

- 重算群体：`ALL`（UKB 4383）+ 1KG 五群体 `AFR/EUR/EAS/SAS/AMR`。
- `NCN`、`SCN` 不再需要：`NCN` 的旧结果与 epo3 结果已删除；`SCN` 本就没有 iHS 运行结果。
- 旧结果与旧图直接删除。原始 selscan 输出 `.ihs.out` 保留，可随时重算。

## New canonical rule

`SDSpipeline/scripts/orthogonal/polarize_norm_ihs_epo3tier.py`

- in_EPO=1（EPO 大写）：正常极化并归一化。
- in_EPO=2（EPO 小写）：用 in_EPO=1 的 DAF 分箱均值/标准差归一化。
- in_EPO=0（无注释/歧义）：`NA`，不参与归一化。
- 输出 `pos DAF raw_ihs norm_ihs extreme AA DA in_EPO flag`，Johnson 约定（负值 = derived 延长）。

脚本已适配新目录：原始输入 `raw/{POP}/`，规范化输出 `epo3/{POP}/`。

## Directory structure

```
results/production/orthogonal/ihs/
├── 1KG_variant_metadata.tsv      # REF/ALT 注释源（重算用）
├── maps/                         # 遗传图谱
├── raw/{ALL,AFR,EUR,EAS,SAS,AMR}/  # selscan 原始输出 chr*.ihs.out + 运行日志
├── epo3/{ALL,AFR,EUR,EAS,SAS,AMR}/ # chr*.ihs.100bins.norm.epo3（各 22 条）
├── plots/
│   ├── manhattan/                # 6 单体图 + 6 面板合并图
│   ├── sfs/                      # sfs_sds_ihs_comparison.png
│   └── compare/                  # r1r2_sds_ihs_ALL.txt
├── analysis/                     # genome-wide overlap 结果、known_sweeps.bed、legacy crossref
└── logs/                         # epo3 运行日志
```

## Deleted

- `ihs/{ALL,AFR,EUR,EAS,SAS,AMR}/chr*.ihs.100bins.norm`（旧规则 100 bins）
- `ihs/NCN/`（含旧 `.ihs.out.100bins.norm` 与 epo3 结果）
- `ihs/{POP}/chr*.ihs.top_hits.tsv`、`.meta_cache_*.tsv`、`polarize*.log`
- `ihs/plots/ihs_genomewide_for_cmplot.tsv`（319M）、`ihs_genomewide_masked_for_cmplot.tsv`（308M）、两张 CMplot PNG
- `sds_output/ALL/plots/ihs_manhattan/`（13 张旧规则 Manhattan PNG）
- `orthogonal/genome_wide_overlap_*`（NCN 版旧结果）
- `master_snp_table.tsv` 移至 `SDSworkspace/tmp/`（中间产物）
- `ihs_sds_crossref.tsv` 移至 `analysis/legacy_ihs_sds_crossref_oldnorm.tsv`（含旧 iHS 列）

## Results after rerun

### Manhattan

- 6 个群体全部由 epo3 重绘，ALL 有效位点 4,940,425。
- 输出：`ihs/plots/manhattan/ihs_manhattan_{POP}.png` 与合并图。

### Genome-wide overlap（ALL，100 kb，|SDS|>4 vs |iHS|>4）

- SDS 极值信号 3,035；iHS 极值信号 9,780。
- SDS-only：2,060（67.9%）；shared：975（32.1%）。
- DAF：SDS-only 中位数 0.237，shared 中位数 0.246。
- shared 符号一致率（SDS 正/iHS 负）：398/975（40.8%）。

### SFS（`plots/sfs/sfs_sds_ihs_comparison.png`）

- 上：SDS 极值按 DAF 分 20 bin，堆叠显示 SDS-only（67.9%）与 shared。
- 下：iHS 极值同分 bin，iHS-only 8,743/9,780（89.4%）。

### chr19 R1/R2（ALL）

- R1：19 leads，|SDS| 均值 6.17，|iHS| 均值 0.52，0 个 |iHS|>4 → 全部 young-cand。
- R2：14 leads，|SDS| 均值 5.64，|iHS| 均值 1.02，0 个 |iHS|>4 → 13 young-cand + 1 SDS-only。
- 与 NCN 结论一致：R1/R2 仍是“SDS 强、iHS 弱”的年轻信号。

## Caveats

1. ALL 的 iHS 极值数量远多于原 NCN（9,780 vs 432）。对 chr1 用旧式频段归一化复算得到 937 个极值，与 epo3 的 555 同量级，说明是 ALL 数据本身重尾，而非 epo3 造成。
2. 因此 shared 比例随群体样本量与统计功效变化，NCN 版 9.7% 与 ALL 版 32.1% 不可直接比较。
3. “nearest iHS within 100 kb” 并不保证同一变异/同一 sweep，shared 符号一致率不能单独作为独立起源证据。
4. in_EPO=0 位点输出 `NA`，Manhattan/SFS/overlap 均跳过；ALL 每条约 5% 位点因此不参与归一化。

## Canonical outputs

- `ihs/epo3/{POP}/chr*.ihs.100bins.norm.epo3`
- `ihs/plots/manhattan/ihs_manhattan_{POP}.png`
- `ihs/plots/manhattan/ihs_manhattan_ALL_plus_1KG_5pop.png`
- `ihs/plots/sfs/sfs_sds_ihs_comparison.png`
- `ihs/plots/compare/r1r2_sds_ihs_ALL.txt`
- `ihs/analysis/genome_wide_overlap_window100kb.tsv`
- `ihs/analysis/genome_wide_overlap_summary.txt`

## Post-plot QA (2026-10-21, interpretation round)

### Is |norm_iHS| > 4 too permissive?

Yes — it is a nominal per-SNP cutoff, not an FDR/genome-wide threshold.

- ALL: 9,780 extreme sites over 4,940,425 tested sites (0.20%).
- Exact N(0,1) expectation for |z|>4: 0.0063% → ~310 sites. Observed is ~31x that,
  and the old-rule normalization on chr1 gives 937 vs epo3 555, i.e. the heavy tail
  is inherent to selscan iHS, not introduced by epo3.
- Extreme sites are strongly autocorrelated: 100 kb clumping of the 9,780 sites gives
  2,647 clumps (2,832 when per-chromosome boundaries are treated separately).
  Top 10 clumps hold 14.6% of sites, top 50 hold 30.0%; largest is MHC
  chr6:29.67-30.00 Mb with 482 sites.
- Known sweeps (EDAR, ALDH2, ADH1B, MHC, OCA2/HERC2) contain only 9.9% of ALL
  extremes (965 of those in MHC). Masking known sweeps does not explain the bulk.

Recommended: report peak-level counts and use an empirical null / permutation or
q-value, not the raw |z|>4 site count.

### Extreme rate by DAF (ALL iHS)

0.05-0.10: 0.494%; 0.10-0.15: 0.305%; 0.15-0.20: 0.201%; mid-range 0.04-0.12%;
0.85-0.90: 0.147%; 0.90-0.95: 0.554%.

The low-DAF excess and the 0.90-0.95 bump are both enriched for in_EPO=2
(low-confidence ancestral annotation): 2,202/3,785 low-DAF extremes and
561/881 high-DAF extremes. Restricting to in_EPO=1 reduces extremes from 9,780
to 4,745 and 100 kb peaks from 2,832 to 1,467.

### Data defects found and fixed

- EUR chr19: a header-only `.meta_cache_19.tsv` left by an interrupted run was
  silently reused, so all 141,793 sites became `no_meta` and the chromosome had no
  normalized values. Cache rebuilt and chr19 rerun; epo3 now has 97,800 in_EPO=1
  sites. Script now rebuilds caches < 1 KB and aborts on an empty metadata dict.
- AMR chr20/21/22 raw selscan outputs are truncated (progress bars stopped at
  23%/68%/69%): chr20 ends at 12.52 Mb (19.4% of chromosome), chr21 at 31.92 Mb,
  chr22 at 34.96 Mb. The 1KG VCFs are not present on this server, so these cannot
  be rerun locally; AMR chr20-22 must be excluded from cross-population density
  comparisons until the upstream selscan run is repeated.

## SFS restricted to in_EPO=1 and LD-clump analysis (2026-10-21)

### SFS (iHS now in_EPO=1 only)

- iHS extreme signals drop from 9,780 to 4,745; iHS-only 4,521 (95.3%), shared 224 (4.7%).
- SDS panel unchanged: 3,035 extremes, SDS-only 2,060 (67.9%).
- The 0.90-0.95 DAF bump shrinks from 881 to ~300 sites, consistent with much of it
  being in_EPO=2 polarization noise.

### LD clumping (`scripts/orthogonal/clump_sds_ihs_signals_ld.py`)

- Genotypes: ALL dedup phased VCFs; greedy PLINK-style clump, |z| ranking,
  window 250 kb, r^2 >= 0.1. SDS and iHS clumped separately.
- SDS: 3,035 sites -> 246 clumps.
- iHS: 4,745 sites -> 1,831 clumps (low-DAF iHS signals are mostly single-site
  because rare variants have weak r^2).
- Sharing definitions: lead within 100 kb; any member within 100 kb; LD-shared
  (any cross-method member pair within 100 kb with r^2 >= 0.1).

| method | clumps | lead-shared | any-member-shared | LD-shared |
|--------|--------|-------------|-------------------|-----------|
| SDS | 246 | 41 (16.7%) | 52 (21.1%) | 13 (5.3%) |
| iHS | 1,831 | 33 (1.8%) | 36 (2.0%) | 16 (0.9%) |

- Co-localization permutation (circular shift of leads within chromosome, 1000x):
  SDS observed 41 vs expected 23.7 (p=0.058); iHS 33 vs 19.7 (p=0.076).
  Weak/borderline excess co-localization, not significant at 0.05.
- LD-shared vs only clumps: significantly larger (SDS median 10 vs 1 member,
  p=0.002; iHS 5 vs 1, p=1.5e-5) and wider (SDS p=0.003; iHS p=1.6e-4).
  DAF is not different (SDS p=0.20, iHS p=0.53); max|z| slightly higher for iHS
  (p=0.04) but not SDS (p=0.24).
- Size-stratified sharing: SDS 1.6% (size 1) -> 13.5% (size >=10); iHS 0.4% -> 5.6%.
  Among size>=2 clumps DAF is still not different (p=0.73 / 0.49). The size effect
  is therefore partly mechanical (more members = more chances to hit the other
  method), so clump-level "shared" is not independent evidence of a shared sweep.
- Outputs: `ihs/analysis/ld_clumps/{clumps_sds.tsv,clumps_ihs.tsv,pattern_tests.txt,clump_patterns.png}`.
