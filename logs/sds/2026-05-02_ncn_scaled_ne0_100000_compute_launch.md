# NCN `scaled_ne0_100000` genome-wide compute launch

Date: 2026-05-02
Topic: sds
Type: production
Question: 现有 `scaled_ne0_100000` 的 full gamma pieces 能否直接组装成 minimal `g_file`，并启动不覆盖 baseline 的 genome-wide SDS 重算？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/full_gamma`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/chr4.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/chr4_p.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/chr4_q.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/logs/chr4_p.compute.csv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/logs/chr4_q.compute.csv`
- `/data/home/grp-wangyf/xuyuan/sds/logs/sds_ne100k_compute_v2_484533_*.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/sds_ne100k_compute_v2_484533_*.err`

## Parameters

- `scenario=scaled_ne0_100000`
- `present_diploid_pop_size=100000`
- `full_gamma_piece_count=10`
- `chr4_smoke_mode=test`
- `chr4_smoke_rows=30532 data rows`
- `compute_out_root=/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000`
- `compute_array_job=484533`
- `postprocess_job=484534`
- `array_concurrency=6`
- `threads_per_task=2`

## Demographic model

- `Demographic model=SDS-MODEL-002 (phlash_q50_scaled_ne100k)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv`
- `Present-day Ne(0)=100000`
- `Model role=rejected`
- `Model note=shape 来自 phlash q50 curve，显式缩放到 Ne(0)=100000；这条轨道后续被 gamma-track compare 否决`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426 --scenario-include scaled_ne0_100000 --skip-high-diagnostics --skip-q97p5-smoke --finalize-only`
- `bash /data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh --pop NCN --chr 4 --test --out-root /data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000 --g-file /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv --force`
- `job 484531`: first compute array submission; failed immediately because `LSB_JOBINDEX` quoting was wrong in the `bsub` command.
- `job 484533`: corrected compute array submission, `sds_ne100k_compute_v2[1-22]%6`
- `job 484534`: dependent postprocess submission, `sds_ne100k_postprocess_v2`

## Key plots

- None archived

## Key result files

- `SDS-009`: `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv`
- `SDS-010`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scaled_ne0_100000/NCN/chr4.sds.tsv`

## Conclusion

- `scaled_ne0_100000` 的 `10` 个 full gamma pieces 已经足够生成 minimal `g_file`，不需要再等 gamma cluster 补跑。
- `run_sds_compute.sh` 已补 `--out-root`，因此 `100k` 结果可以写到独立目录，不会覆盖 baseline `NCN` 输出。
- chr4 smoke 已经跑通，`chr4.sds.tsv` 产出 `30532` 行数据；修正 quoting 后，genome-wide array `484533` 已进入 `RUN`，前 `6` 条染色体正在计算，dependent postprocess `484534` 正在等待。

## Next step

- 等 `484533` 完成后检查 `484534` 是否自动开始；若 postprocess 成功，立刻审查新的 Manhattan、`frequency_bins.tsv`、`normalized.tsv` 和 chr1 / chr4 的形状是否仍异常。
