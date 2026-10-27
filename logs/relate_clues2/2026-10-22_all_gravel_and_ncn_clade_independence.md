# 2026-10-22 — ALL cohort Gravel EAS pipeline + NCN genealogical independence test

## 0. Context / decisions

- **aDNA route (HRA000123, Fu 2020) rejected for these 7 chr19 leads.**
  - HRA000123 = 24 BAMs (HRR051935–HRR051958), ~0.20× WGS; processed Eigenstrat files 404 on the current mirror.
  - Expected derived reads per lead at 24 × 0.20×: 0.16–0.40 for the five 3–8% signals; ~4.6 only for ARRDC5(−) (f=0.955).
  - To see ≥10 derived reads at a 5% allele: ~989 individuals at 0.2× or ~100 at 2×. HRA000123 is ~2 orders of magnitude short.
  - CLUES2 does support ancient time series natively (`--ancientSamps` / `--ancientHaps`; `inference.py:115-132`) — the blocker is data power, not software.
- **Modern-only CLUES2 s(t) is prior-dominated.** With a single modern time point, more `--timeBins` adds no information; the epoch MLEs are the only defensible s(t) output.
- **ALL cohort chr19 Relate ARG already exists** and can be reused; Gravel EAS is applied at the branch-length sampling stage.
- **Whole chr19 for ARG inference; per-signal windows for downstream.** Splitting the ARG into R1/R2+flanking would create boundary artefacts, lose long-range trees, and cannot be merged.

## 1. ALL cohort inputs / existing ARG

| item | path | note |
|---|---|---|
| ALL `.anc` | `SDSworkspace/results/production/orthogonal/relate_clues2/ALL/run_work_chr19_chb_pyrho/UKBQC_ALL_chr19.anc` | 18 GB, 4,383 samples (8,766 haplotypes) |
| ALL `.mut` | same dir `UKBQC_ALL_chr19.mut` | 123 MB |
| ALL haps | `ALL/prepared/UKBQC_ALL_chr19.haps.gz` | 286 MB |
| ALL sample | `ALL/prepared/UKBQC_ALL_chr19.sample.gz` | 4,383 samples |
| map | `refs/maps/chr19_CHB_pyrho.relate.map` | pyrho CHB |
| coal used by ARG | `refs/coal/NCN.coal` (symlink `ALL.constant.coal`) | **not** Gravel EAS |
| ARG job | 505938 | 32 threads, peak 2.1 TB, 6.11 M CPU-s, 3 days wall |

**Key point:** `SampleBranchLengths` (`RelateCoalescentRate --mode SampleBranchLengths`) resamples branch lengths under `--coal`; therefore Gravel EAS can be applied to the existing ARG topology **without re-running Relate**. Only if Gravel-consistent `.anc` branch lengths / `.mut` ages / RelateSelection are required should one run `RelateCoalescentRate --mode ReEstimateBranchLengths` (cheaper than a full re-run).

## 2. ALL Gravel-EAS pipeline (launched)

Directory: `SDSworkspace/results/production/orthogonal/relate_clues2/ALL_gravel/`
(`clues/`, `intermediate/`, `logs/`, `clade_independence/`)

Scripts:
- `SDSpipeline/scripts/orthogonal/prepare_all_gravel_subregions.sh`
- `SDSpipeline/scripts/orthogonal/run_clues2_all_gravel_signal.sh`
- `SDSpipeline/scripts/orthogonal/submit_all_gravel_chr19.sh`

Flow per signal (Gravel EAS coal, 200 samples, `tCutoff=600`, `timeBins 50 200`):
1. `RelateExtract --mode AncMutForSubregion` on `chr19_4650000_5000000` (6 signals) or `chr19_10700000_10820000` (DNM2), focal `POS-1..POS+1`.
2. `RelateCoalescentRate --mode SampleBranchLengths --coal CLUES2/Gravel_EAS.coal --num_samples 200 --format n`.
3. `CLUES2/prepare_clues_inputs.py` → `_derived.txt`, `_popfreq.txt`, `_times.txt`.
4. `CLUES2/RelateToCLUES.py`.
5. `CLUES2/inference.py --coal Gravel_EAS.coal`.

