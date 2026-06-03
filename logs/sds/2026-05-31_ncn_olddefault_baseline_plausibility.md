Date: 2026-05-31
Topic: sds
Type: summary

Question

在 `NCN` 上，`historical old/default gamma + rebuilt input` 是否已经足够作为当前 plausible baseline？剩下的 chr1 强峰更像 biology，还是 rebuilt input 新引入的技术假象？

Input paths

- current `NCN olddefault + rebuilt input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN`
- historical acceptable `NCN olddefault mainline`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN`
- chr1/chr6 lineage audit:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/input_lineage_audit_20260515`

Key files

- rebuilt-input baseline:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.frequency_bins.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.significant_regions.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/diagnostics/NCN.olddefault_newinput.chrom_tail_counts.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/diagnostics/NCN.olddefault_newinput.top_hits.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/cmplot/NCN.cmplot_p5e4_maf001.png`
- historical acceptable baseline:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.significant_regions.tsv`
- chr1/chr6 input-lineage audit:
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-15_chr1_chr6_input_lineage_audit.md`

Result

## 1. The rebuilt-input `NCN olddefault` track is not showing genome-wide failure

Chrom-tail diagnostics for the rebuilt-input baseline:

- `chr1`: `>=20/50/100 = 103 / 33 / 1`
- `chr6`: `>=20/50/100 = 0 / 0 / 0`
- `chr2-22`: essentially quiet, with only one `chr9 >=20` count and no multi-chromosome extreme tail

This is fundamentally different from the rejected `Gravel_CHB@100k + rebuilt input` behavior, where multiple chromosomes carry large `>=20`, `>=50`, `>=100` tails.

Interpretation:

- current `NCN olddefault + rebuilt input` no longer has the genome-wide pathological tail that defined the broken Manhattan family;
- the residual structure is concentrated mainly in chr1, not broadcast across the genome.

## 2. The dominant chr1 peaks are stable across historical acceptable baseline and rebuilt-input baseline

Historical mainline acceptable `olddefault` regions:

- `chr1:125138602-125177684`, top SNP `chr1:125170712:A:T`
- `chr1:143209700-143286983`, top SNP `chr1:143272460:T:C`

Rebuilt-input `olddefault` regions:

- `chr1:125148025-125176827`, top SNP `chr1:125170712:A:T`
- `chr1:143210672-143284926`, top SNP `chr1:143272460:T:C`

These two major chr1 peaks therefore persist with:

- the same top SNPs
- nearly the same region boundaries
- the same `olddefault gamma family`
- despite the switch from historical old input to rebuilt input

Interpretation:

- the two main chr1 peaks are not newly created by the rebuilt input;
- they are inherited from the previously accepted `olddefault` baseline.

## 3. chr1 rebuilt input is essentially identical to current old-split input

From the 2026-05-15 chr1/chr6 lineage audit:

- `chr1 oldsplit vs rebuilt NCN`
  - `peak125`: `2953 vs 2953`, shared IDs `2953`, a-only `0`, b-only `0`
  - `peak143`: `1693 vs 1693`, shared IDs `1693`, a-only `0`, b-only `0`
  - `control35`: `3797 vs 3797`, shared IDs `3797`, a-only `0`, b-only `0`

Interpretation:

- for chr1, the rebuilt main-contract input did not introduce a new test-SNP universe relative to the current split-pop old input;
- the remaining large peaks therefore cannot be blamed on the 2026-05 rebuild itself.

## 4. The true lineage discontinuity is legacy/unpop-like vs current pop-specific, not oldsplit vs rebuilt

The same audit showed that the large structural difference is:

- `legacy/unpop-like` vs `current pop-specific`

not:

- `oldsplit` vs `rebuilt`

This is why the repaired `NCN olddefault + rebuilt input` baseline should be interpreted carefully:

- it is plausible in the sense that it removes the rejected genome-wide artifact family;
- but it does not recreate the exact same chr1 site universe as the old unpop-like lineage.

## 5. Current NCN conclusion

The strongest evidence-supported interpretation is:

1. `NCN olddefault + rebuilt input` is the current plausible baseline.
2. Its remaining major chr1 peaks are not a rebuilt-input artifact:
   - they are already present in the historical acceptable `olddefault` mainline.
3. The open question for `NCN` is no longer “why is the whole Manhattan broken?”;
   it is “whether the surviving chr1 peaks are true biology or residual lineage/site-universe effects inherited from the broader olddefault family.”

## 6. Follow-up refinement for chr1 `125M / 143M`

Follow-up log:

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_ncn_chr1_peak_followup.md`

What the follow-up adds:

- at the significant-region layer, the two chr1 peaks remain nearly identical between historical `NCN olddefault` and rebuilt-input `NCN olddefault`;
- at the raw chr1 SDS-window layer, they also remain strong in both `NCN` baselines;
- but in the repaired current `SCN olddefault` chr1 SDS table, both windows have `row_count=0`.
- a later root-cause audit showed that this `SCN` gap is caused by early-q-arm boundary skipping in `compute_SDS.py`, not by those sites being absent from rebuilt `SCN` input.

Interpretation:

- this further weakens any “rebuilt input created the chr1 peaks” explanation;
- but it also means current `SCN` output is not yet a usable same-site negative control for those two windows.

Repair-path update from the same-day follow-up:

- the `SCN` gap is now traced to a concrete `compute_SDS.py` boundary-skip parameter;
- a full local rerun of the missing `SCN chr1 q_chunk_0` under `skip_boundary_missing_fraction=0.35` recovered `rows_out=9242` and restored both common top SNPs;
- a formal `SCN chr1 relaxed035` line is now running to promote that repair into the standard chunked pipeline.

Further update from the formal repaired `SCN q_chunk_0`:

- the standard array-produced `SCN` `q_chunk_0` also recovered `rows_out=9242`;
- in the repaired same-site windows:
  - `peak125`: `row_count=2118`, `max_abs_rSDS=17.1859`
  - `peak143`: `row_count=2502`, `max_abs_rSDS=16.6941`
- the merged repaired outputs now exist:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chr1_q.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chr1.sds.tsv`
- repaired `chr1_q.sds.tsv` minimum position has moved back from `145551991` to `125148509`

Interpretation:

- the previous `SCN row_count=0` asymmetry was a compute artifact, not a biological absence;
- after repairing that artifact, the two chr1 windows show strong same-region signal in `SCN` too, and that recovery is now present in the merged chr1 output rather than only in a standalone chunk rerun.

So the strongest defensible wording is slightly narrower than “confirmed biology”:

- the chr1 peaks are stable and biologically plausible within `NCN`;
- but the contribution of `NCN`-specific inherited site universe / lineage is still not fully excluded.

Conclusion

- `NCN olddefault + rebuilt input` should be treated as the current repaired/plausible `NCN` baseline.
- Unlike the rejected `Gravel` family, it does not exhibit genome-wide multi-chromosome inflation.
- The surviving chr1 peaks are stable relative to the historical acceptable baseline, so they are not evidence that the rebuild itself broke `NCN`.
- The repaired `SCN` q-arm evidence now also argues against treating those windows as `NCN`-specific.
- The strongest current phrasing is:
  - plausible shared regional signal, with final merged-chr1 confirmation still pending;
  - not rebuilt artifact, and no longer well-described as `NCN`-only residual structure.

Next step

- treat `NCN olddefault + rebuilt input` as the production baseline for current comparative reporting
- if further work is needed, first finish the targeted `SCN chr1 relaxed035` repair line, then use the repaired chr1 track to test biological follow-up vs site-universe inheritance for the `125M` and `143M` peaks
