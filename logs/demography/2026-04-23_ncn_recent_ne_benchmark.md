# NCN recent Ne benchmark

Date: 2026-04-23
Topic: demography
Type: production
Question: 当前 NCN 的 recent `Ne` benchmark 用了哪些输入、产出了哪些核心文件和图，后续应该从哪里接入？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/sample_pool.intersection.txt`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/subset_100.samples.txt`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/subset_vcf`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/shapeit5`

## Output paths

- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/benchmark_manifest.json`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/recent_ne_inputs_manifest.json`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/phlash/NCN_model_full.pkl`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/phlash/NCN_full_Ne_plot.png`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/analysis/NCN.final.json`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/NCN_fine_smcpp.png`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/recent_sensitivity/NCN_recent_resolution_overlay.png`
- `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/relate_recent/popsize/NCN_relate_recent_smoke_chr21_22_cachedfix.png`

## Parameters

- `population=NCN`
- `eligible_count=1255`
- `subset_size=100`
- `num_distinguished_pairs=4`
- `pair_mode=self`
- `chromosomes=1-22`
- `seed=20260423`

## Scripts / commands / job IDs

- benchmark metadata captured in `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/benchmark_manifest.json`
- recent-input metadata captured in `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/recent_ne_inputs_manifest.json`
- scheduler job IDs were not backfilled into this note

## Key plots

- [PLT-DEMO-001](../../plots/demography/20260423_NCN_demography_phlash_full_Ne_v01.png)
- [PLT-DEMO-002](../../plots/demography/20260423_NCN_demography_smcpp_fine_v01.png)
- [PLT-DEMO-003](../../plots/demography/20260423_NCN_demography_smcpp_recent_overlay_v01.png)
- [PLT-DEMO-004](../../plots/demography/20260423_NCN_demography_relate_recent_smoke_chr21_22_cachedfix_v01.png)

## Key result files

- `DEMO-001`: `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/benchmark_manifest.json`
- `DEMO-002`: `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/recent_ne_inputs_manifest.json`
- `DEMO-003`: `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/phlash/NCN_model_full.pkl`

## Conclusion

- NCN recent `Ne` benchmark 的核心入口已经集中到 `benchmark_manifest.json` 和 `recent_ne_inputs_manifest.json`。
- phlash、SMC++、Relate 三条线都有可直接引用的核心产物和关键图。
- 后续所有 recent `Ne` 相关讨论应直接引用这条记录，而不是重新去目录里翻找入口文件。

## Next step

- 明确 paper-facing 的主比较口径，并把最终采用的 `Ne(t)` / `Ne(0)` 决策继续回填到 `logs/sds`。