Jobs:
- prepare: **567184** (`all_grav_prep`, 8 threads, 64 GB, 24 h) — DONE in 42 s; extracted the two subregions from the 18 GB `.anc`.
- prepare resubmit: **567185** — DONE (subregions existed, skipped).
- 7 signal jobs: **567186–567192** (MIR7-3, UHRF1, PLIN3, DPP9, ARRDC5−, ARRDC5+, DNM2), submitted 16:40, all RUN on `smp` (4 threads, 64 GB, 12 h). Expected ~3.5–4 h each.

**Tooling fix:** `CLUES2/RelateToCLUES.py` is O(nodes²) (per-leaf `tree.depths()`, per-node `get_terminals()`) and effectively unusable at 8,766 haplotypes. Added `CLUES2/RelateToCLUES_fast.py`, an O(nodes) reimplementation; validated **line-by-line identical** to the production NCN `_times.txt` (400/400 lines, 0 mismatches, rtol 1e-6). The ALL signal script uses the fast version.

**End-to-end ALL validation:** MIR7-3 with `NSAMP=2` completed in 5.7 min (sampling 121 s → prepare_clues_inputs → fast RelateToCLUES → CLUES2 inference). ALL popFreq at chr19:4,768,268 = 0.0675 (vs NCN 0.0402).

**Caveat:** ALL = SLE (1,965) + non-SLE (2,367). CLUES2 assumes a single population; disease/region structure can confound. Use NCN (1,255) as the clean reference and ALL as the larger-power replication; split SLE/non-SLE if ALL diverges.

## 3. NCN genealogical independence test (modern-only)

Directory: `NCN_gravel/clade_independence/`
Report: `report.md`; figures `clade_independence_summary.png`, `cross_clade_preservation.png`.

### 3.1 Method

- 7 chr19 leads; carrier sets from 2,510 phased NCN haplotypes (`_derived.txt`).
- **Carrier-clade age**: `SampleBranchLengths --coal Gravel_EAS.coal`, 200 samples; age = max coalescence time of derived lineages (95% CI). This is the Gravel-EAS resampling, not the NCN.coal `.mut` age.
- **Carrier co-occurrence**: 2×2 table across 2,510 haplotypes vs expectation; Fisher exact, Bonferroni 0.05/21 = 0.00238.
- **Cross-position clade preservation**: in the local tree at A, compute the MRCA age of B's carriers; ratio to B's own clade age. ratio ~1 = shared haplotype; ratio ≫1 = clade not preserved.
- **Star-likeness**: mean pairwise coalescence / clade TMRCA (1.0 = hard sweep).
- Local trees: `cluster_full.newick`/`.pos` extracted from the full NCN `.anc` for chr19:4.70–4.96 Mb (155 trees); branch-length scale of these trees is ~31× the `SampleBranchLengths` generation scale, but preservation ratios are scale-free.

### 3.2 Carrier-clade ages (Gravel EAS)

| signal | f | age (gen) | 95% CI | star |
|---|---|---|---|---|
| MIR7-3 | 0.040 | 396 | 275–543 | 0.66 |
| UHRF1 | 0.058 | 511 | 427–608 | 0.66 |
| DNM2 | 0.032 | 693 | 578–818 | 0.59 |
| PLIN3 | 0.082 | 1,231 | 565–6,348 | 0.71 |
| DPP9 | 0.060 | 1,283 | 740–4,181 | 0.61 |
| ARRDC5(+) | 0.358 | 4,500 | 902–11,401 | 0.44 |
| ARRDC5(−) | 0.955 | 44,245 | 15,265–90,758 | 0.47 |

Tiers: young (MIR7-3/UHRF1/DNM2, ~400–700), intermediate (PLIN3/DPP9, ~1,200 with wide CI), old (ARRDC5+, ARRDC5−).

### 3.3 Carrier co-occurrence

- **No pair reaches Bonferroni-significant positive enrichment.**
- Significant depletion: UHRF1×ARRDC5(+) enrichment 0.39, p=1.9e-9; PLIN3×DPP9 0.33, p=8.2e-3; PLIN3×ARRDC5(+) 0.79, p=1.9e-2.
- DNM2×PLIN3: nominal enrichment 1.97, p=0.02, not Bonferroni; 6 Mb apart → noise.
- ARRDC5(−)×ARRDC5(+): Fisher p=7.8e-21 only because ARRDC5(−) is near-fixed; enrichment 1.05, r²=0.025.

### 3.4 Cross-position clade preservation

