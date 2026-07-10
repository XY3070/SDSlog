# SDS Input Pipeline: AA/DA Polarization, Real Observability, Singleton QC

Date: 2026-07-10
Status: COMPLETE — Three critical input pipeline fixes applied.

## Audit Findings

### 1. Singleton outlier sample/region exclusion

**Finding:** Neither `generate_s_file.sh` nor `run_sds_input.sh` excludes singleton
outlier samples or regions. The capability exists in `postfilter_sds_input_qc.py`
(Z-score > 4 on per-sample singleton count; sliding-window Z-score > 4 on singleton
density) but was NOT integrated into the main workflow.

**Assessment:** Necessary. Singleton outlier samples indicate sample quality issues
(contamination, low coverage). Singleton outlier regions flag problematic genomic
segments (segdups, low-complexity) beyond fixed masks. Both are standard practice.

**Fix:** Added `--post-qc` flag to `run_sds_input.sh` and `run_sds_workflow.sh`.
When set, runs `postfilter_sds_input_qc.py` with `--apply-singleton-density-qc`,
`--apply-excess-sample-qc`, `--apply-mask`, and `--observability-mode callrate`
after generating the 4 base input files. Default: OFF (must opt in).

### 2. Real AA/DA annotation

**Finding:** `generate_t_file.sh` writes REF as AA and ALT as DA (`bcftools query
-f '%ID\t%REF\t%ALT\t%POS...'`). This is a placeholder — ~50% of sites have
the ancestral allele on ALT, reversing the SDS sign. The EPO 6-primate alignment
FASTA (`homo_sapiens_ancestor_GRCh38/`) has real ancestral alleles but was unused.

**Assessment:** Critical. Wrong polarization flips rSDS sign for sites where
ALT=ancestral, creating systematic bias in both single-SNP interpretation and
window-level signal aggregation. EPO reference: ~82% of autosomal bases have
high-confidence ancestral calls (chr1: 82.4%), ~5% low-confidence, ~11% no coverage.

**Fix:** Created `input/polarize_ancestral.py`:
- Queries EPO FASTA (via pysam) for ancestral allele at each variant position
- REF == ancestral → keep as-is
- ALT == ancestral → swap AA↔DA AND flip genotypes (0↔2, NA→NA)
- Neither matches (or ancestral is N/dot/gap/lowercase) → DROP marker
- Wired into `run_sds_input.sh` after t_file arm generation, before concatenation
- Graceful fallback: if EPO FASTA missing, warns and uses REF/ALT placeholder
- Verified: 4/4 test cases correct (ref=anc, alt=anc, no-cov, mismatch)

### 3. Real observability

**Finding:** `generate_o_file.sh` writes ALL ONES: `for (i=1; i<=n; i++) printf "1"`.
This treats every sample as having perfect singleton detection probability.
`compute_SDS.py` uses observability to weight singleton intervals:
`intervals = (upstream + downstream) * self.observability`.

**Assessment:** Necessary. Samples with lower genotype call rates have reduced
power to detect singletons. All-ones observability inflates the contribution of
low-quality samples to the composite likelihood.

**Fix:** Rewrote `generate_o_file.sh` to compute call-rate-based observability
from the t_file: `observability = 1.0 - NA_count/total_count` per sample.
- Reads t_file genotype columns, counts NA per sample
- Outputs per-sample call rate as observability (0-1)
- Warns if any sample has observability < 0.5
- Fallback: if t_file not provided, uses all-ones with a WARNING
- `run_sds_input.sh` updated to pass `$T_OUT` as 5th argument

## Files Changed

| File | Change |
|------|--------|
| `scripts/input/polarize_ancestral.py` | NEW — EPO-based AA/DA polarization (152 lines) |
| `scripts/input/generate_o_file.sh` | REWRITTEN — call-rate observability instead of all-ones |
| `scripts/input/run_sds_input.sh` | MODIFIED — EPO setup, polarization step, o_file t_file arg, --post-qc flag |
| `scripts/run_sds_workflow.sh` | MODIFIED — --post-qc flag propagated to input phase |
| `manifests/canonical_scripts.tsv` | UPDATED — new entries, revised descriptions |

## Usage

```bash
# Standard run (AA/DA polarization ON by default if EPO FASTA exists):
bash run_sds_workflow.sh --pop NCN --chromosomes "1-22"

# With singleton outlier QC:
bash run_sds_workflow.sh --pop NCN --chromosomes "1-22" --post-qc

# Dry-run to verify:
bash run_sds_workflow.sh --pop NCN --chromosomes "1-22" --post-qc --dry-run
```

## Verification

- `bash -n`: 0 errors on all active .sh files
- `polarize_ancestral.py`: 4/4 test cases correct (ref=anc, alt=anc+flip, no-cov=drop, mismatch=drop)
- `generate_o_file.sh`: call-rate computation correct (verified against manual counts)
- EPO chr1 FASTA: 82.4% bases high-conf, ~5% low-conf, ~11% no-coverage, <0.1% fail
