Date: 2026-05-22
Topic: sds
Type: summary

Question

把 2026-04-25 之后围绕 SDS 结果稳定性做的工作，整理成一份可直接向老师汇报的控制变量报告。重点回答：

1. 哪些因素可能影响 SDS 结果；
2. 针对每个因素做了哪些对照实验；
3. 每个实验对应哪些 Manhattan 图；
4. 目前最合理的技术结论是什么。

Report focus

本阶段的核心不是“再多跑几版 SDS”，而是系统拆解以下 4 类变量对 `rSDS / norm SDS` 和 Manhattan 形态的影响：

1. `SDS input`
2. `Ne(0)`
3. `demographic model`（即 `Ne(t)` / gamma shape）
4. `pop-split`

其中，`SDS input` 里最关键的是 `singleton universe / s_file`；`pop-split` 这条线则通过 `NCN / SCN / merged` 与 raw-joint 重建来判断 split 语义是否改变了结果。

Executive summary

1. 目前最强的技术判断不是“某一个变量单独导致坏图”，而是：
   - `legacy/unpop-like` 与 `current pop-specific` 的 input lineage / site universe 差异是底层问题；
   - `Gravel_CHB@100k` 会在当前 pop-specific input 上放大这些问题，形成 diffuse tail。
2. `Ne(0)` 会影响 `rSDS` 数值，但现阶段没有证据支持它是 genome-wide 坏图的单独主因。
3. 现阶段能直接配上全基因组 `CMplot` Manhattan 的对照实验，主要集中在：
   - `demographic model`
   - `old input` vs `new input`
   - `NCN` vs `SCN`
4. `pop-split` 的 merged 重建目前最强证据来自 chr1 审计，不是全基因组 `CMplot`。

Models and labels used in this report

| Label | Demographic model | Gamma shape source | `Ne(0)` | Input label | Population label |
| --- | --- | --- | --- | --- | --- |
| `olddefault_oldinput_NCN` | `SDS-MODEL-001 historical_olddefault_gamma_table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown` | `old` | `NCN` |
| `phlash100k_oldinput_NCN` | `SDS-MODEL-002 phlash_q50_scaled_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | `old` | `NCN` |
| `gravel100k_oldinput_NCN` | `SDS-MODEL-003 Gravel_CHB_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `old` | `NCN` |
| `olddefault_newinput_NCN` | `SDS-MODEL-001 historical_olddefault_gamma_table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown` | `new` | `NCN` |
| `gravel100k_newinput_NCN` | `SDS-MODEL-003 Gravel_CHB_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `new` | `NCN` |
| `gravel100k_newinput_SCN` | `SDS-MODEL-003 Gravel_CHB_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | `new` | `SCN` |

Control-variable experiment master table

