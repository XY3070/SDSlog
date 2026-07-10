# SDS Pipeline Scripts Reorganization & Workflow Optimization

Date: 2026-07-10
Status: COMPLETE — Scripts reorganized into functional subdirectories; central workflow
        enhanced with auto-detection and cluster-aware skip logic.

## Motivation

`SDSpipeline/scripts/` had 49 files in a flat layout spanning input generation, core
compute, postprocessing, demography, gamma estimation, population structure, orthogonal
methods, diagnostics, and cluster submission. The flat layout made it hard to:
- Identify entrypoints vs helpers
- Understand cross-script dependencies
- Run only needed pipeline stages (manual `--skip-*` flags required)

## Changes

### 1. Subdirectory organization

```
scripts/
  common_env.sh         # INFRA (top-level, sourced by everything)
  sense_cluster.sh      # INFRA (top-level, cluster resource sensing)
  run_sds_workflow.sh   # CENTRAL ORCHESTRATOR
  submit_sds_genomewide.sh  # Genome-wide compute submitter

  input/ (7)            # b/s/t/o file generation
  compute/ (10)         # Core SDS, chunking, merging, LSF submission
  postprocess/ (4)      # Normalization, Manhattan, diagnostics
  demography/ (7)       # SMC++, backward simulation
  gamma/ (4)            # Gamma shape estimation
  population_structure/ (2)  # ADMIXTURE
  orthogonal/ (1)       # Relate reference download
  diagnostics/ (3)      # QC filtering, targeted analysis
  archive/ (5)          # Deprecated scripts
  admin/ (1)            # Bootstrap
```

### 2. Path migration

All scripts moved to subdirectories add:
```bash
SCRIPTS_ROOT="$(cd "$SCRIPT_DIR/.." && pwd)"
PIPELINE_ROOT="$(cd "$SCRIPT_DIR/../.." && pwd)"
```

Cross-subdirectory references use `$SCRIPTS_ROOT/`; same-directory calls use
`$SCRIPT_DIR/` (unchanged). `common_env.sh` exports `SDS_SCRIPTS_ROOT` for
global availability.

### 3. Central workflow auto-detection

`run_sds_workflow.sh` now auto-detects which phases need running:

- **Input**: checks `chr{N}_{b,s,t,o}_file.txt` all exist → skip
- **Compute**: checks `chr{N}.sds.tsv` (or `chr{N}_{p,q}.sds.tsv`) → skip that chr
- **Postprocess**: checks `{pop}.normalized.tsv` + `significant_regions.tsv` → skip

New flags:
- `--force-{input,compute,postprocess}`: re-run even if complete
- `--skip-{input,compute,postprocess}`: skip even if pending (unchanged)
- `--dry-run`: print what would execute, exit
- `--help`: print usage

Default behavior (no flags): auto-detect and run only pending phases.
Compute phase automatically filters chromosome list to only those without
existing SDS output before calling `submit_sds_genomewide.sh`.

### 4. Archived scripts

- `compute_SDS.R`: our fixed version (logsum R>=4.2 fix, NA-genotype handling)
  replaces the old buggy copy in `archive_sds/legacy_scripts/`
- `generate_all_inputs.sh`: superseded by workflow Phase 1
- `run_sds_compute.sh`: superseded by `submit_sds_compute_chunked_chr.sh`
- `plot_ancestry_one_figure.py`, `plot_chr_mirror_compare.py`: single-use scripts

### 5. Cluster-aware resource allocation

Already in place via `submit_sds_genomewide.sh` → `sense_cluster.sh` chain.
Workflow passes `--queue` and `--max-concurrent-chrs` through; genome-wide
submitter re-senses cluster state between waves and auto-derives chunk
parameters (chunk_rows, numba_threads, array_parallel) from queue type and
free slots.

## Verification

- `bash -n`: 0 errors on all 31 active .sh files
- `run_sds_workflow.sh --dry-run`: correctly reports pending phases
- `canonical_scripts.tsv`: all 52 paths resolve to existing files
- `git mv`: preserves file history (`git log --follow` works)
- No dangling references to old flat paths found in active scripts

## Commits

- SDSpipeline: `7d93df4` — refactor(sds): reorganize scripts into functional
  subdirectories with auto-detection workflow
- SDSlog: this entry
