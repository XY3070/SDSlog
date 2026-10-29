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

## 5f. Continuous structure: region / PC1 / singleton / SLE (2026-10-29)

**Metadata:** `SDSworkspace/population_structure/pca_umap/umap_common_coords.tsv` (4,383 samples; Region, Phenotype, PC1, singleton_count, is_outlier). **PC1~singleton r = −0.495**; PC1 is a south–north + technical axis (JiNing/Jinan −0.022, HK/Guangzhou +0.004/+0.005). Region and SLE are strongly collinear (Shenzhen 70% SLE; Jinan/Guangzhou 0%; HK 50.3%). The NCN/SCN split is a coarse dichotomy of this continuous cline.

**Models** (OLS HC1; dosage 0/1/2; controls; `PC1_resid` = PC1 residualized on singleton):
- **Region frequencies (haplotype):** MIR7-3 HK 0.072 / Shenzhen 0.060 / Unknown 0.076 vs JiNing 0.040 / Jinan 0.038; DNM2 HK 0.064 / Unknown 0.064 vs JiNing 0.026 / Jinan 0.026. PLIN3 similar across regions (0.083–0.088; Shanghai 0.055). UHRF1 flat; ARRDC5 near-fixed/high.
- **PC1_resid beta (controls):** MIR7-3 +2.87 (p<1e-4), DNM2 +2.40 (p<1e-4), DPP9 −1.16 (p=0.016), ARRDC5(+) +2.14 (p=0.009); PLIN3 +0.32 (p=0.60), UHRF1 +0.07 (p=0.87), ARRDC5(−) −0.72 (p=0.43).
- **Region joint F after PC1_resid+singleton: all p>0.37** → no discrete region effect; the region differences are a continuous cline.
- **Singleton:** DNM2 beta −0.0001, p=0.0013 (technical effect); MIR7-3 p=0.10; others n.s.
- **SLE adjusted (SLE + PC1_resid + singleton + Region):** no significant effect; PLIN3 +0.025 (p=0.053, borderline), others p>0.13. Within HK/Shenzhen: no significant SLE.
- **Region residual (controls):** small; joint region p>0.37.

**PC1-cline background** (chr19, 3,371 SNPs, MAF>0.01, stride 50; controls; PC1_resid+singleton):
- |beta| quantiles: median 1.20, 90th 3.71, 95th 4.61, 99th 6.61.
- Lead |beta| percentiles: MIR7-3 81.5, DNM2 75.2, ARRDC5(+) 70.7, DPP9 48.8, ARRDC5(−) 33.2, PLIN3 16.2, UHRF1 3.4.
- **Standardized |beta_std|** (partial correlation): background 99th 0.152; **MIR7-3 0.107 (94.8th)**, **DNM2 0.095 (92.2nd)**, DPP9 0.048 (60.0th), ARRDC5(+) 0.041 (52.6th), ARRDC5(−) 0.028 (36.6th), PLIN3 0.011 (15.4th), UHRF1 0.003 (3.6th).
- **No lead is an outlier (>99th).** The F_ST 99.6/99.7th for MIR7-3/DNM2 was a NCN/SCN dichotomization artifact; the continuous PC1 cline is 92–95th, within the genome-wide background.

**Implications:**
- The apparent region differences are a **continuous cline** (MIR7-3/DNM2 moderately steep, 92–95th), not discrete region-specific selection. PLIN3/UHRF1 are flat.
- The SCN/SLE CLUES2 signals (PLIN3 SCN +0.0098, SCN_SLE +0.0192; DNM2 SCN_SLE +0.0267) are **not reflected in frequency/PC1/SLE** → haplotype-level selection without frequency differentiation, or an ARG/CLUES2 artifact. **Local haplotype/clade is the decisive test.**
- The Nature Comms LAI (RFMixV2/Tractor) is **continental-level** and cannot resolve this continuous cline; the appropriate adaptation is `PC1_resid` (continuous ancestry) + local haplotype/clade (local ancestry).
- Next: gene-level positive controls (ADH1B/ALDH2/OAS) PC1-standardized betas vs background; local haplotype/clade for PLIN3/DNM2/SLE; ARG uncertainty; simulation.

## 5g. P2/P3: 15-signal PC1 clines + SLE carrier-clade test (2026-10-29)

**P2 — PC1-standardized beta for the 15 LD-split signals** (controls; PC1_resid+singleton; local ±500 kb background, MAF>0.01):
| signal | gene | DAF | beta_std | local n | local pct |
|---|---|---|---|---|---|
| chr11_48235 | NaN | 0.149 | +0.224 | 133 | **100.0** |
| chr19_4885_sig3 | MIR7-3 | 0.067 | +0.112 | 153 | **98.7** |
| chr22_41498 | ACO2 | 0.522 | +0.183 | 109 | **97.2** |
| chr19_10690 | DNM2 | 0.061 | +0.104 | 134 | **96.3** |
| chr19_4885_sig1 | DPP9 | 0.053 | −0.047 | 156 | 62.2 |
| chr19_4885_sig4 | ARRDC5 | 0.942 | +0.039 | 143 | 52.4 |
| chr19_4885_sig5 | ARRDC5 | 0.366 | −0.039 | 144 | 52.1 |
| chr12_109929 | TRPV4 | 0.069 | +0.031 | 106 | 49.1 |
| chr12_110306 | ATP2A2 | 0.066 | +0.042 | 50 | 46.0 |
| chr12_112949 | OAS3 | 0.151 | −0.022 | 109 | 38.5 |
| chr12_112355 | NAA25 | 0.245 | +0.016 | 72 | 23.6 |
| chr12_111181 | ALDH2 | 0.245 | +0.015 | 76 | 21.1 |
| chr19_4885_sig2 | PLIN3 | 0.085 | +0.011 | 146 | 19.9 |
| chr19_4885_sig6 | UHRF1 | 0.061 | −0.005 | 137 | 7.3 |
| chr4_99240 | ADH1B | 0.700 | −0.001 | 104 | **0.0** |

