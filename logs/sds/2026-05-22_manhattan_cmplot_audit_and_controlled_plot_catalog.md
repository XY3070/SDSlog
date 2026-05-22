Date: 2026-05-22
Topic: sds
Type: summary

Question

检查 `SDSlog` 中登记的 Manhattan 图是否都已经是 `CMplot` version；同时把控制变量实验中，和 `Ne(0)` 以及 demographic model 直接对应的图整理出来，便于后续汇报和追溯。

Conclusion

1. **不是所有登记在 `SDSlog` 的 Manhattan 都是 `CMplot` version。**
2. 目前 `SDSlog` 中的 Manhattan 图可以分成 3 类：
   - `legacy non-CMplot genome-wide Manhattan`
   - `CMplot filtered / official Manhattan`
   - `custom chr1 local compare`（不是 genome-wide Manhattan，也不是 `CMplot`）
3. 当前和控制变量实验最相关、最值得汇报的图，应该优先看：
   - `PLT-SDS-006`, `PLT-SDS-007`, `PLT-SDS-008`
   - `PLT-SDS-011`, `PLT-SDS-012`, `PLT-SDS-013`
4. `Ne(0)` 这条线，目前**真正有全基因组 Manhattan 的显式 production track 只有 `Ne(0)=100000`**；其他 `Ne(0)` 场景主要停留在 region / single-SNP sensitivity，没有对应的全基因组 Manhattan。

Manhattan audit

| Plot ID | Figure | Scope | Model | `Ne(0)` | Render type | CMplot? | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `PLT-SDS-001` | `20260409_NCN_sds_genomewide_manhattan_v01.png` | genome-wide | `SDS-MODEL-001` | implicit / unknown | legacy official | no | 旧归档图，保留原始结果形态 |
| `PLT-SDS-003` | `20260503_NCN_sds_olddefault_mainline_manhattan_v01.png` | genome-wide | `SDS-MODEL-001` | implicit / unknown | legacy official | no | gamma-track compare 时归档的旧正式图 |
| `PLT-SDS-004` | `20260503_NCN_sds_gravel_chb_ne100k_manhattan_v01.png` | genome-wide | `SDS-MODEL-003` | `100000` | legacy official | no | compare 时的旧正式图 |
| `PLT-SDS-005` | `20260503_NCN_sds_phlash_scaled_ne100k_manhattan_v01.png` | genome-wide | `SDS-MODEL-002` | `100000` | legacy official | no | compare 时的旧正式图 |
| `PLT-SDS-006` | `20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png` | genome-wide filtered | `SDS-MODEL-001` | implicit / unknown | filtered display | yes | `CMplot`, `p<0.0005`, `0.01<DAF<0.99`, `MAF>0.01` |
| `PLT-SDS-007` | `20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png` | genome-wide filtered | `SDS-MODEL-003` | `100000` | filtered display | yes | `CMplot`, rejected track |
| `PLT-SDS-008` | `20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png` | genome-wide filtered | `SDS-MODEL-002` | `100000` | filtered display | yes | `CMplot`, rejected track |
| `PLT-SDS-009` | `20260511_chr1_local_compare_raw_joint_v2_v01.png` | chr1 local compare | `SDS-MODEL-001` | implicit / unknown | custom R compare | no | 不是 genome-wide Manhattan |
| `PLT-SDS-010` | `20260511_chr1_local_compare_legacy_adjudication_v01.png` | chr1 local compare | `SDS-MODEL-001` | implicit / unknown | custom R compare | no | 不是 genome-wide Manhattan |
| `PLT-SDS-011` | `20260522_NCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png` | genome-wide | `SDS-MODEL-003` | `100000` | official replaced by `CMplot` | yes | rebuilt new input, `NCN` |
| `PLT-SDS-012` | `20260522_SCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png` | genome-wide | `SDS-MODEL-003` | `100000` | official replaced by `CMplot` | yes | rebuilt new input, `SCN` |
| `PLT-SDS-013` | `20260522_NCN_sds_olddefault_newinput_cmplot_v01.png` | genome-wide filtered | `SDS-MODEL-001` | implicit / unknown | filtered display | yes | rebuilt new input, same-input model compare baseline |

`Ne(0)` controlled plot catalog

