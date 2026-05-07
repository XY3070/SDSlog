# NCN chr1 input fix and chunked compute launch

Date: 2026-05-07
Topic: `sds`
Type: `test`

Question: 当前 `NCN` split 后 input 是否已经修到符合 official SDS input contract，并且能否用修正后的 chr1 input 直接挂一条 chunked old/default compute 链，自动回看 `125.1 Mb`、`143.2 Mb` 和 control window？

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr1.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/sds_input.gamma_shapes.NCN.final`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_o_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_sample_order.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_s_file.audit.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_t_file.audit.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_o_file.audit.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.chr1_inputfix_olddefault.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/review/NCN.chr1_inputfix_olddefault.top_hits.tsv`

Parameters:
- `t_file`: all test SNPs, `PASS + biallelic SNP`
- `s_file`: subset-specific singleton positions from `PASS + biallelic SNP`, filtered by `AC=1` and `F_MISSING<=0.005`
- `o_file`: single-row all `1`
- `test input outdir`: `data/processed/sds_input/test/NCN`
- `production chr1 chunking`: `chunk_rows=20000`, `array_parallel=64`
- `gamma`: `old/default`
- review windows:
  - `peak125=125100001-125200000`
  - `peak143=143200001-143300000`
  - `control35=35735220-36063259`

Scripts / commands / job IDs:
- patched scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_s_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_t_file.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/filter_t_to_feather.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/summarize_sds_windows.py`
- local input rebuild:
  - `bash /data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh --pop NCN --chr 1 --force`
- chr1 chunked compute chain:
  - `484730`: `SDS_NCN_chr1_p_inputfix_olddefault[1-88]%64`
  - `484731`: `SDS_NCN_chr1_q_inputfix_olddefault[1-77]%64`
  - `484732`: `SDS_NCN_chr1_p_merge_inputfix_olddefault`
  - `484733`: `SDS_NCN_chr1_q_merge_inputfix_olddefault`
  - `484739`: `SDS_NCN_chr1_final_inputfix_olddefault`
  - `484741`: `SDS_NCN_chr1_review_inputfix_olddefault_v2`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN/chr1_sample_order.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/chunk_submit/chr1_inputfix_olddefault_20260507_143727/chr1_p.manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_inputfix_olddefault_chr1/NCN/chunk_submit/chr1_inputfix_olddefault_20260507_143727/chr1_q.manifest.tsv`

Conclusion:
- input contract 已收紧到 official SDS-compatible 主线：`o_file=1`，`t_file` 保留 all test SNP，但限定到 `PASS + biallelic SNP`，`s_file` 改为 subset-specific singleton universe。
- `NCN` chr1 的窗口级计数已经对账通过：
  - `t_file`:
    - `125.1 Mb`: expected `2953`, observed `2953`
    - `143.2 Mb`: expected `1693`, observed `1693`
    - `35.7 Mb control`: expected `3797`, observed `3797`
  - `s_file`:
    - `125.1 Mb`: expected `1703`, observed `1703`
    - `143.2 Mb`: expected `736`, observed `736`
    - `35.7 Mb control`: expected `2502`, observed `2502`
- `run_sds_input.sh --test` 现在会写到 `data/processed/sds_input/test/NCN`，不再覆盖正式 input。
- chr1 old/default chunked compute 已提交，review job 会在 `chr1.sds.tsv` 合并后自动输出三个窗口的 summary 和 top hits。

Next step:
- 等 `484739` 和 `484741` 完成，第一时间检查：
  - `peak125` 与 `peak143` 的 `max_abs_rSDS`
  - `control35` 是否仍异常抬高
  - top hits 是否仍是 manifest-like tower
- 若 chr1 input-fix 后显著改善，再把同一套 input contract 推到全基因组。