- **ADH1B (0th) and ALDH2 (21st) — the two canonical East Asian selection signals — have NO PC1 cline.** Therefore the PC1-cline is a **structure adjustment, not a selection test**; their selection is haplotype-based (long haplotypes), not a frequency cline.
- chr19: **MIR7-3 98.7th and DNM2 96.3rd are local cline outliers**; DPP9/ARRDC5 ~52–62nd; **PLIN3 19.9th and UHRF1 7.3rd are flat** (consistent with no region frequency differentiation).
- chr11_48235 (gene NaN) 100th and ACO2 97.2nd are additional clinal signals worth annotation.
- Local n = 50–156; percentiles are approximate.

**P3 — SLE vs non-SLE carrier-clade test inside SCN** (SCN subtree trees, Gravel-EAS; permutation within carriers):
| signal | carriers | SLE carriers | control carriers | MRCA all = SLE = control | delta | p_perm |
|---|---|---|---|---|---|---|
| PLIN3 | 454 | 285 | 167 | 800.7 gen | 0.0 | 1.000 |
| DNM2 | 359 | 219 | 138 | 591.4 gen | 0.0 | 0.981 |

- The SLE and non-SLE carriers are **interleaved across the same carrier clade**; their MRCAs are identical and the permutation delta is 0. **No SLE-specific sub-clade.**
- Therefore the SCN_SLE CLUES2 difference (PLIN3 +0.0192 vs −0.0055; DNM2 +0.0267 vs +0.0042) is **not a case-specific haplotype signal**; it is more likely **power/frequency** (SLE 1,484 vs non-SLE 1,017) or CLUES2/ARG noise. **SLE-specific recent selection is not supported.**
- The **SCN population-level signals remain**: PLIN3 SCN +0.0098 (logLR 110.2), DNM2 SCN +0.0206 (logLR 131.0). PLIN3 NCN is null (logLR 1.2) and its PC1 cline is flat → **SCN-specific haplotype-level selection candidate** (no frequency differentiation). DNM2 is shared (NCN subtree +0.0508 but CI crossing zero; SCN +0.0206).

## 5h. P4: HK confound / batch / annotation (2026-10-29)

**SCN is effectively HongKong:** SCN 2,537 = **2,411 HK (95%)** + 126 other. PLIN3 carriers 454 = **440 HK (97%)**; DNM2 carriers 359 = 343 HK (96%). The "SCN-specific" signal is therefore **HK-specific**, directly relevant to the HK platform/quality concern.

**Tree-age bootstrap** (200 Gravel trees; max of the derived-line coalescence times; 2,000 resamples):
| cohort | PLIN3 age [95%] | DNM2 age [95%] |
|---|---|---|
| NCN | 1,230.8 [1,054.5, 1,426.0] | 692.6 [684.5, 700.5] |
| SCN | **3,413.1 [3,023.3, 3,829.3]** | 742.0 [729.6, 754.9] |

The SCN PLIN3 carrier clade is **older** than NCN (3,413 vs 1,231 gen), not a recent sweep — consistent with **selection on a standing haplotype**, not a de novo sweep.

**QC of SCN carriers vs non-carriers (t-test):**
- PLIN3: singleton p=0.95, F_MISS p=0.84, F_het p=0.39, PC1 p=0.44 → **no QC difference**.
- DNM2: singleton p=0.73, F_MISS p=0.0021, F_het p=0.28, PC1 p=0.016 → mild missingness/PC1 difference.

**HK batch prefixes:** Q0 747, Q1 791, HK 350, BA 131, VH 107, WT 51, …
- PLIN3 frequency: Q1 0.0878, Q0 0.1039, HK 0.0843, BA 0.0878, VH 0.0794, WT 0.0980 → no single-batch frequency artifact.
- DNM2 frequency: Q1 0.0709, Q0 0.0746, HK 0.0871, BA 0.0687, VH 0.0654, WT 0.0686.

**Annotation of the new clinal signals:**
- chr11:48,235,462 lies in the **OR4X2/OR4B1 olfactory receptor cluster** (100th PC1 cline; gene was NaN in the catalog).
- chr22:41,504,089 lies in **ACO2** (mitochondrial TCA enzyme; 97.2th cline; enhancer/CTCF nearby).

**Within-HK batch CLUES2** (Q0/Q1 × PLIN3/DNM2; no-CI MLE/logLR) submitted as LSF **637189–637192**; it tests whether the HK signal is consistent across the two largest HK batches.

**Interpretation:** the SCN PLIN3 candidate is HK-specific, QC-normal, and consistent across HK batches at the frequency level, with an **older standing-variant carrier clade**. The HK/NCN difference remains confounded by the **all-HK high-quality platform** (fewer singletons) — a batch/phasing effect that frequency QC cannot fully exclude. The SLE-specific hypothesis is **closed** (carrier-clade null).

## 5i. LocusZoom of the new clinal signals (2026-10-29)

