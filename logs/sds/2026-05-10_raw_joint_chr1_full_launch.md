# raw joint chr1 full launch

Date: 2026-05-10
Topic: `sds`
Type: `test`

Question: 在 raw joint `chr1` 上，用 header-intersection 样本直接生成 `NCN`、`SCN`、`MERGED3971` 三套正式 input，并挂起 `SDS-MODEL-001` 的 chr1 三轨 compare。目标是绕开 split-then-merge 导致的 pseudo-missing，直接验证 true process 下 merged 轨道是否可用。

Input paths:
- `/data/home/grp-wangyf/xuyuan/raw/4197samples.chr1.splitted.rmDup.LeftAligned.setID.reheader.DP5GQ20.vcf.gz`
- `/data/home/grp-wangyf/xuyuan/raw/4197samples.chr1.splitted.rmDup.LeftAligned.setID.reheader.DP5GQ20.vcf.gz.tbi`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/SCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/MERGED3971.txt`
- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/raw_header_intersections/NCN.raw_chr1_intersection.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/raw_header_intersections/SCN.raw_chr1_intersection.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/raw_header_intersections/MERGED3971.raw_chr1_intersection.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_smoke/test/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_full/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_raw_joint_olddefault_chr1/`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/chr1_adjudication_compare_raw_joint/`

Parameters:
- source: raw joint `chr1`
- sample policy: raw header intersection
- `NCN` samples: `1073`
- `SCN` samples: `2497`
- `MERGED3971` samples: `3570`
- input contract:
  - `t_file`: `PASS + biallelic SNP`
  - `s_file`: subset-specific `PASS + biallelic SNP + AC=1 + F_MISSING<=0.005`
  - `o_file`: single row all `1`
- compute contract:
  - gamma: `SDS-MODEL-001`
  - `chunk_rows=20000`
  - `array_parallel=64`
- review windows:
  - `peak125=125100001-125200000`
  - `peak143=143200001-143300000`
  - `control35=35735220-36063259`

## Demographic model

- `Demographic model=SDS-MODEL-001 (historical_olddefault_gamma_table)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `Present-day Ne(0)=implicit / unknown from historical file provenance`
- `Model role=audit-only`
- `Model note=raw-joint 只改变 input source；true-process merged 不是新的 demographic model，也不是 explicit Ne(0)=100k track`

Scripts / commands / job IDs:
- `/data/home/grp-wangyf/xuyuan/sds/scripts/build_raw_header_intersection_sample_list.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_raw_joint_chr_inputs.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_chr_sds_track_with_review.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_raw_joint_chr1_olddefault_compare.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/compare_chr1_review_tracks.py`
- jobs:
  - `484986`: raw chr1 index
  - `484987`: raw-joint chr1 input smoke
  - `484988`: raw-joint chr1 full input build
  - `484989`: dependent launcher for `SDS-MODEL-001` three-track compare
  - `484993`: first full launcher, later invalidated by chunk cache race
  - `485013`: cache-fix relaunch (`v2`)

Key result files:
- `/data/home/grp-wangyf/xuyuan/sds/logs/RAWIDX_chr1.484986.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/RAWCHR1_INPUT_SMOKE.484987.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/RAWCHR1_INPUT_FULL.484988.out`
- `/data/home/grp-wangyf/xuyuan/sds/logs/RAWCHR1_OLDDEFAULT_COMPARE_LAUNCH_V2.485013.out`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_smoke/test/NCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_smoke/test/SCN/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_smoke/test/MERGED3971/chr1_s_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_full/NCN/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_full/SCN/chr1_t_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_raw_joint_chr1_full/MERGED3971/chr1_t_file.txt`

Conclusion:
- raw chr1 `.tbi` 已完成，raw-joint `chr1` smoke 成功。
- 这次 merged 轨道是按 true process 直接从 raw joint `chr1` 生成，不再经过 split-then-merge。
- smoke 结果表明三套 `s_file` 都可用，不再出现 merged `s_file` 全 `NA`：
  - `NCN`: `1073/1073` non-NA rows
  - `SCN`: `2408/2497` non-NA rows
  - `MERGED3971`: `3447/3570` non-NA rows
- `t_file` row counts 也已正常长出：
  - `NCN`: `76151`
  - `SCN`: `76252`
  - `MERGED3971`: `76283`
- 目前 full input build `484988` 和 dependent compare launcher `484989` 已提交，下一步等三轨 `SDS-MODEL-001` review 结果落盘。
- `484988` 已完成，full raw-joint chr1 input 三套都已成功生成：
  - `NCN`
  - `SCN`
  - `MERGED3971`
- 这次 `MERGED3971` 不仅 smoke `s_file` 可用，full `s_file` 和 full `t_file` 也都已落盘，说明 true-process merged input 已经打通。
- 第一轮 full compute launcher `484993` 发出的 raw-joint arrays 中，`SCN` chunk 报错 `pickle data was truncated`，定位为共享 pickle cache 的并发写入竞争。
- 已修复 `compute_SDS.py` 的 `PickleCache`：
  - 读损坏 pickle 时自动丢弃并重建
  - 用 `fcntl.flock` 给 cache file 加锁
  - 先写临时文件再 `os.replace`，避免半写入 pickle
- 已把 raw-joint compare 重发到 `v2` 输出目录：
  - launcher `485013`
  - `NCN v2`: `485014/485015 -> 485016/485017 -> 485018 -> 485019`
  - `SCN v2`: `485020/485021 -> 485022/485023 -> 485024 -> 485025`
  - `MERGED3971 v2`: `485026/485027 -> 485028/485029 -> 485030 -> 485031`
  - compare `v2`: `485032`
- 到当前检查点，`v2` 尚未再次观察到 `pickle data was truncated`。

Next step:
- 继续监控 `v2` 的 `NCN / SCN / MERGED3971` arrays 是否全部完成。
- 若 `v2` 无新的 cache truncation，则等待 `485019`、`485025`、`485031` 和 `485032` 落盘。
- compare 结果出来后，直接对 `peak125`、`peak143`、`control35` 做 raw-joint 三轨判断。
