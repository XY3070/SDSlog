Date: 2026-05-31
Topic: sds
Type: summary

Question

在 `SCN` rebuilt new input 上，`historical old/default gamma` 的全基因组修复线是否已经真正恢复到 plausible biology？如果是，当前最合理的 `SCN` 修复口径应如何表述？

Input paths

- old `Gravel_CHB@100k + new input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_newinput_20260511/SCN`
- sample-size matched `Gravel_CHB@100k + new input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526/SCN`
- repaired `SCN olddefault + rebuilt input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN`

Key outputs

- repaired full postprocess outputs:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/SCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/SCN.frequency_bins.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/diagnostics/SCN.gravel_chb_ne100k_submitter_array.chrom_tail_counts.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/SCN.manhattan.png`
- repaired filtered `CMplot`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/cmplot/SCN.cmplot_p5e4_maf001.manhattan.png`
- three-track compare:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_coarse_compare_20260528/SCN_old_vs_matched_vs_olddefault.summary.md`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_coarse_compare_20260528/SCN_old_vs_matched_vs_olddefault.chrom_tail_compare.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_coarse_compare_20260528/SCN_old_vs_olddefault.frequency_bins_compare.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_coarse_compare_20260528/SCN_matched_vs_olddefault.frequency_bins_compare.tsv`

Result

## 1. The repaired `SCN olddefault` line completed end-to-end

The coarse-chunk submitter-array line is no longer pending or partial. It produced:

- `22` chromosome archive summaries
- `64` chromosome-level `*.sds.tsv` outputs
- finalized `normalized.tsv`
- finalized `frequency_bins.tsv`
- diagnostics
- full Manhattan plot
- filtered `CMplot`

This means the `SCN olddefault + rebuilt input` repair line is no longer a smoke or chr1/chr6-only test; it is a completed genome-wide result.

## 2. `SCN olddefault` removes the genome-wide extreme tail that defined the bad Manhattan

From the three-track summary:

- `old` (`Gravel_CHB@100k + rebuilt input`)
  - chr1 `>=20/50/100 = 207 / 71 / 36`
- `matched` (`Gravel_CHB@100k`, corrected `SAMPLE_SIZE=5432`)
  - chr1 `>=20/50/100 = 201 / 62 / 29`
- `olddefault`
  - chr1 `>=20/50/100 = 0 / 0 / 0`

Genome-wide chrom-tail comparison is even clearer. For `olddefault`, every chromosome has:

- `count_neglog10_ge_20 = 0`
- `count_neglog10_ge_50 = 0`
- `count_neglog10_ge_100 = 0`

while the old and matched `Gravel` tracks still retain many extreme-tail points across multiple chromosomes.

Interpretation:

- correcting `SCN` `SAMPLE_SIZE` helped, but only partially;
- switching to `olddefault gamma family` is what actually collapses the implausible multi-chromosome extreme tail.

## 3. The worst low-frequency common bins are repaired, not merely attenuated

`SCN matched vs olddefault` bin compare:

- `[0.01,0.02)`:
  - mean `2.037950853 -> -0.3296884407`
  - sd `6.557090373 -> 0.2273623037`
- `[0.02,0.03)`:
  - mean `1.490079273 -> -0.3221499854`
  - sd `7.081476205 -> 0.2213404315`
- `[0.03,0.04)`:
  - mean `0.2528482731 -> -0.3128759275`
  - sd `3.153318674 -> 0.2291013517`

`old vs olddefault` shows an even larger correction:

- `[0.01,0.02)`:
  - mean `4.392295135 -> -0.3296884407`
  - sd `10.50532175 -> 0.2273623037`

Interpretation:

- the repaired line does not just shave off a few top outliers;
- it restores the low-frequency common bins away from the massively inflated regime that drove the old broken Manhattan.

## 4. The repaired Manhattan is visually plausible

Two repaired plots were inspected:

- full genome-wide Manhattan:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/SCN.manhattan.png`
- filtered `CMplot`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/cmplot/SCN.cmplot_p5e4_maf001.manhattan.png`

Both now show:

- no two-layer / gap-separated p-value distribution
- no diffuse `40+` suspended tail across many chromosomes
- sparse, localized peaks rather than genome-wide inflation

This satisfies the working criterion of “plausible biology” substantially better than either `old Gravel` or `matched Gravel`.

## 5. Current repair conclusion for `SCN`

For `SCN`, the best-supported repaired production interpretation is now:

- input: rebuilt new input
- gamma family: `historical old/default`
- role of `SAMPLE_SIZE` mismatch:
  - real technical contributor
  - but not the dominant remaining explanation
- dominant remaining difference:
  - gamma family itself

So the current `SCN` repair answer is:

**`SCN olddefault + rebuilt input` is the first genome-wide repaired line that returns to plausible biology.**

Conclusion

- `SCN` anomaly is now effectively explained as:
  - `SAMPLE_SIZE` mismatch was real but partial;
  - the larger failure mode was the `Gravel_CHB@100k` gamma family on the rebuilt pop-specific input.
- `SCN olddefault + rebuilt input` completed genome-wide and repaired both:
  - extreme chromosomal tails
  - low-frequency common-bin inflation
- therefore `SCN olddefault + rebuilt input` should be used as the current repaired `SCN` baseline in downstream reporting and figure cataloging.

Next step

- register the repaired `SCN` `CMplot` and diagnostics into `SDSlog` plot/report indexes
- keep `NCN olddefault + rebuilt input` as the current plausible `NCN` baseline while `NCN` root-cause work continues separately