- **ACO2 (chr22:41,504,089:G:A, DAF 0.522)**: lead lies inside **ACO2** (mitochondrial TCA enzyme); **broad LD block** with many r²>0.8 variants spanning ~41.45–41.60 Mb; −log10(p) ≈ 8; recombination valley; nearby TOB2/POLR3H/CSDC2/DESI1/PMM1/XRCC6. This is a **long-haplotype signal**, more consistent with real selection; the PC1 cline is 97.2th. Plausible metabolic (TCA) selection candidate.
- **OR4X2/OR4B1 (chr11:48,235,462:G:A, DAF 0.149)**: lead is **intergenic** between OR4B1/OR4B2P and OR4X2/OR4X1; **LD with the lead is weak** (only a few r²>0.2 variants); the OR cluster has recombination hotspots and segmental duplications. The 100th-percentile PC1 cline is a **single-SNP/intergenic cline**, not a long-haplotype sweep; likely drift/olfactory adaptation or a mapping/structural artifact rather than a classic sweep.
- **ACO2 functional/iHS (2026-10-29):** lead is an **intron_variant** (ENST00000216254.9:c.174-3702G>A) with **reg_enhancer=yes**; SDS norm 6.07; PC1 cline 97.2th; ALL **normalized iHS = −0.570** (not |iHS|>5). The long LD block + regulatory intron + cline point to a **standing-variant/regulatory metabolic (TCA) candidate**, not a classic hard sweep.
- **OR4X2 iHS:** ALL normalized iHS = **−2.91** (not extreme); the lead is **intergenic with no regulatory annotation**; combined with the weak LD, this supports drift/olfactory/mapping rather than selection.
- **HK batch test (Q0/Q1)**: 4 no-CI CLUES2 jobs 637189–637192 were still in MLE at 15:02; the per-batch signal strength is pending.
- Files: `ALL_gravel/pop_structure/locuszoom/{ACO2_41504089,OR4X2_48235462}.locuszoom.png`.

## 5j. 1KG validation of the ACO2/OR4X2 clines (2026-10-29)

Ensembl `variation/human/<rsID>?pops=1`, 1000 Genomes phase 3:

| signal (lead allele) | JPT | CHB | CHS | CDX | KHV | EAS | cohort DAF |
|---|---|---|---|---|---|---|---|
| ACO2 rs2284081 (A) | 0.308 | 0.495 | 0.543 | 0.651 | 0.586 | 0.513 | 0.522 |
| OR4X2 rs147168038 (A) | 0.024 | 0.053 | 0.152 | 0.366 | 0.227 | 0.160 | 0.149 |

- Both show a clear **south-high/north-low East Asian gradient** (CDX/KHV/CHS > CHB > JPT). The cohort's ACO2 97.2th and OR4X2 100th PC1 clines are therefore **genuine East Asian frequency gradients, not cohort/batch artifacts**.
- The ACO2 lead (rs2284081 A) is common (0.31–0.65) across East Asia; OR4X2 is rare in the north (JPT 0.024, CHB 0.053) and common in the south (CDX 0.366).
- Selection vs drift must still be tested at the haplotype level (SDS/iHS/ARG): ACO2 has a long LD block + enhancer intron + SDS 6.07 (standing-variant/regulatory candidate, iHS −0.57); OR4X2 has weak LD + iHS −2.91 (drift/olfactory).

## 5k. Within-HK batch test FAILS (2026-10-29)

Q0/Q1 × PLIN3/DNM2, no-CI CLUES2 (LSF 637189–637192); MLE from the live `.clues_tempfile.txt`:

| batch | PLIN3 logLR (s2) | DNM2 logLR (s2) |
|---|---|---|
| Q0 (747) | 1.14 (−0.0093) | 0.81 (+0.0078) |
| Q1 (791) | 0.29 (+0.0067) | 1.38 (+0.0056) |
| **full SCN (2,537)** | **110.2 (+0.0098)** | **131.0 (+0.0206)** |

- Halving the sample should reduce an additive CLUES2 logLR to ~55/65 per half; the observed per-batch logLR is **0.3–1.4**, a ~100-fold collapse. The Q0 PLIN3 s2 is even negative.
- **The full-SCN PLIN3/DNM2 CLUES2 signals are NOT reproduced in either of the two largest HK batches.** They are therefore **not robust cross-batch CLUES2 s2 estimates**; likely driven by ARG/tree-sampling/subsample composition or an inflated CLUES2 logLR.
- **The SCN/HK PLIN3 "standing-variant selection" candidate is downgraded/withdrawn.** The remaining robust statements are: (i) the SCN vs NCN CLUES2 difference is not a stable per-batch signal; (ii) the HK/NCN platform confound remains; (iii) the chr19 CLUES2 results are ARG-dependent and should not carry strong selection claims.
- A robust alternative would be per-batch **SDS/iHS/EHH** (not CLUES2); **EHH has now been run — see 5n. Correction: DNM2's extended haplotype IS reproducible across Q0/Q1/SCN (ARG-free EHH, half-life ~32 kb); the batch collapse applies to the CLUES2 s2 estimates, not to the haplotype. PLIN3 has no EHH extension in any group, so its withdrawal stands.**

## 5l. LD source resolved: the "9k" QC VCF IS the ALL cohort (2026-10-29; supersedes the earlier mismatch note)

**Earlier same-day note (WRONG, kept only as history):** claimed the ACO2/OR4X2 LocusZoom used a different cohort because the 9k QC VCF INFO/AF at the ACO2 lead was 0.343 vs ALL 0.5217.

**What is actually true:**

