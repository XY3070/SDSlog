# 2026-08-06 SLE-family lead loci LocusZoom (ALL cohort, 2 genetic maps)

Date: 2026-08-06
Status: **COMPLETE**
Role: `mainline` — user-requested LocusZoom for 3 lead loci × 2 genetic maps (pyrho CHB + HapMap GRCh38)
Parent entries: 2026-08-05_oas_sle_vs_nonsle_check.md, 2026-08-05_mirror_manhattan_sle_vs_nonsle.md

## Request

3 lead positions, each drawn as its own LocusZoom with the lead labeled as
lead SNP (chr14 two points 190 bp apart — separate plots, separate leads):
`14:104928332`, `14:104928522`, `7:50266267`. Each plot in two versions —
recombination rate from **pyrho** (CHB) and from **HapMap** (plink
GRCh38 map). Output: `ALL/plots/SLE_family/`.

## Data / inputs

- SDS data: `ALL/all.normalized.tsv` (production release, t_file built 2026-07-18)
- LD: per-chromosome `snp_singleton_qc_chr{7,14}_v1/snp_qc_pass.vcf.gz`
  (contig without chr prefix; same VCF family as the OAS two-cohort LocusZoom)
- Maps: pyrho `input/reference/maps/CHB/` (pos, rate, cM) vs HapMap —
  **converted** from `plink.GRCh38.map.zip` layout
  (`no_chr_in_chrom_field/plink.chr{N}.GRCh38.map`, 4-col chr rsID cM bp) to
  pyrho-style 3-col via `oneoff/hapmap_map_to_pyrho_format.py` → new
  `input/reference/maps/HAPMAP_GRCh38/chr{N}.txt` (central-difference
  cM/Mb, negatives clipped, dup positions collapsed; chr7 183,805 markers,
  chr14 107,673).
- Bonferroni: 1.099916164e-08 (ALL genome-wide), red dashed line.

## ⚠️ Variant data gap: chr14:104928332:C:T has NO SDS p-value in the release

- **NOT a phasing issue.** All three cohorts' SDS inputs are built from
  `raw/9k/snp_singleton_qc_chr{N}_v1/snp_qc_pass.vcf.gz` (unphased, post-QC
  VCFs) — verified via the LSF build job command (ALL chr14: job 502018,
  `--vcf .../snp_singleton_qc_chr14_v1/snp_qc_pass.vcf.gz
   --sample-list .../frozen_samples_in_vcf.txt`) and
  `run_sle_nonsle_sds_workflow.sh` `resolve_vcf()` for SLE/non-SLE. The
  `input/raw/vcf/ALL/shapeit5/UKBQC_ALL_*.phased.vcf.gz` files are only used
  by the orthogonal modules (EHH/iHS/Relate), never by SDS input generation.
- **Real cause: EPO ancestral-allele polarization drop.** The t_file build
  log shows `[polarize] t_q.txt: 174437 in, 155722 out (10724 low-conf,
  4075 no-cov, 3916 mismatch, 50206 flipped) = 10.7% dropped`. This variant
  passes all pre-polarization filters in the ALL subset (ALT freq 0.61,
  HWE p=0.87, missing ~0) but its EPO ancestor base at chr14:104928332 is
  **lowercase `c` = low-confidence** (EPO convention: uppercase ACGT =
  high-confidence use, lowercase = DROP) → dropped from the site universe
  by design. Neighbor 104928522:G:A has uppercase `G` (kept); chr7:50266267
  uppercase `C` (kept). Site-universe contract, not a bug.
- Plot handling: canonical `plot_locuszoom.py` **patched** to draw a hollow
  lead diamond at baseline (y=0) + "lead not in SDS data" annotation instead
  of silently skipping (commit `8a88a0a`). LD r² vs this lead still computed
  from VCF (all 587 window variants have LD values).

## Code (canonical)

- `SDSpipeline/scripts/postprocess/plot_locuszoom.py` (committed):
  - `8a88a0a` — missing-lead fallback: hollow diamond at baseline +
    annotation + legend entry "Lead SNP (no p)"
  - `744d926` — `--ld-timeout` arg (default 300s; chr7 hapmap run needed
    900s on the 9k-sample VCF)
