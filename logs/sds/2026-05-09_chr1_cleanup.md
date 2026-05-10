# chr1 cleanup

Date: 2026-05-09
Topic: `sds`
Type: `summary`

Question: `chr1` adjudication 已完成，需要清理失败链和可再生中间件，保留最终 comparison、review 和最小失败证据，减少后续管理负担与磁盘占用。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971PL/`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/`

Parameters:
- cleanup policy:
  - keep final `chr1.sds.tsv`
  - keep `review/*.summary.tsv` and `review/*.top_hits.tsv`
  - keep small failure evidence for direct `MERGED3971` (`chr1_s_file.txt`, `o_file`, `b_file`, logs)
  - remove regenerated / duplicative middle files: `chunk_submit`, `cache`, arm-level `chr1_p/q.sds.*`, merged `*.parquet`

Scripts / commands / job IDs:
- local cleanup only; no new cluster job

Key result files:
- kept:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/chr1.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/chr1.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/chr1.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/review/SCN.chr1_inputfix_olddefault.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/review/MERGED3971PL.chr1_parentlike_olddefault.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN_MERGED3971PL.summary.tsv`
- kept failure evidence:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/chr1_s_file.txt`

Conclusion:
- 已删除 direct `MERGED3971` 的大体积失败中间件：
  - `sds_output_parent3971_olddefault_chr1/MERGED3971/chunk_submit`
  - `sds_output_parent3971_olddefault_chr1/MERGED3971/cache`
  - `sds_input/MERGED3971/chr1_t_file.txt`
- 已删除三条成功轨道的可再生产物：
  - `NCN / SCN / MERGED3971PL` 的 `chunk_submit`
  - `cache`
  - `chr1_p/q.sds.tsv`
  - `chr1_p/q.sds.parquet`
  - `chr1.sds.parquet`
- `MERGED3971PL` input 仍保留完整 `chr1_t_file.txt`，因为这是当前唯一可复用的 merged parent-like 成功输入。
- 清理前后目录体积对比：
  - `sds_output_inputfix_olddefault_chr1`: `7.7G -> 444M`
  - `sds_output_parent3971_olddefault_chr1`: `3.4G -> 371K`
  - `sds_output_parent3971pl_olddefault_chr1`: `4.9G -> 348M`
  - `sds_input/MERGED3971`: `53G -> 26K`
- 本轮约释放 `68G+`，且没有删除支撑当前结论的 final/review/compare 文件。

Next step:
- 若还要继续缩减，可下一轮再删 `NCN / SCN / MERGED3971PL` 的 per-chunk `.out/.err`，仅保留 final/merge/review logs。
- 当前不建议删 `MERGED3971PL/chr1_t_file.txt`，否则 merged parent-like 成功链无法直接复跑。
