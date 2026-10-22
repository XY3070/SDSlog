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

## 6. Next

1. Wait for 567186–567192 (~3.5–4 h); inspect the ALL CLUES2 inference tables.
2. Extract ALL `cluster_full.newick`/`.pos` from `ALL_gravel/intermediate/chr19_4650000_5000000.anc` and run the same clade-preservation test.
3. Compare NCN vs ALL ages/co-occurrence/preservation; if ALL diverges, split SLE/non-SLE.
