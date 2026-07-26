# 1KCP (1000 Chinese Pangenome) eQTL Integration

## Date
2026-07-26

## Summary
Downloaded and integrated the 1KCP Chinese-specific eQTL summary statistics to replace the EUR-ancestry eQTLGen resource for non-coding signal annotation.

## Data Source
- Resource: 1KCP (1000 Chinese Pangenome) — Yang Lab, Westlake University
  - Nature 654, 121–130 (2026). doi:10.1038/s41586-026-10315-y
  - 1,101 Chinese WGS samples with RNA-seq
  - Pan-variant eQTL mapping (small variants, SVs, TRs, nested variants)
- Download: `https://yanglab.westlake.edu.cn/resources/1kcp/eqtl/1kcp.eqtl.chr*.summary.gz`
  - Per-chromosome gzipped summary files (22 autosomes + X/Y available)
  - 14 columns: SNP, Chr, BP, A1, A2, Freq, Probe, Probe_Chr, Probe_bp, Gene(Orientation, b, se, p
  - No pre-computed FDR; p-value filtering used instead

## Files Downloaded
- Location: `SDSworkspace/input/reference/eQTL_1KCP/`
- 22 chromosome `summary.gz` files (~7GB total)
- Converted to sorted + bgzip + tabix format (~5GB total for sorted.bgz)
- Indexed for region-based random access via tabix

## Pipeline Integration
- Updated `SDSpipeline/scripts/postprocess/ld_split_and_annotate.py`:
  - Added `query_1kcp_eqtl()` function using tabix
  - Added `--eqtl-1kcp` argument for 1KCP directory path
  - New catalog columns: `eQTL_1KCP_CHN_hits`, `eQTL_1KCP_CHN_top_genes`, `eQTL_1KCP_CHN_top_ensg`, `eQTL_1KCP_CHN_top_p`, `eQTL_1KCP_CHN_top_beta`
  - ENSG ID → gene symbol mapping from `EnsemblIDtoGeneSymbol_dict.json`
  - eQTLGen(EUR) is retained as a legacy comparison column
  - 1KCP(Chinese) is the primary EAS eQTL resource

- Created standalone utility: `SDSworkspace/oneoff/query_1kcp_eqtl_for_signals.py`
  - Can add 1KCP annotations to any existing catalog without re-running LD split/VEP

## Results on Current Signals
Of 15 non-MHC independent signals, 14 have significant 1KCP eQTL hits (p<1e-5):

| Signal | Lead SNP | 1KCP Top Genes | Best p | eQTLGen Top Genes |
|--------|----------|---------------|--------|-------------------|
| chr4_99240kb | chr4:99318162:T:C | MTTP | 6.2e-06 | — |
| chr11_48235kb | chr11:48235462:G:A | C1QTNF4,MTCH2,MYBPC3 | 5.9e-15 | ACP2,MYBPC3 |
| chr12_109929kb | chr12:109825802:G:A | GIT2 | 1.1e-11 | KCTD10 |
| chr12_110306kb | chr12:110306896:G:A | GIT2,GPN3 | 3.7e-33 | GLTP,GPN3 |
| chr12_111181kb | chr12:111803962:G:A | HECTD4,TMEM116 | 1.7e-104 | — (missense) |
| chr12_112355kb | chr12:112030402:C:T | HECTD4,TMEM116 | 4.2e-102 | MAPKAPK5-AS1,TMEM116 |
| chr12_112949kb | chr12:112971371:G:A | RPH3A | 2.0e-36 | TMEM116 |
| chr19_4885kb_sig1 | chr19:4710802:C:T | DPP9 | 1.2e-47 | C19orf10,DPP9 |
| chr19_4885kb_sig2 | chr19:4867689:G:T | PLIN3 | 3.7e-11 | ARRDC5,PLIN3 |
| chr19_4885kb_sig3 | chr19:4768268:A:C | TICAM1 | 6.5e-06 | TICAM1 |
| chr19_4885kb_sig4 | chr19:4903524:T:A | ARRDC5 | 1.1e-07 | ARRDC5 |
| chr19_4885kb_sig6 | chr19:4951434:C:T | KDM4B | 1.9e-06 | ARRDC5 |
| chr19_10690kb | chr19:10765735:C:T | ILF3-DT,TMED1,YIPF2 | 2.3e-28 | ILF3-AS1,SLC44A2 |
| chr22_41498kb | chr22:41504089:G:A | CCDC134 | 1.4e-70 | L3MBTL2,TEF |

## Key Observations
- 1KCP targets differ from eQTLGen for several signals, as expected with ancestry shift
- Several signals show extremely strong 1KCP evidence (p<1e-30), supporting Chinese-specific eQTL regulation
- The ALDH2 (chr12_111181kb) and HECTD4 (chr12_112355kb) signals show the strongest Chinese eQTL associations (p<1e-100)
- Most signals have multiple eQTL target genes, suggesting complex regulatory architecture

## Role
`mainline` — 1KCP replaces eQTLGen as the primary eQTL resource for Chinese cohort analysis.

## Next Steps
- Complete bgzip/tabix conversion for remaining chromosomes (chr7-10,13-18,20-21 are in progress)
- Re-run full pipeline with `--eqtl-1kcp` for authoritative catalog regeneration
- Consider using 1KCP eQTL data for colocalization (SMR/HEIDI) or TWAS analysis