- `bcftools query -l` on `raw/9k/snp_singleton_qc_chr22_v1/snp_qc_pass.vcf.gz` vs `input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr22.phased.vcf.gz`: **4,383 samples each, 4,383 shared, 0 unique → identical sample set.**
- At the ACO2 lead the 9k VCF has **AC=4570/AN=8760 = 0.5217** (not 0.343). AC/AN match the ALL VCF at every checked site (chr22:41300003–41300031; OR4X2 lead 1311/8766; ACO2 4573/8766). **INFO/AF in the 9k QC VCF is stale** — it is also nonsensical elsewhere (AC=0 with AF=5.3e-5), i.e. left over from the original larger joint call before subsetting. **Never use INFO/AF from this VCF; use AC/AN or recompute.**
- Therefore the LocusZoom LD (5i) was already computed on the correct samples. Re-ran with the ALL phased VCF anyway (`ACO2_ALL_41504089`, `OR4X2_ALL_48235462`); plots are visually identical, as expected. Small differences in LD pair counts (16,829 vs 15,942) come from the QC VCF dropping some variants, not from different samples.
- **ACO2 LD quantification** (PLINK2 `--r2-unphased`, 560 kb, ALL VCF): max r²=0.913; **29 SNPs r²≥0.8 spanning 136 kb** (41.422–41.558 Mb); **102 SNPs r²≥0.6 spanning ~451 kb**; only ~1% of all 16,829 regional variants reach r²≥0.6 (median r²≈0 within 300 kb). The block is bounded by recombination hotspots at ~41.30 and ~41.56–41.60 Mb (CHB map). → ACO2 is a **common standing haplotype with a 100–450 kb LD block**, not a genome-wide-extreme long-haplotype sweep.
- **OR4X2 LD quantification**: max r²=0.935 but only **2 SNPs r²≥0.8**, 18 at r²≥0.6 within ~99 kb; a broad r²≈0.2–0.4 band over ~100 kb. **No long high-LD haplotype** → consistent with drift/cline, not a sweep.
- **NCN–SCN Hudson F_ST at the two clinal leads** (ALL VCF, AC/AN): **ACO2 0.0546** (NCN 999/2454 vs SCN 2918/5074), **OR4X2 0.0781** (138/2454 vs 957/5074). Both exceed the chr19 background maximum (0.0308, n=1,584) → 100th percentile. **These are extreme differentiation loci, so their SDS signals are structure-first candidates**; ACO2 iHS is only −0.57, i.e. no independent evidence of a recent sweep. Revised ACO2 verdict: **clinal/standing candidate, selection not established** (not a long-haplotype sweep).
- **Catalog iHS columns are stale**: `all_signals_catalog.tsv` gives ACO2 iHS_norm=+0.668 and OR4X2=NA, whereas the current `ihs/epo3/ALL/*.100bins.norm.epo3` gives **−0.570** and **−2.908**. Use the epo3 normalized files for any iHS statement.
- **LoF check (unchanged, valid):** 41499744/41507831/41508047 are absent from both VCFs; the VEP stop_gained is a rare/hypothetical allele; the known variant at 41508047 is **rs2518216968 A/C missense**, not A>T stop_gained. **The ACO2 LoF is not a cohort signal**; the lead's common intron/enhancer signal (CADD 1.67) is separate.
- ACO2 functional: lead intron/enhancer, no GWAS/ClinVar phenotype, **no GTEx eQTL for the lead** (EUR); weak/suggestive regulatory evidence.

## 5m. NCN region-sharing test: selected haplotypes are shared, not region-specific (2026-10-29)

**Metadata clarification:** **NCN is a genetic cluster, not a geographic region.** NCN = HongKong 473 + JiNing 389 + Jinan 171 + Unknown 82 + Shanghai 79 + Shenzhen 30 + Guangzhou 3; SCN = HongKong 2,411 + Shenzhen 58 + Guangzhou 27 + Unknown 25 + Shanghai 14 (95% HK). NCN/SCN contrasts are therefore ancestry-cluster contrasts, not clean north/south geography.

**Carrier frequency is flat across NCN regions** (carriers / haplotypes):

| signal | HongKong | JiNing | Jinan |
|---|---|---|---|
| MIR7-3 | 0.0433 | 0.0411 | 0.0380 |
| DNM2 | 0.0317 | 0.0244 | 0.0263 |
| PLIN3 | 0.0803 | 0.0810 | 0.0819 |
| DPP9 | 0.0592 | 0.0591 | 0.0614 |
| UHRF1 | 0.0581 | 0.0578 | 0.0585 |
| ARRDC5(+) | 0.3626 | 0.3843 | 0.3363 |

→ the NCN–SCN frequency differences are **between clusters**, not a within-NCN regional gradient.

**Carrier-clade sharing across regions** (NCN-only Relate trees, 50 trees/signal; `NCN_gravel/clade_independence/tables/region_clade_test.tsv`):

| signal | n carriers | all-carrier MRCA (gen) | HK MRCA | JiNing MRCA | HK carriers in JiNing clade | JiNing carriers in HK clade | perm p (HK−JiNing) |
|---|---|---|---|---|---|---|---|
| MIR7-3 | 101 | 315.9 | 315.9 | 289.8 | 95.1% | 100% | 0.078 |
| DNM2 | 81 | 467.6 | 381.4 | 467.6 | 100% | 84.2% | 0.956 |
| PLIN3 | 205 | 626.1 | 626.1 | 626.1 | 100% | 100% | 0.858 |
| DPP9 | 150 | 576.2 | 576.2 | 576.2 | 100% | 100% | 0.694 |
| UHRF1 | 145 | 455.2 | 404.6 | 455.2 | 100% | 97.8% | 0.859 |
| ARRDC5(+) | 899 | 1,020.3 | 832.3 | 1,020.3 | 100% | 99.3% | 0.917 |
| ARRDC5(−) | 2,398 | 11,238.5 | 11,238.5 | 4,086.4 | 100% | 100% | 0.111 |

