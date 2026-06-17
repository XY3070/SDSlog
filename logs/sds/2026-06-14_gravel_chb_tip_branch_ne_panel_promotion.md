Date: 2026-06-14
Topic: sds
Type: audit

Question

将 2026-05-25 的 `Gravel_CHB` `Ne(0)` tip-branch panel 从 `tmp` 归档到 `SDSworkspace`，并生成一张可以直接引用的图，作为“`Ne(0)` 主要改 scale，不明显改 shape”的论据。

Input paths

- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel_manifest.tsv`

Canonical workspace paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel_manifest.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.md`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/tip_branch_panel.summary.json`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/gravel_chb_tip_branch_ne_panel_overview.png`

Provenance paths

- `/data/home/grp-wangyf/xuyuan/SDSworkspace/provenance/runs/gravel_chb_tip_branch_ne_panel_20260525/run_manifest.yaml`

Conclusion

- 这份 panel 的数值足够稳定，适合作为论据：
  - `shape_recomputed` 只在 `0.6178156122381454-0.6488616268151765` 之间波动；
  - `gamma_shape_file` 只在 `0.656892376852678-0.725245522728335` 之间波动；
  - `tip_branch mean` 则随着 `Ne(0)` 从 `60000` 增到 `270000` 显著上升。
- 这和 `olddefault g_file` 的 `DAF=0.05` 首行 shape `0.246827025658222` 形成明显对照，支持下面这个判断：
  - `Ne(0)` 在当前 `Gravel_CHB` family 内主要改变 tip branch 的 **scale**；
  - 它不能单独解释 `olddefault` 与当前 `Gravel_CHB@100k` 之间的 shape 差异。
- 已把结果归档到 `SDSworkspace/results/audit/gravel_chb_tip_branch_ne_panel_20260525/`，并生成可直接引用的图。

Next step

- 若要继续追根因，应优先追 `historical old/default g_file` 的完整 provenance，而不是继续把差异主要归到 `Ne(0)`。
