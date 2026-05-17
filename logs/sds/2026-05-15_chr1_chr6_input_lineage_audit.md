Date: 2026-05-15
Topic: sds
Type: summary

Question

在用户确认 `old gamma + old input + unpop + Gravel CHB` 的 Manhattan 形态合理后，重新审计当前 `NCN/SCN` 问题的主因：到底是 demographic model 本身，还是 input lineage / site universe / split-pop 语义与 `Gravel CHB` gamma 的交互。

Input paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/legacy`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN`
- `/data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr6.vcf.gz`

Output paths

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/summary.md`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/input_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/model_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/frequency_bin_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/tail_compare.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/chr6_oldsplit_only_filter_breakdown.tsv`

Demographic tracks

| Track | Demographic model | Gamma shape source | Ne(0) | Role | Note |
| --- | --- | --- | --- | --- | --- |
| `NCN olddefault newinput` | `SDS-MODEL-001 historical old/default gamma table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | `audit-only` | 与 `Gravel_CHB@100k` 在同一套 rebuilt input 上直接做 same-input compare |
| `NCN Gravel_CHB@100k newinput` | `SDS-MODEL-003 Gravel_CHB` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `audit-only` | 用于判断 `Gravel CHB` 是否在 fixed input 上放大 artifact |
| `SCN Gravel_CHB@100k newinput` | `SDS-MODEL-003 Gravel_CHB` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `audit-only` | 低频 common bins 极不稳定，用于确认 amplification 不是 NCN 独有 |

Parameters

- audit panel: `chr1 + chr6`
- chr1 windows:
  - `peak125 = chr1:125100001-125200000`
  - `peak143 = chr1:143200001-143300000`
  - `control35 = chr1:35735220-36063259`
- chr6 peak windows:
  - `chr6_peak_1 = chr6:32600001-32700000`
  - `chr6_peak_2 = chr6:32500001-32600000`
- low-frequency bin check: `[0.01,0.02)`, `[0.02,0.03)`

Scripts / commands / job IDs

- audit script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/audit_input_lineage_chr_panel.py`
- targeted chr6 filter check:
  - `bcftools view -r chr6:32500001-32600000 -Ou /data/home/grp-wangyf/xuyuan/sds/data/vcf/NCN/UKBQC_NCN_chr6.vcf.gz | bcftools query -f '%CHROM\t%POS\t%REF\t%ALT\t%FILTER\n'`

Key result files

- `SDS-038`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/summary.md`
- `SDS-039`: `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/chr6_oldsplit_only_filter_breakdown.tsv`

Conclusion

1. `chr1` 上，`oldsplit NCN` 与 `rebuilt NCN` 几乎相同；这说明 2026-05 的 rebuilt main-contract input 并不是 chr1 主问题来源。
2. `chr1` 真正大的 lineage 差异在 `legacy/unpop-like` vs `current pop-specific`。`legacy chr1_t_file = 223,505`，而 `current/rebuilt NCN chr1_t_file = 3,278,034`，同时 `b_file` 边界也明显不同。
3. `chr6` 上，`oldsplit` 与 `rebuilt` 的主差异集中在 `chr6:32500001-32600000`。`oldsplit` 比 `rebuilt` 多出的 `862` 个 test SNP 全部是 `non-PASS`，且主要是 `LowQD` / `LowAAScore`。
4. 在同一套 rebuilt input 上，`Gravel_CHB@100k` 相对 `historical old/default` 明显放大 chr1 / chr6 的 `rSDS` tail。chr6 在 olddefault 下安静，但在 `Gravel_CHB@100k` 下变成全染色体高 tail。
5. 因此当前最合理的技术判断不是“demographic model 单独导致一切”，而是：
   - `legacy/unpop-like` 与 `current pop-specific` 的 input lineage 差异是底层问题；
   - `Gravel_CHB@100k` 在当前 pop-specific site universe 上会把这些问题系统性放大。

Next step

1. 不再把 `Gravel_CHB@100k` 当作当前 `NCN/SCN` 的主 production track。
2. 如果要继续追 `Gravel CHB`，优先恢复更接近 `old unpop` 的全基因组 input lineage，再与当前 pop-specific input 做 same-model compare。
3. 继续核对 `old unpop` 全基因组 input 是否存在可恢复路径，重点是 `chr1/chr6` 的 test-SNP universe 和 `b_file` 语义。
