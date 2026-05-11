# NCN region Ne0 scenario panel

Date: 2026-04-26
Topic: sds
Type: production
Question: 用于 positive-region follow-up 的 `Ne(0)` scenario panel 具体有哪些场景，哪些进入主扫描，哪些只是诊断或 smoke？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/benchmark_context.json`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/phlash_percentiles.npz`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/region_panel.t.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_aliases.json`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/submission_plan.json`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/submitted_jobs.json`

## Parameters

- `q50_present_diploid_pop_size=1743549.3215311412`
- `posterior_context=q2.5,q25,q50`
- `main_scaled_ne0=100000,152000,250000,500000,750000,1000000`
- `diagnostic_ne0=2500000,5000000`
- `smoke_only=q97.5`
- `interval_eligible_main=6`

## Demographic model

- `Model family=SDS-MODEL-005 (region_ne0_panel_family)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv`
- `Present-day Ne(0)=mixed by scenario`
- `Model role=audit-only`
- `Model note=这里不是单一 production gamma；包含 phlash posterior context、scaled Ne(0) main panel、高侧 diagnostics 和 smoke`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/task_plan.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/submission_plan.json`
- `/data/home/grp-wangyf/xuyuan/sds/logs/region_ne0_parallel_full_direct_484305_1.out`
- `job_array=484305`

## Key plots

- None archived

## Key result files

- `SDS-005`: `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv`

## Conclusion

- 当前 panel 把 phlash posterior context、scaled `Ne(0)` 主面板、高侧 diagnostic 和 `q97.5` smoke 明确分开了。
- 主扫描应集中在 `Ne(0)=100k, 152k, 250k, 500k, 750k, 1e6` 这 6 个 interval-eligible 场景。
- `q50` 是当前缩放模板，`q97.5` 只用于能否跑通的 smoke，不进入主比较。

## Scenario classes

| Class | Labels | Use |
| --- | --- | --- |
| Posterior context | `q2.5_curve`, `q25_curve`, `q50_curve` | 参照组，不进入主 `Ne(0)` 区间判断 |
| Main scaled panel | `scaled_ne0_100000`, `152000`, `250000`, `500000`, `750000`, `1000000` | 正式比较 |
| High-side diagnostic | `scaled_ne0_diag_2500000`, `scaled_ne0_diag_5000000` | 只做高侧诊断 |
| Smoke | `q97p5_smoke` | 只看能否跑通 |

## Next step

- 用这套 panel 跑 sentinel region / positive region，后续把每个正式结果写成单独的 detailed log。
