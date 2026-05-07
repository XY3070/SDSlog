# legacy_0506 `o_file` Fix and chr1 q-arm Launch

Date: 2026-05-06
Topic: sds
Type: test
Question: `legacy_0506` 的 `o_file` 是否符合 SDS 标准输入；修正后能否启动 chr1 q-arm 对照计算？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_o_file.callable_backup_20260506.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_b_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_o_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy_0506/chr1_o_file.callable_backup_20260506.txt`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy0506_q_20260506/chr1_legacy0506_q.484688.out`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy0506_q_20260506/logs/chr1_q.compute.csv`

## Parameters

- `sample_count=3971`
- `o_file_policy=all_ones`
- `gamma_source=old_default`
- `job_old=484687`
- `job_new=484688`
- `legacy0506_chr1_t_q_rows=2854878`
- `legacy0506_chr1_s_na_rows=3971`
- `legacy0506_chr1_s_non_na_rows=0`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_chr1_legacy0506_q.sh`
- `bkill 484687`
- `bsub 484688`

## Key plots

- None archived

## Key result files

- `SDS-027`: `/data/home/grp-wangyf/xuyuan/sds/tmp/chr1_legacy0506_q_20260506/chr1_legacy0506_q.484688.out`

## Conclusion

- `legacy_0506` 初始 `chr1_o_file.txt` 含 `3971` 个 callable counts，不符合当前 SDS 输入约定。
- 已备份 callable 版，并把正式 `chr1_o_file.txt` 改成单行 `3971` 个 `1`。
- 用修正后的 `o_file` 重提了 chr1 q-arm 计算，但 `484688` 仍失败；失败点不在 `o_file`，而在 `s_file`。
- `legacy_0506/chr1_s_file.txt` 的 `3971` 行全部是 `NA`，没有任何非空 singleton 轨道；`compute_SDS.py` 因 singleton 矩阵宽度为 `0` 在 `get_intervals()` 崩溃。
- 当前这套 `legacy_0506` 输入不能直接用于 SDS：q-arm `t_file` 已成功抽出 `2854878` 行 test SNP，但 `s_file` 完全为空，无法提供 singleton distance。

## Next step

- 回查 `legacy_0506` 的上游 `VCF/site universe`：确认这份 `t_file` 是否来自基本只有 common/test SNP 的位点集，因此 3971-sample cohort 下没有 singleton。
- 如果要在 3971-sample 输入上跑 SDS，`s_file` 必须改为从包含 rare variants / singleton candidates 的更完整位点集生成，而不是沿用当前这份全 `NA` 的结果。
