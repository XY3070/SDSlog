# iHS Polarization by SDS AA/DA + Plot AA/DA Annotation + Catalog Update

Date: 2026-08-09
Status: COMPLETE — iHS re-polarized + re-normalized (all 20 chr), plots annotated, catalog updated
Role: mainline — orthogonal evidence (iHS/EHH) must follow SDS AA/DA polarization
Sample policy: ALL (4383); Input lineage: ALL phased VCF → selscan → SDS-polarized norm

## 1. Problem

selscan `--ihs` treats the **VCF ALT allele as derived**. The SDS pipeline
polarizes by EPO ancestral allele (AA/DA in `all.normalized.tsv` — NOTE: the
TSV ID column is chr:pos:ref:alt, the AA/DA columns are the authoritative
polarization). Wherever VCF ALT ≠ SDS DA, the raw iHS sign referred to the
ancestral allele — i.e. flipped relative to SDS convention.

**Scale: chr4 alone had 107,550/403,636 loci (26.6%) needing a flip.**

### Concrete impact found
- **chr4 ADH1B (chr4:99318162:T:C)**: VCF ALT=C = SDS **AA**; SDS DA=T (DAF 0.70).
  - iHS was reported -2.46 → after polarization **+2.02**
  - EHH (same ALT-as-derived issue): "no_extension" was wrong — the SDS-derived
    (T) allele EHH@500kb = **0.324** (plateau) → **extended_haplotype**
- chr19_4885kb_sig5: -0.0001 → +0.0095 (flip)
- ALDH2 (DA=A = VCF ALT) and most chr19 signals: no flip needed; values shifted
  slightly from re-normalization on SDS DAF bins (ALDH2 2.54 → 2.72)

## 2. Implementation

`SDSpipeline/scripts/orthogonal/polarize_norm_ihs.py` (registered):
- load (chr,pos) → (AA, DA, DAF) from `all.normalized.tsv` (one pass)
- load VCF ALT per pos from the dedup'd VCF (bcftools query, full path)
- flip raw iHS where ALT == AA; flag allele_mismatch (third allele at
  multiallelic site) and no_sds (absent from SDS site universe)
- re-normalize in 100 bins on **SDS DAF**; output columns:
  `pos DAF(SDS) raw_ihs norm_ihs extreme AA DA flag`
- runs: all 20 chr with iHS (chr19 included after the AC-dedup rerun 504724)

`build_clump_catalog.py` additions:
- `--data-tsv` AA/DA map; `--ihs-root` fresh iHS lookup (already existed)
- **EHH polarization**: recompute fresh + swap der/anc columns when the lead's
  SDS DA == VCF REF (selscan ALT = AA). Fixed chr4 ADH1B EHH classification.

Plot scripts (AA/DA annotated directly):
- `plot_locuszoom.py` (SDSpipeline/scripts/postprocess/): title now includes
  `AA=X DA=Y (DAF Z)`
- `plot_ehh_single.py` → **archived** to `SDSpipeline/scripts/orthogonal/`
  (registered): metrics box shows AA/DA/DAF; der/anc curves swapped to SDS
  polarity and labeled `DA=… (derived)` / `AA=… (ancestral)` when needed

## 3. Catalog (signals_clump/all_signals_catalog.tsv, 16 signals) — iHS updated

| signal | norm_iHS | DAF | EHH class |
|---|---|---|---|
| chr4 ADH1B | **+2.02** (was -2.46) | 0.700 | **extended_haplotype** (was no_extension) |
| chr12 ALDH2 | +2.72 | 0.245 | hard_sweep_extended |
| chr11 intergenic | +2.03 | 0.149 | no_extension |
| chr19_10690kb_sig4 (DNM2) | +1.91 | 0.061 | no_extension |
| chr19_4885kb_sig8 | +1.12 | 0.317 | NA (EHH pending) |
| chr19_4885kb_sig3 (4867638) | **-0.44** (was NA; AC-dedup fix) | 0.925 | no_extension |
| chr19_10690kb_sig3 (10555015) | -1.65 | 0.944 | NA (EHH pending) |
| chr19_4885kb_sig6 | -0.72 | 0.073 | no_extension |
| chr19_4885kb_sig7 (UHRF1) | -0.29 | 0.061 | no_extension |
| chr22 ACO2 | +0.67 | 0.522 | no_extension |
| others (chr19) | +0.01..+0.60 | — | no_extension |

iHS extremes: none |norm_iHS|>4 among the 16 leads.

## 4. Plots regenerated

- `plots/SLE_family/chr7_50266267_ehh.png` — replotted with AA/DA/DAF box +
  SDS-polarized curve labels (chr7:50266267:T:G: DA=G = VCF ALT, no swap)
- LocusZoom: title AA/DA for future runs (`--lead-label` unchanged)

## 5. Caveats / follow-up

- **Old catalog (signals/) EHH/iHS columns carry the same polarity issue** —
  production catalog iHS/EHH values were ALT-as-derived. Only signals_clump
  catalog is now SDS-polarized. Recommend re-deriving the production catalog
  annotation the same way before paper use.
- EHH pending for 3 new leads: chr19_10690kb_sig2/sig3, chr19_4885kb_sig8
- chr19 iHS rerun (AC-aware dedup, 504724) completed — 4867638 now has iHS

## Files

- `SDSpipeline/scripts/orthogonal/polarize_norm_ihs.py` (new, registered)
- `SDSpipeline/scripts/orthogonal/plot_ehh_single.py` (archived from tmp/, registered)
- `SDSpipeline/scripts/postprocess/build_clump_catalog.py` (EHH swap + data-tsv)
- `SDSpipeline/scripts/postprocess/plot_locuszoom.py` (title AA/DA)
- Outputs: `results/production/orthogonal/ihs/ALL/chr*.ihs.100bins.norm`
  (polarized), `signals_clump/all_signals_catalog.tsv`
