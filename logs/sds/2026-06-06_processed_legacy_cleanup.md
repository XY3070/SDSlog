# 2026-06-06 Processed Legacy Cleanup

## Context

`/data/home/grp-wangyf/xuyuan/sds/data/processed` 长期同时承载了：

- scheduler manifest / submitter 残留
- compare / audit 工作目录
- 试验性 input 目录
- 一批零引用但名称过长的 legacy output

这样会带来两个直接问题：

- 顶层路径名本身已经混合“用途 + 时间戳 + job id”，很难一眼看出对象类型
- 即使不删除任何结果，后续也很难区分哪些是 canonical 目录，哪些只是旧实验残留

## Decision

采用保守 cleanup，而不是激进删除：

- 只硬删除确认为空壳的对象
- 对仍有 provenance 价值的目录统一收口到 grouped canonical roots
- 旧路径默认保留 compat symlink，避免打断旧日志或手工记录

本次在 `processed/` 下新建的 canonical roots 是：

- `legacy_scheduler/`
- `legacy_review/`
- `legacy_input/`
- `legacy_output/`
- `legacy_misc/`

## Implementation

已执行以下整理：

- 删除 5 个空壳对象：
  - `gravel_chb_ne100k_scn5432matched_chrlist_20260526_023616_17964`
  - `sds_output_gravel_chb_ne100k_grid001_ncn_20260525_101831`
  - `sds_output_gravel_chb_ne100k_grid001_ncn_20260525_101927`
  - `sds_output_gravel_chb_ne100k_grid001_ncn_20260525_102008`
  - `sds_output_scn_olddefault_genomewide_coarse_smp_20260528`
- 将 31 个 `*_chrlist_*` / `*_launch_*` / `*_relaunch_*` / `*_submitter_array_*` / `*_manual_*` 目录统一迁入 `legacy_scheduler/<track>/<kind>/<stamp>`
- 将 8 个 compare 目录与 1 个 audit 目录统一迁入 `legacy_review/compare/...` 与 `legacy_review/audit/...`
- 将 5 个试验性 input 目录统一迁入 `legacy_input/raw_joint/...` 与 `legacy_input/rebuilt_main_contract/...`
- 将 7 个零引用且名称过长的 legacy output 统一迁入 `legacy_output/...`
- 将 `test_freq_compare_workspace.tsv` 迁入 `legacy_misc/freq_compare_workspace.tsv`

逐项 old-path 到 canonical-path 的映射记录在：

- [2026-06-06_processed_cleanup_manifest.tsv](/data/home/grp-wangyf/xuyuan/SDSworkspace/provenance/datasets/2026-06-06_processed_cleanup_manifest.tsv:1)

## Consequence

现在 `processed/` 的 canonical 结构已经变成“按对象类型分组”，而不是继续把所有实验名直接摊在顶层。

兼容策略如下：

- 被保留的 legacy 名称现在大多只是 symlink
- 真正的实体目录已经下沉到 `legacy_*` roots
- 顶层不再保留普通文件；`test_freq_compare_workspace.tsv` 也已转成 compat symlink

## Verification

- `processed/` 顶层当前为 `34` 个 real directories、`53` 个 symlinks、`0` 个 plain files
- 检查结果为 `0` broken symlinks
- 五个被删空壳都已确认不存在
- 新分组目录当前体量为：
  - `legacy_input`: `998G`
  - `legacy_output`: `101G`
  - `legacy_review`: `2.5G`
  - `legacy_scheduler`: `227K`
  - `legacy_misc`: `8.5K`

## Remaining gap

这次没有动以下对象：

- 仍被 `SDSlog` 明确引用或尚未判定 superseded 的正式 output 目录
- `sds_input/` 这个通用入口
- `relate_clues2/` 这类尚未纳入本轮 cleanup contract 的大目录

因此这次 cleanup 的目标不是把 `processed/` 彻底做成“小而干净”，而是先把最混乱、最冗长、最难分辨类型的部分收口为可追溯的 grouped canonical layout。
