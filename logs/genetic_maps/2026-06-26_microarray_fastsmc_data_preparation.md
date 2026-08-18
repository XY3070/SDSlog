# Microarray Data Preparation for FastSMC (Multi-Population)

**Date**: 2026-06-24 ~ 2026-06-26
**Status**: Complete (110/110 files)

## Motivation

Prepare CHN100k-imputed microarray data from 5 populations for FastSMC IBD-based selection scan. Reference workflow: Science Advances (doi:10.1126/sciadv.adi8419). The data had imputation-based phasing from minimac4 but no proper statistical phasing or QC.

## Populations & Samples

| Population | N samples | Genetic Map | Rationale |
|-----------|-----------|-------------|-----------|
| CC (华中)  | 5,136 | CHB (Han Chinese Beijing) | Northern/Central Han |
| GZ (广州)  | 2,589 | CHS (Southern Han Chinese) | Southern Han |
| HK (香港)  | 4,928 | CHS (Southern Han Chinese) | Southern Han |
| JN (济南)  | 1,506 | CHB (Han Chinese Beijing) | Northern Han |
| Thai       | 3,830 | KHV (Kinh Vietnamese) | Southeast Asian, closest available |

## Pipeline

### Step 1: QC Filtering

**Criteria:**
- Imputation R² ≥ 0.3
- MAF ≥ 0.01
- HWE p ≥ 1e-6
- Genotype missingness ≤ 5%
- Biallelic SNPs only

**Method:** `bcftools view` for R²/MAF/biallelic, then `plink2 --hwe --geno --make-just-pvar` to identify failing sites, then `bcftools view --targets-file` to filter while preserving phased genotypes from imputation.