- All informative low-frequency pairs: ratio **5.6–43.8** → carrier clades are **not preserved** across the interval → **no shared sweep haplotype**.
- ARRDC5(−) pairs: ratio 1.2–2.2, but ARRDC5(−) is near-fixed (250-tip subsample is not a meaningful clade) → uninformative.

### 3.5 Interpretation

- Modern-only data strongly disfavor a single shared sweep for the low-frequency signals; they sit on distinct haplotype backgrounds with distinct ages across tiers.
- Within the young tier (MIR7-3/UHRF1/DNM2) ages overlap; cannot prove separate selection events. Honest claim: **"no evidence for a shared sweep; different mutational origins / haplotype backgrounds; independent selection events are plausible but not proven."**
- Star-likeness 0.44–0.71 (not ~1) → not classic hard sweeps; consistent with soft/older/standing-variant selection.

### 3.6 Caveats

- Local trees ~5 kb; cross-locus sharing over larger distances not tested.
- ARG topology from NCN.coal; Gravel only used for age resampling.
- Low-frequency variants → limited co-occurrence power.
- Population structure could also produce depletion.
- The test can reject a shared sweep, but cannot by itself prove independent selection events; need per-signal branch-length selection tests + functional evidence.

## 4. Files

- NCN: `NCN_gravel/clade_independence/{README.md, report.md, scripts/, tables/, figures/, intermediate/, logs/}`
- ALL: `ALL_gravel/{README.md, clues/, intermediate/, logs/, clade_independence/}`
- Scripts: `SDSpipeline/scripts/orthogonal/{prepare_all_gravel_subregions.sh, run_clues2_all_gravel_signal.sh, submit_all_gravel_chr19.sh}`
- Fast converter: `CLUES2/RelateToCLUES_fast.py`

## 5. Cleanup (2026-10-22)

- Deleted `ALL/run_work_chr19` (45 GB abandoned chunked run) after preserving its unique results:
  - `ALL/results/UKBQC_ALL_chr19_relate_sds.SDS` (57 MB)
  - `ALL/results/UKBQC_ALL_chr19_selection.{freq,lin,sele}` (192 MB)
  - `ALL_gravel/intermediate/UKBQC_ALL_chr19.dist` and `ALL.data_estimated.coal`
  - `plot_relate_sds_mirror.py` updated to the new `.SDS` path.
- Deleted `tmp/chr19_phased.vcf` (5.9 GB; canonical `.vcf.gz` exists). Total freed ≈ 51 GB.
- Reorganized `NCN_gravel/clade_independence/` into `scripts/` (archive/ for superseded + flawed), `tables/` (archive/ for flawed `cross_mrca`), `figures/`, `intermediate/`, `logs/`; added `README.md`.
- Archived old NCN visualizations to `NCN_gravel/clues/archive/{scripts,plots}/`.
- Archived superseded `run_clues2_all_chr19.sh` to `SDSpipeline/scripts/orthogonal/archive/`.

## 5b. Results: ALL vs NCN (CLUES2 epoch-2, Gravel EAS)

**Cohort relationship:** NCN is almost entirely a subset of ALL — overlap **1,227 / 1,255**; ALL-only = **3,156** samples. ALL is therefore a *superset robustness check*, not an independent replication.

Derived-allele frequencies (NCN / ALL / ALL-only):
| signal | NCN | ALL | ALL-only |
|---|---|---|---|
| MIR7-3 | 0.0402 | 0.0662 | **0.0773** |
| UHRF1 | 0.0578 | 0.0564 | 0.0558 |
| DNM2 | 0.0323 | 0.0581 | **0.0691** |
| PLIN3 | 0.0817 | 0.0861 | 0.0882 |
| DPP9 | 0.0598 | 0.0530 | 0.0502 |
| ARRDC5(−) | 0.9554 | 0.9403 (corrected T) | 0.9355 |
| ARRDC5(+) | 0.3542 | 0.3658 (corrected T; pending) | — |

