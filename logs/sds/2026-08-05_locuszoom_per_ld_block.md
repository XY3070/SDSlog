# Per-LD-Block LocusZoom Plots (15 Independent Signals)

Date: 2026-08-05
Status: **COMPLETE**
Role: mainline — visualization refinement of the 15-signal connected-components LD split
Parent entries: 2026-07-28_orthogonal_evidence_final_report.md, 2026-07-26_ld_split_functional_annotation_pipeline.md, 2026-07-22_chr19_locuszoom_ld_structure.md

## Motivation

Existing locuszoom plots were per-peak (10 non-MHC peaks, 7 PNGs in `plots/`). After the connected-components LD split (r²>0.1) defined 15 LD-independent signals, we needed one plot per LD block. Pain point: the gene track drew **one row per gene** with fixed panel height (~2.45 in), so gene-dense chr19 windows (~30-40 genes in a 400 kb window) collapsed and labels overlapped (e.g. chr19 DNM2 window: 30 rows ≈ 0.08 in/row).

## Code changes

`SDSpipeline/scripts/postprocess/plot_locuszoom.py`, commit `7d26659`:
- **Lane packing**: new `compute_gene_lanes()` — greedy interval-graph first-fit (gene starts a new lane only if it overlaps the previous gene in every existing lane; lane count == max overlap depth). `plot_gene_track()` now draws one row per lane, labels centered in-lane, fontsize 6.5 (≤4 lanes) / 5.5, 5.0 for genes <8 kb.
- **Dynamic panel height**: gene panel ratio `max(1.5, n_lanes*0.5)`; figsize scaled via `9*(4+gene_ratio)/5.5` — identical to old layout at gene_ratio=1.5, so legacy runs unaffected.
- **`--lead-label`**: optional title annotation (e.g. `ADH1B (missense)`).

Measured lane counts (vs rows before): chr12 ALDH2 window **78 → 3 lanes**, chr19 DNM2 **30 → 2**, chr22 ACO2 **24 → 2**, chr11 **4 → 1**, chr4 **7 → 1**. All chr19 4885kb-window signals: 2 lanes each.

New driver: `SDSworkspace/oneoff/batch_locuszoom_all_signals.py` — reads `signals/all_signals_catalog.tsv`, per signal calls plot_locuszoom.py with the LD block's own start/end + lead SNP; **adaptive flank** `min(500, max(50, block_span_kb//2))` (window ≈ 2-3× block; 14 kb ARRDC5 blocks → ~120 kb window vs 515 kb before; ALDH2 1.34 Mb block → ~2.3 Mb window); writes `manifest.tsv`; resume-safe (skip if PNG exists).

## Run

- Inputs: `all.significant_regions.tsv` (10 non-MHC peaks) → `signals/all_signals_catalog.tsv` (15 signals); `all.normalized.tsv`; per-chrom QC VCFs `raw/9k/snp_singleton_qc_chr{chr}_v1/snp_qc_pass.vcf.gz` (4383 samples); pyrho CHB maps; Ensembl REST gene cache (`plots/locuszoom_cache/`, reused).
- Env: conda `sds`; plink2 `--r2-unphased` LD vs each block's lead SNP.
- Output: `plots/locuszoom_per_signal/{signal_id}.locuszoom.png` (150 dpi) + `manifest.tsv`; old per-peak plots untouched.
- Runtime: 15 signals ≈ 40 min serial (plink2 full-chrom VCF scan dominates).
- Note: chr4 ADH1B first attempt hit Ensembl REST 500 ×3 retries → no gene panel; rerun after API recovery OK. Driver now flags `API error`/`All retries exhausted` as WARN.

## Results

