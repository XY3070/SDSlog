# chr1 SCN and parent3971 adjudication launch

Date: 2026-05-08
Topic: `sds`
Type: `test`

Question: `NCN` chr1 input-fix 后，`125.1 Mb`、`143.2 Mb` 和 `35.7 Mb control` 仍然过高。需要补两条 chr1 sanity/adjudication 轨道：`SCN` 对称重建，以及 `NCN+SCN=3971` parent-like 合并轨道。目标是判断 chr1 问题是 `NCN` split/regeneration-specific，还是 shared input/data artifact。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/SCN/UKBQC_SCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/SCN.txt`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/sds_input.gamma_shapes.NCN.final`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/MERGED3971.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/MERGED3971/UKBQC_MERGED3971_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/SCN/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971_olddefault_chr1/MERGED3971/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN.summary.tsv`

Parameters:
- `chr`: `1`
- `gamma`: `old/default`
- `chunk_rows=20000`
- `array_parallel=64`
- windows:
  - `peak125=125100001-125200000`
  - `peak143=143200001-143300000`
  - `control35=35735220-36063259`
- `SCN input contract`:
  - `t_file`: all test SNPs, `PASS + biallelic SNP`
  - `s_file`: subset-specific singleton positions from `PASS + biallelic SNP`, `AC=1`, `F_MISSING<=0.005`
  - `o_file`: single-row all `1`
- `MERGED3971` uses the same input contract after merging chr1 VCFs.

Scripts / commands / job IDs:
- added scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/merge_split_chr_vcfs.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_chr_sds_track_with_review.sh`
- running / pending jobs:
  - `484772`: `SCN_chr1_inputfix_rebuild`
  - `484773`: `MERGED3971_chr1_merge_v2`
  - `484774`: `SCN_chr1_olddefault_submitter` (depends on `484772`)
  - `484775`: `MERGED3971_chr1_inputfix_rebuild` (depends on `484773`)
  - `484776`: `MERGED3971_chr1_olddefault_submitter` (depends on `484775`)
  - `484777`: `SCN_chr1_olddefault_submitter_v3`
  - `484778`: `MERGED3971_chr1_olddefault_submitter_v3`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.top_hits.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/review/SCN.chr1_inputfix_olddefault.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/SCN/review/SCN.chr1_inputfix_olddefault.top_hits.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN.summary.tsv`

Conclusion:
- `NCN` input-fix 只做到了局部收紧，不是换回了 pre-split 风格的 site universe。
- `SCN` 和 `MERGED3971` 需要用同一套 fixed input contract 跑 chr1，对称比较 `peak125`、`peak143` 和 `control35`。
- 现有手工单行 merge 命令不可靠：生成出的 `MERGED3971 chr1 VCF` 没有 BGZF EOF marker，也没有索引。已改成 `part.vcf.gz -> index -> atomic move` 的流程。
- compute/review 改成依赖式 submitter，不再手工等 `FINAL_JOB`。
- 第一次自动依赖提交没有真正发出 compute，因为 `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh` 缺少执行权限；`484774` 和 `484776` 都以 `Permission denied` 退出。
- `SCN` input rebuild (`484772`) 与 `MERGED3971` input rebuild (`484775`) 都已成功完成；执行权限修复后，compute/review 已经通过 `484777` 和 `484778` 重新提交。
- `SCN` chr1 old/default 已完成，窗口 summary 为：
  - `peak125`: `row_count=0`
  - `peak143`: `row_count=0`
  - `control35`: `row_count=5542`, `max_abs_rSDS=13.3459`
- `NCN` vs `SCN` 对比表已写到 `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN.summary.tsv`。当前证据不支持“NCN 有峰而 SCN 干净”的解释；`SCN` 的 `control35` 同样异常高。
- `MERGED3971` 主线不是 scheduler 依赖 bug，而是 input 语义直接失效：`/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/chr1_s_file.txt` 共 `3971` 行且全部为 `NA`，导致所有 chunk 在 `compute_SDS.py` 中以 `index 0 is out of bounds for axis 1 with size 0` 失败。
- 已在 `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh` 和 `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh` 增加 fail-fast 检查：若 `s_file` 全 `NA`，直接报错，不再浪费整轮 chunk compute。

Next step:
- `MERGED3971` 不能继续用当前 split 后 `NCN/SCN` VCF 合并来近似 parent，因为它们不能提供全 cohort 的 singleton universe。
- 下一步若还要做 parent-like adjudication，必须回到更上游、保留 rare/singleton 信息的 pre-split VCF / BCF / site universe。

Update: `MERGED3971PL` parent-like reconstruction

Date: 2026-05-09

Question:
- 直接合并 split 后 `NCN/SCN` VCF 得到的 `MERGED3971` 无法生成可用 `s_file`。需要在现有 split cohort 产物上重建一个 parent-like singleton universe，至少拿到可运行的 `NCN / SCN / merged` 三轨 chr1 对照。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/SCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/SCN/UKBQC_SCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/chr1_o_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971/chr1_b_file.txt`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/MERGED3971PL.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971PL/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN_MERGED3971PL.summary.tsv`

Parameters:
- `chr=1`
- `gamma=old/default`
- `chunk_rows=20000`
- `array_parallel=64`
- `MERGED3971PL s_file` construction:
  - start from split-cohort singleton positions in `NCN` and `SCN`
  - query the opposite cohort VCF for each singleton position
  - drop positions that are observed as variant in the opposite cohort
  - keep only positions unique to one side, then concatenate to `3971` rows

Scripts / commands / job IDs:
- added:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/build_parentlike_s_file_from_split_cohorts.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/build_chr1_parentlike_input_from_split.sh`
- jobs:
  - `484926`: `MERGED3971PL_chr1_input_build`
  - `484927`: `MERGED3971PL_chr1_olddefault_submitter`
  - `484932`: `P_ARRAY_JOB`
  - `484933`: `Q_ARRAY_JOB`
  - `484934`: `P_MERGE_JOB`
  - `484935`: `Q_MERGE_JOB`
  - `484936`: `FINAL_JOB`
  - `484937`: `REVIEW_JOB`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/MERGED3971PL/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/review/MERGED3971PL.chr1_parentlike_olddefault.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_parent3971pl_olddefault_chr1/MERGED3971PL/review/MERGED3971PL.chr1_parentlike_olddefault.top_hits.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare/NCN_SCN_MERGED3971PL.summary.tsv`

Conclusion:
- `MERGED3971PL` 修复后，`s_file` 不再全 `NA`：`NA_rows=0`, `nonNA_rows=3971`，chr1 compute/review 能完整跑通。
- 但三轨对照并没有恢复成“`NCN` 有峰、`SCN` 和 merged 干净”的形态：
  - `NCN peak125`: `row_count=2953`, `max_abs_rSDS=14.328`
  - `NCN peak143`: `row_count=1693`, `max_abs_rSDS=14.1092`
  - `NCN control35`: `row_count=3797`, `max_abs_rSDS=12.7735`
  - `SCN peak125/peak143`: `row_count=0/0`
  - `SCN control35`: `row_count=5542`, `max_abs_rSDS=13.3459`
  - `MERGED3971PL peak125/peak143`: `row_count=0/0`
  - `MERGED3971PL control35`: `row_count=7730`, `max_abs_rSDS=13.3459`
- `SCN` 与 `MERGED3971PL` 在 `control35` 的 top site 和 `max_abs_rSDS` 完全一致，说明 shared artifact 仍在；当前 split-cohort 派生的 parent-like 重建并没有把 chr1 问题修掉。

Next step:
- 当前已经拿到了 `NCN / SCN / merged` 的可运行三轨对照，问题不再是 pipeline 挂掉，而是 biology 仍不合理。
- 下一步不该继续在 split 后 VCF 上做近似重建，而应回到更上游、保留 rare/singleton 信息的 pre-split VCF / BCF / site universe，重建真正的 parent baseline。
