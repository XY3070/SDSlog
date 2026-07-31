# Phasing Gap and Relate Debugging Roadmap

Date: 2026-07-30
Status: DIAGNOSED — phasing gap identified, Relate failure root cause found
Role: audit-only — documents infrastructure gaps and debugging findings

## 1. Phasing Gap: NCN-only → Need ALL

### Discovery

iHS genome-wide scan and Relate chr19 pilot both used **NCN-only** (1255 samples) SHAPEIT5-phased VCFs:
- iHS VCF: `SDSworkspace/input/raw/vcf/NCN/shapeit5/UKBQC_NCN_chr{N}.phased.vcf.gz`
- Relate chr19 source: converted from same NCN phased VCF

**No ALL-population (4383 samples) phased VCF exists.** The project has:
- NCN phased: 22 chr (BCF + VCF.gz), 1255 samples
- SCN phased: 22 chr, 2716 samples  
- ALL: NOT phased

### Impact

All orthogonal methods (iHS, EHH, Relate) used NCN-only data. For paper-facing results:
- iHS should be re-run on ALL-phased VCF (4383 samples → more power for rare variants)
- Relate should use ALL-phased VCF (larger sample size improves tree inference)
- EHH at lead SNPs should use ALL-phased VCF

### Phasing Resources Available

| Resource | Path |
|----------|------|
| SHAPEIT5 phase_common | `shapeit/bin/phase_common_static` |
| SHAPEIT5 ligate | `shapeit/bin/ligate_static` |
| CHB pyrho map (chr19) | `shapeit/share/shapeit5/maps/pyrho_hg38/CHB/CHB_recombination_map_hapmap_format_hg38_chr_19.txt` |
| CHS pyrho map (chr19) | `shapeit/share/shapeit5/maps/pyrho_hg38/CHS/CHS_recombination_map_hapmap_format_hg38_chr_19.txt` |
| Generic b38 map | `shapeit/share/shapeit5/maps/b38/chr19.b38.gmap.gz` |

### Population Choice

ALL cohort = NCN (North Chinese) + SCN (South Chinese). Use **CHB** pyrho map for phasing. CHB and CHS maps differ measurably even on same chromosome (chr19 first cM positions: CHB=80848 vs CHS=69983). CHB is the larger reference panel and better matches the Northern Chinese majority in the cohort.

### Phasing Pipeline

```
1. Locate raw joint-called ALL VCF (all samples, biallelic SNPs)
2. Filter: PASS + biallelic SNPs + MAF > 0.01
3. SHAPEIT5 phase_common_static --map CHB pyrho
4. Output: SDSworkspace/input/raw/vcf/ALL/shapeit5/UKBQC_ALL_chr{N}.phased.vcf.gz
5. Pilot chr19 first → verify → extend to all 22 chr
```

---

## 2. Relate chr19 Failure: 9-Attempt Debugging History

### Timeline

| # | Job ID | Date | Error | Stage | Root Cause |
|---|--------|------|-------|-------|-----------|
| 1 | 502945 | Jul 26 23:02 | Output needs to be in working directory | Startup | Absolute paths used |
| 2 | 502946 | Jul 26 23:02 | Same | Startup | Still absolute paths |
| 3 | 502947 | Jul 26 23:04 | bp_pos[snp] == mbp assertion | MakeChunks | Wrong map file (chr19.map vs chr19_relate.map) |
| 4 | 502948 | Jul 26 23:14 | Same assertion | MakeChunks | --dist flag + map/data position mismatch |
| 5 | 502949 | Jul 26 23:18 | Ne: unbound variable | BuildTopology | -N parameter not passed to RelateParallel |
| 6 | 503040 | Jul 28 10:20 | Output needs to be in working directory | Startup | Absolute paths again |
| 7 | 503067 | Jul 28 13:29 | bp_pos[snp] == mbp | MakeChunks | --dist re-added with mismatched map |
| 8 | 503068 | Jul 28 13:32 | Killed by owner | Painting | Manually killed |
| 9 | 503069 | Jul 28 13:42 | **OOM + assertion pfile** | **BuildTopology** | 16 threads × 33GB/section = need ~528GB; LSF -M 32000 (32MB) |

### Root Cause #1: BuildTopology OOM (Attempt 9)

Each BuildTopology section requires ~33GB RAM (confirmed from completed sections 37,39,40,41,42 which each reported 33.6-33.9 GB).

With `--threads 16`, RelateParallel spawns 16 concurrent BuildTopology processes.
43 sections / 16 threads ≈ 3 batches.
Peak memory demand: 16 × 33GB = 528GB.

**LSF requested `-M 32000`** — this is 32 MB, not 32 GB! The actual system limit caused 38/43 sections to be killed by OOM killer. The 5 surviving sections produced .anc files but the missing sections (0-36, 38) caused FindEquivalentBranches to crash with `Assertion 'pfile' failed` when trying to read section 38's .anc file.

### Root Cause #2: Map/Data Position Mismatch (Attempts 3,4,7)

Genetic map (`chr19_relate.map`) covers positions 256,859-58,585,793.
Haps data covers positions 68,565-58,592,630.
188 kb gap at start, 6.8 kb gap at end.

