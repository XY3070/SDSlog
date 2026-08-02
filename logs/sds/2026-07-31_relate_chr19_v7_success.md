# Relate chr19 v7: First Successful Tree Inference

Date: 2026-07-31
Status: **DONE** — Relate chr19 tree inference complete
Role: mainline — pilot for Relate-based orthogonal validation

## Summary

Relate chr19 v7 completed successfully after 9 failed attempts. This is the **first successful Relate tree inference** in this project.

## Fixes Applied

| # | Problem | Fix |
|---|---------|-----|
| 1 | `Ne: unbound variable` | Added `-N 100000` |
| 2 | `bp_pos[snp] == mbp` assertion | Removed `--dist` flag |
| 3 | Working directory requirement | `cd` into workdir, symlink inputs |
| 4 | BuildTopology OOM (16 threads) | Reduced to `--threads 2` |
| 5 | LSF memory 32MB (!) | Set `-M 256000` (256GB) |

## Key Insight

v6's OOM crash (33GB/section) was caused by using **9466 samples** (full joint-call v2 without filtering). With NCN 1255 samples, BuildTopology needs only ~300MB per section.

## Run Statistics

| Metric | Value |
|--------|-------|
| Job ID | 503210 |
| Runtime | Jul 30 17:02 → Jul 31 13:07 (~20 hours) |
| Exit code | 0 |
| Nodes | 4 × c028 |
| Samples | 1255 NCN (2510 haplotypes) |
| SNPs | 777,789 |
| Sections | 21 (0-20) |
| BuildTopology peak memory | 211-299 MB per section |
| Finalize memory | 5.2 GB (CombineSections + Finalize) |
| Not-mapping SNPs | 43,233 |
| Flipped SNPs | 8,081 |

## Output Files

| File | Size | Description |
|------|------|-------------|
| `UKBQC_NCN_chr19.anc` | 3.7 GB | Ancestral trees with branch lengths |
| `UKBQC_NCN_chr19.mut` | 57 MB | Mutation-to-tree mappings |

## Next Steps — RelateSelection

With completed `.anc` and `.mut` files, can now run:

```
# 1. Relate SDS (direct comparison with our SDS)
RelateSelection --mode SDS \
    -i UKBQC_NCN_chr19 -o NCN_chr19_relate_sds
# DONE — 27MB .SDS output, 728K SNPs, rSDS range: -0.39 to 8.48

# 2. Relate Selection p-values
DetectSelection.sh \
    -i UKBQC_NCN_chr19 -o NCN_chr19_selection -m 1.25e-8
# Running

# 3. CLUES2 for time-resolved selection coefficients
# Script ready: SDSpipeline/scripts/orthogonal/run_clues2_all_chr19.sh
```

## ALL-population Relate (chr19)

- **Phasing**: chr19 phased VCF complete (458MB, 4383 samples, 3.8M SNPs)
- **Haps/Sample**: Converted (29GB haps.gz, 72KB sample.gz)
- **Relate job**: 503507 (4 nodes, 512GB, 72hr, --threads 2)
- **CLUES2 script**: `SDSpipeline/scripts/orthogonal/run_clues2_all_chr19.sh` — run after Relate completes

## 2026-08-02: RelateSelection NCN chr19 results

RelateSelection --mode SDS completed on NCN chr19 trees:
- 728,248 SNPs with rSDS values
- All 7 chr19 SDS signals present in Relate SDS output
- Raw rSDS values (not standardized) — needs frequency bin normalization
- Top Relate rSDS signals cluster at chr19:54.6Mb (different from our SDS at 4.7-10.7Mb)
- RelateSelection --mode Selection running for p-values
- CLUES2 script adapted for ALL chr19: `run_clues2_all_chr19.sh`

## Lesson: Data Scale Matters

The 9 failed attempts were a cascade of:
1. Missing parameters (-N, workdir)
2. Map/data mismatch (--dist)
3. **Sample count mismatch**: v6 used 9466 samples from unfiltered joint-call VCF v2, while v7 correctly used the NCN prepared data (1255 samples). BuildTopology scales with sample count — 9466 samples → ~33GB/section; 1255 samples → ~300MB/section.

For ALL-population Relate (4383 samples): estimate 300MB × (4383/1255)² ≈ 3.7GB per section. With --threads 2, need ~7.5GB peak. This is feasible with -M 64000.