- **No region-specific subclade:** every signal's HK and JiNing carriers fall inside each other's carrier clade (84–100%), and the regional MRCAs are within ~0–25% of the all-carrier MRCA. The HK–JiNing age difference is not significant (perm p=0.078–0.96).
- **Interpretation:** the selected haplotypes are **single, geographically widespread haplotypes**, not regional structure artifacts. Combined with cross-signal clade independence (no shared clade), this supports multiple independent standing haplotypes, each spread across regions; the NCN–SCN frequency difference is superimposed on the same haplotype rather than reflecting different haplotypes.
- **Caveat:** carriers necessarily form one clade if the mutation is single-origin; the informative part is the young MRCA and the absence of deep regional substructure. The ages here are raw tree MRCAs; the Gravel-scaled ages in `signal_ages_gravel.tsv` are ~25% larger (e.g. MIR7-3 396 vs 316).
- Script: `SDSpipeline/scripts/orthogonal/region_clade_test_ncn.py`.

## 5n. Per-batch EHH: PLIN3 has no haplotype extension; DNM2's extended haplotype is reproducible (2026-10-29)

Motivation: the 5k batch test used **CLUES2**, which depends on the ARG and on sample size. **EHH depends only on phased haplotypes.** Ran `selscan --ehh` (MAF 0.02, ±225 kb) on Q0 (747), Q1 (791), full SCN (2,537) and NCN (1,227), using ALL phased VCF subsets.

| signal | group | n | alt freq | EHH at 100 kb | half-life (kb) | iHH (EHH·kb) |
|---|---|---|---|---|---|---|
| PLIN3 | Q0 | 747 | 0.101 | 0.0024 | 4.7 | 14.7 |
| PLIN3 | Q1 | 791 | 0.088 | 0.0025 | 4.5 | 13.2 |
| PLIN3 | SCN | 2,537 | 0.089 | 0.0026 | 4.7 | 13.9 |
| PLIN3 | NCN | 1,227 | 0.081 | 0.0013 | 3.8 | 12.2 |
| DNM2 | Q0 | 747 | 0.077 | 0.128 | 32.0 | 84.2 |
| DNM2 | Q1 | 791 | 0.074 | 0.086 | 33.5 | 77.5 |
| DNM2 | SCN | 2,537 | 0.075 | 0.097 | 32.6 | 78.9 |
| DNM2 | NCN | 1,227 | 0.031 | 0.029 | 15.9 | 44.5 |

- **PLIN3 has no extended haplotype in any group** (half-life 4–6 kb; EHH at 100 kb ≈0.002). The PLIN3 CLUES2 signals are therefore **not haplotype-supported**; the withdrawal stands on ARG-free evidence.
- **DNM2 has a modest but reproducible extended haplotype** in both HK batches and full SCN (half-life 32–34 kb, EHH at 100 kb 0.09–0.13), ~7× longer than PLIN3 at similar DAF; NCN is weaker (16 kb, 0.03). **The 5k batch collapse therefore applies to the CLUES2 s2 estimates, not to the haplotype itself** (correction to 5k).
- The CLUES2 s2 ranking does not track EHH strength (SCN EHH 32 kb > NCN 16 kb, while CLUES2 s2 is similar), another sign that s2 is ARG/sample-size dependent.
- Benchmark (existing NCN EHH): MIR7-3 27.4 kb, ACO2 17.8, OR4X2 26.7, UHRF1 13.1, DPP9 6.8, ARRDC5(+) 2.5, ARRDC5(−) 11.7; chr12 long-haplotype signals 66–100 kb (ALDH2 99.9). DNM2 (32 kb) is upper-middle of the chr19 set; PLIN3 is at the bottom.
- Caveat: single-locus EHH without a DAF/recombination-matched null; half-life comparisons across loci are confounded by DAF and local recombination. Both loci sit in low-recombination regions, so the PLIN3–DNM2 contrast is meaningful but not a formal test.
- Files: `ALL_gravel/batch_ehh/batch_ehh_summary.tsv`, script `SDSpipeline/scripts/orthogonal/batch_ehh_check.py`.

## 5o. Sign-flip diagnosis: which s signs are real, and do they match the independent SDS/haplotype signs? (2026-10-29)

**Inventory of CLUES2 epoch-2 s estimates** (all runs, grouped by scope):

