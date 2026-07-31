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
# 1. Re-estimate branch lengths (optional, uses coalescent rate)
RelateCoalescentRate --mode ReEstimateBranchLengths \
    -i UKBQC_NCN_chr19 -o UKBQC_NCN_chr19_reestimated --coal NCN.coal

# 2. Relate SDS (direct comparison with our SDS)
RelateSelection --mode SDS \
    -i UKBQC_NCN_chr19 -o chr19_relate_sds

# 3. Relate Selection p-values
DetectSelection.sh \
    -i UKBQC_NCN_chr19 -o chr19_selection -m 1.25e-8
```

Then generate mirror Manhattan: our SDS (top) vs Relate SDS/Selection (bottom).

## Lesson: Data Scale Matters

The 9 failed attempts were a cascade of:
1. Missing parameters (-N, workdir)
2. Map/data mismatch (--dist)
3. **Sample count mismatch**: v6 used 9466 samples from unfiltered joint-call VCF v2, while v7 correctly used the NCN prepared data (1255 samples). BuildTopology scales with sample count — 9466 samples → ~33GB/section; 1255 samples → ~300MB/section.

For ALL-population Relate (4383 samples): estimate 300MB × (4383/1255)² ≈ 3.7GB per section. With --threads 2, need ~7.5GB peak. This is feasible with -M 64000.