| `Ne(0)` condition | Track / Model | Figure | Scope | Status | Note |
| --- | --- | --- | --- | --- | --- |
| implicit / unknown | `SDS-MODEL-001 historical old/default` | `PLT-SDS-003`, `PLT-SDS-006`, `PLT-SDS-013` | genome-wide | keep | 当前主线或 same-input baseline；不是显式 `Ne(0)` production track |
| `100000` | `SDS-MODEL-002 phlash_q50_scaled_ne100k` | `PLT-SDS-005`, `PLT-SDS-008` | genome-wide | rejected | 只有 `NCN` 全基因组图；过滤后仍 diffuse tail |
| `100000` | `SDS-MODEL-003 Gravel_CHB_ne100k` old compare | `PLT-SDS-004`, `PLT-SDS-007` | genome-wide | rejected | 旧 input lineage 下的 compare 图 |
| `100000` | `SDS-MODEL-003 Gravel_CHB_ne100k` rebuilt new input `NCN` | `PLT-SDS-011` | genome-wide | rejected | 官方图已替换成 `CMplot` |
| `100000` | `SDS-MODEL-003 Gravel_CHB_ne100k` rebuilt new input `SCN` | `PLT-SDS-012` | genome-wide | rejected | 官方图已替换成 `CMplot` |
| `152000` | single-site constant `Ne` sensitivity | no Manhattan archived | single-SNP only | sensitivity only | 见 `SDS-003`, `SDS-004`；没有全基因组 Manhattan |
| `250000/500000/750000/1000000/...` | `SDS-MODEL-005 region_ne0_panel_family` | no Manhattan archived | region only | audit-only | 这些场景只做 region panel，没有正式全基因组 Manhattan |

Demographic-model controlled plot catalog

| Demographic model | Figure(s) | Input lineage | Scope | Main observation |
| --- | --- | --- | --- | --- |
| `SDS-MODEL-001 historical_olddefault_gamma_table` | `PLT-SDS-003`, `PLT-SDS-006`, `PLT-SDS-013` | old mainline / rebuilt new input | genome-wide | 当前最稳；same-input compare 下明显优于 `SDS-MODEL-003` |
| `SDS-MODEL-002 phlash_q50_scaled_ne100k` | `PLT-SDS-005`, `PLT-SDS-008` | old compare lineage | genome-wide | rejected；过滤后仍 diffuse tail |
| `SDS-MODEL-003 Gravel_CHB_ne100k` | `PLT-SDS-004`, `PLT-SDS-007`, `PLT-SDS-011`, `PLT-SDS-012` | old compare lineage / rebuilt new input | genome-wide | 在当前 pop-specific input 上会显著放大 tail；`NCN/SCN` 都不 plausible |
| `SDS-MODEL-001` + raw-joint / parent-like adjudication | `PLT-SDS-009`, `PLT-SDS-010` | raw-joint / legacy adjudication | chr1 local compare | 这些是 input-lineage compare，不是新的 demographic model 图 |

Interpretation for reporting

1. 若汇报主题是“不同 `Ne(0)` 下图形怎样变化”，应明确说明：
   - 目前真正有全基因组 Manhattan 的显式 `Ne(0)` production track 只有 `100000`；
   - 其他 `Ne(0)` 主要停留在 region / single-site sensitivity。
2. 若汇报主题是“不同 demographic model 下图形怎样变化”，应优先展示：
   - `PLT-SDS-006` vs `PLT-SDS-008` vs `PLT-SDS-007`
   - 以及 `PLT-SDS-013` vs `PLT-SDS-011`
3. 若汇报主题是“现在 `SDSlog` 里 Manhattan 是否都已经 `CMplot` 化”，答案是：
   - **否。**
   - `PLT-SDS-001/003/004/005` 仍是 non-`CMplot` 或 legacy official 图；
   - `PLT-SDS-006/007/008/011/012/013` 是 `CMplot`；
   - `PLT-SDS-009/010` 是 custom chr1 local compare。

Files added to `SDSlog/plots/sds`

- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260522_NCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260522_SCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png`
- `/data/home/grp-wangyf/xuyuan/SDSlog/plots/sds/20260522_NCN_sds_olddefault_newinput_cmplot_v01.png`