| Factor tested | Controlled setup | Compared tracks | Fixed variables | Corresponding Manhattan / compare plot | Current result |
| --- | --- | --- | --- | --- | --- |
| `demographic model` | same `NCN`, same old input | `olddefault_oldinput_NCN` vs `phlash100k_oldinput_NCN` vs `gravel100k_oldinput_NCN` | same cohort, same old input lineage | `PLT-SDS-006`, `PLT-SDS-008`, `PLT-SDS-007` | `olddefault` 最稳；`phlash100k` 和 `Gravel_CHB@100k` 都出现 diffuse tail |
| `SDS input` | same `NCN`, same `SDS-MODEL-001` | `olddefault_oldinput_NCN` vs `olddefault_newinput_NCN` | same gamma track, same cohort | `PLT-SDS-006` vs `PLT-SDS-013` | new input 更规范，但没有恢复出 old/unpop-like 那套更 plausible 的 site universe |
| `SDS input` + `demographic model` interaction | same `NCN`, same `Gravel_CHB@100k` | `gravel100k_oldinput_NCN` vs `gravel100k_newinput_NCN` | same model, same explicit `Ne(0)=100000` | `PLT-SDS-007` vs `PLT-SDS-011` | `Gravel_CHB@100k` 在两套 input 上都不理想；说明 model 会放大 lineage 差异，但不是唯一主因 |
| `demographic model` on same new input | same rebuilt `NCN` input | `olddefault_newinput_NCN` vs `gravel100k_newinput_NCN` | same new input, same cohort | `PLT-SDS-013` vs `PLT-SDS-011` | 在同一 input 上，`olddefault` 明显比 `Gravel_CHB@100k` 更 plausible，说明 `Gravel` 在 current input 上是 amplifier |
| `pop-split` | same rebuilt new input, same `Gravel_CHB@100k` | `gravel100k_newinput_NCN` vs `gravel100k_newinput_SCN` | same gamma track, same explicit `Ne(0)=100000`, same input contract | `PLT-SDS-011` vs `PLT-SDS-012` | `NCN` 更集中出现异常峰，`SCN` 更分散，但两者都不够 plausible；说明问题不是 `NCN` 单独独有 |
| `Ne(0)` | region / single-SNP sensitivity | `100k`, `152k`, `250k`, `500k`, `750k`, `1e6` 等 | same loci / region panel | no genome-wide `CMplot`; region-level only | `Ne(0)` 会影响 `rSDS` 数值，但目前没有全基因组图证据支持它是坏图主因 |
| `pop-split` + merged semantics | chr1 targeted adjudication | `NCN`, `SCN`, `MERGED3971PL`, raw-joint merged | same chr1 target windows | `PLT-SDS-009`, `PLT-SDS-010` | merged / split 语义确实会改变 singleton universe，但这条线目前是 chr1 local compare，不是全基因组 Manhattan |

CMplot figure catalog for reporting

| Plot ID | Figure | Demographic model | `Ne(0)` | Input | Population | Scope | Recommended use in report |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `PLT-SDS-006` | [20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_olddefault_mainline_manhattan_p5e4_v01.png) | `old gamma` | `implicit / unknown` | `old` | `NCN` | genome-wide filtered `CMplot` | old-input baseline |
| `PLT-SDS-007` | [20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_gravel_chb_ne100k_manhattan_p5e4_v01.png) | `default CHB / Gravel_CHB` | `100000` | `old` | `NCN` | genome-wide filtered `CMplot` | same old input 下的 `Gravel` compare |
| `PLT-SDS-008` | [20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png](../../plots/sds/20260503_NCN_sds_phlash_scaled_ne100k_manhattan_p5e4_v01.png) | `phlash q50 scaled` | `100000` | `old` | `NCN` | genome-wide filtered `CMplot` | same old input 下的 phlash compare |
| `PLT-SDS-011` | [20260522_NCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png](../../plots/sds/20260522_NCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png) | `default CHB / Gravel_CHB` | `100000` | `new` | `NCN` | genome-wide official `CMplot` | new-input `NCN` `Gravel` compare |
| `PLT-SDS-012` | [20260522_SCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png](../../plots/sds/20260522_SCN_sds_gravel_chb_ne100k_newinput_cmplot_v01.png) | `default CHB / Gravel_CHB` | `100000` | `new` | `SCN` | genome-wide official `CMplot` | pop-split compare |
| `PLT-SDS-013` | [20260522_NCN_sds_olddefault_newinput_cmplot_v01.png](../../plots/sds/20260522_NCN_sds_olddefault_newinput_cmplot_v01.png) | `old gamma` | `implicit / unknown` | `new` | `NCN` | genome-wide filtered `CMplot` | same new input 下的 olddefault baseline |

Factor-by-factor interpretation

## 1. `SDS input`

What was controlled

- same cohort, same gamma, compare `old input` vs `new input`
- same `Gravel_CHB@100k`, compare `old input` vs `new input`
- targeted chr1 / chr6 lineage audit

What the plots show

- `PLT-SDS-006` vs `PLT-SDS-013`:
  - current rebuilt new input 在工程上更规范；
  - 但它没有恢复出 old/unpop-like 那种更 plausible 的 Manhattan 背景。
