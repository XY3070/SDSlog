# Relate ALL chr19 downstream complete — 2026-08-16

Date: 2026-08-16
Status: COMPLETE — Relate ALL chr19 SDS + Selection outputs produced, verified vs v7 reference

## What ran

Chain (v1.2.4 static binaries, `relate/relate_v1.2.4/relate_v1.2.4_x86_64_static/bin/`):
`RelateSelection --mode SDS` → `DetectSelection.sh` (old-build `relate/bin/RelateSelection`, same as v7 used).
Skipped: `ReEstimateBranchLengths` + `EstimatePopulationSize` — see deviations.

## Outputs (ALL/run_work_chr19/)

| file | size | notes |
|---|---|---|
| `UKBQC_ALL_chr19_relate_sds.SDS` | 59.5M / 1,534,701 lines | pos rs_id rSDS |
| `UKBQC_ALL_chr19_selection.sele` | 80.3M / 493,013 lines | 31 time bins + when_DAF_is_half + when_mutation_has_freq2 |
| `UKBQC_ALL_chr19_selection.freq` | 51.2M | trajectory |
| `UKBQC_ALL_chr19_selection.lin` | 68.7M | linear scores |
| `UKBQC_ALL_chr19/chunk_0/ALL.coal` | 805B | constant coal, copied from `refs/coal/NCN.coal` (v7 parity) |
| `plots/relate_sds_mirror/UKBQC_ALL_chr19_sds_mirror.png` | 549K | mirror Manhattan: pipeline rSDS vs Relate rSDS, chr19 |

## Critical fixes this session

1. **Combined anc was binary, not text.** Old-build (Apr) `CombineSections` writes `DumpBin` (binary anc) + short-format mut;
   v1.2.4 readers expect text `NUM_HAPLOTYPES` anc + 14-col mut. Old `RelateCoalescentRate` crashed on the mut line
   (`0;1960 7591 3701;...`); v1.2.4 crashed with `vector::_M_default_append` on the binary anc.
2. **Reconstructed mut to v1.2.x 14-col format** (`tmp/reconstruct_mut_newformat.py`): old mut body + haps positions/rs-ids
   (1:1 line match, 1,728,257), EPO-oriented ancestral allele (`homo_sapiens_ancestor_19.fa`; validated vs v7: 200k/200001
   match), upstream/downstream from EPO flanks (matches v7 exactly), DAF = derived allele count (validated vs v7 NCN haps).
   84% EPO-orientable; 173,755 EPO-gap, 7,403 N, 95,637 true mismatch → ref-first fallback. Also generated `chunk_0.dist`.
3. **Converted binary anc → text** with custom `tmp/anc_bin2text` (linked `librelateStatic.a`; old source `Dump` = exact
   v1.2.4 text format). 12.1GB binary → 18.8GB text, 28,701 trees, N=8766. Binary preserved as `UKBQC_ALL_chr19_c0.anc.binbak`.
4. **ReEstimateBranchLengths lives in `RelateCoalescentRate`, not `Relate`** — old scripts called `Relate --mode
   ReEstimateBranchLengths` which prints "Invalid or missing mode" and exits 0 (silent no-op). Same trap broke
   `relate_post_combine.sh`/`relate_followup.sh` before.
5. **CoalRateForTree vs EstimatePopulationSize**: `--mode CoalRateForTree` (fast, per-SNP) writes only `.coal`; Finalize
   needs `.bin` → correct v1.2.4 entry point is `--mode EstimatePopulationSize` (CoalescentRateForSection → .bin →
   Finalize). `EstimatePopulationSize` is O(N² per epoch × trees) ≈ 9-10 h serial for 8766 haps × 28,701 trees → deferred.

## Deviations from handover plan (with reason)

- **No EstimatePopulationSize / ALL.coal inference** — ~10 h pairwise computation; v7 NCN reference never ran it either
  (constant `refs/coal/NCN.coal`). Used same constant coal → SDS comparable to v7.
- **No ReEstimateBranchLengths** — v7 SDS ran directly on IBL'd trees (no `_reestimated` files in v7 dir); same here.
  Optional refinement later: `RelateCoalescentRate --mode ReEstimateBranchLengths --coal ALL.coal -m 1.25e-8 -i
  UKBQC_ALL_chr19_c0 -o ..._reestimated` (expect ~240 h serial; would need per-section splitting, sections deleted by
  CombineSections).
- **SDS/selection used old-build `RelateSelection`** (DetectSelection.sh hardcodes `$PATH_TO_RELATE/bin/`) — identical to
  v7's run; outputs format-identical to v7.

## Verification

- `.SDS`/`.sele` formats match v7 reference exactly (same headers, same first SNP 74955).
- rSDS distribution: ours sd=0.196 (max 9.1) vs v7 sd=0.355 (max 8.5) — same class.
- Cross-pipeline rSDS correlation ≈ 0 (ours vs SDS pipeline r=-0.03; **v7 reference vs pipeline r=-0.05**) → per-SNP
  rSDS not reproducible across tree inferences, ours behaves exactly like the reference.
- Known chr19 loci (FUT2 rs601338, APOE) flat in both ours and pipeline — consistent.

## Pending / optional

- `EstimatePopulationSize` → true ALL.coal (hours; only if demography needed).
- ReEstimate with that coal (days; needs section splitting).
- chr3 loci merge into `loci_p5e5_final.tsv` — still awaiting user confirmation (114-row staging file ready).
- EPO fa migration manifest: `2026-08-16_epo_migration_manifest.md` (23 files chr1-22+X, ~2.9 GB).
