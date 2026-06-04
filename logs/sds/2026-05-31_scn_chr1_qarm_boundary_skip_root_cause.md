Date: 2026-05-31
Topic: sds
Type: summary

Question

为什么 repaired `SCN olddefault` 在 `chr1 125M / 143M` 两个 `NCN` peak 窗口里是 `row_count=0`？这是因为 `SCN` 当前 input/site universe 本身没有这些位点，还是因为后续 `compute_SDS.py` 在 q-arm 起始端把它们系统性丢掉了？

Input paths

- rebuilt `SCN` input:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN`
- repaired `SCN olddefault` genome-wide coarse output:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN`
- current `NCN olddefault + rebuilt input` baseline:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_olddefault_newinput_20260514/NCN`

Key files

- `SCN` rebuilt input:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr1_t_file.txt`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr1_s_file.txt`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr1_b_file.txt`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_input_rebuilt_main_contract_20260511/SCN/chr1_o_file.txt`
- repaired `SCN` chr1 outputs:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chr1.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chr1_q.sds.tsv`
- q-arm chunk diagnostics:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chunk_submit/chr1_scn_olddefault_genomewide_submitter_array_chr1_20260528_183758/chr1_q.filter.csv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chunk_submit/chr1_scn_olddefault_genomewide_submitter_array_chr1_20260528_183758/q_chunks/q_chunk_0.csv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chunk_submit/chr1_scn_olddefault_genomewide_submitter_array_chr1_20260528_183758/q_chunks/q_chunk_1.csv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chunk_submit/chr1_scn_olddefault_genomewide_submitter_array_chr1_20260528_183758/q_chunks/q_chunk_1.sds.tsv`
- new diagnostics generated here:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/boundary_skip_diag_20260531/SCN_qchunk0_positions.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/boundary_skip_diag_20260531/SCN_qchunk1_positions.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/boundary_skip_diag_20260531/SCN_qchunk0_positions_relaxed035.tsv`
- script changes:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/diagnose_boundary_skip.py`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/compute_SDS.py`

Result

## 1. The missing `SCN` coverage is not an input-site-universe absence

Current rebuilt `SCN` input still contains the relevant q-arm windows and the two common top SNP positions:

- rebuilt `SCN chr1_t_file`:
  - `peak125=4280`
  - `peak143=2502`
  - `pos125170712=1`
  - `pos143272460=1`
- rebuilt `SCN chr1_s_file` singleton support:
  - `peak125=1491`
  - `peak143=468`

Interpretation:

- `SCN` does have both test SNPs and singleton support in the `125M / 143M` windows;
- so `row_count=0` is not caused by those windows being absent from rebuilt input.

## 2. The loss occurs after q-arm input filtering, not before

`chr1_q.filter.csv` for the repaired genome-wide coarse line shows:

- q-arm input start: `124932725`
- q-arm input end: `248956422`
- q-arm input rows: `2065086`

So q-arm filtering/feather conversion still preserved the early q-arm range.

At the final chr1 output layer:

- repaired `SCN chr1.sds.tsv`:
  - `p_rows=2335713`
  - `q_rows=2003967`
  - q-arm minimum output position: `145551991`

Interpretation:

- q-arm as a whole is not missing;
- but all output rows before `145551991` are gone.

## 3. The first q-arm chunk is completely eaten at compute stage

Chunk-level summaries from the repaired coarse line:

- `q_chunk_0.csv`:
  - `rows_in=20000`
  - `rows_out=0`
- `q_chunk_1.csv`:
  - `rows_in=20000`
  - `rows_out=468`

And the first produced rows in `q_chunk_1.sds.tsv` start at:

- `chr1:145551991:G:T`

Interpretation:

- the missing `125M-145.5M` band is not a merge omission;
- it is because the first q-arm chunk is fed into `compute_SDS.py` and produces zero retained rows.

## 4. The direct mechanism is the boundary-missing guard in `compute_SDS.py`

`compute_SDS.py` drops a SNP when either:

- `np.isnan(upstream).mean() > skip_boundary_missing_fraction`
- or `np.isnan(downstream).mean() > skip_boundary_missing_fraction`

with current default:

- `skip_boundary_missing_fraction = 0.10`

Exact diagnostics from `diagnose_boundary_skip.py`:

- strict threshold `0.10`
  - `chr1:125170712:A:T`
    - `upstream_nan_frac=0.3310014728`
    - `downstream_nan_frac=0.0`
    - `skip_boundary=True`
  - `chr1:143272460:T:C`
    - `upstream_nan_frac=0.3140648012`
    - `downstream_nan_frac=0.0`
    - `skip_boundary=True`
  - first kept q-arm site `chr1:145551991:G:T`
    - `upstream_nan_frac=0.09977908689`
    - `downstream_nan_frac=0.0`
    - `skip_boundary=False`

Interpretation:

- the early q-arm loss is specifically caused by too many samples lacking an upstream singleton before those SNPs;
- once the upstream-missing fraction drops below `0.10`, rows start to survive.

## 5. Minimal repair path is now explicit

I added a CLI parameter to `compute_SDS.py`:

- `--skip-boundary-missing-fraction`

default remains unchanged:

- `0.10`

With a relaxed threshold diagnostic:

- `--skip-fraction 0.35`

the two previously dropped SNPs become:

- `chr1:125170712:A:T` -> `skip_boundary=False`
- `chr1:143272460:T:C` -> `skip_boundary=False`

Interpretation:

- there is now a concrete, testable repair path for recovering q-arm early coverage;
- this is no longer just a speculative explanation.

## 6. The repair path has already been validated on the full 20k-row `q_chunk_0`

I completed a one-off local rerun of the original repaired-line `q_chunk_0` with:

- `skip_boundary_missing_fraction=0.35`

Input/output paths:

- source chunk:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528/SCN/chunk_submit/chr1_scn_olddefault_genomewide_submitter_array_chr1_20260528_183758/q_chunks/chr1_q_0000.feather`
- rerun output:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/scn_chr1_qchunk0_relaxed035_20260531/q_chunk_0.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/scn_chr1_qchunk0_relaxed035_20260531/q_chunk_0.csv`

Observed result:

- `rows_in=20000`
- `rows_out=9242`

Recovered target rows include:

- `chr1:125170712:A:T`
  - `DAF=0.5017`
  - `rSDS=0.7054`
- `chr1:143272460:T:C`
  - `DAF=0.4915`
  - `rSDS=0.1037`
- `chr1:145551991:G:T`
  - `rSDS=1.1006`

Interpretation:

- this is no longer just a diagnostic prediction based on `skip_boundary=False`;
- the exact `q_chunk_0` that previously yielded `rows_out=0` now yields `rows_out=9242` under the relaxed threshold and directly recovers both comparison SNPs.

## 7. A formal `SCN chr1 relaxed035` repair line is now running on LSF

Submitted output root:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN`

Working directory:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chunk_submit/chr1_scn_olddefault_relaxed035_20260531_022532`

Submitted jobs:

- `P_ARRAY_JOB=487913`
- `Q_ARRAY_JOB=487914`
- `P_MERGE_JOB=487915`
- `Q_MERGE_JOB=487916`
- `FINAL_JOB=487917`

Current interpretation:

- the repair path is no longer an ad hoc local rescue only;
- it has been promoted into the standard chunked chr1 pipeline with explicit provenance and job tracking.

Update after the first formal q-arm chunk completed:

- formal `q_chunk_0.csv` now reports:
  - `rows_in=20000`
  - `rows_out=9242`
- formal `q_chunk_0.sds.tsv` contains the same recovered comparison SNPs:
  - `chr1:125170712:A:T`, `rSDS=0.7054`
  - `chr1:143272460:T:C`, `rSDS=0.1037`

This means the standard array pipeline reproduces the local one-off repair, not just the diagnostic script.

Update after merge/final jobs completed:

- final repaired outputs now exist:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chr1_q.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_relaxed035_20260531b/SCN/chr1.sds.tsv`
- repaired `chr1_q.sds.tsv` position range:
  - `min=125148509`
  - `max=248463050`
- old broken coarse-line `chr1_q.sds.tsv` position range:
  - `min=145551991`
  - `max=247870058`

Interpretation:

- the repaired formal chr1 output no longer starts at `145551991`;
- the missing early-q-arm band is now restored all the way back into the `125M` region in the merged q-arm table itself.

## 8. The same boundary-skip signature appears beyond chr1 in the broken coarse line

I compared, for the broken repaired coarse line, each chromosome's q-arm start from `chr*_b_file.txt` against the minimum retained position in `chr*_q.sds.tsv`.

Largest observed q-arm start offsets in `SCN olddefault` coarse output:

- `chr1`: `q_start=124932725`, `q_min_output=145551991`, `delta=20619266`
- `chr9`: `q_start=45518559`, `q_min_output=64058436`, `delta=18539877`
- `chr16`: `q_start=38265670`, `q_min_output=47520459`, `delta=9254789`
- `chr15`: `q_start=19725255`, `q_min_output=22853332`, `delta=3128077`
- `chr14`: `q_start=18173524`, `q_min_output=20217164`, `delta=2043640`

Many additional chromosomes show smaller but still non-zero q-arm start shifts around `0.7-1.6Mb`.

Interpretation:

- the `chr1` early-q-arm hole is the clearest case, but it is not unique;
- the same boundary-skip mechanism likely suppresses early q-arm coverage on multiple chromosomes in the broken coarse line;
- this argues that `skip_boundary_missing_fraction=0.10` is not merely a chr1-specific edge case for `SCN`.

Refinement:

- the `q_start -> q_min_output` comparison is a useful triage signal, but it can overstate the compute-stage gap when the filtered q-arm input itself has no very-early test SNPs.
- `chr9` is the clearest example:
  - `q_start=45518559`
  - but the first test SNP in `q_chunk_0` is already `60518620`
  - so the true first-chunk compute gap is closer to `64058436 - 60518620 = 3539816`, not the full `18539877`.

## 9. Broader targeted `relaxed035` repairs show heterogeneous rescue across other flagged q arms

I ran targeted `SCN` relaxed-threshold repairs for the other most-truncated q arms:

- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_relaxed035_flaggedq_20260531/SCN`
- targeted chromosomes:
  - `chr9`
  - `chr14`
  - `chr15`
  - `chr16`

All four targeted lines completed and produced merged outputs:

- `chr9.sds.tsv`, `chr9_q.sds.tsv`
- `chr14.sds.tsv`, `chr14_q.sds.tsv`
- `chr15.sds.tsv`, `chr15_q.sds.tsv`
- `chr16.sds.tsv`, `chr16_q.sds.tsv`

Split sizes:

- `chr9`
  - `p_chunks=52`, `q_chunks=74`
- `chr14`
  - `p_chunks=0`, `q_chunks=91`
- `chr15`
  - `p_chunks=1`, `q_chunks=84`
- `chr16`
  - `p_chunks=42`, `q_chunks=53`

First repaired q-chunk output sizes:

- `chr9 q_chunk_0`: `rows_out=7089`
- `chr14 q_chunk_0`: `rows_out=2253`
- `chr15 q_chunk_0`: `rows_out=8911`
- `chr16 q_chunk_0`: `rows_out=10782`

Merged q-arm start comparison:

- `chr9`
  - `q_start=45518559`
  - old `q_min_output=64058436`, `delta=18539877`
  - new `q_min_output=63611494`, `delta=18092935`
- `chr14`
  - `q_start=18173524`
  - old `q_min_output=20217164`, `delta=2043640`
  - new `q_min_output=19732789`, `delta=1559265`
- `chr15`
  - `q_start=19725255`
  - old `q_min_output=22853332`, `delta=3128077`
  - new `q_min_output=20232026`, `delta=506771`
- `chr16`
  - `q_start=38265670`
  - old `q_min_output=47520459`, `delta=9254789`
  - new `q_min_output=46780567`, `delta=8514897`

First test-SNP positions in repaired `q_chunk_0`:

- `chr9`: `first_input_pos=60518620`
- `chr14`: `first_input_pos=18223607`
- `chr15`: `first_input_pos=19783901`
- `chr16`: `first_input_pos=38266114`

Compute-stage gap relative to the first actual test SNP:

- `chr9`
  - old: `64058436 - 60518620 = 3539816`
  - new: `63611494 - 60518620 = 3092874`
- `chr14`
  - old: `20217164 - 18223607 = 1993557`
  - new: `19732789 - 18223607 = 1509182`
- `chr15`
  - old: `22853332 - 19783901 = 3069431`
  - new: `20232026 - 19783901 = 448125`
- `chr16`
  - old: `47520459 - 38266114 = 9254345`
  - new: `46780567 - 38266114 = 8514453`

Boundary-fraction sampling across `q_chunk_0` also shows why simply raising the threshold is not enough for the worst cases:

- `chr9`
  - at `pos=60518620`, `upstream_nan_frac=1.0000`
  - at `pos=61373121`, `upstream_nan_frac=0.8015`
  - at `pos=62910678`, `upstream_nan_frac=0.4529`
  - at `pos=63686678`, `upstream_nan_frac=0.2780`
- `chr16`
  - at `pos=38266114`, `upstream_nan_frac=1.0000`
  - at `pos=46411245`, `upstream_nan_frac=0.9867`
  - at `pos=46525139`, `upstream_nan_frac=0.6303`
  - at `pos=46831436`, `upstream_nan_frac=0.3196`
  - at `pos=47401489`, `upstream_nan_frac=0.1186`

Interpretation:

- `skip_boundary_missing_fraction=0.35` is clearly not a no-op outside chr1:
  - all four chromosomes now produce non-empty first q chunks;
  - `chr15` is strongly rescued;
  - `chr14` is moderately rescued.
- But the same parameter is not sufficient to fully repair every severe q-arm truncation:
  - `chr9` remains heavily truncated;
  - `chr16` remains heavily truncated;
  - both improve only modestly at the merged q-arm start.
- The sampled `upstream_nan_frac` values show why:
  - for the earliest `chr9` and especially `chr16` q-arm positions, the upstream missing fraction stays near `1.0` for a long stretch;
  - even a much looser threshold like `0.8` would only recover part of the remaining gap;
  - so the unresolved severe cases are not likely to be fixed by threshold inflation alone.

Pragmatic conclusion:

- the boundary-skip parameter is a real part of the broken-Manhattan root cause family;
- but `0.35` should not yet be treated as a universal one-size-fits-all SCN repair setting.
- Current evidence supports:
  - `chr1 relaxed035` as a validated repair;
  - broader flagged-q reruns as useful diagnostics and partial rescue;
  - additional per-chromosome tuning or deeper mechanism checks for the still-severe cases (`chr9`, `chr16`).

Sanity check on the newly recovered-only regions (`new POS < old_q_min_output`):

- `chr9`
  - `recovered_rows=12182`
  - `max_abs_rSDS=13.3661`
  - `>=20/50/100 = 0 / 0 / 0`
- `chr14`
  - `recovered_rows=12604`
  - `max_abs_rSDS=13.4512`
  - `>=20/50/100 = 0 / 0 / 0`
- `chr15`
  - `recovered_rows=17982`
  - `max_abs_rSDS=14.8796`
  - `>=20/50/100 = 0 / 0 / 0`
- `chr16`
  - `recovered_rows=12892`
  - `max_abs_rSDS=13.4004`
  - `>=20/50/100 = 0 / 0 / 0`

Interpretation:

- the relaxed-threshold partial rescues are not obviously reintroducing the catastrophic extreme-tail phenotype that characterized the broken Manhattan family;
- the remaining issue for `chr9/16` is insufficient early-q-arm recovery, not newly created ultra-extreme outliers.