| signal_id | gene | block span (kb) | flank (kb) | lanes | status |
|---|---|---|---|---|---|
| chr4_99240kb_sig1 | ADH1B | 262 | 131 | 1 (7 genes) | OK |
| chr11_48235kb_sig1 | intergenic | 99 | 50 | 1 (4) | OK |
| chr12_109929kb_sig1 | TRPV4 | 264 | 132 | 2 (11) | OK |
| chr12_110306kb_sig1 | ATP2A2 | 37 | 50 | 1 (2) | OK |
| chr12_111181kb_sig1 | ALDH2 | 1341 | 500 | 3 (52) | OK |
| chr12_112355kb_sig1 | NAA25 | 748 | 374 | 3 (29) | OK |
| chr12_112949kb_sig1 | OAS3 | 142 | 71 | 1 (5) | OK |
| chr19_4885kb_sig1 | DPP9 | 36 | 50 | 2 (3) | OK |
| chr19_4885kb_sig2 | PLIN3 | 146 | 73 | 2 (9) | OK |
| chr19_4885kb_sig3 | MIR7-3 | 19 | 50 | 2 (5) | OK |
| chr19_4885kb_sig4 | ARRDC5 | 14 | 50 | 2 (4) | OK |
| chr19_4885kb_sig5 | ARRDC5 | 14 | 50 | 2 (5) | OK |
| chr19_4885kb_sig6 | UHRF1 | 163 | 81 | 2 (5) | OK |
| chr19_10690kb_sig1 | DNM2 | 720 | 360 | 3 (42) | OK |
| chr22_41498kb_sig1 | ACO2 | 146 | 73 | 2 (24) | OK |

All 15/15 rc=0; no LD failures (0 warnings); lead labels: `ADH1B (missense)`, `ALDH2 (missense)`, others `<gene> (consequence_class)`.

## Verification

- Lane counts confirmed from plot stdout (`[plot] Gene track: N lanes, M genes`) and by recomputing `compute_gene_lanes` on cached Ensembl JSON.
- Regression: chr19_10690kb_sig1 vs old per-peak chr19_10690kb.locuszoom.png — same variants/LD/recomb; gene panel now 3 lanes instead of 30 rows.
- 15 PNGs + manifest in `plots/locuszoom_per_signal/`; old `plots/` files untouched.

## Round 2 (same day): label placement + x-range mismatch fixes

User review of round 1 plots flagged: (1) gene name labels drawn ON the gene bodies obscured them; (2) gene panel x-range did not match the variant panel on chr12_111181kb_sig1, chr12_112949kb_sig1, chr19_4885kb_sig3.

**Root cause of (2):** Ensembl REST `overlap/region` returns genes *overlapping* the window, some spanning its edges (e.g. RPH3A 112,570,380-112,898,881 extends 299kb left of the OAS3 window; DPP9/UHRF1/PLIN3 cross chr19 block boundaries). Round 1 drew full gene bodies + midpoint labels → bodies/labels outside the panel → visible mismatch.

**Fixes** (commit `7a50a24`):
- Gene bodies, exons, and strand chevrons **clipped to the plot window** (`draw_start/draw_end`); genes with zero visible span skipped.
- Labels placed over the **visible portion** (pushed inward at window edges, `ha` switches left/right); vertical placement **above the lane** (one side, sparse tracks ≤2 lanes) or **alternating above/below lanes** (both sides, ≥3 lanes).
- Dense tracks get taller gene panel: `gene_ratio = max(1.5, n_lanes*0.7)` for n_lanes≥3.
- Permanent debug prints: `[main] Variants extent` / `[main] Genes extent (unclipped)` per run.

**Verification (in-memory, all 15):** 11/15 signals have edge-overlapping genes now clipped; **0 label collisions** across all lanes (label bbox math in data coords). 3-lane signals (ALDH2, NAA25, DNM2) render taller gene panels (1213px vs 1100px). All 15 re-plotted: OK=15 WARN=0 FAILED=0, `run_20260805_v2.log`, manifest regenerated.

## Next steps

- Visual review of the 6 chr19 plots (gene-dense region — the original pain point).
- Any signal can be re-plotted with `--dpi 300` or custom `--flank-kb` for paper figures.
