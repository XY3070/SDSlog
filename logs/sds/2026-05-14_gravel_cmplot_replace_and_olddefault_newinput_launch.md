Date: 2026-05-14
Topic: sds
Type: summary

Question
`SDS-MODEL-003` (`Gravel_CHB`, `Ne(0)=100000`) 的 `NCN/SCN` 全基因组 Manhattan 图极不合理。需要先把官方图替换成可审图的 `CMplot` 版本，再判断问题主要来自 plotting、input，还是 demographic model / gamma track。

Demographic model

Track | Demographic model | Gamma shape source | Ne(0) | Role | Note
NCN Gravel run | Gravel_CHB | /data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt | 100000 | rejected-under-review | 图和 raw rSDS 分布都不合理
SCN Gravel run | Gravel_CHB | /data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt | 100000 | rejected-under-review | 图和 raw rSDS 分布都不合理
NCN olddefault rerun | historical old/default gamma table | /data/home/grp-wangyf/xuyuan/sds/g_file.txt | implicit / unknown from historical file provenance | audit-only | 用同一套 rebuilt new input 复核是 input 问题还是 model 问题
SCN olddefault rerun | historical old/default gamma table | /data/home/grp-wangyf/xuyuan/sds/g_file.txt | implicit / unknown from historical file provenance | audit-only | 用同一套 rebuilt new input 复核是 input 问题还是 model 问题

Input paths
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/NCN
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.normalized.tsv
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.normalized.tsv

Output paths
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.manhattan.png
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.manhattan.png
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.manhattan.matplotlib_backup_20260513.png
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.manhattan.matplotlib_backup_20260513.png
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514

Parameters
CMplot redraw filters: `plot_p_threshold=5e-4`, `0.01<DAF<0.99`, `MAF>0.01`
Olddefault rerun input root: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511`
Olddefault rerun output root: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514`

Scripts / commands / job IDs
`/data/home/grp-wangyf/xuyuan/sds/scripts/report_cmplot_manhattan.R`
`/data/home/grp-wangyf/xuyuan/sds/scripts/submit_olddefault_newinput_chrlist.sh`
`NCN olddefault newinput chr1` chunk arrays / merge / final started as `485556`, `485557`, `485558`, `485559`, `485560`

Key plots
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/cmplot/NCN.cmplot_p5e4_maf001.manhattan.png
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/cmplot/SCN.cmplot_p5e4_maf001.manhattan.png

Key result files
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.frequency_bins.tsv
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.frequency_bins.tsv
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/diagnostics/NCN.gravel_chb_ne100k_manual.top_hits.tsv
/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/diagnostics/SCN.gravel_chb_ne100k_manual.top_hits.tsv

Conclusion
先前官方 `NCN.manhattan.png` / `SCN.manhattan.png` 是 matplotlib 版，而且 summary 中 `plot_p_threshold` 为空，约 `7M` common points 全部被画出。现已用 `CMplot` 版本覆盖官方 `manhattan.png`，并保留 matplotlib 备份。

但 plotting 不是主因。后处理逻辑本身是标准的 genome-wide pooled MAF-bin normalization；真正的病灶在上游 raw `rSDS` 分布。`SCN` 的低频 common bins 已经整体偏移（如 `[0.01,0.02)` bin mean `4.392`, sd `10.505`），`NCN` 也出现大量 raw `rSDS` 极端值（如 `rSDS 21~42`），说明 `SDS-MODEL-003` 在这批数据上产生了不合理的 genome-wide tail。

目前没有看到 rebuilt new input contract 足以单独解释这种广泛 common-variant 爆炸；更合理的解释是 `Gravel_CHB + Ne(0)=100000` 这条 gamma track 本身不适配。为拆分 input 与 model，已启动 `same new input + historical old/default gamma table` 的全基因组复跑。

Next step
等待 `NCN/SCN olddefault newinput` launcher 完成全染色体提交与后处理。
对比 `same new input` 下的 `old/default` 与 `Gravel_CHB@100k` Manhattan、tail counts、top hits。
若 `old/default newinput` 恢复明显正常，则把当前问题归到 demographic model / gamma track，而不是 input。
