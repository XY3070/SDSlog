Date: 2026-05-25
Topic: sds
Type: test

Question

围绕当前 `Gravel_CHB@100k` 坏 Manhattan，继续按用户要求完成三步中的前两步半：

1. 对照 SDS GitHub 原始 `Makefile`，排除“我们后来改坏了 Makefile 才导致 gamma 模拟误差”的简单解释；
2. 固定现有结果，验证 `MAF >= 0.05` 是否明显改善 Manhattan；
3. 尝试把 gamma grid 从 `0.05` 扩到 `0.01`，并检查当前 pipeline 是否能实际产出低频 gamma piece。

Input paths

- `https://github.com/yairf/SDS/blob/master/Makefile`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/Makefile`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/snapshots/20260405_1528/Makefile.bak`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/legacy_Makefile`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.normalized.tsv`

Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/cmplot/NCN.cmplot_maf005_allp.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/cmplot/SCN.cmplot_maf005_allp.manhattan.png`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_grid001_run4/parallel_submit/chunks_tasks.jsonl`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_grid001_run4/parallel_submit/aggregates_tasks.jsonl`

Parameters

- `demographic_model=Gravel_CHB`
- `present_ne=100000`
- `grid001_attempt_daf=0.01,0.02,0.03,0.04`
- `grid001_attempt_sim_reps=1000`
- `maf_cutoff_for_plot_check=0.05`

Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `NCN/SCN gravel newinput` | `Gravel_CHB` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `rejected` | 当前坏图主轨道 |
| `grid001_run2` | `Gravel_CHB` | low-frequency extension attempt from `0.01` to `0.04`, to be merged with baseline `0.05-0.95` grid | `100000` | `audit-only` | 用于验证 `<0.05` 覆盖是否能改善 gap |

Scripts / commands / job IDs

- GitHub original `Makefile` was inspected directly from `yairf/SDS`
- local single-replicate probes:
  - `make my_simulation1 DAF=0.01`
  - `make my_simulation1 DAF=0.02`
  - `make my_simulation1 DAF=0.03`
  - `make my_simulation1 DAF=0.04`
- stale pre-fix low-frequency gamma extension pipeline:
  - `486832`: `gravel_chb_ne100k_grid001_run2_chunks`
  - `486833`: `gravel_chb_ne100k_grid001_run2_aggs`
  - `486834`: `gravel_chb_ne100k_grid001_run2_finalize`
  - `486835`: `gravel_chb_ne100k_grid001_run2_build`
  - `486836`: dependent NCN rerun launcher
- fresh post-fix low-frequency gamma extension pipeline:
  - `486837`: `gravel_chb_ne100k_grid001_run4_chunks`
  - `486838`: `gravel_chb_ne100k_grid001_run4_aggs`
  - `486839`: `gravel_chb_ne100k_grid001_run4_finalize`
  - `486840`: `gravel_chb_ne100k_grid001_run4_build`
  - `486841`: dependent NCN rerun launcher

Key result files

- `/tmp/gravel_grid001_probe/res_0.01_1.tab`
- `/tmp/gravel_grid002_probe/res_0.02_1.tab`
- `/tmp/gravel_grid003_probe/res_0.03_1.tab`
- `/tmp/gravel_grid004_probe/res_0.04_1.tab`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_grid001_run4/gravel_chb_present100000.grid001.g_file.txt`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_grid001_ncn_run4/NCN/logs/chr1.submit.out`

Conclusion

- SDS GitHub 原始 `Makefile` 本身就把 `PRESENT_DAF_LIST` 设为从 `0.05` 起步；这一点没有问题，也解释了为什么官方原链并不覆盖 `0.01–0.04`。
- 但本地 `Makefile` 确实存在一个**后来引入的回归**：在 `my_simulation_trees_2_tip_branch_stats` 里，`tip_branches_$(DAF)_$(I).tab` 在被 `cat` 进 `res_$(DAF)_$(I).tab` 之前就被删除了。这个删除来自后加的 `TIP_BRANCH_DUMP` 逻辑，而 `2026-04-05` 快照没有这个问题。
- 修掉这个回归后，单 replicate 的 `<0.05` probe 已经可以实际产出结果：
  - `DAF=0.01`: `0.561989615236484`
  - `DAF=0.02`: `0.622326518783633`
  - `DAF=0.03`: `0.716869002388415`
  - `DAF=0.04`: `0.339707326888122`
  这说明先前“`<0.05` 本身不被支持”这个判断过强；真正先被排到前面的原因，是本地 `Makefile` 回归导致 tip-branch 统计文件被提前删掉。
- `MAF >= 0.05` 的定量改善已经成立：
  - `NCN Gravel_CHB@100k + new input`
    - 全部 common variants：`ge40 = 2840`
    - `MAF >= 0.05`：`ge40 = 163`
  - `SCN Gravel_CHB@100k + new input`
    - 全部 common variants：`ge40 = 680`
    - `MAF >= 0.05`：`ge40 = 60`
  这说明坏 Manhattan 的大头确实来自 `<0.05` 的 low-frequency common bin。
- `run4` 的 low-frequency pieces、DAF-level aggregates 与 combined `g_file` 已经全部生成；新的 extended gamma 前几行为：
  - `0.01 -> 0.9683906760369475`
  - `0.02 -> 0.7928868498917352`
  - `0.03 -> 0.7660406730590298`
  - `0.04 -> 0.7400089414091944`
  - `0.05 -> 0.7103425643116044`
- `run4` 的 dependent NCN rerun 已经完整收口：
  - `postprocess = 486955`: `DONE`
  - `diagnostics = 486956`: `DONE`
  - 结果目录：`/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_grid001_ncn_run4/NCN`
  - 关键结果文件：
    - `NCN.normalized.tsv`
    - `NCN.frequency_bins.tsv`
    - `NCN.manhattan.png`
    - `NCN.manhattan_points.tsv`
- `grid=0.01 + interpolation` **没有改善** Manhattan，反而把 `<0.05` 的极端尾部推得更重。对比同一套 `NCN` new input：
  - 原始 `Gravel_CHB@100k`：
    - `<0.05`: `lt005_ge20 = 2685`, `lt005_ge40 = 2677`
    - `>=0.05`: `ge005_ge20 = 176`, `ge005_ge40 = 163`
    - `max_neglogp = 309.612935`
  - `grid001_run4`：
    - `<0.05`: `lt005_ge20 = 3918`, `lt005_ge40 = 3648`
    - `>=0.05`: `ge005_ge20 = 175`, `ge005_ge40 = 162`
    - `max_neglogp = 309.855356`
- 这说明 `grid=0.01` 的影响几乎完全落在 `<0.05` 低频 common bin 上，并且方向是**变差**而不是改善；`>=0.05` 的背景几乎不变。
- 因此，当前最稳的技术结论是：
  1. `Makefile` 问题真实存在，但它解释的是“我们为什么之前拿不到 `<0.05` 的 gamma piece”，不是当前坏 Manhattan 的最终主因。
  2. `MAF >= 0.05` 明显改善 Manhattan，是当前最直接、最强的硬证据。
  3. 在修复 `Makefile` 之后，把 gamma grid 扩到 `0.01` 再加插值，**并不能改善** Manhattan；按当前 `Gravel_CHB@100k` gamma 生成链，它反而会进一步加重 low-frequency artifact。
  4. 因此，问题已经不能再归因于“仅仅缺少 `<0.05` gamma grid”或“只是 `Makefile` 改坏了”；更深层的嫌疑应转到 gamma provenance / low-frequency shape family 本身。

Next step

- 这三步验证已经完成：
  - 排查 `Makefile`
  - 验证 `MAF >= 0.05`
  - 验证 `grid=0.01 + interpolation`
- 如果继续往下，最合理的下一步不再是重扫 `grid`，而是直接追 `olddefault g_file` 的真实 provenance，并把它与当前 `Gravel_CHB@100k` 的 low-frequency shape family 做逐频率对照。
