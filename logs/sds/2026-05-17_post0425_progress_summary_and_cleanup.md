Date: 2026-05-17
Topic: sds
Type: summary

Question

整理 2026-04-25 之后的主要工作，用于向老师汇报；同时在核对 `SDSlog` 后清理可无害删除的冗余中间文件、备份图和重复 manifest。

Progress summary

| Date | Work package | What was done | Key outputs | Current conclusion |
| --- | --- | --- | --- | --- |
| 2026-04-26 | `Ne(0)` scenario panel | 为 positive-region follow-up 整理 phlash posterior context、scaled `Ne(0)` main panel、high-side diagnostics 与 smoke 场景 | `scenario_manifest.tsv`, `scenario_aliases.json`, `submission_plan.json` | 形成正式 `Ne(0)` panel，后续 `100k/152k/...` 比较有统一入口 |
| 2026-05-02 | chunked gamma acceleration | 把 `scaled_ne0_100000` gamma generation 改成 chunked parallel path，并做 local smoke | `task_plan.tsv`, `submission_plan.json`, local chunk smoke canonical piece | gamma generation 不再是串行瓶颈，`100k` 轨道可直接组装 minimal `g_file` |
| 2026-05-02 | `100k` sensitivity + mainline reset | 启动 `scaled_ne0_100000` genome-wide compute；随后把主线切回 `historical old/default`，并并行发出 `Gravel_CHB@100k` compare 轨道 | `scaled_ne0_100000.minimal_g.tsv`, olddefault / Gravel launch logs | 主线正式从 phlash 100k 切回 `SDS-MODEL-001` |
| 2026-05-03 | gamma-track compare | 比较 `SDS-MODEL-001/002/003` 三条 Manhattan、tail counts、top windows 和 top hits | `NCN_gamma_tracks.summary.md`, olddefault / Gravel / phlash compare plots | 只有 `SDS-MODEL-001` 保留；`SDS-MODEL-002/003` 因 diffuse tail 被否决 |
| 2026-05-03 | filtered Manhattan redraw | 按 `p < 0.0005` 与 `0.01 < DAF < 0.99`, `MAF > 0.01` 重画过滤版 Manhattan | `cmplot/NCN.cmplot_p5e4_maf001.*` | olddefault 过滤后仍最像 real biology |
| 2026-05-05 ~ 2026-05-06 | chr1 `t_file` / legacy audit | 对 chr1 legacy/current input 做差分；测试 sparse `t_file`；确认 `legacy_0506` 的 `s_file` 全空失败不是 compute 问题 | `chr1_legacy_vs_current.../summary.md`, `chr1_sparse_shadow_q...`, `legacy0506` failure log | 简单稀疏化 `t_file` 不能修好 chr1；`legacy_0506` 因 singleton universe 为空不可直接用 |
| 2026-05-07 | chr1 input-fix | 收紧 `s/t/o` contract 到 `PASS + biallelic SNP`、subset-specific singleton、`o=1`，并发起 chr1 olddefault chunked compute | input audit sidecars, `NCN.summary.tsv` | input-fix 通过计数对账，但 chr1 异常峰只轻微回落 |
| 2026-05-08 ~ 2026-05-10 | chr1 adjudication | 补 `SCN` 和 merged/parent-like/raw-joint chr1 轨道，判断问题是 `NCN` 特异还是 shared artifact | `NCN_SCN.summary.tsv`, `NCN_SCN_MERGED3971PL.summary.tsv`, raw-joint compare | `SCN` 和 merged 也有异常 control；问题不是单纯 `NCN` 特异峰 |
| 2026-05-11 | chr1 local compare plots | 做两组 chr1 local pseudo-Manhattan compare 图（raw-joint 和 legacy adjudication） | `chr1_compare_raw_joint_v2.png`, `chr1_compare_legacy_adjudication.png` | `NCN` 的 `peak125/143` 仍高，`SCN/MERGED` 的 `control35` 也不安静 |
| 2026-05-11 | `Gravel_CHB@100k` + rebuilt new input genome-wide | 用重建 new input 启动 `NCN/SCN` 全基因组 Gravel 轨道；先做 smoke，再改成 chunked compute-only rerun | `sds_output_gravel_chb_ne100k_newinput_20260511/{NCN,SCN}` | 流程跑通，但图形 plausibility 仍差 |
| 2026-05-14 | CMplot replace + olddefault newinput rerun | 把 `Gravel` 官方 Manhattan 换成 CMplot 版；在同一套 rebuilt input 上复跑 olddefault | `NCN.manhattan.png`(CMplot), `sds_output_olddefault_newinput_20260514/NCN` | plotting 不是主因；同一 input 下 olddefault 明显更稳 |
| 2026-05-15 | chr1 + chr6 input-lineage audit | 系统比较 `legacy / oldsplit / rebuilt` 的 `site universe`，并做 same-input `olddefault vs Gravel` amplification audit | `input_compare.tsv`, `model_compare.tsv`, `tail_compare.tsv`, `chr6_oldsplit_only_filter_breakdown.tsv` | `chr1` 主差异在 `legacy/unpop-like` vs current pop-specific；`chr6` 的 old-only 位点全部是 non-PASS；`Gravel_CHB@100k` 会放大这些差异 |

Cleanup policy

- 保留：
  - `normalized.tsv`, `frequency_bins.tsv`, `significant_regions.tsv`
  - 官方 `CMplot` Manhattan
  - 合并后的 `chr*.sds.tsv`
  - diagnostics summary / top hits / top windows
  - 所有被 `SDSlog` 直接引用的失败证据、summary 和归档图
- 删除：
  - completed run 的 `chunk_submit/`, `cache/`
  - `*.parquet`
  - `chr*_p.sds.tsv`, `chr*_q.sds.tsv`
  - 未被 `SDSlog` 索引的临时 postprocess 目录
  - 重复 launch manifest
  - 已被 `CMplot` 官方图替代的 matplotlib 备份图
- 不删除：
  - repo 正式脚本
  - `SDSlog` 直接引用的 smoke / failure 证据
  - 尚未收口或仍可能作为追溯证据的 partial output

Deleted paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_postprocess_maf_daf_p5e4`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_postprocess_smoke`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_launch_20260511_150150`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/chunk_submit`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/cache`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/chunk_submit`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/cache`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/chunk_submit`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/cache`
- `sds_output_gravel_chb_ne100k_newinput_20260511/{NCN,SCN}` 下所有 `*.parquet`, `chr*_p.sds.tsv`, `chr*_q.sds.tsv`
- `sds_output_olddefault_newinput_20260514/NCN` 下所有 `*.parquet`, `chr*_p.sds.tsv`, `chr*_q.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.manhattan.matplotlib_backup_20260513.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.manhattan.matplotlib_backup_20260513.png`

Retained checkpoints

- `SDS-MODEL-001` mainline:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.normalized.tsv`
- `Gravel_CHB@100k` newinput:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.normalized.tsv`
- same-input olddefault rerun:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.normalized.tsv`
- chr1 / chr6 audit:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/summary.md`

Cleanup result

- 关键正式结果全部保留并复核通过：
  - `NCN/SCN Gravel newinput` 的 `normalized.tsv`, `CMplot Manhattan`, `chr*.sds.tsv`
  - `NCN olddefault newinput` 的 `normalized.tsv`, `CMplot Manhattan`, `chr*.sds.tsv`
- 近似释放空间：`~123.7G`
- 本轮未删除 repo 正式脚本；后续若要再清理，优先从未被 `SDSlog` 索引的 `tmp/` smoke / debug 目录入手。
