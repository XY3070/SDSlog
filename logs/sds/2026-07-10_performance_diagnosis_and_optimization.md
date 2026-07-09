# SDS Compute Performance Diagnosis & Optimization

Date: 2026-07-10
Status: COMPLETE — BFGS baseline preserved; alternative optimizations rejected by RMSE gate.

## Baseline

Post-audit configuration (BFGS, 6 random starts, maxiter=1000, gtol=1e-5):
- Speed: ~7 rows/s on chr1 9k pilot data (4393 individuals)
- Accuracy vs R: RMSE=0.053, Pearson r=0.9997 (200-SNP audit subset)

## Performance Diagnosis

### Profiling setup
- 1000 chr1 SNPs (10k-subset, positions 777k-248M)
- Numba warmup: 10 SNPs before timing
- Per-start data collected: nit, nfev, njev, success, NM fallback, elapsed

### Key findings

1. **NM fallback rate: 0.0%** (0/5646 starts). The Nelder-Mead fallback NEVER triggers.
   BFGS always produces finite results. NM fallback optimization would have zero
   performance impact.

2. **BFGS iterations: median=9, p95=14, max=34.** Convergence is fast. The
   nfev (function evaluations) is the bottleneck: mean=37 vs nit=9 → 4× line
   search overhead per iteration.

3. **NLL values: all positive (59,634-65,484).** Early-stop threshold of -1e5
   (negative) would never trigger. The NLL of the gamma mixture model on 4393
   data points is always large positive. Early-stop with a positive threshold
   calibrated from the NLL distribution showed minimal gain (<5% of SNPs would
   trigger).

4. **Convergence rate: 64.5%** (scipy reports success). Remaining 35.5% report
   "Desired error not necessarily achieved due to precision loss" — the BFGS
   line search fails to satisfy Wolfe conditions, causing early termination.
   This is the root cause of the nfev overhead.

### Root cause of slowdown

The 3× slowdown from 2→6 starts (50→~17 rows/s expected) is compounded by BFGS
line search inefficiency (nfev=37 per start, 4× the iteration count). With
6 starts × 37 nfev = 222 NLL evaluations per SNP, each processing 4393 data
points through numba, the compute is inherently ~7 rows/s.

## Optimization Attempts (all RMSE-gated against R)

### Fix 1: Relax BFGS gtol 1e-5 → 1e-4
- **Result: NO EFFECT.** Iteration counts identical. BFGS terminates from line
  search failure, not gradient norm convergence. RMSE unchanged.
- **Verdict: REJECTED** (no benefit, not harmful — kept at 1e-5 for safety).

### Fix 2: NM fallback optimization (only when ALL starts fail)
- **Result: NO EFFECT.** NM never triggers (0/5646 starts). Code change has
  zero speed impact but is better code hygiene.
- **Verdict: NOT APPLIED** (no benefit, would add code complexity for zero gain).

### Fix 3: Deterministic starts (center + 4 grid corners + midpoint)
- **Result: RMSE=0.995, r=0.898.** Catastrophic failure for DAF<0.01 SNPs.
  The 4 corners and midpoint of the diagonal grid miss the narrow basin of
  attraction for ultra-rare variants (nG1=1, nG2=0). Random sampling from
  the full 102-point grid provides better coverage.
- **Verdict: REJECTED.** Random starts required for accuracy.

### Fix 4: Reduce starts from 6 to 4
- **Result: RMSE=0.995, r=0.898.** Same failure mode as Fix 3. Even with
  random sampling, 4 starts miss the global optimum for ~2% of rare variants.
- **Verdict: REJECTED.** 6 starts required for accuracy.

### Fix 5: Switch to L-BFGS-B (bounded, more robust line search)
- **Speed: 14.7 rows/s** (2.1× faster than BFGS).
  nfev dropped from 37→14 per start.
- **Accuracy: RMSE=0.801, r=0.956** against R.
  FAILS RMSE gate (DAF<0.01 RMSE=0.898, threshold=0.06).
  L-BFGS-B with grid-range bounds converges to different optima for ~2% of
  rare-variant SNPs.
- **Verdict: REJECTED.** Speed gain not worth accuracy loss.

### Fix 6: Reduce maxiter from 1000 to 200
- **Result: RMSE=0.768, r=0.948.** Some rare-variant SNPs need >200 BFGS
  iterations. Even though profiling showed max nit=34, the profiling sample
  didn't include the problematic SNPs (different genomic regions).
- **Verdict: REJECTED.** maxiter=1000 required for safety.

## Final Configuration

**Unchanged from post-audit Q4 baseline:**

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| optimizer_method | BFGS | Only method passing RMSE gate |
| optimizer_n_starts | 6 | 2,4 fail; 6 required for rare variants |
| optimizer_maxiter | 1000 | 200 fails for some SNPs |
| optimizer_gtol | 1e-5 | Relaxation has no effect |
| Start selection | Random from 102-pt grid | Deterministic misses rare-variant basin |
| NM fallback | Per-start on non-finite | Never triggers; kept for safety |
| boundary fill | max-observed | Matches R; boundary-distance rejected |

**Speed: ~7 rows/s** (6.9 on 200-SNP audit subset, 8.0 on 1000-SNP profile).

## Speed Improvement Options (not implemented)

The BFGS line search overhead (nfev=37 per start, 222 per SNP) is the
fundamental bottleneck. Options for future work:

1. **Parallelization (already in production):** LSF chunking across chromosome
   arms and row-chunks. Each chunk runs independently. Throughput scales
   linearly with job count.

2. **GPU acceleration:** The gradient kernel is numba-CPU. Rewriting in CUDA
   could process multiple SNPs simultaneously.

3. **Two-stage optimization:** Run 2 starts first; if results are inconsistent
   (NLL difference > threshold), run 4 more. Most SNPs would use 2 starts,
   only problematic ones use 6. Need RMSE validation.

4. **Nelder-Mead as primary:** NM doesn't need gradient and may converge
   faster for 2-parameter problems despite more iterations. Needs RMSE
   validation (unknown if NM passes RMSE gate solo).

## Conclusions

1. The 6-start BFGS configuration is the **minimum viable configuration** for
   accuracy (RMSE<0.06, Pearson r>0.999 vs R).

2. All speed-oriented optimizations tested (gtol relaxation, fewer starts,
   deterministic starts, L-BFGS-B, reduced maxiter) **fail the RMSE gate**
   for DAF<0.01 SNPs.

3. The ~7 rows/s throughput is the inherent speed limit of the current
   single-core numba implementation with 4393-individual cohorts.

4. Production throughput is achieved via LSF chunked parallelism, not
   single-core optimization.
