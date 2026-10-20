# chr19 locus97 rs281422: pyrho LocusZoom + Relate marginal tree — 2026-08-26

## Target

- `chr19:10325679:T:C`, rsID `rs281422`, `chr19_10246kb_locus97`, R2
- annotated locus gene: RAVER1; nearby TYK2 signal context
- Relate tree `4237`, mutation branch `17515`
- `.mut` SNP index `219804:219802`; TreeView uses genomic BP `10,325,679`
- derived carriers `3868/8766`; age interval `12421–18695.9` generations

## LocusZoom

The ±100 kb window is chr19:10,225,679–10,425,679. It contains 178 SDS
variants and 6,071 lead-centered PLINK2 LD pairs. Recombination uses the CHB
pyrho map. The gene panel uses the local Ensembl GRCh38.112 GTF fallback,
with 19 plottable genes in 3 lanes, including RAVER1 and TYK2.

## Relate marginal tree

The target is on a duplicated BP. A direct BP-only TreeView lookup selected a
different same-position mutation (TreeView branch 8064), so that output was
not used. The final figure uses tree 4237 and the authoritative `.mut` branch
17515, with its descendant set reconstructed from the marginal-tree
coordinates. Purple marks branch 17515; orange marks its descendant
branches. This is local genealogy visualization, not proof of a shared
selection origin with other R2 leads.

## EHH

The existing selscan run covers the same ±100 kb window (14,117 points). The
raw selscan `derEHH` is VCF-ALT based; here VCF ALT=C=AA, while SDS
polarization is `AA=C, DA=T`, so the plotted curves explicitly swap the raw
columns: DA=T uses raw `ancEHH`, AA=C uses raw `derEHH`. At ±50 kb, DA EHH
is 0.035517 and AA EHH is 0.007737; at ±100 kb they are 0.002668 and
0.003155. Half-decay distances are AA: L=3.41 kb, R=1.40 kb; DA: L=8.13
kb, R=8.46 kb. Thus the EHH result is consistent with negative polarized
iHS: the SDS-selected DA=T has the more extended haplotype. EHH remains
supporting haplotype evidence, not a genealogy or causal-selection test.

## Outputs

- `SDSworkspace/results/production/orthogonal/relate_clues2/ALL/genealogy_r1_r2/locus97_rs281422/locus97_rs281422.locuszoom.png`
- `.../marginal_tree_rs281422.png`
- `.../marginal_tree_rs281422.pdf`
- `.../locus97_rs281422.ehh.png`
- `.../manifest.tsv`

Scripts: `SDSworkspace/oneoff/plot_locuszoom_locus92_gtf.py` and
`SDSworkspace/oneoff/render_relate_locus97.py`.