Epoch-2 (50–200 gen) selection coefficient:
| signal | NCN s2 [95% CI] | ALL s2 [95% CI] | verdict |
|---|---|---|---|
| DNM2 | +0.0210 [0.0148, 0.0272] | +0.0313 [0.0251, 0.0375] | **robust, ALL stronger** |
| DPP9 | +0.0293 [0.0239, 0.0346] | +0.0226 [0.0206, 0.0246] | **robust, ALL slightly weaker** |
| MIR7-3 | +0.0191 [0.0130, 0.0253] | +0.0611 [0.0132, 0.1090] | sign robust; effect size unstable (ALL-only f=0.077) |
| PLIN3 | +0.0173 [0.0153, 0.0193] | +0.0067 [−0.1856, 0.1990] | **not robust (CI crosses 0; flat likelihood surface)** |
| UHRF1 | +0.0123 [−0.0483, 0.0730] | +0.0024 [−0.0555, 0.0603] | null in both |
| ARRDC5(−) | −0.0408 [−0.0470, −0.0346] | +0.0033 [0.0013, 0.0053] (corrected T) | **sign flip; not robust** |
| ARRDC5(+) | +0.0087 [0.0011, 0.0164] | pending (orientation fix) | — |

**PLIN3:** the ALL likelihood surface is flat (top grid points differ by <0.1 logL over s2 0.0067–0.0126; MLE at s1=−0.01, s2=s3=0.00667). The ALL-only frequency is similar (0.088 vs 0.082), so the loss is due to haplotype structure, not frequency. This signal is **not robust to adding 3,156 samples**.

**MIR7-3 / DNM2:** the ALL-only samples have much higher derived frequencies (0.077 / 0.069 vs 0.040 / 0.032); the stronger ALL effect sizes may partly reflect this frequency shift, not a stronger selection coefficient.

**ARRDC5 orientation bug (fixed in a separate dir):** the ALL `.mut` uses REF/ALT (`T/A`, `T/C`) while NCN used EPO (`A/T`, `C/T`). `prepare_clues_inputs.py` then produced a derived vector for T (n=8243) but a popFreq for the opposite allele (0.0592). Fix: patch `.mut` to the EPO orientation and overwrite popFreq with `n_derived/n_haplotypes`; re-run `RelateToCLUES_fast` + inference in `ALL_gravel/clues_flipfix/`. 4903524 corrected (derived T, f=0.9403): s2=**+0.0033 [0.0013, 0.0053]**, vs NCN **−0.0408 [−0.0470, −0.0346]** — a **sign flip** (ARRDC5(−) signal is not robust). 4889675 (derived T) pending. **The original ALL ARRDC5 results are invalid for comparison.**

**ALL post-Relate assets (NCN-equivalent):** ALL Relate ARG complete (job 505938; 18 GB `.anc`, 123 MB `.mut`, 4,383 samples, whole chr19). `RelateSelection --mode SDS` → `ALL/results/UKBQC_ALL_chr19_relate_sds.SDS` (1,534,702 SNPs, chr19:71k–58.6M). `RelateSelection --mode Selection` → `ALL/results/UKBQC_ALL_chr19_selection.{freq,lin,sele}` (493,013 SNPs, full chr19). Both are from the earlier full-chromosome chunked ARG; re-running on the exact full ARG (`run_work_chr19_chb_pyrho`) is optional. CLUES2: 7/7 main inference + posteriors done. Clade independence: inputs ready (`cluster_full.newick`, 218 trees; `clues_analysis` symlinks).

**Job status:** all 7 original jobs DONE; all 7 `_post.txt` written (8.7 MB each). ARRDC5 correction: 4903524 done, 4889675 running.

Outputs: `ALL_gravel/comparison/{ncn_vs_all_clues2.tsv,ncn_vs_all_s2_forest.png}`.

## 5c. ALL clade independence (completed)

Gravel-EAS carrier-clade ages (gen): MIR7-3 314, UHRF1 611, DNM2 690, PLIN3 3,928, DPP9 415, ARRDC5(−) 2,617, ARRDC5(+) 2,408.

- **Stable vs NCN:** MIR7-3 (396), UHRF1 (511), DNM2 (693).
- **Changed:** DPP9 1,283→415, PLIN3 1,231→3,928, ARRDC5(−) 44,245→2,617, ARRDC5(+) 4,500→2,408.
- Star-likeness 0.46–0.75 (not hard sweeps).
- **No pair meets the "same-sweep" criteria.** Low-frequency pairs are independent or significantly **depleted** (MIR7-3×DPP9 enrichment 0.42, p=2.4e-4; PLIN3×ARRDC5(+) 0.66, p=3.3e-14; UHRF1×ARRDC5(+) 0.62, p=1.7e-11).
- Cross-position preservation: UHRF1×PLIN3 15.0, UHRF1×DPP9 20.4, PLIN3×DPP9 10.8 → not preserved (independent). ARRDC5 pairs (~1–2) are uninformative (near-fixed/high frequency).
- ARRDC5(−) (f=0.940) and ARRDC5(+) (f=0.366) are not useful for carrier co-occurrence/preservation.
- Outputs: `ALL_gravel/clade_independence/{report.md, figures/*.png, tables/*.tsv}`.

