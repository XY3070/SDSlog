# Phasing Complete (chr1/4) + Relate ALL chr19 v5 Failure Diagnosis + chr4 Catalog Registration

Date: 2026-08-09
Status: IN PROGRESS — chr4/1 phased + validated; chr3 resubmitted; Relate resumed via per-section array; iHS/EHH launched
Role: mainline (phasing/orthogonal infrastructure)
Sample policy: ALL (4383); Input lineage: joint-call v2 QC → SHAPEIT5 phase_common (CHB pyrho)

## 1. Phasing results

| chr | Status | Size | Validation |
|---|---|---|---|
| chr4 | ✅ DONE (Aug 8 21:40) | 1.65 GB | AN=8766 (4383) ✓ GT-only ✓ chr-prefixed ✓ 13,286,784 variants ✓ **ADH1B lead chr4:99318162:T:C present (AC=2632/8766, DAF 0.3002)** ✓ |
| chr1 | ✅ DONE (Aug 8 22:16) | 1.81 GB | file complete; validation (variant count) streaming |
| chr3 | ❌ KILLED — 48h walltime at Finalization (HAP solving) | — | phase_common.log shows main iteration done; resubmitted **504425** with `-W 96:00` |

chr3 root cause: `submit_phase_all_genome.sh` uses `-W 48:00`; chr3 (largest autosome, ~248Mb) takes ~48h. chr1/4 each took ~47h.

## 2. Relate ALL chr19 v5 — failure diagnosis (job 504350)

**Exited with exit code 1** at 25.4h wall (Aug 8 17:08), silent (empty lsf err, sec log truncated).

