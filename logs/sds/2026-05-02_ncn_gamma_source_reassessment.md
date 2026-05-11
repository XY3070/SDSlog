# NCN gamma source reassessment

Date: 2026-05-02
Topic: sds
Type: summary
Question: 目前 split-cohort SDS 主线是否应继续使用 phlash-derived gamma，还是应切回 old/default gamma 作为更符合 real biology 的主锚点？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `/data/home/grp-wangyf/xuyuan/g_file.txt`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/sds_input.gamma_shapes.NCN.final`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_ablation_20260411_080350/summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/legacy_chr1_peak_check/summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/input_order_audit_20260411_074550/summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/benchmark_context.json`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-02_ncn_gamma_source_reassessment.md`

## Parameters

- `old_default_gamma=/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `old_default_gamma_rows=91`
- `old_default_gamma_daf_grid=0.05-0.95 step 0.01`
- `phlash_current_gamma=/data/home/grp-wangyf/xuyuan/ms/scripts/sds_input.gamma_shapes.NCN.final`
- `phlash_current_present_Ne0=1743549.3215311412`
- `phlash_scaled_100k_job=484533`
- `phlash_scaled_100k_postprocess_job=484534`
- `phlash_scaled_100k_role=sensitivity_only`

## Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `old_default_anchor` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `mainline` | 当前应切回的主线 |
| `phlash_current_gamma` | `phlash current q50-like shape` | `/data/home/grp-wangyf/xuyuan/ms/scripts/sds_input.gamma_shapes.NCN.final` | `1743549.3215311412` | `audit-only` | 这是当时正在使用的 phlash 形状文件，不应再混写成 old/default |
| `phlash_scaled_ne100k` | `SDS-MODEL-002` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | `rejected` | 只保留为 sensitivity/reference |

## Scripts / commands / job IDs

- `bjobs 484533 484534`
- `job 484533`: `sds_ne100k_compute_v2[1-22]%6`
- `job 484534`: `sds_ne100k_postprocess_v2`

## Key plots

- None archived

## Key result files

- `SDS-011`: `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `SDS-012`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_ablation_20260411_080350/summary.md`
- `SDS-013`: `/data/home/grp-wangyf/xuyuan/sds/tmp/legacy_chr1_peak_check/summary.md`

## Conclusion

- 当前正在运行的 `484533/484534` 使用的是 phlash `q50` shape 缩放到 `Ne(0)=100k` 的 gamma，不是 old/default gamma，也不是 repo 内显式标注的 `CHS` file。
- 现有审计支持把问题优先归到 gamma / demography 口径，而不是输入错序或 compute engine bug：input audit `172/172` checks 通过，legacy compute 能完整复现 chr1 异常峰，而 old vs new gamma ablation 显示换 gamma 会大幅改变 chr1 peak。
- 当前主线判断应切回 old/default gamma；phlash-derived `100k` run 只保留为 sensitivity/reference，不再参与 main biology judgment。

## Next step

- 用 old/default gamma 作为 mainline A，单独落到新的 output root。
- old/default 的 `Ne(0)=100k` 缩放版本作为 mainline B，但在没有明确 provenance 前不要把它直接命名为 `CHS` 主文件。
