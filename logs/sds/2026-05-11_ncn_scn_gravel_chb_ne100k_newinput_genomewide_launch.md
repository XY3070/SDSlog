# NCN / SCN `SDS-MODEL-003` genome-wide launch with rebuilt input

Date: 2026-05-11
Topic: `sds`
Type: `production`

Question: 用重建后的全基因组 input，而不是旧的 `sds_input/NCN` 与 `sds_input/SCN`，分别为 `NCN` 和 `SCN` 跑一版 `SDS-MODEL-003`，看新的全基因组 Manhattan 图是否仍然表现为 diffuse tail。

Input paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/SCN/`
- `/data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/SCN.txt`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_input.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/diagnose_sds_scan.py`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`

Output paths:
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_smoke_20260511`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_launch_20260511_150204/launch_manifest.tsv`

Parameters:
- `demographic_model=SDS-MODEL-003`
- `gamma_shape_source=/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- `present_day_Ne0=100000`
- `input_contract`:
  - `t_file=PASS + biallelic SNP, all test SNP`
  - `s_file=PASS + biallelic SNP + AC=1 + F_MISSING<=0.005`
  - `o_file=single-row all 1`
- `smoke_chr=10`
- `queue=normal`
- `input_array_cap=4`
- `compute_array_cap=6`

## Demographic model

- `Demographic model=SDS-MODEL-003 (Gravel_CHB_ne100k)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`
- `Present-day Ne(0)=100000`
- `Model role=production`
- `Model note=这次不复用旧的全基因组 input；先按当前 main contract 重建 NCN / SCN input，再分别跑全基因组 compare`

Scripts / commands / job IDs:
- launch script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_gravel_chb_ne100k_newinput_genomewide.sh`
- status script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/check_gravel_chb_ne100k_newinput_status.sh /data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_launch_20260511_150204/launch_manifest.tsv`
- smoke:
  - `485070`: `NCN` input smoke chr10
  - `485071`: `SCN` input smoke chr10
  - `485072`: `NCN` compute smoke chr10
  - `485073`: `SCN` compute smoke chr10
- full input:
  - `485074`: `NCN` rebuilt input array `[1-22]%4`
  - `485075`: `SCN` rebuilt input array `[1-22]%4`
- full compute:
  - `485076`: `NCN` genome-wide compute array `[1-22]%6`
  - `485077`: `SCN` genome-wide compute array `[1-22]%6`
- postprocess:
  - `485078`: `NCN` postprocess
  - `485079`: `SCN` postprocess
- diagnostics:
  - `485080`: `NCN` diagnostics
  - `485081`: `SCN` diagnostics

Key result files:
- smoke input:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/test/NCN/chr10_t_file.txt`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/test/SCN/chr10_t_file.txt`
- smoke compute:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_smoke_20260511/NCN/chr10.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_smoke_20260511/SCN/chr10.sds.tsv`
- launch manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/gravel_chb_ne100k_newinput_launch_20260511_150204/launch_manifest.tsv`

Conclusion:
- `SDS-MODEL-003` 的 `g_file` 沿用现有 `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt`，不再重新生成 gamma。
- 这次 production 不复用旧的全基因组 `sds_input/NCN` 与 `sds_input/SCN`，而是落到新的 rebuilt input root：
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511`
- `NCN` 与 `SCN` 的 chr10 smoke 都已经产出 `chr10.sds.tsv`，说明新 input 路径和 `SDS-MODEL-003` 的 compute 路径在 production 前是打通的。
- 当前第一轮状态检查显示：
  - `NCN` full input array 已开始并至少部分完成
  - `SCN` full input array 正在跑
  - `NCN` genome-wide compute array 已启动
  - `SCN` genome-wide compute array 仍在等待 full input 依赖满足

Next step:
- 持续用 `check_gravel_chb_ne100k_newinput_status.sh` 盯 `485074-485081`。
- 等 `485078` / `485079` 完成后，直接审两张标准 Manhattan：
  - `NCN.manhattan.png`
  - `SCN.manhattan.png`
- 若两边的 full Manhattan 都已出，再决定是否补过滤版展示图。
