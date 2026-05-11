# chr4:99317841:T:C gamma sensitivity

Date: 2026-04-16
Topic: sds
Type: test
Question: `chr4:99317841:T:C` 的 raw `rSDS` 对 gamma shape 和 constant `Ne` 有多敏感，方向是否稳定？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/target_snp.t.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/scenario_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/chr4_q.sds.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/final_report.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/final_report.json`

## Parameters

- `target_snp=chr4:99317841:T:C`
- `nG0=64; nG1=487; nG2=704`
- `DAF=0.7549800796812749`
- `1-DAF=0.2450199203187251`
- `scenarios=current_gfile,q2.5,q25,q50,q75,q97.5,constNe152000`
- `baseline_raw_rSDS=1.1458`
- `baseline_norm_SDS=3.3155`

## Demographic model

- `Control model=SDS-MODEL-001 (historical_olddefault_gamma_table)`
- `Control gamma shape source=/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `Control present-day Ne(0)=implicit / unknown from historical file provenance`
- `Scenario family=SDS-MODEL-005 (region_ne0_panel_family)`
- `Model role=audit-only`
- `Model note=q2.5/q25/q50/q75/q97.5 是 phlash posterior context；constNe152000 是显式 constant Ne(0)=152000；这条记录是 sensitivity，不是主 production track`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/scripts/scan_single_snp_gamma_sensitivity.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/report_single_snp_gamma_sensitivity.py`
- `/data/home/grp-wangyf/xuyuan/sds/logs/gamma_single_snp_finalize_fix_475262.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/gamma_single_snp_report_fix2_475263.out`

## Key plots

- None archived

## Key result files

- `SDS-003`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/summary.tsv`
- `SDS-004`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gamma_single_snp_adh_chr4_20260416_110231/final_report.md`

## Conclusion

- 在所有已完成情景下，raw `rSDS` 范围是 `0.5501` 到 `1.2539`，没有发生 sign flip。
- `q50` 的 raw `rSDS=1.1227`，和当前 `g_file` baseline `1.1458` 很接近；`constNe152000` 会把数值拉低到 `0.6868`。
- 和外部 Nyuwa 记录相比，NCN 在这个位点的方向一致，说明这里更应该关注方向与 rank stability，而不是 raw magnitude 的绝对一致。

## Scenario summary

| Scenario | rSDS | Delta vs control | Projected norm_SDS |
| --- | ---: | ---: | ---: |
| `current_gfile` | 1.1458 | 0.0000 | 3.3155 |
| `q2.5` | 0.5501 | -0.5957 | 1.8533 |
| `q25` | 1.2539 | 0.1081 | 3.5808 |
| `q50` | 1.1227 | -0.0231 | 3.2588 |
| `q75` | 1.0440 | -0.1018 | 3.0656 |
| `q97.5` | 0.6321 | -0.5137 | 2.0546 |
| `constNe152000` | 0.6868 | -0.4590 | 2.1889 |

## Next step

- 后续 `Ne(0)` sensitivity 主面板继续优先看 `norm_SDS` 和 peak rank，不把单个位点 raw `rSDS` 的波动当作主结论。
