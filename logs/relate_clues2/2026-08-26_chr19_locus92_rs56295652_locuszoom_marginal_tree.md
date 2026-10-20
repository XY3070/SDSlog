# chr19 locus92 rs56295652: pyrho LocusZoom + Relate marginal tree — 2026-08-26

## Target QC

The current authoritative lead table identifies locus92 as:

- `chr19:10378293:G:T`
- rsID `rs56295652`
- gene `TYK2`
- ancestral allele `G`, derived allele `T`
- DAF `0.0652`, norm_SDS `7.5922`

`rs281422` is a different variant (`chr19:10325679:T:C`) and is locus97 in
the current lead table. It was not used for this figure.

## Pyrho LocusZoom

Window: chr19:10,268,293–10,488,293 (±100 kb around the lead). Inputs were
the ALL normalized SDS table, ALL chr19 QC VCF, and CHB pyrho map. The window
contains 179 variants and 6,435 PLINK2 LD pairs. The gene panel is restored
using the local Ensembl GRCh38.112 GTF fallback (16 plottable genes, 3 lanes)
because the Ensembl REST request returned HTTP 500; TYK2 is present in the
gene track and title.

Visual reading: rs56295652 is the dominant SDS peak (`p=3.15e-14`,
norm_SDS=7.59). Several nearby variants around 10.46–10.49 Mb are also
significant, but the lead-centered LD coloring does not form a broad red/orange
high-r² block. This is consistent with a strong local signal and limited
lead-level LD sharing, not evidence that the entire R2 macro-region is one
genealogy.

## Relate marginal tree

RelateTreeView extracted the tree at BP `10,378,293` from the full chr19
CHB-pyrho Relate output. The mutation record is:

- tree index `4265`
- mutation SNP index `221546`
- mutation branch `17413`
- Relate age interval `1283.61–2204.08` generations
- derived carrier subtree `572/8766` haplotypes

The rendered figure highlights the mutation branch in purple and all branches
below the derived mutation in orange. It shows one coherent derived-carrier
subtree at this marginal tree. This supports a local TYK2 carrier genealogy,
but one marginal tree cannot establish that locus92 shares genealogy with the
other R2 leads or prove a common selection origin.

## Outputs

- `SDSworkspace/results/production/orthogonal/relate_clues2/ALL/genealogy_r1_r2/locus92_rs56295652/locus92_rs56295652.locuszoom.png`
- `.../locus92_rs56295652.no_genes.locuszoom.png` (preserved pre-fallback figure)
- `.../marginal_tree_rs56295652.png`
- `.../marginal_tree_rs56295652.pdf`
- `.../manifest.tsv`

Rendering script: `SDSworkspace/oneoff/render_relate_marginal_tree.py`.
