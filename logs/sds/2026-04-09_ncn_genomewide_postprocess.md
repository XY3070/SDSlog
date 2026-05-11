# NCN genome-wide SDS postprocess

Date: 2026-04-09
Topic: sds
Type: production
Question: 当前 NCN genome-wide SDS postprocess 产出了哪些主结果文件，规模多大，后续汇报应该引用哪里？

## Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/chr1_p.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/chr22_q.sds.tsv`

## Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.significant_regions.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.manhattan_points.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.postprocess_summary.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/SDS_Analysis_Result_Manhattan.png`

## Parameters

- `population=NCN`
- `maf_threshold=0.01`
- `bonferroni_threshold(summary)=6.919457927e-09`
- `plot_points_written=7191531`
- `normalized_rows=41225226 data lines`
- `significant_region_rows=2703 data lines`

## Demographic model

- `Demographic model=SDS-MODEL-001 (historical_olddefault_gamma_table)`
- `Gamma shape source=/data/home/grp-wangyf/xuyuan/sds/g_file.txt`
- `Present-day Ne(0)=implicit / unknown from historical file provenance`
- `Model role=mainline`
- `Model note=这是项目历史沿用的预计算 gamma table；当前不能把它记成显式 Ne(0)=100k`

## Scripts / commands / job IDs

- `/data/home/grp-wangyf/xuyuan/sds/logs/postprocess_sds_20260409_032753.log`
- `/data/home/grp-wangyf/xuyuan/sds/logs/postprocess_sds_20260409_032823.log`

## Key plots

- [PLT-SDS-001](../../plots/sds/20260409_NCN_sds_genomewide_manhattan_v01.png)
- [PLT-SDS-002](../../plots/sds/20260409_NCN_sds_binned_tail_enrichment_v01.png)

## Key result files

- `SDS-001`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.normalized.tsv`
- `SDS-002`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output/NCN/NCN.significant_regions.tsv`

## Conclusion

- genome-wide normalized SDS 主表已经产出，文件约 `6.4G`，适合作为后续所有 paper-facing downstream analysis 的主输入。
- significant region 表已有 `2703` 行数据窗口，可以直接作为 peak follow-up 的起点。
- `NCN.postprocess_summary.tsv` 里有空字段，且它记录的 Bonferroni threshold 与 `NCN.significant_regions.tsv` 内嵌阈值不完全一致，这个差异需要在最终 freeze 前核对。

## Next step

- 补齐 postprocess 的最终命令和配置快照，并确定后续统一引用的 genome-wide figure set。
