# chr1 local compare plots

Date: 2026-05-11
Topic: `sds`
Type: `test`

Question: 在已经完成的 chr1 adjudication 结果上，做两组 chr1-only local-normalized pseudo-Manhattan compare 图：一组是 raw-joint `v2` 的 `NCN / SCN / MERGED3971`，另一组是 legacy adjudication 的 `NCN / SCN / MERGED3971PL`。目标是把 `peak125`、`peak143`、`control35` 三个固定窗口在三轨中的形态差异直接可视化。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_raw_joint_olddefault_chr1_v2/NCN/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_raw_joint_olddefault_chr1_v2/SCN/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_raw_joint_olddefault_chr1_v2/MERGED3971/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare_raw_joint_v2/NCN_SCN_MERGED3971_raw_joint_v2.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN_MERGED3971PL.summary.tsv`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/raw_joint_v2/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/legacy_adjudication/`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260511_chr1_local_compare_raw_joint_v2_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260511_chr1_local_compare_legacy_adjudication_v01.png`

Parameters:
- scope: `chr1` only
- local normalization:
  - `MAF = min(DAF, 1-DAF)`
  - common variants: `MAF > 0.01`
  - `maf_bin_width = 0.01`
  - within-bin `norm_SDS = (rSDS - BIN_MEAN) / BIN_SD`
  - `p_bothside = 2 * min(Phi(norm_SDS), 1 - Phi(norm_SDS))`
- plot metric: `-log10(p_bothside)`
- windows:
  - `peak125 = chr1:125100001-125200000`
  - `peak143 = chr1:143200001-143300000`
  - `control35 = chr1:35735220-36063259`
- note: 这是 chr1-local pseudo-significance，用于轨道间形态比较，不是 paper-facing genome-wide `p`

## Demographic model

| Track group | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `raw_joint_v2` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `audit-only` | raw-joint 只改变 input lineage |
| `legacy_adjudication` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `audit-only` | `MERGED3971PL` 只改变 parent-like merged lineage |

Scripts / commands / job IDs:
- `/data/home/grp-wangyf/xuyuan/sds/scripts/normalize_chr1_local_sds.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/report_chr1_compare_manhattan.R`
- normalization 在 `sds` mamba env 下完成
- 绘图最初失败于系统 `Rscript` 不在 `PATH`，随后改用 `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/Rscript`
- 绘图脚本还修复了一个 `data.table` 下标 bug：`load_summary_value()` 中参数名与列名 `key` 冲突

Key plots:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/raw_joint_v2/chr1_compare_raw_joint_v2.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/legacy_adjudication/chr1_compare_legacy_adjudication.png`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/raw_joint_v2/ncn_rawjoint_v2_chr1.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/raw_joint_v2/scn_rawjoint_v2_chr1.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/raw_joint_v2/merged3971_rawjoint_v2_chr1.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/legacy_adjudication/ncn_inputfix_chr1.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/legacy_adjudication/scn_inputfix_chr1.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_local_compare/legacy_adjudication/merged3971pl_chr1.summary.tsv`

Conclusion:
- raw-joint `v2` compare 图直接确认：
  - `NCN` 在 `peak125` 和 `peak143` 都有突出的高峰
  - `SCN` 与 `MERGED3971` 在这两个窗口没有对应峰
  - `control35` 在三轨都不安静，尤其 `SCN` 和 `MERGED3971` 仍有很高的 local pseudo-significance
- legacy adjudication 图与 raw-joint `v2` 方向一致：`NCN` 的两处 chr1 峰不是因最近这轮 cache race 或 raw-joint 重建才出现；shared artifact 仍存在
- 这两组图让 `peak125/peak143` 与 `control35` 的角色更清楚：
  - `peak125/peak143` 是当前专门盯的异常峰窗口
  - `control35` 原本是 quiet-control 候选，但现在已被证伪为“不安静”

Next step:
- 若继续追 chr1 机制，应转去更上游的 chr1 VCF / site-level forensic audit，而不是再反复微调 gamma 或 split-then-merge 近似。
