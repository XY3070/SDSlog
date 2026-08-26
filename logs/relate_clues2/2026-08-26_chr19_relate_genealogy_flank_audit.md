# chr19 R1/R2 Relate genealogy + flank audit — 2026-08-26

## Scope

Use completed ALL chr19 Relate output with CHB pyrho map:

- `.anc/.mut`: `ALL/run_work_chr19_chb_pyrho/UKBQC_ALL_chr19.{anc,mut}`
- haplotypes: 8,766
- Relate trees: 28,788
- leads: R1=19, R2=14
- flank: ±100 kb around every lead
- flank tree sampling: dense, every tree in each lead's ±100 kb interval

This audit tests local genealogy/carrier-clade sharing. It does not prove
causality, a shared sweep, or independence by itself.

## Execution

- `506212/506213`: sparse preview and manifest-header correction; superseded.
- `506214`: dense rerun on `smp`, `FLANK_TREE_STRIDE=1`, mapped-lead persistence;
  completed successfully, peak memory about 3 GB. It overwrites only audit
  outputs, never Relate `.anc/.mut`.

Script:

`SDSworkspace/oneoff/analyze_chr19_relate_genealogy.py`

Wrapper:

`SDSpipeline/scripts/orthogonal/run_chr19_relate_genealogy.sh`

## QC

- 33/33 leads matched exactly to one `.mut` row by rs-id/POS-REF-ALT.
- 32 target trees: two lead pairs share one Relate tree.
- locus88 has `is_not_mapping=1` and multiple branch indices; its clade
  metrics are retained for QC but must not drive biological conclusions.
- Final Relate reported 192,570 not-mapped SNPs genome-wide and 935 flipped
  SNPs; lead-level mapping QC is therefore mandatory.

## Preliminary carrier-clade result

Pairwise Jaccard uses derived-carrier clades at each lead's own Relate tree.
It is a useful local sharing diagnostic, not a same-tree proof when lead trees
differ.

- R1 strongest mapped overlaps: locus86–locus78 Jaccard 0.703,
  locus84–locus78 0.535. locus86–locus84 is 0.371. This points to a local
  genealogy-sharing network, not all 19 R1 leads on one genealogy.
- R2 strongest mapped overlap: locus97–locus94 Jaccard 0.422; other notable
  local pairs include locus93–locus94 0.239, locus98–locus93 0.222, and
  locus101–locus102 0.215. No broad 14-lead shared-clade pattern appears.

## Dense ±100 kb flank persistence

For each lead, branches in every ±100 kb tree were compared by descendant
haplotype set (branch IDs are not assumed stable across recombination). The
dense scan contains 942 flank-tree observations across 28,788 Relate trees;
32 target trees are retained separately. Strongest persistence is R1 locus78
(median best-clade Jaccard 0.925) and locus86 (0.751), with locus73 also
showing a stable but largely distinct local clade (0.657). R2 shows strong
local clades at locus94 (0.944), locus95 (0.885), locus92 (0.779), and locus91
(0.722), but these clades have low mutual carrier overlap in key comparisons.

The dense flank result therefore supports R1 as several local networks, with
the clearest 86–78 relation and a possible nested 84 relation, rather than one
genealogy spanning all leads. R2 contains multiple related-but-distinct local
blocks: 94/97/93 is one candidate related block, while 95, 92, and 91 retain
distinct stable clades. Do not collapse either macro-region into one selected
allele before branch-level selection analysis.

## Pairwise shared-branch audit

The lead-specific persistence test was followed by a stricter pairwise test.
For every pair, all trees in the intersection of their two ±100 kb flanks were
examined. A tree is counted as joint support when one internal branch has
Jaccard >=0.5 to both lead-specific carrier clades. Pairs with fewer than 20
shared flank trees are not used for block adjudication.

- R1: 86–78, 86–84, and 84–78 all have joint support in every shared-flank
  tree tested (fraction=1.0); carrier Jaccard values are 0.703, 0.371, and
  0.535, respectively. This supports an 86–78–84 shared local branch network,
  with 86–78 the strongest carrier-sharing pair. It still does not support a
  single genealogy spanning all R1 leads.
- R2: no pair reaches joint-support fraction >=0.5 among pairs with >=20
  shared flank trees. The lead-tree carrier overlaps around 94/97/93 therefore
  do not establish one shared local branch; they remain related-carrier/local
  clade clues only.

This test is still not a proof of common selection origin: a shared ancestral
branch can predate selection, and a recombination event can separate selected
backgrounds. It does, however, provide the strongest current Relate-based
partitioning evidence and explicitly incorporates flanking trees.

## Outputs

- `SDSworkspace/results/production/orthogonal/relate_clues2/ALL/genealogy_r1_r2/manifest.tsv`
- `lead_mut_qc.tsv`
- `lead_tree_carrier_clades.tsv`
- `lead_carrier_pairwise.tsv`
- `lead_flank_clade_persistence.tsv`
- `lead_genealogy_adjudication.tsv`
- `lead_pair_shared_branch_persistence.tsv`
- `genealogy_block_adjudication.tsv`
- `lead_carrier_clade_jaccard_square.png`
- `genealogy_adjudication_overview.png`
- `lead_pair_shared_branch_persistence_square.png`

## Decision status

R1: supports several local shared-genealogy/proxy networks, especially the
86–78–84 area; does not support one genealogy spanning all R1 leads.

R2: dense flank and pairwise shared-branch evidence reject treating all 14
leads as one common genealogy. Retain multiple related-haplotype and multiple
selection-event models. Next: run RelateSelection/CLUES2 separately for the
R1 86–78–84 network and for R2 candidate local blocks, using mapped leads only
and preserving ±100 kb flanking context; do not run one macro-region-wide
selected-branch model as the sole analysis.