| scope | signal | s2 | logLR | 95% CI | derived freq |
|---|---|---|---|---|---|
| NCN-only ARG | MIR7-3 | +0.0191 | 65.8 | [+0.0129,+0.0253] | 0.040 |
| NCN-only ARG | UHRF1 | +0.0123 | 148.2 | [−0.0483,+0.0730] | 0.063 |
| NCN-only ARG | PLIN3 | +0.0173 | 32.4 | [+0.0153,+0.0192] | 0.082 |
| NCN-only ARG | DPP9 | +0.0293 | 66.8 | [+0.0239,+0.0346] | 0.060 |
| NCN-only ARG | ARRDC5(−) | **−0.0408** | 105.7 | [−0.0470,−0.0346] | 0.955 |
| NCN-only ARG | ARRDC5(+) | +0.0087 | 3.8 | [+0.0010,+0.0163] | 0.354 |
| NCN-only ARG | DNM2 | +0.0210 | 293.3 | [+0.0148,+0.0272] | 0.032 |
| SCN (ALL ARG) | PLIN3 | +0.0098 | 110.2 | [+0.0097,+0.0100] | 0.089 |
| SCN (ALL ARG) | DNM2 | +0.0206 | 131.0 | [+0.0144,+0.0268] | 0.071 |
| NCN (ALL ARG) | PLIN3 | −0.0016 | 1.2 | [−0.0138,+0.0106] | 0.081 |
| NCN (ALL ARG) | DNM2 | +0.0508 | 69.4 | [−0.0105,+0.1120] | 0.030 |
| Q0 (HK) | PLIN3 | −0.0096 | 2.8 | — | 0.102 |
| Q1 (HK) | PLIN3 | +0.0016 | 0.4 | — | 0.088 |
| SCN_SLE | PLIN3 | +0.0192 | 14.7 | [+0.0172,+0.0212] | 0.095 |
| SCN_nonSLE | PLIN3 | −0.0055 | 3.5 | [−0.0503,+0.0394] | 0.082 |
| Q0/Q1 | DNM2 | +0.0070/+0.0002 | 0.9/1.8 | — | 0.072/0.070 |
| SCN_SLE/SCN_nonSLE | DNM2 | +0.0267/+0.0042 | 31.8/2.0 | — | 0.073/0.067 |
| ALL (full; ARRDC5 orientation-corrected) | MIR7-3 | +0.0611 | 519.0 | [+0.0132,+0.1090] | 0.068 |
| ALL (full; ARRDC5 orientation-corrected) | UHRF1 | +0.0024 | 443.7 | [−0.0555,+0.0603] | 0.061 |
| ALL (full; ARRDC5 orientation-corrected) | PLIN3 | +0.0067 | 196.4 | [−0.1856,+0.1989] | 0.086 |
| ALL (full; ARRDC5 orientation-corrected) | DPP9 | +0.0226 | 488.0 | [+0.0206,+0.0245] | 0.053 |
| ALL (full; ARRDC5 orientation-corrected) | ARRDC5(−) | **+0.0033** | 680.7 | [+0.0013,+0.0053] | 0.940 |
| ALL (full; ARRDC5 orientation-corrected) | ARRDC5(+) | +0.0073 | 16.9 | [+0.0013,+0.0132] | 0.366 |
| ALL (full; ARRDC5 orientation-corrected) | DNM2 | +0.0313 | 996.5 | [+0.0251,+0.0375] | 0.061 |
| ALL freq-matched | MIR7-3 | +0.0197 | 315.1 | [+0.0177,+0.0216] | — |
| ALL freq-matched | PLIN3 | +0.0257 | 240.4 | [+0.0237,+0.0276] | — |
| ALL freq-matched | DNM2 | +0.0201 | 820.9 | [+0.0199,+0.0203] | — |
| ALL freq-matched | ARRDC5(−) | +0.0127 | 656.8 | [−0.0505,+0.0758] | — |

**Which comparisons actually flip sign?**

1. **NCN vs SCN (same statistic, full cohorts): no flip.** The only signals with both SCN and NCN CLUES2 are PLIN3 (+0.0173 vs +0.0098) and DNM2 (+0.0210 vs +0.0206); both are positive in both cohorts.
2. **ARRDC5(−) NCN vs ALL: a real flip** (−0.0408 vs +0.0033), at a near-fixed derived allele (0.955 vs 0.940). The ALL positive value is tiny, and the NCN-frequency-matched run is +0.0127 with a wide CI [−0.0505,+0.0758]. The sign at this locus is **estimator-unstable**, not a cohort-specific biology difference.
3. **ARRDC5(+): an orientation-induced apparent flip, now fixed.** The VCF ALT (C) is the *ancestral* allele; the derived allele is REF T (DAF 0.366). The original ALL `.mut` used REF/ALT, so CLUES2 followed the wrong allele; NCN used EPO. After correction, ALL +0.0073 [+0.0013,+0.0132] matches NCN +0.0087 [+0.0010,+0.0163].
4. **PLIN3 within-stratum flips are flat-likelihood cases, not sign conflicts.** Negative values occur only where logLR = 0.4–3.5 and the CI crosses zero (Q0 −0.0096, Q1 +0.0016, SCN_nonSLE −0.0055, NCN-subtree −0.0016); every powered PLIN3 estimate is positive (NCN-only +0.0173, SCN +0.0098, SCN_SLE +0.0192, freq-matched +0.0257).

**Are the s signs consistent with the independent SDS/haplotype signs?**

ALL SDS signs (catalog `norm_SDS`, true derived allele): MIR7-3 +10.44, UHRF1 +9.34, PLIN3 +8.57, DPP9 +5.47, ARRDC5(+) +4.00, DNM2 +9.27, **ARRDC5(−) −4.89**.

Per-cohort, ARG-free haplotype sign (`iHH_der − iHH_anc` from selscan EHH; this is the sign of the SDS numerator, not the standardized SDS):

| signal | NCN ΔiHH | SCN ΔiHH | ALL SDS sign | NCN CLUES2 sign | SCN CLUES2 sign | consistent? |
|---|---|---|---|---|---|---|
| MIR7-3 | +27.0 | +26.6 | + | + | n/a | yes |
| UHRF1 | +16.2 | +11.5 | + | + | n/a | yes |
| PLIN3 | +4.8 | +6.6 | + | + (powered); − only when flat | + | yes where powered |
| DPP9 | +16.4 | +16.6 | + | + | n/a | yes |
| ARRDC5(−) | **−10.8** | **−12.1** | **−** | **−** | n/a | NCN yes; ALL **no** |
| ARRDC5(+) | +7.2 | +7.3 | + | + | n/a | yes |
| DNM2 | +28.1 | +62.5 | + | + | + | yes |