**Conclusion:** same as NCN — no evidence for a shared sweep; the low-frequency signals sit on distinct haplotype backgrounds. The ALL ages for DPP9/PLIN3/ARRDC5 differ substantially from NCN, so those genealogical ages are not robust to cohort expansion.

## 5d. Structure vs selection diagnostics (2026-10-23)

**NCN/SCN × SLE/non-SLE:** NCN 247 SLE / 973 non-SLE; SCN 1,484 / 1,017; other 234/377. Strong region–phenotype collinearity (NCN 20% SLE, SCN 58% SLE).

**Within-region SLE vs non-SLE** (haplotype + carrier 2×2, Mantel–Haenszel stratified by region): **null at all 7 leads** (MH p = 0.16–0.99). No SLE allele-frequency effect at these leads; the crude SLE difference is region-driven. The selection-strength test (SCN_SLE vs SCN_nonSLE CLUES2) is separate and was running.

**Hudson F_ST NCN vs SCN** (chr19 background n = 1,584; corrected formula):
| signal | p_NCN | p_SCN | F_ST | percentile |
|---|---|---|---|---|
| MIR7-3 | 0.0375 | 0.0769 | 0.0140 | **99.6th** |
| DNM2 | 0.0297 | 0.0708 | 0.0172 | **99.7th** |
| PLIN3 | 0.0807 | 0.0895 | 0.0002 | 76.7th |
| DPP9 | 0.0603 | 0.0526 | 0.0002 | 77.1st |
| UHRF1 | 0.0579 | 0.0572 | −0.0003 | 2.3rd |
| ARRDC5(−) | 0.953 | 0.933 | 0.0032 | 94.6th |
| ARRDC5(+) | 0.356 | 0.372 | 0.0002 | 76.8th |

- **MIR7-3/DNM2 are extreme F_ST outliers (>99.5th)** → strong NCN/SCN differentiation.
- **PLIN3/DPP9/UHRF1/ARRDC5(+) are not differentiated (<95th)** → PLIN3 instability is not frequency/region-driven; it is genealogy/topology or selection heterogeneity.

**ARRDC5 orientation correction complete (derived T):**
- ARRDC5(+) (f=0.366): ALL s2 = **+0.0073 [0.0013, 0.0132]**, NCN +0.0087 [0.0011, 0.0164] → consistent.
- ARRDC5(−) (f=0.940): ALL s2 = **+0.0033 [0.0013, 0.0053]**, NCN **−0.0408 [−0.0470, −0.0346]** → sign flip; near-fixed allele unstable.

**Subgroup trees extracted from the cluster/DNM2 ARG:** NCN 2,454 haps; SCN 5,074; SCN_SLE 2,968; SCN_nonSLE 2,034. First subgroup batch failed on the subgroup `.mut` header/data mismatch (14 vs 11 columns); fixed by using the main ALL 11-column focal `.mut`. Resubmitted as LSF **567388–567395** (PLIN3/DNM2 × NCN/SCN/SCN_SLE/SCN_nonSLE). Frequency-matched inference (ALL times + NCN popFreq; MIR7-3/DNM2/PLIN3/ARRDC5−) resubmitted as LSF **567396**.

**Diagnostic roadmap:** (1) frequency/association [done]; (2) subgroup CLUES2 + frequency matching [running]; (3) local tree/carrier clade + NCN-subtree-from-ALL-ARG + cross-region clade sharing [next]; (4) subgroup-specific coal + constant-vs-Gravel + sMax/df sensitivity [next]; (5) neutral two-population simulation with observed F_ST → CLUES2, to calibrate flat/spurious signals [gold standard].

**Outputs:** `ALL_gravel/pop_structure/{subgroup_frequencies.tsv,sle_tests.tsv,lead_fst_corrected.tsv,fst_background_corrected.tsv}`; `ALL_gravel/freq_matched/`; `ALL_gravel/subgroup/`; `ALL_gravel/comparison/{ncn_vs_all_clues2.tsv,ncn_vs_all_s2_forest.png}`.

