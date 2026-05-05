# NCN region `Ne(0)=100k` chunked gamma acceleration

Date: 2026-05-02
Topic: sds
Type: test
Question: 在保留 `sim_reps=1000` 的前提下，怎样先把 `scaled_ne0_100000` 的 gamma generation 改成更快的 chunked parallel path，并确认它能产出可被 finalize / SDS compute 复用的 canonical piece？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_region_ne0_parallel.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_chunk.sh`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/full_gamma`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/submission_plan.json`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/task_plan.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502/scenario_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502/scaled_ne0_100000/full_chunks/0p09440000000000004`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502/scaled_ne0_100000/full_gamma/scaled_ne0_100000.full.0.09440000000000004`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502/scaled_ne0_100000/full_gamma/scaled_ne0_100000.full.0.09440000000000004.meta.json`

## Parameters

- `scenario_include=scaled_ne0_100000`
- `gamma_generation_mode=chunked`
- `forced_dry_run_sim_reps=1000`
- `forced_dry_run_gamma_chunk_size=100`
- `chunk_array_cap=64`
- `full_array_cap=16`
- `skip_high_diagnostics=True`
- `skip_q97p5_smoke=True`
- `local_smoke_sim_reps=10`
- `local_smoke_gamma_chunk_size=5`
- `local_smoke_frequency=0.09440000000000004`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_region_ne0_parallel.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_chunk.sh`
- `dry-run reuse cache: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/submit_region_ne0_parallel.py --dry-run --chunk-array-cap 64 --full-array-cap 16 --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426 --scenario-include scaled_ne0_100000 --gamma-generation-mode chunked --gamma-chunk-size 100 --skip-high-diagnostics --skip-q97p5-smoke`
- `dry-run force regenerate: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/submit_region_ne0_parallel.py --dry-run --chunk-array-cap 64 --full-array-cap 16 --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426 --scenario-include scaled_ne0_100000 --gamma-generation-mode chunked --gamma-chunk-size 100 --skip-high-diagnostics --skip-q97p5-smoke --no-reuse-existing-gamma`
- `local prepare-only smoke: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py --prepare-only --outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502 --scenario-include scaled_ne0_100000 --target-ne0-grid 100000 --skip-high-diagnostics --skip-q97p5-smoke --sim-reps 10 --gamma-chunk-size 5`
- `local chunk worker 1: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502 --scenario-include scaled_ne0_100000 --target-ne0-grid 100000 --skip-high-diagnostics --skip-q97p5-smoke --sim-reps 10 --gamma-chunk-size 5 --chunk-worker --scenario-label scaled_ne0_100000 --gamma-purpose full --frequency 0.0944 --chunk-index 1 --chunk-start-rep 1 --chunk-end-rep 5`
- `local chunk worker 2: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502 --scenario-include scaled_ne0_100000 --target-ne0-grid 100000 --skip-high-diagnostics --skip-q97p5-smoke --sim-reps 10 --gamma-chunk-size 5 --chunk-worker --scenario-label scaled_ne0_100000 --gamma-purpose full --frequency 0.0944 --chunk-index 2 --chunk-start-rep 6 --chunk-end-rep 10`
- `local aggregate worker: /data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/benchmark_adh_ne0.py --resume-outdir /data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502 --scenario-include scaled_ne0_100000 --target-ne0-grid 100000 --skip-high-diagnostics --skip-q97p5-smoke --sim-reps 10 --gamma-chunk-size 5 --aggregate-worker --scenario-label scaled_ne0_100000 --gamma-purpose full --frequency 0.0944`

## Key plots

- None archived

## Key result files

- `SDS-006`: `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/submission_plan.json`
- `SDS-007`: `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/parallel_submit/task_plan.tsv`
- `SDS-008`: `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_chunk_smoke_20260502/scaled_ne0_100000/full_gamma/scaled_ne0_100000.full.0.09440000000000004`

## Conclusion

- `submit_region_ne0_parallel.py` 现在支持 `--scenario-include`、`--gamma-generation-mode chunked`、chunk array、aggregate array 和 finalize dependency chain；在 `scaled_ne0_100000` forced dry-run 下会展开成 `100` 个 chunk task 和 `10` 个 aggregate task。
- 同一 rescue outdir 的默认 dry-run 会提交 `0` 个 task，因为 `scaled_ne0_100000` 已经有 `10` 个可复用的 full gamma pieces；cache reuse 逻辑保留不变。
- 本地 `sim_reps=10` smoke 已经实测通过：两个 `5`-rep chunk 可以并行完成，并成功聚合成 canonical gamma piece；worker 侧的 frequency 解析也已修成 tolerant match，`0.0944` 这种简写不会再被拒绝。

## Next step

- 直接把现有 `scaled_ne0_100000` 的 full gamma pieces 组装成 minimal `g_file`，先做 `run_sds_compute.sh --g-file ...` 的单染色体 smoke，再决定是否需要 full genome-wide `norm_SDS` 重算。