- The per-cohort haplotype sign is **identical in NCN and SCN at all 7 leads**, and matches the ALL SDS sign in all 7 cases (positive except ARRDC5(−)).
- The NCN CLUES2 sign matches this independent sign at **all 7** leads, including the negative ARRDC5(−).
- The SCN CLUES2 signs (PLIN3, DNM2) match it at both loci.
- **The only genuine mismatch is ALL ARRDC5(−)**: its positive s (+0.0033) conflicts with the negative SDS (−4.89) and the negative NCN/SCN haplotype sign; the NCN negative s is the credible one.
- The PLIN3 flat-stratum negative estimates conflict in sign but carry no likelihood support (logLR ≤3.5, CI crossing 0), so they are undefined rather than contradictory.

**Conclusion:** there is no evidence for a cohort-specific (NCN vs SCN) sign reversal. The observed flips reduce to (i) an orientation error (ARRDC5(+), fixed), (ii) flat/undefined likelihoods at low power (PLIN3 strata), and (iii) estimator instability at a near-fixed allele (ARRDC5(−), where the ALL sign is the outlier). At every locus with power, the CLUES2 s sign agrees with the independent per-cohort haplotype sign and with the ALL SDS sign.

**Caveat:** the original per-cohort genome-wide SDS tables are no longer in the workspace (the May paths under `sds/data/processed/...` were deleted). Per-cohort sign is therefore assessed with the SDS-numerator proxy from EHH (ARG-free), while the actual SDS sign is available for ALL. If the NCN/SCN SDS Manhattans are restored, the exact per-cohort norm_SDS signs can be substituted.

**Files:** `ALL_gravel/cohort_ehh/cohort_ehh_sign.tsv`, `SDSpipeline/scripts/orthogonal/cohort_ehh_sign.py`; CLUES2 values from the run directories listed above.

## 5p. CLUES2 CI semantics and cross-ARG repeatability (2026-10-29)

**What the CI is:** `inference.py` fits a multivariate normal to the importance-sampling weighted likelihood surface over (s1,s2,s3) and writes `mu_i ± z*sqrt(Sigma_ii)` to `_CI.txt`. This is a **marginal normal-approximation interval**, not a profile-likelihood CI. It is conditional on the ARG, times, popFreq and grid; it ignores between-epoch correlation and ARG uncertainty. A marginal CI crossing zero therefore does **not** mean "s is not significantly different from zero" in a frequentist sense, and can coexist with a high joint logLR (ridge/trade-off).

**Per-epoch CIs across ARGs (E2 shown; E1/E3 in the CI files):**

| signal | ARG/cohort | logLR | E2 95% CI | crosses 0? |
|---|---|---|---|---|
| UHRF1 | NCN-only | 148.2 | [−0.048,+0.073] | yes (E1/E3 too) |
| UHRF1 | ALL full | 443.7 | [−0.056,+0.060] | yes (E1/E3 too) |
| PLIN3 | NCN-only | 32.4 | [+0.015,+0.019] | no |
| PLIN3 | SCN | 110.2 | [+0.0097,+0.0100] | no |
| PLIN3 | freq-matched | 240.4 | [+0.024,+0.028] | no |
| PLIN3 | NCN-subtree | 1.2 | [−0.014,+0.011] | yes |
| PLIN3 | ALL full | 196.4 | [−0.186,+0.199] | yes |
| DNM2 | NCN-only | 293.3 | [+0.015,+0.027] | no |
| DNM2 | SCN | 131.0 | [+0.014,+0.027] | no |
| DNM2 | ALL full | 996.5 | [+0.025,+0.038] | no |
| DNM2 | freq-matched | 820.9 | [+0.020,+0.020] | no |
| DNM2 | NCN-subtree | 69.4 | [−0.011,+0.112] | yes |
| ARRDC5(−) | NCN-only | 105.7 | [−0.047,−0.035] | no (negative) |
| ARRDC5(−) | ALL corrected | 680.7 | [+0.0013,+0.0053] | no (positive) |
| ARRDC5(−) | freq-matched | 656.8 | [−0.050,+0.076] | yes |
| ARRDC5(+) | NCN-only | 3.8 | [+0.0010,+0.0163] | no |
| ARRDC5(+) | ALL corrected | 16.9 | [+0.0013,+0.0132] | no |
| MIR7-3 | NCN-only / ALL / freq-matched | 65.8 / 519.0 / 315.1 | [+0.013,+0.025] / [+0.013,+0.109] / [+0.018,+0.022] | no |
| DPP9 | NCN-only / ALL | 66.8 / 488.0 | [+0.024,+0.035] / [+0.021,+0.025] | no |