## 5e. Final subgroup / frequency-matched CLUES2 (2026-10-24)

**Frequency-matched** (ALL `_times.txt` + NCN popFreq; `--noAlleleTraj`):
| signal | ALL s2 (ALL popFreq) | freq-matched s2 [95% CI] | NCN-only ARG s2 |
|---|---|---|---|
| MIR7-3 | +0.0611 | **+0.0197 [0.0177, 0.0216]** | +0.0191 |
| DNM2 | +0.0313 | **+0.0201 [0.0199, 0.0203]** | +0.0210 |
| PLIN3 | +0.0067 | +0.0257 [0.0237, 0.0276] | +0.0173 |
| ARRDC5(−) | +0.0033 | +0.0127 [−0.0505, 0.0758] | −0.0408 |

→ **MIR7-3/DNM2: the ALL s2 inflation is frequency-driven**; setting the NCN popFreq reproduces the NCN s2 almost exactly. PLIN3/ARRDC5 are unstable to popFreq.

**Subgroup CLUES2** (subtrees of the ALL ARG; Gravel EAS):
| group | DNM2 s2 [95% CI] (logLR) | PLIN3 s2 [95% CI] (logLR) |
|---|---|---|
| NCN | +0.0508 [−0.0105, 0.1120] (69.4) | −0.0016 [−0.0138, 0.0106] (1.2) |
| SCN | +0.0206 [0.0144, 0.0268] (131.0) | +0.0098 [0.0097, 0.0100] (110.2) |
| SCN_SLE | +0.0267 [0.0265, 0.0269] (31.8) | +0.0192 [0.0172, 0.0212] (14.7) |
| SCN_nonSLE | +0.0042 [−0.0125, 0.0210] (1.9) | −0.0055 [−0.0503, 0.0394] (3.5) |

Findings:
- **PLIN3 is SCN-specific and SLE-specific.** The NCN-only ARG signal (+0.0173) is an **ARG artifact**: the NCN subtree from the ALL ARG is null (−0.0016, logLR 1.2). SCN +0.0098 (logLR 110); SCN_SLE +0.0192 (logLR 14.7); SCN_nonSLE null (−0.0055, logLR 3.5).
- **DNM2 is significant in SCN (+0.0206 [0.0144, 0.0268])**, positive but CI-crossing in the NCN subtree (+0.0508 [−0.0105, 0.1120]), and strongest in SCN_SLE (+0.0267). Direction is robust; effect size is ARG-dependent.
- **SLE candidate:** within SCN, both PLIN3 and DNM2 are stronger in SLE than non-SLE. The non-SLE logLR is low (1.9–3.5), so power/frequency need to be excluded; the SCN_SLE vs SCN_nonSLE comparison is within the same ARG, which is the cleanest available design.
- **ARG dependence:** NCN-only ARG vs NCN subtree from the ALL ARG: PLIN3 +0.0173 vs −0.0016; DNM2 +0.0210 vs +0.0508. **CLUES2 s2 is strongly ARG-dependent; cross-ARG comparisons are not valid.**
- **CI caveat:** several profile CIs are implausibly tight (e.g., SCN_SLE DNM2 [0.0265, 0.0269]; SCN PLIN3 [0.0097, 0.0100]) because `inference.py` profiles a sparse `_tempfile` surface. Use the point estimates/logLR as the main evidence; re-estimate CIs with a dense grid or bootstrap.

## 6. Next

1. Finish the ARRDC5 orientation correction (`clues_flipfix/`), then add the corrected s2 values to the comparison.
2. Run the ALL clade-independence analysis on `ALL_gravel/clues_analysis/` (5 original + 2 corrected dirs) with `cluster_full.newick` (218 trees, 55 MB; chr19:4.70–4.96 Mb).
3. Diagnose PLIN3: flat ALL likelihood, stable frequency; test ALL-only (3,156) vs NCN-only (1,227) and SLE/non-SLE split to see whether the NCN signal is cohort-specific.
4. For MIR7-3/DNM2, separate the effect of the higher ALL-only frequency from a genuinely stronger selection coefficient (e.g., frequency-matched subsampling).
5. Wait for the 4 remaining posterior trajectories or re-run with `--noAlleleTraj` if only s(t) tables are needed.
