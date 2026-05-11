# SDS Demographic Model Registry

这份 registry 是 `SDSlog` 里所有 SDS 结果的 demographic-model 单一真值。以后 detailed log、result index、plot index 里不再自由发明 `old/default`、`100k track`、`CHB track` 这类模糊名字；统一引用这里的标准名字和含义。

## Registry

| Model ID | Human-readable name | Gamma file path | Shape provenance | Present-day `Ne(0)` | Status | Used in results | Notes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `SDS-MODEL-001` | `historical_olddefault_gamma_table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | 项目历史沿用的预计算 gamma table；当前能确认 `DAF` 网格是 `0.05-0.95 step 0.01` | `implicit / unknown from historical file provenance` | `mainline` | `SDS-001`, `SDS-002`, `SDS-003`, `SDS-004`, `SDS-011`, `SDS-014`, `SDS-017`, `SDS-018`, `SDS-020`, `SDS-021`, `SDS-026`, `SDS-028`-`SDS-033`, `SDS-036`, `SDS-037` | 当前唯一保留为主线的 gamma 口径；不要把它口头写成 `CHS` 或 `Ne(0)=100k` |
| `SDS-MODEL-002` | `phlash_q50_scaled_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | 以 phlash `q50` curve 为 shape template，缩放到 present-day `Ne(0)=100000` | `100000` | `rejected` | `SDS-005`-`SDS-010`, `SDS-020`, `SDS-023` | 只保留为 sensitivity / rejected track；Manhattan 有 diffuse tail，不用于主解释 |
| `SDS-MODEL-003` | `Gravel_CHB_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `Gravel_CHB` demography shape，chunked gamma generation，present-day `Ne(0)=100000` | `100000` | `rejected` | `SDS-015`, `SDS-016`, `SDS-019`, `SDS-020`, `SDS-022` | 只保留为 sensitivity / rejected track；Manhattan 有 diffuse tail，不用于主解释 |
| `SDS-MODEL-004` | `legacy_parentlike_input_context` | `inherits SDS-MODEL-001` | 不是新的 gamma file；只是 `MERGED3971PL` 这条 chr1 adjudication 轨道的 input lineage 说明 | `inherits SDS-MODEL-001` | `audit-only` | `SDS-032`, `SDS-033`, `PLT-SDS-010` | 只改变 merged 输入的构造方式，不改变 demographic model |
| `SDS-MODEL-005` | `region_ne0_panel_family` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scenario_manifest.tsv` | phlash posterior context + scaled `Ne(0)` scenario family | `mixed by scenario` | `audit-only` | `SDS-005`, `SDS-006`, `SDS-007`, `SDS-008` | 用于 region / single-site sensitivity，不对应单一 genome-wide production track |

## Usage Rules

1. 每个 SDS 结果都必须记录：
   - `Demographic model`
   - `Gamma shape source`
   - `Present-day Ne(0)`
   - `Model role`
2. `old/default` 不能单独出现，必须展开为 `historical old/default gamma table`。
3. 凡是写 `100k` 的条目，必须明确它是显式 present-day scaling，不得和 `historical old/default` 混用。
4. `raw-joint`、`input-fix`、`parent-like` 只描述 input lineage，不是 demographic model 名字。
5. 若某条结果无法从历史 provenance 恢复出单独的 `Ne(0)`，统一写：
   - `implicit / unknown from historical file provenance`
