# SDSlog demography provenance backfill

Date: 2026-05-11
Topic: `sds`
Type: `summary`
Question: 把 `SDSlog` 中所有关键 SDS 结果的 demographic model、gamma shape source 和 `Ne(0)` 口径补成统一、可审计的记录，避免继续出现 `old/default` 或 `100k track` 这类模糊表述。

Input paths:
- `/data/home/grp-wangyf/xuyuan/SDSlog/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_log.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_Chinese_Cohort.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/_log_template.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/result_index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/plot_index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/`

Output paths:
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/README_demographic_models.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-11_sdslog_demography_provenance_backfill.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_log.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/SDS_Chinese_Cohort.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/_log_template.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/result_index.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/plot_index.md`

Parameters:
- backfill scope: `all SDS results`
- registry file: `README_demographic_models.md`
- standard fields:
  - `Demographic model`
  - `Gamma shape source`
  - `Present-day Ne(0)`
  - `Model role`
  - `Model note`
- standard model IDs:
  - `SDS-MODEL-001`: `historical_olddefault_gamma_table`
  - `SDS-MODEL-002`: `phlash_q50_scaled_ne100k`
  - `SDS-MODEL-003`: `Gravel_CHB_ne100k`
  - `SDS-MODEL-004`: `legacy_parentlike_input_context`
  - `SDS-MODEL-005`: `region_ne0_panel_family`

## Demographic model

- `Demographic model=documentation backfill only`
- `Gamma shape source=not applicable`
- `Present-day Ne(0)=not applicable`
- `Model role=audit-only`
- `Model note=这条记录本身不对应一条新的 SDS compute 轨道；它只登记 provenance schema 的回填`

Scripts / commands / job IDs:
- non-mutating audit:
  - `rg -n "old/default|Gravel_CHB@100k|scaled_ne0_100000|Ne\(0\)=100k" /data/home/grp-wangyf/xuyuan/SDSlog -g "*.md"`
- no cluster jobs

Key plots:
- None archived

Key result files:
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/README_demographic_models.md`

Conclusion:
- `SDSlog` 现在新增了一份 SDS demographic model registry，作为所有结果与图的单一真值。
- `README`、template、总览、result index、plot index 和多条关键 SDS detailed log 已回填 model / gamma / `Ne(0)` provenance。
- `README`、template、总览、result index、plot index 和多条关键 SDS detailed log 的高层叙述也已清理到 registry 口径，不再把 input lineage、gamma track 和 explicit `Ne(0)` 混写。
- `historical old/default gamma table` 现在被明确记录为：
  - gamma file=`/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
  - `Ne(0)=implicit / unknown from historical file provenance`
  - role=`mainline`
- `phlash q50 scaled` 与 `Gravel_CHB` 的 `100k` 轨道也已统一记录为显式 present-day scaling，并标为 rejected。

Next step:
- 后续所有新 SDS 记录先引用 `README_demographic_models.md`，再写结果本身。
- 如果以后继续补更早期的历史文档，仍按同一 registry 口径补齐，不再新增自由命名的 gamma track 称呼。