## 10. Experimental `cap_to_boundary` mode cleanly rescues the most difficult early `chr16` segment

To test whether `chr9/16` need a different boundary rule rather than a larger skip threshold, I added an experimental mode to `compute_SDS.py`:

- `--boundary-missing-mode skip`
- `--boundary-missing-mode cap_to_boundary`

`cap_to_boundary` behavior:

- do not hard-drop SNPs solely because upstream/downstream missing fraction exceeds the skip threshold;
- instead, fill missing upstream/downstream intervals with the arm-boundary distance for that SNP.

This mode is fully parameterized through:

- `/data/home/grp-wangyf/xuyuan/sds/scripts/compute_SDS.py`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute_chunk.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/run_sds_compute_manifest_entry.sh`
- `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_sds_compute_chunked_chr.sh`

Micro-test on the hardest case, `chr16 q_chunk_0` first 2000 rows:

- input subchunk:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr16_qchunk0_head2k_captest_20260531/chr16_qhead_0000.feather`
- baseline `skip@0.35`:
  - `rows_in=2000`
  - `rows_out=0`
- experimental `cap_to_boundary@0.35`:
  - `rows_in=2000`
  - `rows_out=2000`
  - output:
    - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr16_qchunk0_head2k_captest_20260531/capmode.sds.tsv`

Recovered early-range summary:

- `min_pos=38266114`
- `max_pos=46411176`
- `max_abs_rSDS=8.9111`
- `>=20/50/100 = 0 / 0 / 0`

Interpretation:

- for the most difficult early `chr16` segment, the problem is not just “threshold too low”;
- a different boundary handling rule can restore coverage immediately, whereas `skip@0.35` keeps zero rows;
- the first 2000 rescued rows do not show any obvious extreme-tail blow-up.

Matching micro-test on `chr9 q_chunk_0` first 2000 rows:

- input subchunk:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr9_qchunk0_head2k_captest_20260531/chr9_qhead_0000.feather`
- baseline `skip@0.35`:
  - `rows_in=2000`
  - `rows_out=0`
- experimental `cap_to_boundary@0.35`:
  - `rows_in=2000`
  - `rows_out=2000`
  - output:
    - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr9_qchunk0_head2k_captest_20260531/capmode.sds.tsv`

Recovered early-range summary:

- `min_pos=60518620`
- `max_pos=61373108`
- `max_abs_rSDS=16.3023`
- `>=20/50/100 = 0 / 0 / 0`

Interpretation:

- the same non-threshold rule change also rescues the problematic earliest `chr9` segment immediately;
- as with `chr16`, the rescued rows do not recreate the catastrophic extreme-tail phenotype.

Current experimental conclusion:

- the unresolved `chr9/16` cases are better framed as a boundary-handling problem than a simple threshold-tuning problem;
- `cap_to_boundary` is now the strongest targeted next-step candidate for those still-severe early q-arm gaps.

Formal targeted `cap_to_boundary` repair launch status:

- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN`
- wrapper did eventually submit formal arrays and dependent merges:
  - `chr16`:
    - `487967` = `p` array
    - `487968` = `q` array
    - `487969` = `p` merge
    - `487970` = `q` merge
    - `487971` = final merge
  - `chr9`:
    - `487977` = `p` array
    - `487978` = `q` array
    - `487979` = `p` merge
    - `487980` = `q` merge
    - `487981` = final merge
- operational correction:
  - an extra manual `chr16` duplicate chain (`487972-487976`) was launched before it became clear that the wrapper had already submitted the same workdir.
  - that duplicate chain was explicitly terminated to avoid concurrent writes into the same chunk paths.

Pre-merge verification on formal `chr16 cap_to_boundary` q chunks:

- workdir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chunk_submit/chr16_scn_olddefault_capboundary_20260531_135220`
- current chunk completion:
  - `p_chunks/*.sds.tsv = 42 / 42`
  - `q_chunks/*.sds.tsv = 53 / 53`
- comparison baseline:
  - old `relaxed035` merged `chr16_q.sds.tsv` starts at `POS=46780567`
- formal `cap_to_boundary` q-chunk aggregate:
  - `rows=1050079`
  - `min_pos=38266114`
  - `max_abs_rSDS=15.1635`
  - `>=20/50/100 = 0 / 0 / 0`
- recovered-only range relative to old `relaxed035` q start:
  - `old_q_min=46780567`
  - `recovered_rows=9218`
  - `recovered_max_abs_rSDS=15.1635`
  - `>=20/50/100 = 0 / 0 / 0`

Interpretation:

- formal whole-`chr16` `cap_to_boundary` has already recovered the early q-arm gap at the chunk-output level, before merge finalization;
- the recovered-only `chr16` region still shows no `>=20` extreme tail, so the repair is not simply trading truncation for a new Manhattan artifact.

Formal merged-output verification on `chr16`:

- final outputs now exist:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr16_q.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr16_p.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr16.sds.tsv`
- wrapper `chr16` chain status:
  - `487967-487971 = DONE`
- merged `chr16_q.sds.tsv` matches the pre-merge aggregate exactly:
  - `rows=1050079`
  - `min_pos=38266114`
  - `max_abs_rSDS=15.1635`
  - `>=20/50/100 = 0 / 0 / 0`
  - recovered-only relative to old `relaxed035` q start:
    - `old_q_min=46780567`
    - `recovered_rows=9218`
    - `recovered_max_abs_rSDS=15.1635`
    - `>=20/50/100 = 0 / 0 / 0`

Current `chr9` formal run status:

- wrapper `chr9` chain is now fully done:
  - `487977-487981 = DONE`
- final outputs now exist:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr9_q.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr9_p.sds.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_capboundary_chr9_16_20260531/SCN/chr9.sds.tsv`
- merged `chr9_q.sds.tsv` summary:
  - old `relaxed035` q start = `63611494`
  - repaired q start = `60518620`
  - `rows=1474780`
  - `max_pos=138318264`
  - `max_abs_rSDS=16.3023`
  - `>=20/50/100 = 0 / 0 / 0`
- recovered-only relative to old `relaxed035` q start:
  - `recovered_rows=12911`
  - `recovered_max_abs_rSDS=16.3023`
  - `>=20/50/100 = 0 / 0 / 0`
- the repaired q-arm starts exactly at the first actual problematic site:
  - `chr9:60518620:C:T`

Local validation merge for full `chr9`:

- before the dependent merge jobs flushed their outputs, a read-only validation merge was built under:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr9_capboundary_validation_20260531/chr9_q.capboundary_validation.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/chr9_capboundary_validation_20260531/chr9.capboundary_validation.tsv`
- validation file sizes:
  - `chr9_q.capboundary_validation.tsv = 1421781` lines at the time of assembly
  - `chr9.capboundary_validation.tsv = 2449715` lines at the time of assembly
- full-chromosome validation summary:
  - `rows=2449714`
  - `min_pos=10485`
  - `max_pos=138318264`
  - `max_abs_rSDS=16.3023`
  - `>=20/50/100 = 0 / 0 / 0`
- the later official merged `chr9.sds.tsv` is consistent with the same qualitative conclusion:
  - `rows=2502714`
  - `min_pos=10485`
  - `max_pos=138318264`
  - `max_abs_rSDS=16.3023`
  - `>=20/50/100 = 0 / 0 / 0`

Cross-chromosome repair-policy summary:

- summary table:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/scn_qarm_repair_policy_summary_20260531.tsv`
- verified chromosome-level outcomes:
  - `chr1`:
    - coarse olddefault `q` start `145551991 -> 125148509` under `relaxed035`
    - recovered-only `28774` rows, `recovered_max_abs_rSDS=17.1859`, `>=20=0`
  - `chr14`:
    - coarse olddefault `q` start `20217164 -> 19732789` under `relaxed035`
    - recovered-only `12604` rows, `recovered_max_abs_rSDS=13.4512`, `>=20=0`
  - `chr15`:
    - coarse olddefault `q` start `22853332 -> 20232026` under `relaxed035`
    - recovered-only `17982` rows, `recovered_max_abs_rSDS=14.8796`, `>=20=0`
  - `chr16`:
    - relaxed035 `q` start `46780567 -> 38266114` under `cap_to_boundary`
    - recovered-only `9218` rows, `recovered_max_abs_rSDS=15.1635`, `>=20=0`
  - `chr9`:
    - relaxed035 `q` start `63611494 -> 60518620` under `cap_to_boundary`
    - recovered-only `12911` rows, `recovered_max_abs_rSDS=16.3023`, `>=20=0`

Minimal broader repair policy supported by current evidence:

- `skip_boundary_missing_fraction=0.35` is sufficient for chromosomes that show moderate early q-arm loss but already rescue cleanly without tail relapse (`chr1`, `chr14`, `chr15`).
- `boundary_missing_mode=cap_to_boundary` is warranted for chromosomes where `0.35` still leaves a large early q-arm truncation (`chr9`, `chr16`).
- Current evidence supports a hybrid policy more strongly than a one-size-fits-all global switch:
  - keep `relaxed035` as the default repair mode
  - escalate to `cap_to_boundary` only on residual flagged q-arm chromosomes with demonstrated severe post-`0.35` gaps
- This hybrid policy is the narrowest repair rule that currently explains the anomaly and fixes it while preserving plausible biology.

Hybrid genome-wide stitch attempt:

- assembled hybrid output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_hybridqarm_20260531/SCN`
- chromosome-source manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_hybridqarm_20260531/hybrid_sources.tsv`
- stitching rule:
  - keep coarse olddefault genome-wide output for untouched chromosomes
  - replace `chr1` with `relaxed035`
  - replace `chr14/15` with `relaxed035 flaggedq`
  - replace `chr9/16` with formal `cap_to_boundary`

Operational note:

- the first local postprocess attempt failed because the hybrid builder naively created `chr14_p.sds.tsv`, but neither the coarse baseline nor the `relaxed035 flaggedq` source contains a real `chr14_p` file.
- after rebuilding the hybrid directory to link only files that actually exist, local postprocess was able to generate:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_hybridqarm_20260531/SCN/SCN.normalized.tsv`
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_hybridqarm_20260531/SCN/SCN.manhattan_points.tsv`
- however, that local run did not finish writing the later artifacts such as `SCN.frequency_bins.tsv`, `SCN.postprocess_summary.tsv`, or `SCN.manhattan.png`; the manual log stayed empty, so this is not yet a clean finalized postprocess chain.

Hybrid normalized / Manhattan-point evidence:

- line counts relative to coarse baseline:
  - hybrid `SCN.normalized.tsv = 49,510,525` lines
  - coarse `SCN.normalized.tsv = 54,164,890` lines
  - hybrid `SCN.manhattan_points.tsv = 6,243,932` lines
  - coarse `SCN.manhattan_points.tsv = 6,984,497` lines
- common-variant extreme-tail counts from `norm_SDS`:
  - hybrid:
    - `common=6,987,648`
    - `|norm_SDS|>=8: 134`
    - `|norm_SDS|>=10: 47`
    - `|norm_SDS|>=15: 7`
    - `|norm_SDS|>=20: 4`
  - coarse:
    - `common=6,984,534`
    - `|norm_SDS|>=8: 1`
    - `|norm_SDS|>=10/15/20: 0 / 0 / 0`
- top hybrid Manhattan hits are not a diffuse multi-chromosome artifact anymore; they concentrate mainly in:
  - `chr1 125.149-125.166 Mb`
  - `chr9 ~60.518-60.550 Mb`
- but the peak magnitudes are now too extreme to accept blindly as final plausible biology:
  - top hybrid `-log10(p)` values reach `172.17`, `171.93`, `166.41`, `156.49`
  - all four strongest hits come from `chr1 ~125.15 Mb`
  - a representative common variant is:
    - `chr1:125149434:T:C`
    - `MAF=0.4949`
    - `rSDS=1.6951`
    - `BIN_MEAN=0.00673`
    - `BIN_SD=0.06127`
    - `norm_SDS=27.5547`
    - `p_bothside=3.8896e-167`

Interpretation of the hybrid stitch:

- direct chromosome-level repair plus simple genome-wide re-normalization does recover the missing q-arm coverage, but it also produces a new second-order issue:
  - highly localized repaired signals at `chr1/9` become ultra-extreme common-variant peaks after the global normalization step.
- this is not the old “many chromosomes all explode” failure mode.
- but it also means the hybrid stitched output cannot yet be treated as the final repaired Manhattan that satisfies plausible biology.
- The current state is therefore:
  - boundary-skipping root cause is solved at the chromosome-output level
  - a naive genome-wide stitch is not yet an acceptable final repair product
  - the remaining work is to determine whether the final fix should be:
    - a clean genome-wide rerun under the hybrid boundary policy
    - or an adjustment to the normalization/postprocess layer when repaired chromosomes re-enter the full genome-wide pool

Conclusion

- repaired `SCN olddefault` loses `chr1 125M-145.5M` coverage at compute stage, not input stage.
- The direct cause is the `compute_SDS.py` boundary guard:
  - `skip_boundary_missing_fraction=0.10`
  - too many `SCN` samples have no upstream singleton for early q-arm SNPs.
- This explains why `SCN` had `row_count=0` in the `NCN` `125M / 143M` windows even though:
  - the test SNPs are present in `t_file`
  - the windows do have singleton support in `s_file`
- A concrete repair path now exists:
  - raise the boundary-missing tolerance to `0.35` as the baseline repair
  - then use `cap_to_boundary` specifically on chromosomes where early q-arm truncation remains severe after `0.35`
  - this repairs the affected q-arm chunks without recreating the old extreme-tail Manhattan artifact
- But a second constraint is now also established:
  - simply stitching the repaired chromosomes back into a genome-wide output and re-running postprocess can create new ultra-extreme localized common-variant peaks
  - so the final paper-facing repair still needs one more pass at the genome-wide normalization layer

Next step

- the next meaningful move is no longer root-cause hunting on `chr9/16`; that part is already resolved.
- the remaining step toward a paper-quality repaired `SCN` output is to decide whether to:
  - stitch a hybrid repaired genome-wide `SCN` run from the verified chromosome-level outputs
  - or relaunch a broader genome-wide repair that applies the hybrid rule directly during compute
- the hybrid stitch experiment suggests the second route is now more defensible:
  - use the verified boundary policy during compute
  - then inspect whether the resulting genome-wide normalization still generates the same `chr1/9` over-amplification
- whichever route is chosen, the current evidence already fixes the mechanism:
  - early q-arm Manhattan holes were caused by boundary skipping
  - the narrow repair consistent with plausible biology is `relaxed035` by default plus targeted `cap_to_boundary` escalation on residual hard cases

Clean genome-wide rerun under the hybrid boundary policy:

- launch script added:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_hybridqarm_submitter_array.sh`
- submitter entry script updated to forward boundary policy from environment:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_submitter_array_entry.sh`
- policy encoded in the launch:
  - `relaxed035` group:
    - chromosomes `1,2,3,4,5,6,7,8,10,11,12,13,14,15,17,18,19,20,21,22`
    - `SDS_SKIP_BOUNDARY_MISSING_FRACTION=0.35`
  - `cap_to_boundary` group:
    - chromosomes `9,16`
    - `SDS_SKIP_BOUNDARY_MISSING_FRACTION=0.35`
    - `SDS_BOUNDARY_MISSING_MODE=cap_to_boundary`
- launch manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_hybridqarm_submitter_array_20260531_174923_177002/manifest.tsv`
- live jobs:
  - `487996` = relaxed submitter array
  - `487997` = cap submitter array
  - `487998` = dependent finalizer / postprocess launcher
- current queue state at launch check:
  - `487996` running/pending as expected on `smp`
  - `487997` already running with one task finished
  - `487998` pending on dependency

Interpretation:

- the project is no longer at the “infer policy from stitched hybrids” stage only;
- a clean genome-wide recomputation is now in flight under the narrowest policy currently supported by evidence;
- this rerun is the decisive test for whether the hybrid-policy compute layer avoids the second-order `chr1/9` over-amplification seen in the naive stitch-and-renormalize attempt.

Early evidence from the clean rerun:

- current state check:
  - `487997` (`cap` submitter array) has finished
  - `487996` (`relaxed` submitter array) is still running
  - `487998` (dependent finalizer/postprocess launcher) is still pending on dependency
  - chromosome-level final jobs currently recorded in the state tree:
    - `chr16.final_job.tsv -> 488003`
    - `chr9.final_job.tsv -> 488008`
    - `chr1.final_job.tsv -> 488013`
    - `chr2.final_job.tsv -> 488018`
- live queue inspection shows:
  - `488003` (`chr16` final) is `DONE`
  - `488008` (`chr9` final) is still `PEND`
  - `488013` (`chr1` final) is still `PEND`
  - `488018` (`chr2` final) is still `PEND`
- this explains why the output root currently contains only:
  - `chr16.sds.tsv`
  - `chr16_p.sds.tsv`
  - `chr16_q.sds.tsv`
  - `chr9_p.sds.tsv`
  - and not yet the full set of per-chromosome merged outputs.

`chr16` clean-rerun comparison:

- `chr16_q.sds.tsv` from the clean rerun is byte-identical to the previous standalone `cap_to_boundary` repair:
  - clean rerun md5:
    - `1ba19f9a786e054a122c558067ab0d5e`
  - standalone repair md5:
    - `1ba19f9a786e054a122c558067ab0d5e`
- so the decisive early-q-arm rescue is preserved exactly in the clean recomputation.
- but `chr16_p.sds.tsv` is not identical:
  - clean rerun:
    - `rows=837722`
    - `max_abs_rSDS=14.9386`
    - `>=20/50/100 = 0/0/0`
  - previous standalone `cap_to_boundary` run:
    - `rows=648222`
    - `max_abs_rSDS=14.9386`
    - `>=20/50/100 = 0/0/0`
- interpretation:
  - the clean genome-wide hybrid policy is not only preserving the repaired `chr16_q` signal;
  - it is also changing the `chr16_p` site universe by recovering an additional `189500` rows without creating new extreme tail.
  - this matters because the remaining genome-wide normalization behavior will depend on the full repaired site universe, not only on q-arm rescue.

Implication for the final normalization question:

- the earlier hybrid stitch produced:
  - `common=6987648`
  - `|norm_SDS|>=8: 134`
  - `>=10: 47`
  - `>=15: 7`
  - `>=20: 4`
- while the coarse baseline had:
  - `common=6984534`
  - `|norm_SDS|>=8: 1`
  - `>=10/15/20: 0/0/0`
- the clean rerun now shows an important structural difference from the naive stitch:
  - at least one chromosome already differs at the merged-output level (`chr16_p`) even when the repaired q-arm itself (`chr16_q`) is preserved exactly.
- therefore the clean rerun is not a redundant re-expression of the naive stitch.
- it remains the right decisive test for whether recomputing the full repaired site universe under one policy avoids the stitched normalization failure.

Important confound discovered after inspecting the clean rerun state tree:

- the current hybrid-policy clean rerun is **not** a pure boundary-policy-only validation.
- direct evidence:
  - launch manifest:
    - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_hybridqarm_submitter_array_20260531_174923_177002/manifest.tsv`
    - records `chunk_rows` as blank
  - per-chromosome submitter outputs:
    - `chr16.submitter.out`
    - `chr9.submitter.out`
    - `chr1.submitter.out`
    - all record `CHUNK_ROWS = 5000`
  - by contrast, the earlier coarse genome-wide submitter manifest:
    - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_183757_121492/manifest.tsv`
    - records `chunk_rows = 20000`
  - script-level cause:
    - `launch_scn_olddefault_genomewide_hybridqarm_submitter_array.sh` had left `CHUNK_ROWS=""`
    - `submit_sds_compute_chunked_chr.sh` therefore fell back to:
      - `sds_queue_default_chunk_rows(normal) = 5000`
- this explains the `chr16_p` discrepancy exactly:
  - `chr16_p.filter.csv` is identical between the clean rerun and the earlier standalone `cap_to_boundary` run:
    - both use the same rebuilt input
    - both have `rows=837722`
    - both cover `start=1, end=36311157`
  - but `chr16_p.manifest.tsv` differs:
    - clean rerun: `168` lines
    - standalone `cap_to_boundary`: `42` lines
  - this is the expected `4x` split pattern from `5000`-row versus `20000`-row chunking.

Interpretation of the confound:

- the previously observed `chr16_p` row difference:
  - clean rerun `chr16_p.sds.tsv = 837722 rows`
  - standalone `cap_to_boundary chr16_p.sds.tsv = 648222 rows`
  - cannot yet be attributed purely to a cleaner genome-wide recomputation or repaired site-universe effect.
- the current evidence says something narrower but stronger:
  - the clean rerun changed both:
    - boundary policy
    - chunk granularity
- therefore this in-flight hybrid rerun is scientifically informative, but it is **not** an apples-to-apples decisive test against the earlier `20000`-row repaired chromosome outputs.
- the correct next decisive validation must hold chunking constant at `20000` while applying the hybrid boundary policy.

Corrected decisive rerun launched after fixing the chunking confound:

- script fix:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_hybridqarm_submitter_array.sh`
  - `CHUNK_ROWS` is now explicitly set to `20000` by default so the hybrid-policy launch cannot silently fall back to the queue default `5000`.
- corrected launch command used a fresh output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b`
- corrected manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_hybridqarm_submitter_array_20260531_225228_25405/manifest.tsv`
- corrected manifest records:
  - `chunk_rows = 20000`
  - `relaxed_skip_boundary_missing_fraction = 0.35`
  - `cap_skip_boundary_missing_fraction = 0.35`
  - `cap_boundary_missing_mode = cap_to_boundary`
- corrected live jobs:
  - `488122` = relaxed submitter array
  - `488123` = cap submitter array
  - `488124` = dependent finalizer / postprocess launcher
- immediate queue state at launch check:
  - `488122` running with the first relaxed tasks active
  - `488123` running with both cap tasks active
  - `488124` pending on dependency

Current status after this correction:

- there are now two distinct genome-wide hybrid-policy lines:
  - exploratory `5000`-row line:
    - manifest `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_hybridqarm_submitter_array_20260531_174923_177002/manifest.tsv`
    - jobs `487996 / 487997 / 487998`
    - useful for showing that hybrid boundary handling changes outputs, but confounded by chunk granularity
- corrected `20000`-row line:
  - manifest `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_hybridqarm_submitter_array_20260531_225228_25405/manifest.tsv`
  - jobs `488122 / 488123 / 488124`
  - this is now the decisive apples-to-apples genome-wide validation against the earlier coarse / chromosome-level repaired outputs

Early structural validation of the corrected `20000`-row line:

- `chr16` is the first chromosome to reach the submitter-complete stage in the corrected line.
- its submitter output records:
  - `CHUNK_ROWS = 20000`
  - `SKIP_BOUNDARY_MISSING_FRACTION = 0.35`
  - `BOUNDARY_MISSING_MODE = cap_to_boundary`
  - `P_ARRAY_JOB = 488125`
  - `Q_ARRAY_JOB = 488126`
  - `P_MERGE_JOB = 488127`
  - `Q_MERGE_JOB = 488128`
  - `FINAL_JOB = 488129`
- critical apples-to-apples check:
  - corrected `chr16_p.manifest.tsv` has `42` chunks
  - corrected `chr16_q.manifest.tsv` has `53` chunks
  - these exactly match the earlier standalone `cap_to_boundary` repair:
    - `chr16_p.manifest.tsv = 42`
    - `chr16_q.manifest.tsv = 53`
- interpretation:
  - the corrected line has now removed the `5000`-row chunking confound at the actual chromosome-workdir level, not just at launch-script intent level.
  - this is the first direct proof that the new decisive rerun is structurally comparable to the earlier repaired chromosome outputs.

Live corrected-line status after the first structural validation:

- the corrected result tree now has `3` submitter-complete chromosomes recorded:
  - `chr1.final_job.tsv`
  - `chr9.final_job.tsv`
  - `chr16.final_job.tsv`
- but the corrected output root still has:
  - `root_chr_sds_count = 0`
  - i.e. no merged `chr*.sds.tsv` have landed yet
- this means the decisive line is still in the chromosome-internal array/merge stage, not yet in genome-wide postprocess territory.

Detailed `chr16` live state:

- `chr16` final-job chain exists:
  - `P_ARRAY_JOB = 488125`
  - `Q_ARRAY_JOB = 488126`
  - `P_MERGE_JOB = 488127`
  - `Q_MERGE_JOB = 488128`
  - `FINAL_JOB = 488129`
- current queue state:
  - `488125` (`chr16_p` chunk array) running
  - `488126` (`chr16_q` chunk array) running
  - `488127/488128/488129` still pending on their upstream dependencies
- this is the expected healthy state:
  - no sign of another hidden finalizer or merge failure
  - just active chunk-array execution followed by merge/final jobs

Additional concrete evidence from the `chr16` workdir:

- `chr16_q.filter.csv` is present and records:
  - `start=38265670`
  - `end=90338345`
  - `rows=1050079`
- `chr16_q.manifest.tsv = 53`
- many `q_chunk_*.sds.tsv` files are already materializing under:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/chunk_submit/chr16_scn_olddefault_hybrid_cap_submitter_array_chr16_20260531_225230/q_chunks`
- this confirms the corrected line is not stalled at the split/filter stage anymore; it is actively executing the repaired per-chunk SDS compute at the intended `20000`-row granularity.

Expanded structural validation after additional corrected-line progress:

- the corrected result tree has now advanced to `4` submitter-complete chromosomes:
  - `chr1.final_job.tsv`
  - `chr2.final_job.tsv`
  - `chr9.final_job.tsv`
  - `chr16.final_job.tsv`
- direct submitter outputs confirm the intended policies for all four:
  - `chr1`:
    - `CHUNK_ROWS = 20000`
    - `SKIP_BOUNDARY_MISSING_FRACTION = 0.35`
    - `BOUNDARY_MISSING_MODE = ""`
  - `chr2`:
    - `CHUNK_ROWS = 20000`
    - `SKIP_BOUNDARY_MISSING_FRACTION = 0.35`
    - `BOUNDARY_MISSING_MODE = ""`
  - `chr9`:
    - `CHUNK_ROWS = 20000`
    - `SKIP_BOUNDARY_MISSING_FRACTION = 0.35`
    - `BOUNDARY_MISSING_MODE = cap_to_boundary`
  - `chr16`:
    - `CHUNK_ROWS = 20000`
    - `SKIP_BOUNDARY_MISSING_FRACTION = 0.35`
    - `BOUNDARY_MISSING_MODE = cap_to_boundary`

`chr9` apples-to-apples chunk-structure check:

- corrected `chr9_p.manifest.tsv = 52`
- corrected `chr9_q.manifest.tsv = 74`
- these exactly match the earlier standalone `cap_to_boundary` repair for `chr9`:
  - `chr9_p.manifest.tsv = 52`
  - `chr9_q.manifest.tsv = 74`
- this extends the structural validation beyond `chr16` and shows that the corrected decisive line is reproducing the intended `20000`-row chunk regime on both cap-policy chromosomes.

Current final-job state of the corrected line:

- direct `rtk bjobs` inspection shows:
  - `488129` (`chr16` final) = `PEND`
  - `488134` (`chr9` final) = `PEND`
  - `488139` (`chr1` final) = `PEND`
  - `488144` (`chr2` final) = `PEND`
- interpretation:
  - these are healthy dependency waits, not a recurrence of the old finalizer failure mode.
  - the corrected line still has `root_chr_sds_count = 0`, so the work remains inside per-chromosome array/merge execution, but the active structure is now internally consistent with the repaired chromosome baselines.

Independent engineering blocker discovered and fixed:

- the exploratory hybrid-policy line (`487996 / 487997 / 487998`) did not merely “fail to finish postprocess in time”.
- its finalizer job `487998` exited immediately with:
  - `488003: No matching job found. Job not submitted.`
- root cause:
  - `submit_postprocess_from_final_jobs.sh` constructed a `bsub -w "done(jobid) && ..."` dependency from `chr*.final_job.tsv`
  - once LSF no longer recognized one of those historical final-job IDs, the postprocess submission itself failed
- script fix:
  - `submit_postprocess_from_final_jobs.sh` now waits on the actual required files:
    - `chr*_p.sds.tsv`
    - `chr*_q.sds.tsv`
  - and only then submits postprocess without a direct `done(jobid)` dependency
- this fix is important for the corrected decisive line because `488124` will execute the patched script when its array dependencies clear.

Exploratory line current salvage status:

- it currently has:
  - `22` `chr*.final_job.tsv`
  - `56` `chr*.sds.tsv` files under the output root
- but it is still missing `7` required postprocess inputs:
  - `chr1_p.sds.tsv`
  - `chr1_q.sds.tsv`
  - `chr2_p.sds.tsv`
  - `chr2_q.sds.tsv`
  - `chr13_p.sds.tsv`
  - `chr14_p.sds.tsv`
  - `chr9_q.sds.tsv`
- so even after the finalizer-script fix, that exploratory `5000`-row tree is not yet postprocessable as a complete genome-wide input set.
- this further supports keeping it as an exploratory artifact line, not as the paper-facing repair candidate.

Additional engineering root cause discovered while validating corrected `chr16`:

- the previously observed `chr16_p` row-count discrepancy between:
  - corrected hybrid-policy `chunk20` line:
    - `chr16_p.sds.tsv = 837722` rows
  - old standalone `cap_to_boundary` line:
    - `chr16_p.sds.tsv = 648222` rows
  is not a new biological or algorithmic difference.
- direct chunk-level audit shows that the old standalone `chr16_p` tree is affected by `NUL`-corrupted chunk `TSV`s:
  - `20` `p_chunk_*.sds.tsv` files contain embedded `NUL` bytes
  - total `NUL` burden across old `chr16_p` chunk `TSV`s:
    - `11,285,432` bytes
  - the affected chunks are concentrated in early `p` chunks:
    - `p_chunk_0` through `p_chunk_19`
  - their per-chunk `*.csv` summaries still report the expected full outputs:
    - e.g. old `p_chunk_1.csv` says `rows_in=20000`, `rows_out=20000`
  - but the corresponding old `p_chunk_1.sds.tsv` contains `582,210` `NUL` bytes and is only readable as `10000` rows by line-oriented tools
- summing across old `chr16_p` chunk summaries versus actual old `TSV` rows gives:
  - expected from `*.csv`: `837722`
  - actual readable `TSV` rows: `648222`
  - deficit: `189500`
- this exactly matches the corrected-vs-old `chr16_p` row delta:
  - `837722 - 648222 = 189500`
- scope check:
  - old standalone `chr9_p.sds.tsv`, `chr9_q.sds.tsv`, and `chr16_q.sds.tsv` all have `NUL=0`
  - the `NUL` corruption is therefore localized to old standalone `chr16_p`, not a genome-wide property of the cap-policy repair
  - corrected hybrid-policy merged files currently examined (`chr16_p/q`, `chr9_p/q`) all have `NUL=0`
- interpretation:
  - the old standalone `chr16_p` undercount should not be interpreted as a cap-policy/computation difference
  - it is a file-integrity artifact in the old standalone validation line
  - the corrected hybrid-policy `chunk20` line should be treated as the cleaner source of truth for `chr16_p` while the full decisive genome-wide rerun is still finishing

Latest decisive-line live status (`2026-06-01`):

- chromosome-final jobs now show a clear near-complete frontier:
  - `DONE`: `chr1-19`
    - representative final jobs:
      - `chr1 = 488139`
      - `chr9 = 488134`
      - `chr16 = 488129`
      - `chr19 = 488220`
  - `PEND`: `chr20-22`
    - `chr20 = 488226`
    - `chr21 = 488231`
    - `chr22 = 488236`
- root output tree currently has partial but substantial merged coverage:
  - `chr1-12`, `chr15-17` already have full triplets:
    - `chrN_p.sds.tsv`
    - `chrN_q.sds.tsv`
    - `chrN.sds.tsv`
  - `chr13-14` are missing one arm-level `TSV` each, but their chromosome-level archive logs already exist
  - `chr18-20` currently only have `p`-side root files/logs
  - `chr21-22` have not yet materialized root `TSV`s
- root `chr*.sds.tsv` count at this checkpoint:
  - `21`
- corrected hybrid-policy root files checked so far remain clean:
  - `chr16_p.sds.tsv`: `NUL = 0`, `rows = 837722`
  - `chr16_q.sds.tsv`: `NUL = 0`, `rows = 1050079`
  - `chr9_p.sds.tsv`: `NUL = 0`, `rows = 1027934`
  - `chr9_q.sds.tsv`: `NUL = 0`, `rows = 1474780`
- the new postprocess finalizer `488124` is not blocked by the old dependency bug anymore; its current state is a plain queue delay:
  - `PEND`
  - `PENDING REASON: Job slot limit reached: 34 hosts`
- interpretation:
  - the decisive `chunk20` line is still actively converging to a full genome-wide repaired tree
  - no new evidence currently suggests a relapse of the earlier boundary artifact or the old standalone `chr16_p` file-integrity corruption
  - the next decisive evidence point remains the first successful genome-wide postprocess (`SCN.normalized.tsv`, `SCN.frequency_bins.tsv`, `SCN.manhattan_points.tsv`, `SCN.manhattan.png`) from this corrected line

Second postprocess blocker found and fixed (`2026-06-01`):

- after all chromosome-final jobs converged, the first corrected-line finalizer `488124` entered `RUN` but did not progress to postprocess outputs.
- root cause:
  - `submit_postprocess_from_final_jobs.sh` had been fixed earlier to wait on actual files instead of stale `done(jobid)`, but it still hard-required every `chr*_p.sds.tsv` and `chr*_q.sds.tsv`
  - this is invalid for chromosomes whose `p` arm is legitimately skipped
- direct evidence from corrected submitter state:
  - `chr13.submitter.out`:
    - `P_ARRAY_JOB = SKIP`
    - `P_MERGE_JOB = SKIP`
    - `Q_ARRAY_JOB = 488193`
    - `FINAL_JOB = 488195`
  - `chr14.submitter.out`:
    - `P_ARRAY_JOB = SKIP`
    - `P_MERGE_JOB = SKIP`
    - `Q_ARRAY_JOB = 488202`
    - `FINAL_JOB = 488204`
  - corrected root therefore correctly contains:
    - `chr13_q.sds.tsv` and `chr13.sds.tsv`
    - `chr14_q.sds.tsv` and `chr14.sds.tsv`
  - but does not contain:
    - `chr13_p.sds.tsv`
    - `chr14_p.sds.tsv`
- fix:
  - `submit_postprocess_from_final_jobs.sh` now parses each `chr*.submitter.out` and only waits for arms whose array job is not `SKIP`
- operational recovery:
  - stalled finalizer `488124` was terminated
  - corrected postprocess was resubmitted successfully
  - new jobs:
    - `postprocess_job = 488240`
    - `diag_job = 488241`

Stable full corrected-line postprocess outputs now landed:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/SCN.normalized.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/SCN.frequency_bins.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/SCN.manhattan_points.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/SCN.manhattan.png`

This is the decisive result for the corrected full rerun:

- stable common-variant tail counts from `SCN.normalized.tsv`:
  - `common = 7,109,812`
  - `|norm_SDS| >= 8: 137`
  - `|norm_SDS| >= 10: 48`
  - `|norm_SDS| >= 15: 7`
  - `|norm_SDS| >= 20: 4`
- these counts are effectively the same as the earlier naive hybrid-stitch result:
  - previous hybrid stitch:
    - `common = 6,987,648`
    - `>=8: 134`
    - `>=10: 47`
    - `>=15: 7`
    - `>=20: 4`
- therefore:
  - the earlier ultra-extreme common-variant tail is **not** explained solely by naive chromosome stitch assembly
  - a clean full genome-wide rerun under the same repaired boundary policy reproduces essentially the same inflation pattern
  - the remaining root cause must now be treated as a downstream normalization / site-universe / MAF-bin behavior issue, not merely a stitching artifact

Current stable concentration of the tail:

- `|norm_SDS| >= 8` variants cluster only on:
  - `chr1: 76`
  - `chr9: 39`
  - `chr16: 22`
- the `>=20` ultra-extreme common variants are all in one narrow `chr1` window:
  - `chr1:125149434`
  - `chr1:125159258`
  - `chr1:125160092`
  - `chr1:125166213`
- strongest stable examples:
  - `chr1:125166213:T:A`
    - `MAF = 0.4987`
    - `rSDS = 1.7243`
    - `norm_SDS = 28.0043`
    - `p_bothside = 1.4384e-172`
  - `chr9:61696894:A:C`
    - `MAF = 0.4716`
    - `rSDS = -1.3045`
    - `norm_SDS = -17.6058`
    - `p_bothside = 2.2227e-69`
- interpretation:
  - raw `rSDS` itself is not exploding genome-wide
  - the inflation is being created when moderate raw signals are divided by very small high-MAF bin `SD`s during normalization
  - the full-repair line is therefore still not yet a final paper-facing “plausible biology” Manhattan

Stable diagnostics after `488241`:

- diagnostic outputs landed under:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b/SCN/diagnostics`
- key files:
  - `SCN.gravel_chb_ne100k_submitter_array.summary.tsv`
  - `SCN.gravel_chb_ne100k_submitter_array.chrom_tail_counts.tsv`
  - `SCN.gravel_chb_ne100k_submitter_array.peak_windows.tsv`
  - `SCN.gravel_chb_ne100k_submitter_array.top_hits.tsv`
  - `SCN.gravel_chb_ne100k_submitter_array.chrom_tail_counts.png`
- diagnostic tail summaries show the residual anomaly is not genome-wide diffuse; it is concentrated in a few repaired windows:
  - strongest `100 kb` windows:
    - `chr1:125.10-125.20 Mb`
      - `hit_count = 36`
      - `max_neg_log10_p = 171.84`
      - `top_id = chr1:125166213:T:A`
    - `chr9:61.60-61.70 Mb`
      - `hit_count = 16`
      - `max_neg_log10_p = 68.65`
      - `top_id = chr9:61696894:A:C`
    - `chr16:46.40-46.50 Mb`
      - `hit_count = 7`
      - `max_neg_log10_p = 24.42`
      - `top_id = chr16:46452087:A:C`
    - additional secondary windows:
      - `chr9:61.70-61.80 Mb`
      - `chr1:144.40-144.50 Mb`
- chromosome-level diagnostic tail counts (`-log10 p` thresholds) also concentrate in the same places:
  - `chr1`: `count_neglog10_ge_20 = 40`, `count_neglog10_ge_50 = 4`, `count_neglog10_ge_100 = 4`
  - `chr9`: `count_neglog10_ge_20 = 21`, `count_neglog10_ge_50 = 3`
  - `chr16`: `count_neglog10_ge_20 = 7`
- interpretation update:
  - the corrected full rerun does **not** leave us with a diffuse genome-wide artifact anymore
  - but it still leaves a non-plausible concentration of ultra-significant common-variant peaks in repaired early-q-arm windows, especially:
    - `chr1:125.10-125.20 Mb`
    - `chr9:61.60-61.80 Mb`
    - `chr16:46.40-46.50 Mb`
  - this shifts the remaining root-cause question from “whole-genome Manhattan is broken” to:
    - why the repaired boundary-policy windows themselves carry inflated local common-variant signal after normalization
  - therefore the pipeline is materially closer to plausible biology than before, but the end state is still **not yet achieved**

Window-level comparison against coarse and standalone repaired lines:

- targeted comparison windows:
  - `chr1_peak125 = chr1:125.10-125.20 Mb`
  - `chr9_peak616 = chr9:61.60-61.70 Mb`
  - `chr16_peak464 = chr16:46.40-46.50 Mb`
- direct `chr*.sds.tsv` window summaries show:
  - coarse genome-wide baseline:
    - `chr1_peak125`: `row_count = 0`
    - `chr9_peak616`: `row_count = 0`
    - `chr16_peak464`: `row_count = 0`
  - corrected full rerun:
    - `chr1_peak125`:
      - `row_count = 2118`
      - `max_abs_rSDS = 17.1859`
      - `top_id = chr1:125177188:C:A`
    - `chr9_peak616`:
      - `row_count = 2946`
      - `max_abs_rSDS = 14.3323`
      - `top_id = chr9:61676023:C:G`
    - `chr16_peak464`:
      - `row_count = 3127`
      - `max_abs_rSDS = 15.1635`
      - `top_id = chr16:46432524:A:T`
  - corresponding standalone repaired lines reproduce the same window summaries exactly:
    - `chr1_relaxed035` reproduces corrected `chr1_peak125` exactly
    - `chr9_16_capboundary` reproduces corrected `chr9_peak616` exactly
    - `chr9_16_capboundary` reproduces corrected `chr16_peak464` exactly
- interpretation:
  - the corrected full genome rerun is **not** creating these hotspot windows de novo
  - it is faithfully inheriting them from the standalone repaired chromosome lines
  - therefore the remaining anomaly is now pinned one level deeper:
    - not to full-genome assembly or postprocess orchestration
    - but to the repaired boundary-policy windows themselves
  - in practical terms:
    - `relaxed035` on `chr1` already carries the future `chr1_peak125` hotspot
    - `cap_to_boundary` on `chr9/16` already carries the future `chr9_peak616` and `chr16_peak464` hotspots
  - because the coarse genome-wide baseline has `row_count = 0` in all three comparison windows, these hotspot windows are also entirely `recovered-only` relative to the coarse baseline:
    - `chr1_peak125`: corrected `2118` rows vs coarse `0`
    - `chr9_peak616`: corrected `2946` rows vs coarse `0`
    - `chr16_peak464`: corrected `3127` rows vs coarse `0`
  - this means the residual anomaly is now maximally localized:
    - it is not coming from pre-existing shared chromosome content
    - it is coming from the specific site universe that enters only after the boundary-policy repairs

## 19. Exclusion-aware postprocess repair removes the non-plausible Manhattan tail without discarding the whole repaired line

- `2026-06-03` I patched `/data/home/grp-wangyf/xuyuan/sds/scripts/postprocess_sds_results.py` to support:
  - `--exclude-regions-tsv`
  - consistent first-pass and second-pass exclusion of the same genomic intervals before normalization and plotting
- the artifact window list is now versioned at:
  - `/data/home/grp-wangyf/xuyuan/sds/boundary_artifact_regions_scn_olddefault.tsv`
- current excluded windows:
  - `chr1:125100001-125200000`
  - `chr9:61600001-61800000`
  - `chr16:46400001-46500000`
- rationale:
  - `chr1:125.10-125.20 Mb` is the `relaxed035` borderline recovered-only window
  - `chr9:61.60-61.80 Mb` and `chr16:46.40-46.50 Mb` are `cap_to_boundary`-only windows with `rows_boundary_ok = 0` under skip logic

Repaired postprocess output:

- repaired root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b_boundarymask/SCN`
- repaired files now exist:
  - `SCN.normalized.tsv`
  - `SCN.frequency_bins.tsv`
  - `SCN.manhattan_points.tsv`
  - `SCN.significant_regions.tsv`
  - `SCN.manhattan.png`
  - `SCN.postprocess_summary.tsv`
- summary file records:
  - `exclude_regions_tsv = /data/home/grp-wangyf/xuyuan/sds/boundary_artifact_regions_scn_olddefault.tsv`

Tail-distribution comparison on common variants (`MAF >= 0.05`):

- coarse olddefault baseline:
  - `common = 5119816`
  - `|norm_SDS| >= 8 / 10 / 15 / 20 = 0 / 0 / 0 / 0`
  - `p < 5e-8 / 1e-10 / 1e-20 = 0 / 0 / 0`
  - top `-log10(p) = 6.04`
- corrected hybridpolicy full rerun:
  - `common = 5204483`
  - `|norm_SDS| >= 8 / 10 / 15 / 20 = 31 / 14 / 7 / 4`
  - `p < 5e-8 / 1e-10 / 1e-20 = 131 / 104 / 15`
  - top `-log10(p) = 171.84`
- repaired `boundarymask` postprocess:
  - `common = 5204263`
  - `|norm_SDS| >= 8 / 10 / 15 / 20 = 0 / 0 / 0 / 0`
  - `p < 5e-8 / 1e-10 / 1e-20 = 6 / 2 / 0`
  - top `-log10(p) = 11.70`

Interpretation:

- the repaired `boundarymask` line keeps almost the entire corrected full-rerun common-variant universe:
  - `5204263` vs `5204483` common variants
- but it completely removes the non-plausible ultra-tail generated by the three boundary-artifact windows:
  - corrected `|norm_SDS| >= 8 = 31`
  - repaired `|norm_SDS| >= 8 = 0`
- therefore the current best paper-facing workflow is no longer the coarse baseline and no longer the raw corrected hybridpolicy rerun:
  - coarse is the cleanest null but too conservative because it drops repaired early-q-arm content wholesale
  - raw corrected hybridpolicy retains the repaired content but still carries boundary-artifact towers
  - repaired `hybridpolicy + boundarymask postprocess` is currently the best balance between coverage recovery and plausible p-value behavior

Current top repaired residual hits:

- from `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_hybridpolicy_chunk20_20260531b_boundarymask/SCN/SCN.normalized.tsv`
  - `chr1:144430073:A:C`, `-log10(p) = 11.70`
  - `chr1:144457040:A:G`, `-log10(p) = 11.38`
  - `chr1:144458150:C:T`, `-log10(p) = 8.06`
- this residual `chr1:144.40-144.50 Mb` window is not created by the new mask:
  - it is already present in the raw corrected hybridpolicy line with nearly identical values
  - it remains absent from the coarse baseline only because the coarse line has `0` recovered rows there
  - earlier boundary diagnostics also showed this window is much less boundary-stressed than `chr1:125.10-125.20 Mb`
- therefore the remaining strongest repaired signal is no longer explained by the same boundary-artifact mechanism as the removed `chr1/9/16` towers

Practical workflow ranking as of `2026-06-03`:

- best null-looking p-value distribution:
  - coarse olddefault baseline
- best current balance of repaired coverage plus plausible Manhattan behavior:
  - corrected hybridpolicy full rerun + exclusion-aware `boundarymask` postprocess