- **231 BuildTopology sections** (vs NCN v7's 43!) — ALL has 8766 haps; MakeChunks with `--memory 64` sizes chunks by distance matrix (n_haps²) → ~5× smaller chunks/sections (~254 kb each)
- Section cost: ~4-11h CPU each, **~80 GB RAM each** (8766 haps × matrix; NCN was 33 GB)
- v5 ran 2-way parallelism (`--threads 2`) → 3/231 sections in 25h → **would take ~2 months. Run was doomed, not just unlucky.**
- Progress at death: sections 0-2 complete (.anc+.mut), 3 partial, 4-230 never started. Paint stage complete (231 relate_*.bin).

## 3. Relate resume — per-section LSF jobs (pinning, one per node)

Sections are independent → each BuildTopology = its own LSF job. **Memory finding: per-section peak ~82 GB is intrinsic** — `--memory 8` does NOT bound it (memtest with 12/30 GB caps died instantly); all c-nodes have ~150-176 GB RAM (the "800G" reading was an lsload column artifact) → **exactly 1 section per node**.

Failed attempts (all OOM/bad_alloc from co-placement on shared nodes):
- 504432: array `relate_bt[3-230]%30`, `-n 1 -M 90000` → co-placed 2+/node, ~20 killed
- 504433/504435: + `-R rusage[mem=90000]` → not honored by scheduler; still co-placed (c001 took 8 jobs) → deaths
- select[mem>90000] → mem load index not enforced at dispatch

Working solution — **supervisor with host pinning** (`SDSworkspace/tmp/relate_supervisor.sh`, runs as LSF job 504489 `relate_sup`):
- loop: lsload free-mem >120 GB hosts → skip hosts already running a relate_bt job → `bsub -m <host> -M 100000` one section → refill every 60 s
- worker `relate_resume_section.sh <S>`: `Relate --mode BuildTopology --chunk_index 0 --first_section S --last_section S --painting 0.001,1 -N 100000 -o UKBQC_ALL_chr19` (matches RelateParallel.sh), skips sections with complete .anc
- status at 14:00: 13 RUN (1/node: c001,c005,c009,c015,c023,c026,c028,c029,c030,c036...), ~20 PEND, **0 deaths since 13:52**
- ETA: ~228 × ~5 h / ~20 concurrent ≈ **~2.5-3 days**
- After BuildTopology: FindEquivalentBranches + InferBranchLengths (per section), then RelateSelection (--mode SDS + --mode Selection) → mirror Manhattan

Note: sections submitted before 13:48 died from the earlier arrays — their partial sec*.log/.running markers remain; the worker re-runs them (marker-based skip only honors complete .anc).

## 4. chr4 catalog registration (clump method, ALL LD)

- Reran `clump_signals.py` chr4 (now ALL phased VCF, `ld_source=ALL` — no more 9k fallback): 318 analysis SNPs, **chr4_99240kb_sig1: lead chr4:99318162:T:C (rs1229984), p=2.84e-12, 231 SNPs, 21 sig**
- 2 minor chr4 clumps (99335841, 99364928) non-strict → excluded
- `build_clump_catalog.py` now computes fresh EHH metrics (was carried-only); chr4 EHH: **der@100kb=0.009, class no_extension**
- **signals_clump/all_signals_catalog.tsv = 16 signals complete** (chr4 + 15), all on ALL LD

## 5. Launched (ALL cohort)

| Job | Task |
|---|---|
| 504429 | EHH chr4 ADH1B (rs1229984) ✅ DONE 106s — **15/15 signals now have EHH** |
| 504430-431 | iHS chr1 + chr4 (selscan, ALL phased) |
| 504425 | chr3 phasing (96h walltime) |
| 504432 | Relate BuildTopology sections 3-230 (array %30) |

## 6. Notable findings

- **ADH1B EHH (ALL): no_extension** — derEHH 0.009 at 100 kb (half-life ~100 kb). Classic EAS sweep but no extended haplotype in ALL at DAF 0.30. Cross-check pending: iHS chr4 (504431). Compare ALDH2 (der@500kb=0.45, hard_sweep_extended).
- EHH pending for 3 new leads: chr19_10690kb_sig2/sig3 (10515885, 10555015), chr19_4885kb_sig8 (5023601) — to be filled in the annotation batch.

## Files

- `SDSworkspace/results/production/sds_output/ALL/signals_clump/all_signals_catalog.tsv` (16 signals)
- `SDSworkspace/tmp/relate_resume_section.sh` + `results/production/orthogonal/relate_clues2/ALL/run_work_chr19/bt_*.lsf.*`
- EHH chr4: `results/production/orthogonal/ehh/ALL/chr4_99240kb_sig1/ehh.ehh.99318162.out`

## 2026-08-09 evening updates

### iHS
- **chr1 + chr4 DONE** (jobs 504430/431). chr4 ADH1B lead chr4:99318162:T:C: **norm_iHS = -2.46** (raw -0.57, DAF 0.300) — moderate negative (ancestral-haplotype extension), consistent with weak EHH at this locus
- Genome-wide: 20/22 chr (chr2,5-22 + chr1 + chr4); chr3 pending phasing; chr19 RERUN (504724) for the AC-twin fix
- **Dedup bug found + fixed**: chr19:4867638 multiallelic twins — old `!seen` kept `T:C` (AC=0, monomorphic in ALL) and dropped the signal allele `T:G` (AC=8111) → selscan dropped the position → iHS missing for chr19_4885kb_sig3. Fix: dedup keeps the max-AC record per position + `sort -k2,2n` (first attempt failed: awk hash-order output unsorted → tabix index error). Rerunning chr19 (~1h). After rerun: catalog 16/16 iHS.

### Relate — fat nodes (smp queue)
- **s001-s004 are fat nodes**: 128 cores + 5.2-5.8 TB RAM each (lsload mem col reports T units — supervisor unit bug found: "5.8T"+0=5.8 < 200 threshold; fixed with unit-normalizing awk)
- supervisor v2 (`relate_supervisor.sh`): state-file-based host tracking (v1 BUSY check missed PEND jobs → flooded 210 PEND, killed), c-nodes (normal, ≥120GB free) + s-nodes (smp queue, ≥200GB free), 1 section/node
- status 19:2x: 12 sections done (3+9), 12 RUN (10 c-nodes + s001/s004 on smp), 33 PEND (pinned, serializing)
- ETA with fat nodes: 228 remaining × ~5h / ~30 concurrent ≈ ~40h

### EHH chr7:50266267 (SLE-family locus) — DONE
- `ehh/ALL/chr7_50266kb_sig1/ehh.ehh.50266267.out` (12,187 loci)
- **Fast decay: derEHH@100/250/500kb = 0.0018, half-life ~100 kb, der/anc@500kb = 1.69** — no extended haplotype in ALL at this locus
- Plot: `sds_output/ALL/plots/SLE_family/chr7_50266267_ehh.png` (1440×960, der red / anc blue, 0.5 line, metrics box)
- Plot script: `SDSworkspace/tmp/plot_ehh_single.py` (reusable single-locus EHH plotter)