- `SDSworkspace/oneoff/hapmap_map_to_pyrho_format.py` (new)
- `SDSworkspace/oneoff/plot_sle_family_locuszoom.py` (new, batch runner)

## Run (6 plots, 300 dpi, flank ±250 kb + 5% pad)

| locus | map | variants | LD pairs | recomb range | genes |
|---|---|---|---|---|---|
| chr14:104928332 (lead: no p) | pyrho | 587 | 18,062 | 0.1–57.6 cM/Mb | 20 (2 lanes) |
| chr14:104928332 (lead: no p) | hapmap | 587 | 18,062 | 0.0–9.7 cM/Mb | 20 (2 lanes) |
| chr14:104928522 | pyrho | 587 | 18,061 | 0.1–57.6 | 20 (2 lanes) |
| chr14:104928522 | hapmap | 587 | 18,061 | 0.0–9.7 | 20 (2 lanes) |
| chr7:50266267 | pyrho | 1,111 | 16,457 | 0.1–4.5 | 7 (2 lanes) |
| chr7:50266267 | hapmap | 1,111 | 16,457 | 0.0–34.9 | 7 (2 lanes) |

- Ensembl gene track cached in `ALL/plots/locuszoom_cache/` (chr14 region:
  41 fetched/20 plottable; chr7: 17/7).
- chr7 hapmap LD initially timed out (300 s plink2 limit on 9k-sample VCF) —
  rerun with `--ld-timeout 900` ✓.
- Pixel verification (sampled): LD red r²≥0.8, lead purple diamonds
  (hollow for 104928332, filled for the other two), recomb blue line and
  low-LD gray points present in all 6 PNGs (~3100×2200 px).

## Outputs

- `ALL/plots/SLE_family/chr{14_104928332,14_104928522,7_50266267}_{pyrho,hapmap}.locuszoom.png`
  (6 PNGs) + `run_20260806.log`
- Archived → `SDSlog/plots/sds/2026-08-06_sle_family_locuszoom/`

## Notes / next

- chr14 两 lead 190 bp 间隔 → 两个 plot 窗口几乎相同；差异只在 lead
  diamond、LD 着色锚点与标题。
- chr14:104928332 无 p 值 plot 的 diamond 在基线。该位点被 EPO 极化
  drop (low-conf)，不是 VCF 版本问题 — 若需纳入分析须重评 EPO 置信度
  (site-universe 决策)，不在画图范畴。
- 肉眼看图：确认 hollow diamond 标注、两版 recomb 曲线差异、LD 块结构。

## 2026-08-06 追加: 数据来源审计 (用户质疑)

用户质疑 SDS 输入不应来自 phased VCF。审计结果 — **来源正确，我的
先前说法错误**：

| cohort | input VCF (evidence) |
|---|---|
| ALL | `raw/9k/snp_singleton_qc_chr14_v1/snp_qc_pass.vcf.gz` — LSF job 502018 (2026-07-18) 命令明文 |
| SLE / non-SLE | 同家族 — `run_sle_nonsle_sds_workflow.sh` `resolve_vcf()` → `raw/9k/snp_singleton_qc_chr{chr}_v1/snp_qc_pass.vcf.gz` |
| workflow 默认 | `run_sds_workflow.sh` VCF_PATTERN 默认 = `$SDS_COHORT_9K_ROOT/snp_singleton_qc_chr{CHR}_v1/snp_qc_pass.vcf.gz` |

- t_file 三 cohort sample order 均 ⊆ `frozen_samples_in_vcf.txt`；t_file
  全部 155,722 个 chr14 IDs 同时存在于 snp_qc_pass 与 phased VCF
  (交集，不能判别)，判别靠 build job 命令本体。
- `input/raw/vcf/ALL/shapeit5/UKBQC_ALL_*.phased.vcf.gz` 仅 EHH/iHS/Relate
  正交模块使用；phased VCF 08-03 重新生成与 SDS 输入无关。
- 早先总结中 "t_file 建于 07-18、phased VCF 08-03 重生成 → 位点不在
  site universe" 的因果解释**错误**（当时只看目录名推断）；真实因果 =
  EPO 极化 drop（上节）。
- 附带发现: chr14 p-arm t_file 为空 (t_p: 0 markers) — b_file p-arm
  边界 14:1-15999999 覆盖丝点/NOR 区，属 b_file 契约。