- `PLT-SDS-007` vs `PLT-SDS-011`:
  - 在 `Gravel_CHB@100k` 下，old/new 两套 input 都不够 plausible；
  - 说明 input 确实重要，但 `Gravel` 也在放大问题。

Current conclusion

- `s_file / singleton universe` 是 input 里最关键的变量；
- rebuilt main-contract input 不是“坏 input”，但它和 old/unpop-like 不是同一个 site universe。

## 2. `Ne(0)`

What was controlled

- single-site gamma sensitivity
- region `Ne(0)` panel
- explicit `100000` production tracks

What the plots show

- 目前**真正有全基因组 `CMplot`** 的显式 `Ne(0)` track 只有 `100000`；
- 其它 `152k/250k/500k/...` 主要停留在 region / single-site sensitivity，没有全基因组 Manhattan。

Current conclusion

- `Ne(0)` 会影响 `rSDS` 数值；
- 但现阶段没有证据支持“单独改 `Ne(0)` 就能把 plausible 图变成 diffuse tail”。

## 3. `demographic model`

What was controlled

- same old input, compare `old gamma` / `phlash` / `Gravel_CHB`
- same new input, compare `old gamma` / `Gravel_CHB`

What the plots show

- `PLT-SDS-006` vs `PLT-SDS-008` vs `PLT-SDS-007`:
  - 在 old input 上，`old gamma` 最稳；
  - `phlash` 和 `Gravel_CHB@100k` 都有明显 diffuse tail。
- `PLT-SDS-013` vs `PLT-SDS-011`:
  - 在 same new input 上，`old gamma` 明显比 `Gravel_CHB@100k` 更 plausible。

Current conclusion

- `demographic model` 不是完全无关；
- 但它更像 **amplifier**，不是当前唯一 root cause。

## 4. `pop-split`

What was controlled

- `NCN` vs `SCN` on same rebuilt new input and same `Gravel_CHB@100k`
- merged reconstruction from another raw-joint VCF

What the plots show

- `PLT-SDS-011` vs `PLT-SDS-012`:
  - `NCN` 更集中出现异常峰；
  - `SCN` 也不够安静，只是更分散。
- `PLT-SDS-009/010`:
  - raw-joint / merged 语义会影响 merged singleton universe；
  - 但这条证据目前来自 chr1 targeted adjudication，而不是全基因组 Manhattan。

Current conclusion

- `pop-split` 是需要单独控制的变量；
- 旧 `unpop` 和当前 `NCN/SCN` 不能简单当成同一条 input lineage。

What is still missing

1. 目前没有一套**显式不同 `Ne(0)` 的全基因组 Manhattan**，所以 `Ne(0)` 这条线主要还是 region / single-site evidence。
2. `pop-split` 的 merged 重建目前最强证据是 chr1 local compare，不是全基因组 `CMplot`。
3. 旧 `unpop` baseline 图目前有用户确认的合理图形，但没有在本仓库里恢复成一套可完全追溯的全基因组 input + `CMplot` 产物。

Current best summary for teacher report

1. 影响 SDS 结果的不止 `Ne(0)`、`demographic model` 和 `input`，还包括 `pop-split` 与 source VCF 语义。
2. 现阶段最关键的底层问题是：
   - `legacy/unpop-like` 与 `current pop-specific` 的 input lineage / site universe 不一致。
3. 在这个前提下：
   - `old gamma` 目前仍是最稳的主线；
   - `phlash` 和 `Gravel_CHB@100k` 在当前 pop-specific input 上都会放大问题；
   - `Ne(0)` 会影响数值，但不是当前坏图的单独主因。

Recommended figure set for presentation

1. `demographic model`:
   - `PLT-SDS-006`
   - `PLT-SDS-008`
   - `PLT-SDS-007`
2. `input`:
   - `PLT-SDS-006`
   - `PLT-SDS-013`
3. `same-input model compare`:
   - `PLT-SDS-013`
   - `PLT-SDS-011`
4. `pop-split`:
   - `PLT-SDS-011`
   - `PLT-SDS-012`
5. `merged / split semantics`:
   - `PLT-SDS-009`
   - `PLT-SDS-010`