When `--dist` is passed, MakeChunks enforces `bp_pos[snp] == mbp` (exact position match between data and map boundaries). The first data SNP (68,565) is before the first map entry (256,859), causing assertion failure.

**Fix**: Do NOT use `--dist` flag. Relate will interpolate distances from the map internally.

### Root Cause #3: Missing `-N` (Attempts 1-5)

Original `run_relate_chr19.sh` line 61 calls `RelateParallel.sh` without `-N`. RelateParallel.sh line 243 uses `$Ne` which is unbound (set -u mode) → crash.

**Fix**: Always pass `-N 100000` (recent Ne(0) for Chinese populations).

### Corrected Relate Command

```bash
cd $WORKDIR
RelateParallel.sh \
    --mode All \
    --haps UKBQC_ALL_chr19.haps.gz \
    --sample UKBQC_ALL_chr19.sample.gz \
    --map chr19_relate.map \
    --annot UKBQC_ALL_chr19.annot \
    --coal ALL.coal \
    -N 100000 \
    -m 1.25e-8 \
    --memory 64 \
    --threads 2 \
    -o UKBQC_ALL_chr19
```

Key fixes:
- `--threads 2` (limit BuildTopology parallelism, max ~66GB needed)
- `--memory 64` (64GB allocation parameter)
- `-N 100000` (mandatory)
- NO `--dist` flag
- LSF: `-M 128000` (128GB, safe margin)

---

## 3. RelateSelection Capabilities

### Two Modes for SDS Comparison

| Mode | Output | Mechanism | Comparison with Our SDS |
|------|--------|-----------|------------------------|
| `--mode SDS` | `.SDS` (pos, rs_id, rSDS) | Computes `rSDS = log((aSDS/dSDS) * derived_count/(N-derived_count))` from tree branch lengths | **Direct comparison** — both compute rSDS, but from different inputs (trees vs singleton density) |
| `--mode Selection` | `.sele` (pos, rs_id, pvalue per epoch) | Branch-length test: how unexpectedly fast did allele rise? | **Orthogonal** — p-value from tree-based frequency trajectory vs our SDS z-score |

### Pipeline (after Relate trees complete)

```
RelateCoalescentRate --mode ReEstimateBranchLengths \
    -i UKBQC_ALL_chr19 -o UKBQC_ALL_chr19_reestimated --coal ALL.coal

RelateSelection --mode SDS \
    -i UKBQC_ALL_chr19_reestimated -o chr19_relate_sds

bash DetectSelection.sh \
    -i UKBQC_ALL_chr19_reestimated -o chr19_selection -m 1.25e-8
```

Output:
- `chr19_relate_sds.SDS`: per-SNP rSDS values
- `chr19_selection.sele`: per-SNP selection p-values

### Mirror Manhattan Comparison

Adapt `SDSworkspace/runs/chr1_qc_control_20260606/postprocess_chr1norm/baseline/mirrored_manhattan.py`:
- Top panel: our SDS (ALL chr19, existing output)
- Bottom panel: Relate SDS `.SDS` rSDS values, normalized by DAF bin

---

## 4. EHH Status

14/15 signals successful after fixing VCF format (GT-only, dedup positions, integer POS locus).
ADH1B failed (rs1229984 filtered from NCN phased VCF during SHAPEIT5 QC).

EHH decay patterns: all 14 signals show strong derived allele EHH persistence across 500kb window. Derived allele EHH never drops below 0.5 within the window for any signal — characteristic of recent hard sweeps.

---

## 5. Implementation Log (2026-07-30)

### Phasing
- **Source**: Joint-call v2 QC VCFs at `/data/home/grp-wangyf/xuyuan/raw/9k/all.chr{N}.QC.vcf.gz` (9466 samples, trio-based)
- **Cohort**: Analysis set = 4383 frozen samples (`raw/9k/sample_qc/frozen_4383.txt`)
- **Map**: CHB pyrho (`shapeit/share/shapeit5/maps/pyrho_hg38/CHB/`)
- **Method**: SHAPEIT5 phase_common v5.1.1 (commit 990ed0d, 2023-05-08)
- **chr1**: Already phased (5.9GB, 9466 samples) — replicates approach
- **chr19 pilot**: Job 503213 (16 threads, 64GB, 48hr)
- **Index fixes**: Jobs 503214-503224 (11 chr missing tabix indices)
- **Scripts**:
  - `SDSpipeline/scripts/preprocessing/phase_single_chr.sh` — single-chromosome worker
  - `SDSpipeline/scripts/preprocessing/submit_phase_all_genome.sh` — genome-wide submitter

### Relate chr19 v7
- **Job**: 503210 (4 nodes, 256GB, 48hr, --threads 2)
- **Fixes**: Reduced parallelism (2 threads vs 16), proper LSF memory (256GB vs 32MB)
- **Status**: Running (Paint stage as of 17:04)

### Next
1. Monitor chr19 phasing (503213) → verify output
2. Submit genome-wide phasing (chr2-18,20-22) after chr19 pilot confirms
3. Monitor Relate v7 (503210) → if succeeds, run RelateSelection
4. Re-run iHS on ALL-phased chr19 for comparison with NCN-only results
