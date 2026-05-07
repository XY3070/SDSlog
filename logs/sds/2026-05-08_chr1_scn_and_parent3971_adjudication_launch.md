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

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.top_hits.tsv`

Conclusion:
- `NCN` input-fix 只做到了局部收紧，不是换回了 pre-split 风格的 site universe。
- `SCN` 和 `MERGED3971` 需要用同一套 fixed input contract 跑 chr1，对称比较 `peak125`、`peak143` 和 `control35`。
- 现有手工单行 merge 命令不可靠：生成出的 `MERGED3971 chr1 VCF` 没有 BGZF EOF marker，也没有索引。已改成 `part.vcf.gz -> index -> atomic move` 的流程。
- compute/review 改成依赖式 submitter，不再手工等 `FINAL_JOB`。

Next step:
- 等 `484772` 和 `484773` 完成。
- 若 `484773` 成功，自动重建 `MERGED3971` input 并接上 old/default chr1 compute + review。
- 若 `484772` 成功，自动提交 `SCN` chr1 old/default compute + review。
- 三条轨道齐后，统一比较 `NCN` / `SCN` / `MERGED3971` 在 `peak125`、`peak143`、`control35` 的 `max_abs_rSDS` 和 top hits。
