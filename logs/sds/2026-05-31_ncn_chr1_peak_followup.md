Date: 2026-05-31
Topic: sds
Type: summary

Question

在 `NCN olddefault + rebuilt input` 已被接受为当前 plausible baseline 之后，剩余 `chr1 125M / 143M` 强峰是否已经更像真实 biology，还是仍更像 `NCN` 特有的 site-universe / lineage 残留？

Input paths

- current `NCN olddefault + rebuilt input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN`
- historical acceptable `NCN olddefault mainline`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN`
- repaired `SCN olddefault + rebuilt input`:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN`
- prior lineage audit:
  - `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-15_chr1_chr6_input_lineage_audit.md`

Key files

- region-level significance:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN/NCN.significant_regions.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_mainline/NCN/NCN.significant_regions.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/SCN.significant_regions.tsv`
- chr1 window summaries generated in this follow-up:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_current_chr1.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_historical_chr1.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/SCN_repaired_current_chr1.summary.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_SCN_chr1_followup.summary.tsv`
- chr1 window top hits:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_current_chr1.top_hits.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_historical_chr1.top_hits.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/SCN_repaired_current_chr1.top_hits.tsv`

Result

## 1. The two dominant `NCN` chr1 peaks remain region-stable at the significant-region level

Historical acceptable `NCN olddefault`:

- `chr1:125138602-125177684`, top SNP `chr1:125170712:A:T`
- `chr1:143209700-143286983`, top SNP `chr1:143272460:T:C`

Current rebuilt-input `NCN olddefault`:

- `chr1:125148025-125176827`, top SNP `chr1:125170712:A:T`
- `chr1:143210672-143284926`, top SNP `chr1:143272460:T:C`

Interpretation:

- the common-variant significant peaks are reproduced almost exactly across the historical and rebuilt-input `NCN olddefault` lines;
- this still argues strongly against “2026-05 rebuild newly created these peaks”.

## 2. The raw chr1 SDS windows are also stable within `NCN`

From `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_chr1_followup_20260531/NCN_SCN_chr1_followup.summary.tsv`:

- `NCN current`
  - `peak125`: `row_count=2953`, `max_abs_rSDS=16.3305`
  - `peak143`: `row_count=1693`, `max_abs_rSDS=15.9359`
  - `control35`: `row_count=3797`, `max_abs_rSDS=12.8343`
- `NCN historical`
  - `peak125`: `row_count=4186`, `max_abs_rSDS=16.1465`
  - `peak143`: `row_count=3248`, `max_abs_rSDS=16.7957`
  - `control35`: `row_count=3834`, `max_abs_rSDS=12.8324`

Interpretation:

- the chr1 structure is not just preserved at the final Manhattan/significant-region layer;
- the underlying chr1 raw SDS windows also remain strongly elevated across both `NCN olddefault` baselines.

## 3. `SCN repaired current` is not a same-site negative control for `peak125/peak143`

From the same follow-up summary:

- `SCN repaired current`
  - `peak125`: `row_count=0`
  - `peak143`: `row_count=0`
  - `control35`: `row_count=5542`, `max_abs_rSDS=13.4085`

Interpretation:

- under the current repaired `SCN olddefault` line, the `125M` and `143M` windows are absent from the chr1 SDS table rather than merely “quiet”;
- therefore `SCN` cannot currently be used as a direct same-site negative control for these two `NCN` peaks.

This matters for interpretation:

- it weakens the strongest possible “confirmed biology” statement;
- because the available cross-cohort comparison is currently asymmetric in site universe.

Refinement from the follow-up root-cause audit:

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-05-31_scn_chr1_qarm_boundary_skip_root_cause.md`

That audit shows the asymmetry is more specific than “`SCN` simply lacks those sites”:

- rebuilt `SCN` `t_file` does contain both windows and both common top SNP positions;
- rebuilt `SCN` `s_file` also has singleton support in those windows;
- the actual loss happens later, because `compute_SDS.py` drops early q-arm SNPs when `upstream_nan_frac > 0.10`.

So the current limitation is:

- not “no comparable `SCN` input exists”;
- but “current repaired `SCN` output still has a compute-stage early-q-arm coverage hole”.

Further refinement from the same audit:

- a one-off full `SCN q_chunk_0` rerun with `skip_boundary_missing_fraction=0.35` already recovered `rows_out=9242` and restored both `chr1:125170712:A:T` and `chr1:143272460:T:C`;
- a formal `SCN chr1 relaxed035` repair line is now running under:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN`

This means the remaining asymmetry is now operational rather than conceptual:

- we already have a validated repair setting for the missing early q-arm band;
- what remains is to let the formal chr1 line finish and then test whether the repaired `SCN` output yields a usable same-site comparison.

## 5. Formal repaired `SCN` q-arm output now restores same-site coverage in both windows

From the formal `SCN chr1 relaxed035` run:

- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN`
- working directory:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chunk_submit/chr1_scn_olddefault_relaxed035_20260531_022532`
- repaired final outputs:
  - `chr1_q.sds.tsv`
  - `chr1.sds.tsv`
- repaired seed chunk:
  - `q_chunks/q_chunk_0.sds.tsv`
  - `q_chunks/q_chunk_0.csv`

The formal array-produced `q_chunk_0` reproduces the local recovery exactly, and the merged chr1 output preserves it:

- `q_chunk_0 rows_in=20000`
- `q_chunk_0 rows_out=9242`
- recovered explicit SNPs:
  - `chr1:125170712:A:T`, `rSDS=0.7054`
  - `chr1:143272460:T:C`, `rSDS=0.1037`
- repaired `chr1_q.sds.tsv` position range:
  - `min=125148509`
  - `max=248463050`
- old broken coarse-line `chr1_q.sds.tsv` position range:
  - `min=145551991`
  - `max=247870058`

Window summaries on the repaired merged `SCN chr1.sds.tsv`:

- `peak125` (`125100001-125200000`)
  - `row_count=2118`
  - `max_abs_rSDS=17.1859`
  - top hit `chr1:125177188:C:A`
- `peak143` (`143200001-143300000`)
  - `row_count=2502`
  - `max_abs_rSDS=16.6941`
  - top hit `chr1:143209642:G:C`

Interpretation:

- once the early-q-arm boundary skip is repaired, `SCN` no longer has `row_count=0` in either window;
- the merged repaired chr1 output now explicitly restores q-arm coverage back into `125M`, rather than starting only at `145.5M`;
- the two windows show strong same-region signal in `SCN` as well, with magnitude comparable to `NCN`;
- the exact common top SNPs `125170712` and `143272460` are present but are not themselves the strongest `SCN` hits, so the strongest concordance is currently regional rather than single-SNP exact-match.

## 4. What this follow-up changes

This follow-up does **not** overturn the baseline conclusion:

- `NCN olddefault + rebuilt input` remains the current plausible `NCN` baseline;
- the chr1 peaks are still not a rebuilt-input artifact.

But it does sharpen the remaining uncertainty:

- the best-supported statement is now “stable within `NCN olddefault` across historical and rebuilt-input baselines”;
- and it is no longer true that `SCN` is unusable for same-site comparison in those windows.

What changes after the repaired `SCN q_chunk_0` result:

- the previous `SCN row_count=0` asymmetry was itself a compute artifact;
- after repairing that artifact, both `125M` and `143M` windows remain strongly elevated in `SCN`.

Conclusion

- `NCN chr1 125M / 143M` peaks are stable across historical and rebuilt-input `NCN olddefault` baselines.
- This makes a pure rebuild artifact explanation unlikely.
- The earlier `SCN row_count=0` result was caused by a compute-stage early-q-arm coverage hole, not by missing input sites.
- After repairing that hole on the formal `SCN chr1 relaxed035` line, both windows show strong same-region signal in `SCN` too.
- Therefore the strongest current wording shifts to:
  - `NCN olddefault + rebuilt input` remains a plausible baseline;
  - the chr1 `125M / 143M` structure is no longer well-described as `NCN`-specific;
  - it is more consistent with a shared regional signal that was previously masked in `SCN` by the boundary-skip artifact;
  - exact single-SNP concordance is still weaker than regional concordance, but merged repaired chr1 reporting is now available.

Next step

- do not start another whole-genome repair run;
- use the repaired `SCN chr1 relaxed035` merged table as the new same-site comparison baseline for these windows;
- then decide whether the relaxed boundary-skip setting should stay chr1-targeted or be generalized more carefully.
