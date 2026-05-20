Date: 2026-05-17
Topic: sds
Type: summary

Question

整理 2026-04-25 之后 SDS 相关分析，形成一份可向老师汇报的结果报告。重点回答：

1. 4.25 之后做了哪些分析；
2. 哪些变量会影响 `rSDS` / `norm SDS` 结果；
3. 针对这些变量做了哪些控制变量实验；
4. 每类实验得到的结果和当前判断是什么。

Report objective

这段时间的核心任务不是单纯“多跑几版 SDS”，而是系统拆解影响结果的技术变量，判断当前坏图到底来自：

- `SDS input`
- `present-day Ne(0)`
- `demographic model / gamma file`
- 还是更上游的 `input lineage / site universe / split-pop` 语义

Executive summary

1. 目前确认会显著影响 `rSDS` 的变量，不止 `SDS input`、`Ne(0)`、`demographic model` 三类；还包括：
   - `input lineage / site universe`
   - `population split strategy`
   - `singleton universe` 的定义方式
   - `t_file` / `s_file` 的过滤口径
   - `b_file` 边界语义
   - raw-joint vs split-then-merge 的 VCF 语义
2. 从控制变量实验看，**当前最强的主因不是单独的 `Ne(0)` 数值**。更合理的判断是：
   - `legacy/unpop-like` 与 `current pop-specific` 的 input lineage 差异是底层问题；
   - 某些 gamma track，尤其 `SDS-MODEL-003 Gravel_CHB@100k`，会把这些底层差异放大成坏的 genome-wide Manhattan。
3. `rebuilt main-contract input` 在工程上是目前最规范的 input，但它**没有**恢复出旧 `unpop` 轨道那套更 plausible 的 site universe。
4. `historical old/default gamma table` 目前仍是唯一可保留的主线口径；`phlash_q50_scaled_ne100k` 与 `Gravel_CHB_ne100k` 都不适合作为当前主 production track。

Demographic models used in this period

| Model ID | Human-readable name | Gamma shape source | Present-day `Ne(0)` | Role in this period | Current status |
| --- | --- | --- | --- | --- | --- |
| `SDS-MODEL-001` | `historical_olddefault_gamma_table` | `/data/home/grp-wangyf/xuyuan/sds/g_file.txt` | `implicit / unknown from historical file provenance` | genome-wide mainline, chr1 adjudication, same-input compare baseline | `retain as current mainline` |
| `SDS-MODEL-002` | `phlash_q50_scaled_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/region_ne0_positive_20260426/scaled_ne0_100000/scaled_ne0_100000.minimal_g.tsv` | `100000` | region / genome-wide sensitivity | `rejected` |
| `SDS-MODEL-003` | `Gravel_CHB_ne100k` | `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_gamma_ne100k_20260502/gravel_chb_present100000.g_file.txt` | `100000` | same-input compare, genome-wide sensitivity | `rejected for current production use` |

Variables tested after 2026-04-25

