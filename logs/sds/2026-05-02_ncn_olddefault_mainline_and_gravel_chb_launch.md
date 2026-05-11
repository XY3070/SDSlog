# NCN `SDS-MODEL-001` mainline and `SDS-MODEL-003` launch

Date: 2026-05-02
Topic: sds
Type: production
Question: 在 `SDS-MODEL-002` 被降级为 sensitivity 之后，是否已经把 mainline 切回 `SDS-MODEL-001`，并启动 `SDS-MODEL-003` 的并行 gamma 生成与 downstream compute？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_gamma_parallel.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/compare_sds_diagnostics.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_chunk.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/aggregate_single_snp_gamma_chunks.sh`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/parallel_submit/submission_plan.json`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/chr4.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/chr4_p.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/chr4_q.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/parallel_submit/submission_plan.json`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/parallel_submit/submitted_jobs.json`
- `/data/home/grp-wangyf/xuyuan/sds/logs/sds_olddefault_compute_484541_*.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/sds_gravel_chb_gamma_chunks_484535_*.out`

## Parameters

- `olddefault_mainline_gfile=/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `olddefault_chr4_smoke_mode=test`
- `olddefault_chr4_smoke_rows=102000 data rows`
- `olddefault_out_root=/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline`
- `olddefault_compute_job=484541`
- `olddefault_postprocess_job=484542`
- `gravel_pop_model=Gravel_CHB`
- `gravel_present_ne=100000`
- `gravel_sim_reps=1000`
- `gravel_chunk_size=100`
- `gravel_chunk_task_count=910`
- `gravel_aggregate_task_count=91`
- `gravel_gamma_jobs=484535,484536,484537`
- `gravel_compute_job=484538`
- `gravel_postprocess_job=484539`
- `olddefault_diag_job=484543`
- `gravel_diag_job=484544`
- `phlash_diag_job=484545`
- `track_compare_job=484546`
- `diag_failure_reason=system_python_missing_matplotlib`
- `olddefault_diag_job_v2=484559`
- `gravel_diag_job_v2=484560`
- `phlash_diag_job_v2=484561`
- `track_compare_job_v2=484562`
- `phlash_scaled_100k_jobs=484533,484534`
- `phlash_scaled_100k_role=sensitivity_only`

## Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `olddefault_mainline` | `SDS-MODEL-001` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `mainline` | 这条线只确认 shape file，不额外声称 explicit `Ne(0)` |
| `gravel_chb_ne100k` | `SDS-MODEL-003` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `rejected` | sensitivity / compare only |
| `phlash_scaled_ne100k` | `SDS-MODEL-002` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | `rejected` | 在这条记录里只作为 compare companion |

## Scripts / commands / job IDs

- `bash /data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh --pop NCN --chr 4 --test --out-root /data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline --g-file /data/home/grp-wangyf/xuyuan/sds/g_file.txt --force`
- `job 484541`: `sds_olddefault_compute[1-22]%6`
- `job 484542`: `sds_olddefault_postprocess`
- `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_gamma_parallel.py --outdir /data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502 --present-ne 100000 --sim-reps 1000 --chunk-size 100 --chunk-array-cap 64 --aggregate-array-cap 16`
- `job 484535`: `sds_gravel_chb_gamma_chunks[1-910]%64`
- `job 484536`: `sds_gravel_chb_gamma_aggregates[1-91]%16`
- `job 484537`: `sds_gravel_chb_gamma_finalize`
- `job 484538`: `sds_gravel_chb_ne100k_compute[1-22]%6`
- `job 484539`: `sds_gravel_chb_ne100k_postprocess`
- `job 484543`: `sds_olddefault_diag`, dependency `done(484542)`
- `job 484544`: `sds_gravel_chb_diag`, dependency `done(484539)`
- `job 484545`: `sds_phlash100k_diag`, dependency `done(484534)`
- `job 484546`: `sds_gamma_track_compare`, dependency `done(484543) && done(484544) && done(484545)`
- `jobs 484543/484544/484545`: all exited immediately because cluster `python3` did not have `matplotlib`
- `job 484559`: `sds_olddefault_diag_v2`, direct resubmission with `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python`
- `job 484560`: `sds_gravel_chb_diag_v2`, direct resubmission with `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python`
- `job 484561`: `sds_phlash100k_diag_v2`, direct resubmission with `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python`
- `job 484562`: `sds_gamma_track_compare_v2`, dependency `done(484559) && done(484560) && done(484561)`

## Key plots

- None archived

## Key result files

- `SDS-011`: `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `SDS-014`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/chr4.sds.tsv`
- `SDS-015`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/parallel_submit/submission_plan.json`
- `SDS-016`: `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/parallel_submit/submitted_jobs.json`

## Conclusion

- mainline 已从 `SDS-MODEL-002` 切回 `SDS-MODEL-001`；`SDS-MODEL-002` 继续跑完，但只保留 sensitivity 身份。
- `SDS-MODEL-001` 的 `chr4 --test` smoke 已跑通，`chr4.sds.tsv` 已写入独立 output root，避免覆盖 baseline。
- `SDS-MODEL-003` 的 chunked gamma 任务图和 dependent compute 已全部提交，当前 bottleneck 已转为集群排队与 chunk 完成速度，而不是串行 gamma 生成。
- 三条轨道的 postprocess 后诊断任务也已全部挂好依赖；`normalized.tsv` 一出来就会自动产出 chromosome tail-count、top-hit 和 dense-peak window summary。
- 三条轨道的 cross-track compare 也已排队；三个 diagnostics 完成后会自动生成 merged summary，不需要再手工比对三个目录。
- compare 链第一次没有跑起来不是因为结果文件缺失，而是因为 diagnostics 用错了解释器；`v2` 已改成 `sds` 环境 Python 并重新提交。

## Next step

- 等 `484541/484542` 产出 `SDS-MODEL-001` genome-wide `normalized.tsv` 和 Manhattan。
- 等 `484535/484536/484537` 产出 `gravel_chb_present100000.g_file.txt`，随后检查 `484538/484539` 是否自动开始。
- 三条轨道结果齐全后，统一比较 Manhattan null shape、chr1 tower 和已知 positive peaks 的 local clustering。
