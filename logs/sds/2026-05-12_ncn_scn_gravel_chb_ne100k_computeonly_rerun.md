# NCN / SCN `SDS-MODEL-003` compute-only rerun with new chunked submitter

Date: 2026-05-12
Topic: `sds`
Type: `production`

Question: `SDS-MODEL-003` 的全基因组新 input 生产链在 2026-05-11 仍然走了旧的 per-chromosome direct compute 路径，计算速度过慢。需要在不重跑已完成整染色体结果的前提下，改用新优化的 chunked submitter，只补没有 `chr*.sds.tsv` 的染色体。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/relaunch_gravel_chb_ne100k_newinput_computeonly_chunked.sh`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_relaunch_20260512_043219/relaunch_manifest.tsv`

Parameters:
- `demographic_model=SDS-MODEL-003`
- `gamma_shape_source=/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- `present_day_Ne0=100000`
- `relaunch_mode=compute_only_missing_chromosomes`
- `queue=normal`
- `chunked_defaults_from_common_env`:
  - `chunk_rows=5000`
  - `array_parallel=32`
  - `chunk_job_slots=1`

## Demographic model

- `Demographic model=SDS-MODEL-003 (Gravel_CHB_ne100k)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- `Present-day Ne(0)=100000`
- `Model role=production`
- `Model note=只切换 compute 提交路径；input root 继续复用 2026-05-11 已完成的 rebuilt main contract input`

Scripts / commands / job IDs:
- old chain terminated:
  - `485076`: old `NCN` full compute array, terminated before any final `chr*.sds.tsv`
  - `485077`: old `SCN` full compute array, terminated before any final `chr*.sds.tsv`
  - `485078-485081`: old postprocess / diagnostics, terminated or exited
- new relaunch:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/relaunch_gravel_chb_ne100k_newinput_computeonly_chunked.sh`
  - status:
    - `/data/home/grp-wangyf/xuyuan/sds/scripts/check_gravel_chb_ne100k_relaunch_status.sh /data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_relaunch_20260512_043219/relaunch_manifest.tsv`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_relaunch_20260512_043219/relaunch_manifest.tsv`

Conclusion:
- 2026-05-11 那条 `SDS-MODEL-003` 新 input production 虽然重建了 input，但 compute 入口仍调用了旧的 `run_sds_compute.sh` 直跑链，没有真正进入 2026-05-12 刚优化的 chunked cluster fan-out。
- 截至 rerun 前，输出目录中没有任何已完成的整染色体结果：
  - `NCN`: `chr*.sds.tsv = 0`
  - `SCN`: `chr*.sds.tsv = 0`
- 因此本次 compute-only rerun 不会覆盖已完成整染色体结果；实际上补跑范围就是 `NCN` 和 `SCN` 的 `1-22` 全部染色体，但不再重跑 input。
- 新 relaunch 直接走 `submit_sds_compute_chunked_chr.sh`，并继承 `common_env.sh` 里当前的 `normal` 队列默认 chunk 参数。

Next step:
- 用 `check_gravel_chb_ne100k_relaunch_status.sh` 盯新的 final jobs、postprocess 和 diagnostics。
- 第一批 `chr*.sds.tsv` 落盘后，优先检查 `NCN` 与 `SCN` 的 `chr1`、`chr10` 和全基因组 `normalized.tsv` 是否开始出现。

## Update: 2026-05-12 afternoon

Question: `normal` 队列下，本地 submitter 预处理对 `SCN` 大染色体会触发系统 `Killed`，同时同一 shell 快速连续 `bsub` 会偶发 socket error。需要在不重跑已完成 chr 的前提下，把 `NCN` 和 `SCN` 的缺口补齐，并让 `SCN` 后半段可以一次性提交。

Scripts / commands / job IDs:
- `submit_sds_compute_chunked_chr.sh` patched:
  - empty-arm skip for acrocentric chromosomes (`chr13/14/15/21/22`)
- `relaunch_gravel_chb_ne100k_newinput_computeonly_chunked.sh` patched:
  - `POPS` selector
  - retry on transient submitter failure
- new helper scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_ne100k_chrlist.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr_via_smp.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_submitter_array_entry.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_postprocess_from_final_jobs.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_remaining_submitter_array.sh`
- `NCN` manual final jobs:
  - `chr14=485177`
  - `chr15=485182`
  - `chr16=485202`
  - `chr17=485192`
  - `chr18=485197`
  - `chr19=485187`
  - `chr20=485217`
  - `chr21=485212`
  - `chr22=485207`
- `NCN` downstream:
  - `postprocess=485218`
  - `diagnostics=485219`
- `SCN` manual final jobs now secured:
  - `chr1=485229`
  - `chr2=485224`
  - `chr3=485236`
  - `chr4=485241`
  - `chr5=485253`
  - `chr6=485263`
  - `chr7=485275`
  - `chr8=485281`
  - `chr9=485296`
  - `chr10=485305`
  - `chr11=485310`
- `SCN` remaining `chr12-22` one-shot submitter array:
  - outer array on `smp`: `485312`
  - finalizer: `485313`
  - state dir: `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_submitter_array_manual_20260512_1`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/diagnostics/NCN.gravel_chb_ne100k_manual.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_submitter_array_manual_20260512_1/results`

Conclusion:
- `NCN` 已完成全染色体 submit，并已接上 `postprocess + diagnostics`；`NCN` diagnostics 文件已经开始落盘。
- `SCN` 的稳定策略是：
  - `chr1-11` 用单条 submitter 逐条提交，避免本地内存被并行预处理打爆
  - `chr12-22` 改成 `smp` submitter array，一次性交给 LSF 节点完成预处理，再由各 task 向 `normal` 队列提交 compute arrays
- 这轮之后，`SCN` 不再需要继续手工逐条补 `chr12-22`。

Next step:
- 等 `485312` 的 `smp` submitter array 完成并产出 `chr12-22.final_job.tsv`。
- 让 `485313` 自动提交 `SCN` 的 `postprocess + diagnostics`。
- `NCN` 与 `SCN` 的 `normalized.tsv` 和 Manhattan 图一旦落盘，立即检查图形是否合理。

## Update: 2026-05-12 finalization fix

Question: `NCN` 与 `SCN` 的 `chr1-22.sds.tsv` 都已齐，但 `SCN` 的 finalizer `485313` 退出，导致 `SCN.normalized.tsv`、`SCN.manhattan.png` 没有自动生成。需要确认是不是 compute 未完成，还是只是 finalizer 依赖失效。

Scripts / commands / job IDs:
- verification:
  - `NCN`: `chr1-22.sds.tsv` complete
  - `SCN`: `chr1-22.sds.tsv` complete
- failed finalizer:
  - `485313`: `SDS_SUBMITTER_SCN_FINALIZE_MANUAL` exited
  - stderr showed `485229: No matching job found. Job not submitted.`
- manual recovery:
  - `SCN postprocess=485381`
  - `SCN diagnostics=485382`

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_submitter_array_manual_20260512_1/results`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN`

Conclusion:
- `SCN` 的问题已经从 “compute 没完成” 收敛为 “finalizer 依赖了一批 LSF 中已不可见的历史 final jobs”，不是 SDS 计算本身失败。
- 因为 `SCN chr1-22.sds.tsv` 已全部存在，可以安全跳过旧依赖，直接补发 `postprocess + diagnostics`。

Next step:
- 等 `485381/485382` 结束。
- 产出 `SCN.normalized.tsv`、`SCN.frequency_bins.tsv`、`SCN.significant_regions.tsv`、`SCN.manhattan.png` 后，与 `NCN` 一起审图。
