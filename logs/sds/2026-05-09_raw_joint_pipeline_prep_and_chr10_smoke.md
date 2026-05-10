# raw joint pipeline prep and chr10 smoke

Date: 2026-05-09
Topic: `sds`
Type: `test`

Question: 为后续 `chr1` 问题修复准备 raw-joint 主线。目标是让同一个 raw joint VCF 能直接生成 `NCN`、`SCN`、`MERGED3971` 三套 SDS input，并先用 `chr10` 做最小 smoke，确认 merged 不再依赖 split-then-merge。

Input paths:
- `/data/home/grp-wangyf/xuyuan/raw/4197samples.chr10.splitted.rmDup.LeftAligned.setID.reheader.DP5GQ20.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/SCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/MERGED3971.txt`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/tmp/raw_joint_chr10_smoke_20260509/sample_overlap.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_smoke/`

Parameters:
- code changes:
  - `run_sds_input.sh`: add `--vcf`, `--sample-list`, `--out-pop`, `--out-root`; add VCF-header sample validation
  - `run_sds_compute.sh`: add `--vcf`, `--in-root`
  - `generate_all_inputs.sh`: include `MERGED3971`
  - new helper: `run_raw_joint_chr_inputs.sh`
  - new helper: `build_raw_header_intersection_sample_list.sh`
- smoke:
  - `chr=10`
  - `mode=test`
  - `raw_vcf=4197samples.chr10.splitted.rmDup.LeftAligned.setID.reheader.DP5GQ20.vcf.gz`

Scripts / commands / job IDs:
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_raw_joint_chr_inputs.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/build_raw_header_intersection_sample_list.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/generate_all_inputs.sh`
- smoke command:
  - `bash sds/scripts/run_raw_joint_chr_inputs.sh --chr 10 --vcf raw/4197samples.chr10.splitted.rmDup.LeftAligned.setID.reheader.DP5GQ20.vcf.gz --out-root sds/data/processed/sds_input_raw_joint_smoke --test --force`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/tmp/raw_joint_chr10_smoke_20260509/sample_overlap.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.raw_chr10_intersection.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/SCN.raw_chr10_intersection.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/MERGED3971.raw_chr10_intersection.txt`

Conclusion:
- raw-joint 直连接口已经补齐，后续不需要再先 split VCF 再 merge。
- 但 `chr10` smoke 没有进入 input 生成阶段，因为 raw VCF header 和当前 sample lists 不是同一套样本宇宙。
- overlap 结果：
  - `NCN`: `hits=1073`, `misses=182`
  - `SCN`: `hits=2497`, `misses=219`
  - `MERGED3971`: `hits=3570`, `misses=401`
- 当前 raw chr10 header 前部是数值 /字母样本 ID；缺失样本主要是 `NA185xx` / `NA186xx` 这一类，说明 raw 主源不包含当前 `NCN/SCN` sample lists 的全部成员。
- 用户已接受使用 raw-present intersection 方案。
- 已生成 `chr10` 的交集样本表：
  - `NCN`: `1073`
  - `SCN`: `2497`
  - `MERGED3971`: `3570`
- 当前未完成的仅是 raw chr10 的 `.tbi` 建立；index 完成后即可用交集样本表直接重跑 `MERGED3971` smoke。

Next step:
- 用 `build_raw_header_intersection_sample_list.sh` 和 `run_raw_joint_chr_inputs.sh --use-header-intersection` 处理后续 raw chr1。
- chr1 raw 到位并完成 index 后，直接跑：
  - `NCN.raw_chr1_intersection`
  - `SCN.raw_chr1_intersection`
  - `MERGED3971.raw_chr1_intersection`
- 再进入 chr1 old/default compute 和三轨 compare。