**Special cases:**
- CC chr4: Original `_final.vcf.gz` was corrupted (binary content). Used pgen/pvar directly — extracted R²/MAF from pvar INFO field with awk, exported passing variants from pgen with plink2, then applied HWE filter.
- JN chr10: VCF file was missing. Exported from pgen with `plink2 --export vcf bgz` (preserves phasing); INFO/R² was preserved in pvar so standard QC worked.
- CC chr4 required `bcftools +fill-tags -- -t AC,AN` before SHAPEIT5 (pgen export doesn't produce AC/AN).

### Step 2: Phasing (SHAPEIT5)

**Tool:** `phase_common_static` v5.1.1 (2023-05-08)

**Parameters:**
- `--filter-snp`
- `--thread 8`
- Default MCMC: `5b,1p,1b,1p,1b,1p,5m`
- Default PBWT: window=4cM, depth=auto, mac=5, missing=0.1
- Default HMM: window=4cM, Ne=15000

**Genetic maps:** Population-specific recombination maps from **popgenmethod** (Pyrho-based, hg38), bundled with SHAPEIT5. These are NOT the original HapMap genetic maps — they are population-specific estimates (CHB, CHS, KHV) with higher resolution. Source: `/data/home/grp-wangyf/xuyuan/shapeit/share/shapeit5/maps/hg38/{CHB,CHS,KHV}/`. Converted to 3-column format (pos, rate, cM — no header) required by phase_common.

**Output:** `.bcf` format (SHAPEIT5 requirement; `.vcf.gz` extension not recognized by this version).

### Step 3: Exclusion Mask

**Sources:**
- Centromeres: UCSC hg38 `centromeres` table (mysql genome-mysql.soe.ucsc.edu), merged with 1kb gap tolerance
- Positive selection regions (hg38 coordinates):

| Region | Coordinates | Gene/Locus |
|--------|-------------|------------|
| chr2:108-110M | EDAR | East Asian skin/hair |
| chr2:135-137M | LCT | Lactase persistence |
| chr2:72-74M | EXOC6B region | |
| chr4:41-43M | SLC30A9 region | |
| chr6:28-34M | HLA/MHC | Immune diversity |
| chr11:88-89.5M | TYR | Pigmentation |
| chr14:92-93.5M | SLC24A4 | Pigmentation |
| chr15:48-49.5M | SLC24A5/SLC12A1 | Pigmentation |
| chr15:28-29.5M | HERC2/OCA2 | Eye color |
| chr16:30-31.5M | ALDH2 region | Alcohol metabolism |
| chr4:99-101M | ADH1B | Alcohol metabolism |
| chr1:150-152M | THEM4/S100A | |
| chr5:33-34.5M | SLC45A2 | Pigmentation |
| chr20:33-34.5M | EDN3 | |
| chr10:94-96M | ABCC2 | |

**Result:** 57 merged regions, 86.0 Mb total masked.
**File:** `/data/home/grp-wangyf/xuyuan/fastsmc_prep/masks/exclusion_regions.bed`

### Step 4: Conversion to Oxford haps/sample

**Method:** `bcftools view --targets-file ^mask.bed | bcftools convert --hapsample`

**Post-processing:** Strip `*` phase uncertainty markers from haps output (`sed 's/\*//g'`).

**Verification (GZ chr6):** 427,496 QC variants → 422,296 after mask (HLA+centromere removed 5,200 variants).

## Output Structure

```
/data/home/grp-wangyf/xuyuan/fastsmc_prep/fastsmc_input/
├── {POP}_chr{N}.haps.gz         # Phased haplotypes, masked
├── {POP}_chr{N}.samples         # Sample IDs (Oxford format)
└── maps/{POP}/genetic_map_chr{N}.txt  # pos(bp) rate(cM/Mb) cM
```

**Total size:** ~30 GB

**FastSMC genetic map format:** 3-column tab-separated (physical_position, rate, genetic_position), no header. Derived from popgenmethod (Pyrho) population-specific maps, NOT HapMap. Matches FastSMC requirement of "physical positions in the first column and genetic positions in the third column."

## Key Paths

| Item | Path |
|------|------|
| QC VCFs | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/qc/{POP}_chr{N}_qc.vcf.gz` |
| Phased BCFs | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/phased/{POP}_chr{N}_phased.bcf` |
| FastSMC haps | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/fastsmc_input/{POP}_chr{N}.haps.gz` |
| FastSMC samples | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/fastsmc_input/{POP}_chr{N}.samples` |
| FastSMC maps | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/fastsmc_input/maps/{POP}/genetic_map_chr{N}.txt` |
| Exclusion mask | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/masks/exclusion_regions.bed` |
| SHAPEIT5 maps | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/shapeit_maps/{CHB,CHS,KHV}/chr{N}.txt` |
| Pipeline scripts | `/data/home/grp-wangyf/xuyuan/fastsmc_prep/scripts/` |

## Scripts

| Script | Purpose |
|--------|---------|
| `01_qc_filter.sh` | Sequential QC (standalone) |
| `02_phase.sh` | Sequential phasing (standalone) |
| `03_create_masks.sh` | Generate exclusion BED from UCSC + selection regions |
| `04_convert_haps.sh` | Convert phased BCF to haps/sample with mask |
| `05_prepare_maps.sh` | Convert 4-col SHAPEIT maps to 3-col FastSMC format |
| `submit_01_qc.sh` | LSF batch submission for QC |
| `submit_02_phase.sh` | LSF batch submission for phasing |
| `submit_03_convert.sh` | LSF batch submission for conversion |
| `submit_pipeline.sh` | Master submission (all three steps chained) |
| `fix_cc_chr4_qc.sh` | Special pgen-based QC for corrupted CC chr4 |
| `verify_outputs.sh` | Post-completion verification |

## Notes

- Input VCFs were from CHN100k imputation server (minimac4 with CHN100K reference panel). Phasing from imputation (`0|0` format) was used as initialization for SHAPEIT5.
- Two input files were problematic: CC chr4 (corrupted download, binary content in .vcf.gz) and JN chr10 (missing VCF). Both resolved by falling back to pgen/pvar/psam files.
- SHAPEIT5 v5.1.1 requires 3-column genetic maps (no header) and `.bcf` output extension.
- SHAPEIT5 requires AC/AN fields in VCF — pgen-exported VCFs need `bcftools +fill-tags -- -t AC,AN` first.
