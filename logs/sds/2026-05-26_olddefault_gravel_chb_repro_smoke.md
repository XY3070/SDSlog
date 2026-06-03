Date: 2026-05-26
Topic: sds
Type: test

Question

在用户补充 `olddefault g_file` 的生成背景后，做一个最小复现实验：

- demographic family 固定为 `Gravel_CHB`
- `present-day Ne(0)=219253`
- 比较当前 `ms/scripts/Makefile` 与 `2026-04-05` 快照 `Makefile.bak`
- 只测最关键的 `DAF = 0.05, 0.07, 0.10`

目标不是重跑整条 gamma pipeline，而是先回答：

1. 只把 `Ne(0)` 改成 `219253`，shape 能否接近 `olddefault g_file`？
2. 旧快照 `Makefile` 是否能在同条件下复现出显著更接近 `olddefault` 的 shape？

Input paths

- `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/Makefile`
- `/data/home/grp-wangyf/xuyuan/ms/scripts/snapshots/20260405_1528/Makefile.bak`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/NCN_scaled_ne0_100000.npz`

Output paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/summary.md`

Parameters

- `demographic_model = Gravel_CHB`
- `present_ne = 219253`
- `daf = 0.05, 0.07, 0.10`
- `sim_reps = 100`

Demographic model

| Track | Demographic model | Gamma shape source | Present-day `Ne(0)` | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `olddefault` | likely `Gravel_CHB` family | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `219253` per user recollection | reference | 老师最初让先跑通流程时使用的历史 gamma file |
| `current smoke` | `Gravel_CHB` | current `ms/scripts/Makefile` | `219253` | audit-only | 复现实验 |
| `snapshot-native smoke` | `Gravel_CHB` | `2026-04-05` snapshot `Makefile.bak`, native target | `219253` | audit-only | 检查旧快照是否更接近 olddefault |

Scripts / commands / job IDs

- scripts:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_piece.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_single_snp_gamma_piece_snapshot_native.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/summarize_olddefault_repro_smoke.py`
- current-Makefile jobs:
  - `487056`: `olddefault_repro_current_daf0p05`
  - `487059`: `olddefault_repro_current_daf0p07`
  - `487060`: `olddefault_repro_current_daf0p10`
- snapshot-native jobs:
  - `487063`: `olddefault_repro_snapshotnative_daf0p05`
  - `487064`: `olddefault_repro_snapshotnative_daf0p07`
  - `487065`: `olddefault_repro_snapshotnative_daf0p10`
- failed interface-mismatch jobs retained in manifest as `missing`:
  - `487057`, `487058`, `487061`
  - they failed because `Makefile.bak` does not expose the newer `sim_single_daf` interface expected by `run_single_snp_gamma_piece.sh`

Key result files

- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_current_daf0p05/gamma.0.05`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_current_daf0p07/gamma.0.07`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_current_daf0p10/gamma.0.10`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_snapshotnative_daf0p05/gamma.0.05`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_snapshotnative_daf0p07/gamma.0.07`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/olddefault_repro_smoke_20260525/results/olddefault_repro_snapshotnative_daf0p10/gamma.0.10`

Result

`olddefault g_file` 的对应行：

- `0.05 -> 0.246827025658222`
- `0.07 -> 0.223486576382251`
- `0.10 -> 0.190725152904082`

当前 `Makefile`、`Ne(0)=219253` 的 smoke：

- `0.05 -> 0.705617788029301`
- `0.07 -> 0.664106851010389`
- `0.10 -> 0.667451790728288`

`2026-04-05` 快照 `Makefile.bak`，按 native target 跑出的 smoke：

- `0.05 -> 0.704937770305459`
- `0.07 -> 0.705570208517984`
- `0.10 -> 0.675649480485596`

另外，当前与快照的 `Makefile` 默认 `SAMPLE_SIZE` 都是：

- `SAMPLE_SIZE = 2510`

Conclusion

- 在当前掌握的条件下，**只把 `Ne(0)` 固定成 `219253`，并不能复现 `olddefault g_file` 的 low-frequency / low-DAF shape**。
- 更重要的是，**当前 `Makefile` 和 `2026-04-05` 快照 `Makefile.bak` 在同一 `Gravel_CHB + Ne(0)=219253` 条件下给出的 shape 基本一致**：
  - 都在 `~0.66–0.71`
  - 都明显高于 `olddefault` 的 `~0.19–0.25`
- 这说明：
  1. `olddefault` 与 current/snapshot-native smoke 的差异，**不能简单归因于后续 `Makefile` 修改**；
  2. 也**不能简单归因于 current `Gravel_CHB@100k` 把 `Ne(0)` 取成 100k**，因为在 `219253` 下 shape 仍然维持在 `~0.7`；
  3. 更深层的差异很可能还在 gamma-generation provenance 的别处，例如：
     - 历史生成链与当前 smoke 使用的入口并不完全同源
     - 历史运行时另有未恢复的参数覆盖
     - 不是当前 `Makefile` / 当前快照 `Makefile` 这一级就能解释
- 当前最稳的结论是：
  - **`Makefile` 不是 olddefault/current 差异的主因**
  - **`Ne(0)=219253` 本身也不足以把当前 `Gravel_CHB` family 的 shape 拉回 olddefault**
  - 下一步应把注意力继续转向 `olddefault g_file` 的完整 provenance，而不是继续把问题主要归咎于 `Makefile` 或 `Ne(0)` 单独一项

Next step

- 优先追 `olddefault g_file` 的完整生成链 provenance：
  - 历史调用脚本
  - 是否有额外 `SAMPLE_SIZE` / population-size / aggregation override
  - 是否使用了不同的 `ms` / `backward.py` / wrapper
- 在现有结果层面，`current Makefile` 与 `snapshot-native Makefile` 的 smoke 已足以说明：
  - 当前要解决的问题已经超出“修复 Makefile”这个层级
