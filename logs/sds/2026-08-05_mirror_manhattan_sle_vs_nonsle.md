# 2026-08-05 Mirror Manhattan redraw (SLE top / non-SLE bottom) with peak annotation

Date: 2026-08-05
Status: **COMPLETE**
Role: `mainline` — user-requested redraw of the SLE vs non-SLE mirror Manhattan
Parent entries: 2026-08-02_sle_vs_nonsle_sds_comparison.md (+synthesis, +confounder diagnostic), 2026-08-05_oas_sle_vs_nonsle_check.md

## Motivation

User request: 旧 mirror Manhattan 全错（matplotlib、无 peak 注释），重画 — 上半 SLE、下半 non-SLE，**黑色注释 shared peaks、highlight 注释 different peaks**，用与普通 Manhattan 相同的标准（CMplot, Rule 9）。

## Code

**New canonical script** `SDSpipeline/scripts/postprocess/report_cmplot_mirror_manhattan.R` (committed):
- 与 `report_cmplot_manhattan.R` 同一渲染标准: 相同 filter (is_common_variant==1, DAF/MAF>0.01, p<1)、col c("#4E79A7","#9C755F")、cex 0.28、points.alpha 80、band 0.6、threshold lty 2、amplify FALSE、chr.labels chr1-22。
- CMplot `plot.type="m"` 双 P 列 (SLE, non-SLE) → 两个 stacked panel (SLE 上, nonSLE 下), 共享 x 轴 + ylim。
- 各自 cohort 的 Bonferroni 阈值 (SLE 1.101848726e-08, nonSLE 1.090037774e-08) 作为 threshold 向量。
- **Per-trait highlight (CMplot 4.5.1 支持 list):** shared/MHC = 黑色点, SLE_only = 红 (#E64B35), nonSLE_only = 蓝 (#0072B5); 文字全黑。
- **CMplot 4.5.1 quirk:** m-mode 下 `highlight.text.col` 传 flat 参数（非 per-panel `[[i]]`）→ 文字颜色只能标量 "black"；点色 per-panel 正常。注释在源码中。
- 峰值标注需 SNP 在 join 后数据中存在，否则 CMplot match 产生 NA → length mismatch error（已 trim）。

**Prep script** `SDSworkspace/oneoff/prep_mirror_peaks.py` → `mirror_peaks.tsv`:
- 输入: SLE/nonSLE `significant_regions.tsv`；shared = 区间重叠 (100kb slack)；MHC (chr6 25.0-32.5Mb) 特判为单一 "MHC" 标签（最强 peak chr6:29528038）；基因名: ALL 15-signal catalog 区间优先（含 lead 的 block 优先），未覆盖走 Ensembl REST (±5kb → ±200kb nearest, 缓存于 locuszoom_cache/)。

## Peak annotation (19 行)

| 类型 | 数量 | 标签 |
|---|---|---|
| shared (黑) | 8 | ADH1B, TRPV4, ALDH2, OAS3, intergenic (chr18:30.3Mb), UHRF1, DNM2, MHC |
| SLE-only (红) | 2 | PIP5K1B (near, chr9:68.66), DNM2 (chr19:10.81, SLE 特有 sig SNV) |
| nonSLE-only (蓝) | 9 | PEX11B (chr1:145.9), CARTPT (chr5:71.7), intergenic (chr11:48.2), TRPV4 (chr12:110.03), ATP2A2, PACS2 (chr14:105.4), CHST9 (chr18:27.06), L3MBTL2 (chr22:41.2), ACO2 (chr22:41.5) |

注: 非 MHC 区 SLE 9 峰 vs nonSLE 16 峰 — nonSLE 显著峰更多（n=2367 > 1965 及 singleton burden 差异, 见 Aug-2）。非 MHC shared 7 个。

## Run

```
Rscript SDSpipeline/scripts/postprocess/report_cmplot_mirror_manhattan.R \
  --input-sle SLE/sle.normalized.tsv --input-nonsle non-SLE/non-sle.normalized.tsv \
  --peaks-tsv .../mirror_peaks.tsv \
  --bonferroni-sle 1.101848726e-08 --bonferroni-nonsle 1.090037774e-08 \
  --output-prefix .../mirror_manhattan_SLE_vs_nonSLE_CMplot \
  --width 16 --height 9 --dpi 400
```
- Variants: SLE 4,537,913 → 4,537,828 filtered; nonSLE 4,587,055 → 4,586,997; **joined 4,475,520 shared SNPs** (SLE-only 62,308, nonSLE-only 111,477 dropped — inner join; cohort-unique variants 不画).
- Highlights: SLE panel 10, nonSLE panel 17。
- 输出 6400×3600 px (16×9in @400dpi)。

## Cleanup (Rule 8 流程: 先更新引用再删)

3 张旧图 deleted (记录 superseded 于 3 个 Aug-2 log 的 Output 区):
- `mirror_manhattan_SLE_vs_nonSLE.png`, `delta_manhattan_SLE_minus_nonSLE.png`, `three_panel_comparison.png`

## Outputs

- 图: `sle_vs_nonsle_comparison/mirror_manhattan_SLE_vs_nonSLE_CMplot.png` → 归档 `SDSlog/plots/sds/2026-08-05_mirror_manhattan/`
- 表: `sle_vs_nonsle_comparison/mirror_peaks.tsv` (19 peaks, lead SNP, label, type)
- 脚本: `SDSpipeline/scripts/postprocess/report_cmplot_mirror_manhattan.R` (canonical, committed) + `SDSworkspace/oneoff/prep_mirror_peaks.py`

## Round 2 (same day): panel fix + all-variant plot + interval highlight

User feedback: (1) 图只有上半没有下半; (2) highlight 点全用金色、且整基因都 highlight 不是单点; (3) **所有算过 SDS 的位点都要画，不只共享位点**。

**Root cause of (1):** CMplot 4.5.1 `multracks=FALSE`（默认）→ m-mode 走 Rect per-trait 循环、**不设 mfcol** → 两个 trait 画在同一绘图区互相覆盖 → 只剩最后画的 panel。修: `multracks=TRUE` + 自己开 `png(bg="white")` + `par(mfcol=c(R,1))` + 总高 height×R（multracks 的 !file.output 分支不设 mfcol，依赖现有 device 布局；file.output=TRUE 路径 bg=NA 透明背景不可用）。

**(3) 全位点:** full outer join on SNP（SLE-only 62,308 + nonSLE-only 111,477 也画），cohort-unique 变异在缺失 panel 中 p=1（底部不可见）。总变异 4,649,305（共享 4,475,520）。

**(2) interval highlight:** `mirror_peaks.tsv` 加 start/end — union(sig regions)，<100kb 的 peak 以 lead 为中心扩到 ±50kb；MHC = 全跨度 25.44-31.87Mb。R 脚本 highlight **区间内所有变异**（SLE panel 19,759 / nonSLE panel 20,986 个点），gene 名 text 只在 lead SNP 处。颜色: **different peaks = 金 (#E6A817)，shared/MHC = 深绿 (#009E73)**（与金区分）；文字黑（CMplot 4.5.1 flat text.col quirk 不变）。highlight.cex 0.8 (multracks ×R 后 1.6)。

**验证:** 像素 band 分析 — band 3 (SLE 点群 29%) + band 7 (nonSLE 点群 26%) 双 panel 都在；金 169k + 绿 607k 像素存在。6400×6400 (16×16in @400dpi)。

## Round 3 (same day): true mirror + ALL signal labels

User: "是做不到mirror这个事吗？上半peak往上，下半对称peak往下？" + "把ALL队列的signal也label一下"。

**真 mirror 做到了** (`report_cmplot_mirror_manhattan.R` 加 `--mirror` 默认 TRUE):
- 关键: CMplot multracks 支持 **per-panel ylim list** (`Max/Min <- ylim[[i]]`, `ylim = ylim[[i]]`)。
- 实现: panel 2 列 = `log10(p)`（负的 -log10p），`LOG10=FALSE`，`ylim = list(c(0,ymax), c(-ymax,0))`，`threshold = c(-log10(thr_sle), log10(thr_ns))`（= ±7.96）。
- 验证: 像素 band — band 3 (29.4%) = SLE 点群在下面板**底**（peak 向上），band 4 (29.6%) = nonSLE 点群在上面板**顶**（peak 向下，对称镜像）✓。6400×6400@400dpi。
- `--mirror FALSE` 恢复旧模式（两 panel 都向上）。

**ALL Manhattan signal label** (`report_cmplot_manhattan.R` 加可选 `--peaks-tsv`):
- 单 trait 版也支持 interval highlight + gene 标签（与 mirror 同一视觉语言: 绿点 #009E73 全区间, 黑字标签只在 lead）。
- `all_peaks.tsv`: 15 signals（catalog 块 start/end, lead, gene 名; chr11 空 gene → intergenic）。
- 输出 `ALL/plots/all_manhattan_labeled_CMplot.png` — 4,545,801 变异, **15 peaks, 4,405 highlighted variants**, 6400×2800@400dpi, Bonferroni 1.099916164e-08。

**验证:** 绿信号像素 174k 存在; highlight 日志行确认 15 peaks 全部命中。

## Round 4 (same day): shared x-axis, no middle gap + 流程化注册

User: mirror 应共享同一根 x-axis，中间不应有 gap；修好后固定成流程化脚本进 `SDSpipeline/scripts/`。

**Gap 根因:** CMplot 4.5.1 multracks 每 panel 的 `plot(..., yaxs="r")` 硬编码 — yaxs="r" 在坐标范围两端各加 4% padding → y=0 的点永远到不了 panel 边缘 → 两 panel 之间出现白带（实测 176px = 2.75%）。**不是 par 能解决的**: yaxs 经 plot.default 的 `...` 传递（trace plot.default 也改不到，因为 `...` 只读）。padding 实际发生在 `plot.window` 计算 usr 时。

**Fix:** 临时 patch graphics namespace 的 `plot.window`（unlockBinding → 包一层强制 `yaxs="i"` → 恢复），仅 mirror 模式、仅本次绘图期间。验证: 边界行 (3150-3250) 密度 81-83%（原来 <5%），白带 0。x-axis 仍只在下面板（`i == R || multracks.xaxis` 条件，multracks.xaxis=FALSE）。

**流程化:** `manifests/canonical_scripts.tsv` 注册 `scripts/postprocess/report_cmplot_mirror_manhattan.R`（postprocess, active）。

## Round 5 (same day): per-panel chromosome colors (grays/blues)

User: 中间去掉 x-axis 后难以区分染色体边界 → 用颜色区分: **上片相邻染色体深灰/浅灰间隔，下片深蓝/浅蓝间隔，peak 颜色不变**。

**实现:** CMplot multracks 的 `col` 是 R×染色体矩阵（`colx = col[i, ]`，每 trait 一行，列循环染色体）→ `col = rbind(SLE_gray_pair, nonSLE_blue_pair)`。加 `--sle-colors` / `--nonsle-colors` 参数（逗号分隔，默认 `#6E6E6E,#D9D9D9` / `#1F4E79,#B4C7E7`）。highlight 不变（shared/MHC 绿 #009E73, different 金 #E6A817）。

**验证 (像素):** SLE panel 深灰 415k + 浅灰 407k px; nonSLE panel 深蓝 357k + 浅蓝 356k px; green 631k + gold 167k px 保持; 边界 min density 81.4%（无 gap）。6400×6400@400dpi。

## Next steps

- 肉眼确认: MHC 单标签是否够；chr19 DNM2 SLE-only 红点 (10.81Mb) 与 shared DNM2 黑标 (10.24Mb) 并列是否清楚；shared 黑字与红/蓝点重叠 (ALDH2 大块内)。
- 任何 peak 可重画: `--height`, `--dpi 300` 或调整 `mirror_peaks.tsv`。
