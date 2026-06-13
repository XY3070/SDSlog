# NyuWa SMC++ Resampling: NCN & SCN Demographic History

Date: 2026-06-12
Topic: demography
Type: production
Question: NCN 和 SCN 的 SMC++ resampling demographic history 结果如何？瓶颈时间和深度是否一致？

## Input paths

- Sample files: `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`, `SCN.txt`
- VCF source: `/data/home/grp-wangyf/xuyuan/sds/data/vcf/{NCN,SCN}/`
- Worker script: `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/run_smcpp_nyuwa_worker.py`
- Submit script: `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/submit_nyuwa_smcpp_lsf.sh`
- Average script: `/data/home/grp-wangyf/xuyuan/sds_rebuild/scripts/average_nyuwa_curves.py`

## Output paths

- NCN results: `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp_resampled/`
- SCN results: `/data/home/grp-wangyf/xuyuan/benchmark/demography/SCN/smcpp_resampled/`
- NCN averaged: `NCN_averaged_Ne.csv`, `NCN_resampling_summary.json`
- SCN averaged: `SCN_averaged_Ne.csv`, `SCN_resampling_summary.json`
- Per-shuffle CSVs: `shuffle_XX/{POP}_sXX_smcpp.csv` (10 per population)
- Per-shuffle plots: `shuffle_XX/{POP}_sXX_smcpp.png` (10 per population)

## Parameters

- `population`: NCN, SCN
- `n_shuffles`: 10 (s00-s09)
- `sample_size`: 200 per shuffle
- `n_distinguished_pairs`: 10
- `chromosomes`: 1-22
- `mutation_rate (mu)`: 1.25e-8
- `generation_time`: 29 years
- `estimate params`: `--knots 16 --timepoints 10.0 100000.0`
- `base_seed`: 20260607 (per-shuffle seed = base + shuffle_idx * 1000)
- `cores_per_job`: 32
- `vcf2smc parallelism`: 4 chromosome groups (A:1-6, B:7-12, C:13-18, D:19-22)
- Scheduler: LSF, queue=normal

## Results

### NCN (Northern Chinese)

| Generation | Years | mean_Ne | sd_Ne | CV |
|---|---|---|---|---|
| 0 | 0 | 301,282 | 229,603 | 0.762 |
| 1,000 | 29,000 | 10,443 | 708 | 0.068 |
| 2,000 | 58,000 | 3,106 | 947 | 0.305 |
| 10,000 | 290,000 | 18,887 | 4,972 | 0.263 |
| 100,000 | 2,900,000 | 41,828 | 3,668 | 0.088 |

- **Bottleneck**: Ne = 3,106 ± 947 at gen 2,000 (~58 kya)
- Ne(0) range: 11,368 – 529,048 (CV=0.76, sensitive to distinguished lineage sampling)

### SCN (Southern Chinese)

| Generation | Years | mean_Ne | sd_Ne | CV |
|---|---|---|---|---|
| 0 | 0 | 450,835 | 286,626 | 0.636 |
| 1,000 | 29,000 | 10,900 | 851 | 0.078 |
| 2,000 | 58,000 | 2,696 | 72 | 0.027 |
| 10,000 | 290,000 | 20,652 | 2,366 | 0.115 |
| 100,000 | 2,900,000 | 42,640 | 796 | 0.019 |

- **Bottleneck**: Ne = 2,696 ± 72 at gen 2,000 (~58 kya) — extremely stable across shuffles (CV=0.027)
- Ne(0) range: 10,848 – 984,988 (CV=0.64)

### Comparison

- Bottleneck timing identical (~58 kya, consistent with Last Glacial Maximum)
- SCN bottleneck deeper (2,696 vs 3,106) and more stable (CV 0.027 vs 0.305)
- Deep history (gen ≥ 1,000) nearly identical between populations
- Recent Ne diverges: SCN higher at gen 0, suggesting larger effective population size

## Issues encountered and fixes

1. **shared_vcf bug**: Original submission used `--shared-vcf-dir` pointing to a single VCF shared across all shuffles. Since each shuffle uses a different random seed to select 200 samples, the shared VCF did not contain samples for later shuffles → `RuntimeError: Distinguished lineages not found in data?`
   - Fix: Removed `--shared-vcf-dir`, each job generates its own per-shuffle subset VCF.

2. **Empty SMC files (44-45 bytes)**: Old failed shared_vcf jobs created empty gzip headers before crashing. Worker treated these as "already done" and skipped them.
   - Fix: `find -size -100c -delete` across all shuffle directories (76 files SCN, 21 files NCN).

3. **Truncated gzip files (CRC failure)**: Batch 2 kill during active vcf2smc left partially-written .smc.gz files > 100 bytes.
   - Fix: `gzip -t` integrity check, delete corrupt file, resubmit affected group.

4. **LSF `done()` dependency on purged jobs**: Completed/failed jobs aged out of LSF history, causing `No matching job found` errors for downstream dependencies.
   - Fix: Submit without `done()` dependencies when upstream jobs already completed; resubmit all 4 groups together.

5. **`seq -w 0 9` naming mismatch**: `seq -w 0 9` produces "0"–"9" but submit script uses `printf '%02d'` giving "00"–"09". Job names didn't match.
   - Fix: Use `printf '%02d'` consistently.

## Key plots

- [PLT-DEMO-010](../../plots/demography/20260612_NCN_nyuwa_smcpp_resampling_shuffles.png) — NCN 10-shuffle overlay
- [PLT-DEMO-011](../../plots/demography/20260612_SCN_nyuwa_smcpp_resampling_shuffles.png) — SCN 10-shuffle overlay

## Role

- `mainline` — primary demographic model for NCN and SCN used in SDS analysis

## Conclusion

NCN 和 SCN 共享 ~58 kya 瓶颈信号，深层历史高度一致。SCN 瓶颈更深更稳定。近期 Ne 估计方差大（对 distinguished lineage 选择敏感），属 SMC++ resampling 常见现象。结果可用于 paper。

## Next step

- 生成 NCN vs SCN overlay comparison plot
- 与 phlash/Relate 结果交叉验证瓶颈深度
