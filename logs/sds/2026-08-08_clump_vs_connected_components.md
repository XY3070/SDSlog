# Clumping vs Connected Components: Signal Decomposition Comparison

Date: 2026-08-08
Status: COMPLETE — chaining problem confirmed, clumping method validated
Role: mainline — method comparison for signal identification (LD decomposition)
Input lineage: ALL joint-call v2, phased ALL (4383), normalized ALL
Demographic model: SDS-MODEL-001 (unchanged); Gamma: unchanged
Sample policy: ALL (NCN 1255 + SCN 2716 = 4383); LD panel = ALL phased VCF (chr4: 9k QC VCF fallback — ALL chr4 phasing pending)

## 1. Motivation

Old signal decomposition (`ld_split_and_annotate.py`) used **connected components** on the r²≥0.1 graph (per significant region ±50kb flank). Risk: **chaining** — A-B r²≥0.1, B-C r²≥0.1 with A-C independent get merged into one component. Replaced with **PLINK --clump-style greedy clumping**: most significant unclumped SNP = lead; absorb unclumped SNPs within a distance window (bp or cM) with r² ≥ threshold vs the lead; next lead; repeat.

## 2. Implementation

- Script: `SDSpipeline/scripts/postprocess/clump_signals.py` (imports cc logic from ld_split_and_annotate.py — single source of truth)
- LD: phased ALL VCFs (4383 samples), plink2 `--r2-unphased triangle`, canonical pairs
- Window: bp (100/250/500 kb) + cM (0.1/0.5 cM, pyrho CHB maps interpolated)
- Sweep: r² ∈ {0.05, 0.1, 0.2, 0.5} × 5 windows = 20 configs; primary = r² 0.1 + 250 kb
- Signal rule: ≥1 Bonferroni-significant SNP (strict); parity flag keeps old OR-rule (≥2 SNPs & |SDS|>3)
- Invariant (checked): clump ⊆ one cc component → clump partition refines cc; only identical/split possible
- Outputs: `signals/clump_comparison/` (all_clump_signals.tsv, cc_reference_signals.tsv, clump_vs_cc.tsv, chaining_audit.tsv, clump_vs_production.tsv, config_summary.tsv)

## 3. Validation

- Refinement invariant: 0 merge/new/absent across all 20 configs ✓
- Monotonicity: n_signals non-decreasing in r², non-increasing in window ✓
- Determinism: two independent runs → byte-identical outputs ✓
- r² spot-check vs plink2 --ld-snp: consistent ✓
- Analysis SNPs: 5,690 in group intervals (chr4=318 9k, chr11=285, chr12=2557, chr19=2384, chr22=146)

## 4. Chaining confirmed (user hypothesis verified)

| Evidence | Value |
|---|---|
| cc component chr11 | 285 SNPs chain-connected; clump = 190 (95 have NO direct r²≥0.1 vs lead) |
| cc component chr12 | ALL 5 regions chain into ONE component (2551 SNPs, span 3.25 Mb) |
| Split lead pairs audited | 470 pairs in 8 split components |
| — `chaining` class | **400 (85%)**: r²(leads) < 0.1, median 0.0068, max 0.099 — merged by cc purely through intermediates |
| — `distance` class | 70 (15%): direct LD but leads >250 kb apart (window-boundary split) |
| — `bug` | 0 |
| Notable distance case | ALDH2 (111803962) vs NAA25 (112074109) leads: r²=0.99 at 270 kb = 0.0064 cM (recombination cold spot) — bp window splits, cM window merges |

## 5. Signal comparison (primary config: r² 0.1, 250 kb, ALL LD, parity rule)

Production catalog (9k LD, connected components): **15 signals**. Clumping: **69** (parity) / **25 strict** (with ≥1 Bonferroni SNP).

- **14/15 production lead SNPs preserved exactly** (lead_exact). Only exception: chr12 NAA25 lead (112030402:C:T) absorbed into ALDH2 clump (111803962:G:A; r²≥0.1, 226 kb apart; ALL-LD effect).
- **Signal sizes shrink** — production n_snps inflated by chaining:
  | Region | production n_snps | clump n_snps |
  |---|---|---|
  | chr4 ADH1B | 318 | 231 |
  | chr11 | 285 | 190 |
  | chr12 ALDH2 (111803962) | 913 | 309 |
  | chr19 DNM2 (10765735) | 820 | 41 |
  | chr22 ACO2 | 144 | 48 |
- **chr12**: 1 chained mega-component → **11 strict signals** (TRPV4, ATP2A2, 5 independent LD blocks in ALDH2 region incl. 110976657 with 56 sig SNPs, 3 blocks around NAA25/OAS3, 110672222 block)
- **chr19 4885 kb**: 6 → **9 strict signals** (new: 4838039 p=4.6e-17, 4917007, 5023601 171-SNP block)
- **chr19 10690 kb (DNM2)**: 1 → **4 strict signals** (10378293, 10515885, 10555015, 10765735)
- **chr11 / chr22 / chr4**: same count, same leads
- Production minor signals chr19 sig4 (4903524, p=1.0e-6) and sig5 (4889675, p=6.3e-5) remain non-Bonferroni clumps (minor)

## 6. Sensitivity (20 configs, vs cc on same matrix)

- strict signals: 17–50; parity: 46–147 across configs
- r² 0.5 → finest partition (window-boundary splits dominate; 67 cc comps at r² 0.5)
- cM windows merge cold-spot pairs bp windows split (e.g. 0.0064 cM pair above)
- All configs: refinement invariant 0, monotonicity holds

## 7. Caveats

- **chr4 LD = 9k VCF fallback** (ALL phasing still running, job 504164) — ADH1B clump on 9k LD, `ld_source=9k` column. Re-run when ALL chr4 phased.
- clump_vs_production comparison confounded by LD source (9k vs ALL) — descriptive only.
- NAA25 lead change is LD-source-driven, not algorithm-driven (cc@ALL also merges ALDH2+NAA25 region).

## 8. Recommendation

- Adopt greedy clumping (r² 0.1, 250 kb) as the signal-decomposition method — removes chaining inflation, yields per-LD-block signals
- Run with strict Bonferroni rule for paper catalog; parity rule only for comparisons
- Re-run chr4 on ALL LD when phasing completes
- Follow-up if adopted: re-run VEP/eQTL annotation on new leads (25 strict), update catalog

## Files

- Script: `SDSpipeline/scripts/postprocess/clump_signals.py` (+ registry entry in `manifests/canonical_scripts.tsv`)
- Outputs: `SDSworkspace/results/production/sds_output/ALL/signals/clump_comparison/*.tsv`
- Inputs: `all.significant_regions.tsv`, `all.normalized.tsv` (ALL), phased ALL VCFs, pyrho CHB maps
