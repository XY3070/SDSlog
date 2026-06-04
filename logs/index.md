# Logs Index

| Topic | Scope | Latest log | Status | Notes |
| --- | --- | --- | --- | --- |
| `cohort_qc` | sample QC、population split、freeze decision | No detailed log yet | backfill needed | 先补 1st / 2nd QC 和 NCN / SCN split 的文件路径 |
| `demography` | phlash、SMC++、Relate、recent `Ne` comparison | [2026-04-23_ncn_recent_ne_benchmark.md](demography/2026-04-23_ncn_recent_ne_benchmark.md) | active | 首批 benchmark 文件和关键图已登记 |
| `sds` | genome-wide scan、postprocess、single-SNP sensitivity、region panel | [2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md](sds/2026-06-03_pipeline_reflection_bfile_snp_gamma_demography.md) | **reflection pause** | 2026-06-03 系统性反思，识别出三个 blocking 问题：(1) b_file 把着丝粒包含进 arm 范围（chr1 q-arm 起始在着丝粒内部，chr6 深入 2.3 Mb），chr1 两个”最强峰”（125.1M/143.2M）确认为 1q12 pericentromeric HSATII mapping artifact，峰内零基因；(2) 当前 SNP input contract (PASS+biallelic+F_MISSING) 不足以排除着丝粒 satellite DNA 和极端 rare variant；(3) olddefault gamma shape (0.026-0.25) 与 Gravel_CHB@100k shape (0.63-0.71) 相差 3-27 倍，olddefault provenance 不明且不可复现。同时发现 chr6 MHC 在所有 baseline 中都没有正选择信号（与已知人类选择证据矛盾），pipeline 存在”假阳性敏感、假阴性不敏感”的结构性问题。基因注释确认所有超极端峰（chr1:125M/143M, chr9:61.6M, chr16:46.4M）都不在蛋白编码基因上。暂停新 compute run，优先解决 gamma provenance。之前 SCN boundary-policy 修复工作（relaxed035/cap_to_boundary/hybrid-policy/corrected chunk20 rerun/boundarymask postprocess）的技术结论仍然有效，但 paper-facing 结论需要等 gamma provenance 和 b_file fix 解决后才能出。详见：[peak 基因注释](sds/2026-06-03_peak_gene_annotation_and_biological_assessment.md)、[chr6 MHC 缺失](sds/2026-06-03_chr6_mhc_signal_absence.md)。控制变量总表见 [2026-05-22_sds_control_variable_report_for_teacher.md](sds/2026-05-22_sds_control_variable_report_for_teacher.md)；demographic-model 单一真值见 [README_demographic_models.md](sds/README_demographic_models.md) |
| `ihs_ehh` | iHS、EHH、gene label、peak follow-up | [README.md](ihs_ehh/README.md) | backfill needed | 方向已确定，文件和图未系统登记 |
| `meta_overlap` | meta、GWAS overlap、tSDS | [README.md](meta_overlap/README.md) | backfill needed | 现有结论已写入总览，缺文件级记录 |
| `relate_clues2` | Relate、CLUES2、selection trajectory | [README.md](relate_clues2/README.md) | backfill needed | 需要 paper-facing 结果与图 |

## Priority Backfill

1. `cohort_qc`: 把最终纳入样本表、排除规则和南北划分文件路径补进来。
2. `ihs_ehh`: 把显著位点表、gene label 输出和关键图补进来。
3. `meta_overlap`: 把 Nyuwa overlap / meta 的输入输出路径补进来。