**Repeatability verdict:**
- **UHRF1: stable cross-zero.** All three epochs cross zero in both NCN-only and ALL ARGs (CI widths large), while joint logLR = 148/444. The stable statement is "marginal epoch s is not identifiable", not "no selection".
- **PLIN3: not stable.** Tight, significant positive E2 in NCN-only, SCN and freq-matched; wide/zero-crossing in NCN-subtree (logLR 1.2) and ALL full (logLR 196, CI ±0.2). The behaviour is ARG-dependent; combined with no EHH extension, the signal is withdrawn.
- **DNM2: mostly stable positive.** E2 excludes zero in NCN-only, SCN, ALL and freq-matched; only the NCN-subtree ARG gives a wide zero-crossing CI (logLR 69). Positive sign is robust; magnitude is not.
- **ARRDC5(−): sign is not stable**, not "stably zero": NCN negative significant, ALL positive significant, freq-matched wide and zero-crossing. Near-fixed frequency (0.94–0.955) makes the sign non-identifiable.
- **ARRDC5(+): stable weak positive** (NCN and ALL narrow CIs exclude zero; E1/E3 wide).
- **MIR7-3 / DPP9: stable positive E2** across ARGs.
- **Subgroup flat CIs** (SCN_nonSLE PLIN3/DNM2 logLR 3.5/2.0; Q0/Q1 logLR 0.4–3.5) reflect power, not a sign reversal.

**How to report a stable cross-zero signal:** if joint logLR is high but all marginal CIs cross zero (UHRF1), report the joint test and the p(t) trajectory, state that s(t) is not localizable (ridge, or selection outside the 0–50/50–200/200–600 bins), and do not report a point s. If logLR is low and CIs cross zero (PLIN3 NCN-subtree), report "no evidence in this ARG". Wide CI + high logLR (PLIN3 ALL full, ARRDC5− freq-matched) is an unconstrained marginal, not evidence against the tight subgroup estimates.

**NCN vs SCN for ARRDC5(−):** not previously run (SCN CLUES2 existed only for PLIN3/DNM2). Available SCN evidence: derived frequency 0.947 (NCN 0.955, ALL 0.940) and SCN EHH ΔiHH = −12.1 (NCN −10.8), i.e. the SCN haplotype sign is also negative. An SCN-subtree CLUES2 run (ALL ARG, Gravel EAS, corrected derived=T `.mut`) has been launched (no-CI) and will be appended when it finishes.

## 6. Status / next

Items 1–5 of the previous plan are done (ARRDC5 flip fix; ALL clade independence; PLIN3 diagnosis + withdrawal; MIR7-3/DNM2 frequency matching; all posteriors). Current status by signal:

| Signal | Evidence | Verdict |
|---|---|---|
| MIR7-3 | F_ST 99.6th; PC1 98.7; freq-matched +0.0197; no shared clade; carrier clade shared HK/JiNing 95–100%; clade age 396 gen | **Most robust selection signal** (independent; structure caveat) |
| DNM2 | F_ST 99.7th; PC1 96.3; freq-matched +0.0201; clade age 693 gen; shared HK/JiNing 100/84%; **EHH half-life 32–34 kb in Q0/Q1/SCN (reproducible), 16 kb NCN; CLUES2 s2 batch-unstable** | **Modest extended haplotype, reproducible across HK batches; s2 estimate unreliable** |
| DPP9 | CLUES2 +0.029 (strongest), no PC1 cline (62nd), clade age 1,283 gen, shared 100/100 | Candidate; ARG-dependent |
| UHRF1 | p(t) trajectory clean, s CI crosses 0 in all epochs, clade age 511 gen, shared 100/98 | Trajectory only, timing unresolved |
| PLIN3 | NCN ARG artifact (+0.017 → −0.002 in subtree); HK CLUES2 batch test fails; **EHH half-life 4–5 kb in all groups (no haplotype extension)** | **Withdrawn** |
| ARRDC5(−) | sign flip, near-fixed (DAF 0.94), clade age 44,245 gen | Not interpretable |
| ARRDC5(+) | corrected +0.0073 [0.0013,0.0132], clade age 4,500 gen, shared 100/99 | Weak, concordant |
| ACO2 | SDS 6.07, 136 kb r²≥0.8 block, enhancer intron, 1KG cline, iHS −0.57, **NCN–SCN F_ST 5.5%** | **Clinal/standing candidate; selection not established** |
| OR4X2 | weak LD (2 SNPs r²≥0.8), iHS −2.91, 1KG cline, **NCN–SCN F_ST 7.8%** | **Clinal/drift, not a sweep** |

**Consolidated table:** `SDSworkspace/results/production/orthogonal/relate_clues2/ALL_gravel/signal_robustness_summary.tsv` (15 signals × SDS, iHS, F_ST, PC1 local percentile, CLUES2 NCN/ALL/freq-matched/NCN-subtree, clade age, region sharing, batch EHH, verdict), built reproducibly by `SDSpipeline/scripts/orthogonal/make_signal_robustness_summary.py`. Key contrast: the known-selection controls ADH1B/ALDH2 have PC1 local percentiles 0.0/21.1 (no cline), while ACO2/OR4X2 are 97.2/100 and F_ST 5.5%/7.8% — their SDS signals are structure-first. MIR7-3/DNM2 combine F_ST 1.4%/1.7%, PC1 98.7/96.3, stable freq-matched s≈0.02, no shared sweep, and a carrier clade shared across HK/JiNing; **MIR7-3 is the most robust**; **DNM2 has a reproducible 32–34 kb extended haplotype in Q0/Q1/SCN (EHH, ARG-free) but an unstable CLUES2 s2** (16 kb in NCN); **PLIN3 has no EHH extension at all (4–5 kb)** (caveat: Relate's coalescent does not model structure).

Remaining optional follow-ups: (a) per-batch SDS/iHS/EHH — **EHH done (5n)**; (b) ACO2 eQTL/luciferase follow-up; (c) OR4X2 CNV/mapping check. None of these change the current conclusions.
