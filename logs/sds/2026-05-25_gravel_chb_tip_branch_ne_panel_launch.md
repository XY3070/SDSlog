Date: 2026-05-25
Topic: sds
Type: test

Question

在不直接重跑全基因组 SDS 的前提下，围绕 `Gravel_CHB` 固定 demographic family，做一组小型 `Ne(0)` panel，直接导出 `DAF=0.05/0.07/0.10` 的 tip branch 分布，用来判断当前异常 Manhattan 是否主要来自：

1. `Ne(0)=100000` 在低频边界产生了不合适的 gamma shape；
2. `gamma grid` 只从 `0.05` 起步带来的边界 clamping；
3. 或两者叠加。

Input paths

- `/data/home/grp-wangyf/xuyuan/ms/scripts/Makefile`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_piece.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_tip_branch_panel.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/summarize_tip_branch_panel.py`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/NCN_scaled_ne0_100000.npz`

Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.md`

Parameters

- `demographic_model=Gravel_CHB`
- `present_ne_grid=60000,80000,100000,150000,200000,270000`
- `daf_grid=0.05,0.07,0.10`
- `sim_reps=100`
- `tip_branch_dump=TRUE`
- `queue=smp`
- `array_cap=6`

Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `tip_branch_panel` | `Gravel_CHB` | diagnostic panel generated from `run_single_snp_gamma_piece.sh` with tip-branch dump enabled | `60000,80000,100000,150000,200000,270000` | `audit-only` | 只用于确认低频边界处 `Ne(0)` 如何改变 tip branch distribution 与 gamma shape |

Scripts / commands / job IDs

- `job 486818`: `gravel_tip_panel[1-18]%6`
- `job 486819`: `gravel_tip_panel_summary`
- array worker command uses:
  - `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/submit_gravel_chb_tip_branch_panel.py --run-task-file ... --task-index $LSB_JOBINDEX`
- summary command uses:
  - `/data/home/grp-wangyf/intern/miniforge3/envs/sds/bin/python /data/home/grp-wangyf/xuyuan/sds/scripts/summarize_tip_branch_panel.py --manifest ... --output-prefix ...`

Key result files

- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.md`

Conclusion

- panel 已完成，`486818`/`486819` 正常产出 summary。
- 在固定 `Gravel_CHB` demographic family、只改变 `Ne(0)=60k,80k,100k,150k,200k,270k` 时：
  - `DAF=0.05/0.07/0.10` 的 tip branch `mean`、`sd`、以及各分位数都会随着 `Ne(0)` 增大而近似线性上升；
  - 但 `shape_recomputed` 基本稳定在 `0.62-0.66`；
  - 对应 `gamma_shape_file` 也只在大约 `0.65-0.73` 范围内轻微波动。
- 这说明：在当前 `Gravel_CHB` 生成链里，`Ne(0)` 主要改变的是 tip branch 的 **scale**，而不是把 gamma shape 大幅改型。
- 因此，`historical old/default g_file` 在 `DAF=0.05` 的首行 shape 仅 `0.246827`，**不能仅用“它可能是同一个 `Gravel_CHB` 但 `Ne(0)` 更大（如 ~270k）”来解释**。在本 panel 中，把 `Ne(0)` 从 `60k` 调到 `270k`，`DAF=0.05` 的 gamma shape 仍然维持在 `0.69-0.73`。
- 结合前一轮 Manhattan / p-value 审计，当前最合理的 technical reason 是：
  1. `0.01 < MAF < 0.05` 的 low-frequency common variants 被 `compute_SDS.py` 边界 clamping 到 `0.05` 的 gamma shape；
  2. 当前 `Gravel_CHB@100k` 的低频边界 shape 本身较大；
  3. `Ne(0)` 在同一 `Gravel_CHB` family 内只会轻度改变 shape，不能单独解释 olddefault 与当前 Gravel gamma 的巨大差异；
  4. 因而异常 p-value 分布更像是 **gamma provenance / low-frequency coverage mismatch**，而不是单独的 `Ne(0)` 设定问题。

Next step

- 如需继续修复，应优先做：
  - 把 gamma grid 扩到 `DAF=0.01`，避免 `<0.05` 全部被 clamp 到 `0.05`；
  - 或在解释与作图时先限制到 `MAF >= 0.05`；
  - 再单独追查 `historical old/default g_file` 的真实 provenance，而不是继续假设它只是 `Gravel_CHB + larger Ne(0)`。