| Variable class | Concrete variable | Why it may matter | How it was tested | Main result |
| --- | --- | --- | --- | --- |
| `Ne(0)` | present-day `Ne(0)` scaling | changes gamma table magnitude and possibly raw `rSDS` | region `Ne(0)` panel, `100k` sensitivity, single-site gamma sensitivity | 对单点和 region 的 `rSDS` 数值有波动，但不足以单独解释当前坏图 |
| `Demographic model` | `historical old/default` vs `phlash q50 scaled` vs `Gravel_CHB@100k` | changes gamma shape and hence expected singleton interval likelihoods | same input 下的 gamma-track compare、genome-wide Manhattan、tail compare | `SDS-MODEL-002/003` 在当前数据上都出现 diffuse tail；`SDS-MODEL-001` 最稳 |
| `SDS input` | oldsplit / rebuilt / raw-joint / legacy | affects `t_file`, `s_file`, singleton universe and site universe | chr1 input-fix、raw-joint true-process、legacy/current compare、chr1+chr6 lineage audit | input 确实重要，但 `rebuilt` 不是唯一问题；主差异在 `legacy/unpop-like` vs current pop-specific lineage |
| `Population grouping` | `unpop` vs `NCN/SCN` split | changes cohort-specific AF and singleton definition | chr1 `NCN/SCN/merged` adjudication；raw-joint merged reconstruction | split 后语义与 old unpop 不同；不能直接把旧 `unpop` 和新 `NCN/SCN` 当同一 input lineage |
| `Source VCF semantics` | raw-joint vs split-then-merge | merge 后对侧 cohort 可能变成 `./.` 而不是 `0/0` | `MERGED3971` failure vs raw-joint true-process rebuild | split-then-merge 不能可靠恢复 merged singleton universe；raw-joint true-process 才能生成 non-empty merged `s_file` |
| `t_file` site universe | all test SNP, thinning, PASS/non-PASS | changes tested common-site set and density | sparse `t_file` shadow test；chr1/chr6 site-universe audit | 简单稀疏化不能修问题；chr6 old-only sites 全是 non-PASS |
| `s_file` singleton universe | `AC=1`, `F_MISSING`, subset-specific singleton | directly affects nearest-singleton intervals | chr1 legacy0506 failure、input-fix、raw-joint rebuild | singleton universe 是核心变量；空 `s_file` 会直接让 compute 失效 |
| `b_file` boundary` | p/q arm boundaries and centromere gaps | changes interval truncation and callable range | legacy vs current chr1/chr6 compare | 有影响，但不足以单独解释全图 diffuse tail |
| plotting / postprocess | matplotlib vs `CMplot`, filtered display | affects readability, not raw statistics | `CMplot` replace、filtered Manhattan redraw | plotting 不是主因，只影响图是否可读 |
| compute implementation | chunking, cache race | engineering variable; may corrupt output if buggy | cache-race fix and rerun | 修复后结果可重复；不是当前 biology 问题来源 |

Main control experiments and results

| Date | Controlled factor | Fixed factors | Compared factors | Key outputs | Result |
| --- | --- | --- | --- | --- | --- |
| 2026-04-26 | `Ne(0)` panel | same positive regions, same pipeline | scaled `Ne(0)` scenarios under shared panel | `scenario_manifest.tsv`, `submission_plan.json` | 建立了正式 `Ne(0)` sensitivity panel，便于后续 `100k/152k/...` 比较 |
| 2026-05-02 | gamma generation speed | same `scaled_ne0_100000` target | serial vs chunked gamma generation | `task_plan.tsv`, local chunk smoke | 证明 chunked gamma 可行，gamma generation 不再是主瓶颈 |
| 2026-05-02 ~ 2026-05-03 | demographic model | same `NCN` data and postprocess | `SDS-MODEL-001/002/003` | `NCN_gamma_tracks.summary.md`, compare plots | 只有 `SDS-MODEL-001` 给出相对 plausible 的 Manhattan；`SDS-MODEL-002/003` 都有 diffuse tail |
| 2026-05-03 | plotting filter | same `NCN` result | all points vs `p<5e-4` and `0.01<DAF<0.99` | filtered `CMplot` Manhattan | 过滤后 olddefault 仍最像 real biology；plotting 只改变可读性 |
| 2026-05-05 | `t_file` density | same chr1 q-arm source | current `t_file` vs sparse-like-legacy shadow `t_file` | `chr1_sparse_shadow_q_20260505` | 稀疏化没有实质修好 chr1，高峰和 control 都没有干净回落 |
| 2026-05-06 | `legacy_0506` input sanity | same chr1 compute code | callable `o_file` vs all-1 `o_file`; same `s/t` | `legacy0506` chr1 failure log | 问题不在 `o_file`，而在 `s_file` 全空；说明 singleton universe 才是决定性变量 |
| 2026-05-07 | main-contract input-fix | same `historical old/default` gamma | oldsplit-like current input vs rebuilt `PASS+biallelic`, subset-singleton, `o=1` | audit sidecars, `NCN.summary.tsv` | input-fix 通过计数对账，但 chr1 高峰只轻微回落，说明 rebuilt contract 不是唯一主因 |
| 2026-05-08 ~ 2026-05-10 | population grouping / merged semantics | same chr1 target windows and same gamma | `NCN`, `SCN`, merged/parent-like/raw-joint tracks | `NCN_SCN.summary.tsv`, `NCN_SCN_MERGED3971PL.summary.tsv`, raw-joint compare | `SCN` 与 merged 也有异常 control；不是单纯 `NCN` 特异峰 |
| 2026-05-09 ~ 2026-05-10 | source VCF semantics | same sample lists | split-then-merge vs raw-joint true process | raw-joint smoke and full chr1 rebuild | raw-joint merged 能生成 non-empty `s_file`；split-then-merge merged 不能 |
| 2026-05-11 | same-input model test (launch) | same rebuilt new input | `Gravel_CHB@100k` genome-wide production for `NCN/SCN` | `sds_output_gravel_chb_ne100k_newinput_20260511` | 流程打通，但图形 plausibility 很差 |
| 2026-05-12 | compute-only rerun | same input, same gamma | old direct compute vs new chunked submitter | rerun manifest, finished genome-wide outputs | 加速和收口问题解决；证明坏图不是 compute bug 造成 |
| 2026-05-14 | same input, different model | same rebuilt new input | `Gravel_CHB@100k` vs `historical old/default` | `NCN.frequency_bins.tsv`, `NCN.manhattan.png`, `CMplot` outputs | 同一 input 下 olddefault 明显更稳，说明 model/gamma 在放大当前 artifact |
| 2026-05-15 | chr1 + chr6 lineage audit | same selected chromosomes and audit code | `legacy`, `oldsplit`, `rebuilt`, same-input `olddefault vs Gravel` | `summary.md`, `input_compare.tsv`, `tail_compare.tsv` | 当前主问题最合理解释为：input lineage 差异是底层问题，`Gravel_CHB@100k` 是 amplifier |

Results by variable

## 1. `Ne(0)` 是否是主因

Current assessment

- `Ne(0)` 会影响 gamma table，也会导致 `rSDS` 数值出现波动；
- 但在我们这批实验里，**单独的 `Ne(0)` 数值并不足以解释 genome-wide Manhattan 从 plausible 变成 diffuse tail**。

Evidence

- `100k` sensitivity 和 region `Ne(0)` panel 的结果说明：
  - `rSDS` 数值会波动；
  - 某些 locus 的 sign 在阳性窗口不轻易翻转；
  - 但真正让图形全局失控的，不像是单一 `Ne(0)` 缩放本身。
- 更重要的是：你提供的 `old gamma + old input + unpop + Gravel CHB` 图形是合理的，这直接说明：
  - `Gravel CHB`
  - 以及一个二十多万量级的 `Ne(0)`
  并不会天然把 SDS 弄坏。

Conclusion

- `Ne(0)` 是影响 `rSDS` 的变量，但不是当前主因。
- 目前更合理的表述是：
  - `Ne(0)` 决定 gamma 数值层面的敏感性；
  - 但真正的坏图需要和 input lineage / site universe 的问题一起出现。

## 2. demographic model / gamma file 的影响

Current assessment

- `demographic model` 不是完全无关；
- 但它的作用更接近 **artifact amplifier**，而不是孤立 root cause。

Evidence

- 2026-05-03 的 gamma-track compare：
  - `SDS-MODEL-001 historical old/default gamma table` 的 Manhattan 最稳；
  - `SDS-MODEL-002 phlash_q50_scaled_ne100k` 和 `SDS-MODEL-003 Gravel_CHB_ne100k` 都出现 diffuse tail。
- 2026-05-14 的 same-input compare：
  - 在**同一套 rebuilt new input** 上，
  - `NCN olddefault newinput` 的图明显比 `NCN Gravel_CHB@100k newinput` 更 plausible。
- 2026-05-15 的 `chr1 + chr6` audit：
  - `chr6` 在 same-input `olddefault` 下基本安静；
  - 换成 `Gravel_CHB@100k` 后出现大量高 tail。

Conclusion

- `demographic model / gamma file` 明显会影响 `rSDS` 分布；
- 但当前最合理的因果链不是“model 自己导致一切”，而是：
  - current pop-specific input lineage 先带来底层问题；
  - `Gravel_CHB@100k` 再把这些问题全局放大。

## 3. `SDS input` 的影响

Current assessment

- `SDS input` 是当前最重要的变量之一；
- 但需要拆开看，不能只笼统说“旧 input vs 新 input”。

The input variables that mattered

- `t_file` 的 test-SNP universe
- `s_file` 的 singleton universe
- `b_file` 边界
- source VCF 是 raw-joint 还是 split 后视图
- split 后是 `unpop` 还是 pop-specific (`NCN/SCN`)

Evidence

### 3.1 rebuilt main-contract input 不是唯一问题

- 2026-05-07 的 input-fix 把 current pipeline 收紧到：
  - `t_file = PASS + biallelic SNP`
  - `s_file = PASS + biallelic SNP + AC=1 + F_MISSING<=0.005`
  - `o_file = 1`
- 这套 input 在工程上更规范，也通过了计数对账；
- 但 chr1 高峰只轻微回落，没有根治问题。

Interpretation

- rebuilt main-contract input 不是“错得离谱”；
- 但它没有恢复出 old `unpop` 轨道那套更 plausible 的 site universe。

### 3.2 sparse `t_file` 不是解法

- 2026-05-05 做了 chr1 sparse `t_file` shadow；
- 如果问题只是 test SNP 太密，稀疏化应该会明显变好；
- 实际上没有。

Interpretation

- 问题不是“把全部 test SNP 放进去”这一原则本身；
- 更可能是**什么位点被定义成 test SNP**、以及它们所在的 site universe 变了。

### 3.3 `s_file` / singleton universe 是真正核心

- `legacy_0506` 的 chr1 compute 失败时，`o_file` 改成全 `1` 也无济于事；
- 根因是 `s_file` 全空。
- merged `MERGED3971` 也是同理：只要 merged 视图长不出 merged singleton universe，compute 就会直接失效。

Interpretation

- SDS 最终依赖的是 singleton interval；
- 所以 `s_file` 的语义比 `t_file` 密度更关键。

## 4. `population split strategy` 与 raw-joint / split-pop 语义

Current assessment

- `unpop` vs `NCN/SCN` 不是简单“样本少了或多了”的问题；
- 它改变了 AF、singleton、以及 merged cohort 的恢复方式。

Evidence

- 2026-05-08 ~ 2026-05-10 的 chr1 adjudication：
  - `NCN` 有高峰；
  - `SCN` 没有对应主峰，但 `control35` 也不安静；
  - `MERGED3971PL` 只是补救，不是真 merged baseline。
- raw-joint true-process rebuild 证明：
  - merged cohort 完全可以直接从 raw-joint source 生成 non-empty `s_file`；
  - split-then-merge 不行。

Interpretation

- split 后的 pop-specific VCF 不是不能用；
- 但它不等价于 old `unpop` input lineage；
- 如果要拿旧 `unpop` 图做直接 baseline，就必须承认这两条线的 source VCF / site universe 已经不同。

## 5. `b_file` 边界与 site universe 差异

Current assessment

- `b_file` 有影响；
- 但从目前证据看，它不是当前问题的唯一主因。

Evidence

- `legacy` 与 `current/rebuilt` 的 chr1 `b_file` 明显不同；
- 但 2026-05-15 的 audit 说明，真正更大的差异还是：
  - `legacy/unpop-like chr1_t_file = 223,505`
  - `current/rebuilt NCN chr1_t_file = 3,278,034`
- `chr6` 上，oldsplit 比 rebuilt 多出的 `862` 个位点全部是 non-PASS，主要是 `LowQD` / `LowAAScore`。

Interpretation

- 这说明：
  - 边界是因素之一；
  - 但更大的底层问题是 site universe 变了，而且带入了旧链没有的位点类别。

Additional variables worth noting

| Variable | Scientific meaning | Tested? | Result |
| --- | --- | --- | --- |
| `o_file` weights | sample-level observability | only sanity-tested, not fully optimized | 当前没有证据显示 `o=1` 是主因 |
| plotting engine | `matplotlib` vs `CMplot` | yes | 只影响可读性，不改变统计结果 |
| compute implementation | direct vs chunked submitter | yes | 只影响速度和稳定性；cache race 修复后不再是 scientific confounder |
| merged reconstruction method | direct merged vs parent-like vs raw-joint | yes | raw-joint true-process 最可信；parent-like 只适合诊断，不是 production baseline |

Current overall conclusion

1. 4.25 之后我们已经把最主要的技术变量基本拆开测试了一遍。
2. 目前可确认会影响 `rSDS` / `norm SDS` 结果的变量包括：
   - `present-day Ne(0)`
   - `demographic model / gamma file`
   - `t_file` / `s_file` / `b_file` 这类 `SDS input`
   - `population grouping` 与 raw-joint / split-pop 语义
   - `site universe / input lineage`
3. 当前最合理的技术判断不是“某一个变量单独造成坏图”，而是：
   - `legacy/unpop-like` 与 `current pop-specific` 的 input lineage 差异是底层问题；
   - `Gravel_CHB@100k` 在当前 pop-specific input 上会把这些问题放大成 diffuse tail；
   - `Ne(0)` 会影响数值，但不是当前唯一主因。
4. 因此，若下一步要恢复一条可信的 `Gravel CHB` 轨道，优先级应当是：
   - 先恢复或重建更接近 old `unpop` 的全基因组 input lineage；
   - 再在 **同一套 input** 上比较 gamma / model；
   - 而不是继续只改 `Ne(0)`。

Files to cite in teacher report

- gamma / model compare:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_diagnostic_comparison/NCN_gamma_tracks.summary.md`
- same-input `Gravel` production:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/NCN/NCN.manhattan.png`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN/SCN.manhattan.png`
- same-input olddefault baseline:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.manhattan.png`
- input-lineage audit:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/summary.md`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515/chr6_oldsplit_only_filter_breakdown.tsv`

Appendix: housekeeping done on 2026-05-17

- 已在核对 `SDSlog` 后删除可无害删除的中间件、重复 manifest、备份图与 completed-run cache。
- 近似释放空间：`~123.7G`
- 详细删除清单不再作为本报告主体；如需追溯，以 git history 与当天 cleanup commit/log 为准。
