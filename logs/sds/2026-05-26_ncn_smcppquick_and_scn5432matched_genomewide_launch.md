Date: 2026-05-26
Topic: sds
Type: launch

Question

在 `SCN` 已确认存在 gamma `SAMPLE_SIZE` 错配、而 `NCN` 仍未修复的前提下，进一步回答：

1. `NCN` 的下一条有效 demography 修复线是否应当改用 pop-specific `SMC++` history，而不是错误的 phlash-derived `npz`？
2. 如果用 `NCN` 的 `SMC++` curve，`fine` 与 `fine_quick` 哪一条更值得先扩成 gamma smoke？
3. `SCN` 已经证明有效的 `sample-size matched gamma` 是否已经进入干净的 genome-wide 修复阶段？

Input paths

- invalid phlash-derived npz context:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/smcpp_fine_demography_20260526/smcpp_fine_ncn_scn.npz`
  - user clarified that prior phlash-derived `npz` demographic history is wrong and must not be used as repair evidence
- pop-specific `SMC++` curves:
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/NCN_fine_smcpp.csv`
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/NCN_fine_quick_smcpp.csv`
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography/SCN/smcpp/SCN_fine_smcpp.csv`
- recent-resolution benchmark:
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/recent_sensitivity/NCN_recent_resolution_report.csv`
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography/NCN/smcpp/recent_sensitivity/NCN_recent_resolution_report.json`
- SCN matched gamma:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/gravel_chb_ne100k_scn5432_smokegrid_20260526/gravel_chb_present100000.g_file.txt`

Output paths

- `NCN fine_quick` backward-compatible NPZ:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/smcpp_quick_demography_20260526/smcpp_quick_ncn_scn.npz`
- invalid-but-recorded `NCN fine` smoke gamma:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_smcppfine_smoke_20260526/final_gamma`
- valid `NCN fine_quick` smoke gamma:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_smcppquick_smoke_20260526/final_gamma`
- launched `NCN fine_quick` full-grid smoke outdir:
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_smcppquick_smokegrid_20260526`
- clean `SCN` genome-wide matched-gamma out root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526`
- `SCN` submitter-array manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn5432matched_submitter_array_20260526_134055_36191/manifest.tsv`

Parameters

- `NCN fine` present-day `Ne(0)` extracted from curve: `137410.06482379386`
- `NCN fine_quick` present-day `Ne(0)` extracted from curve: `87281.8167082614` (submitted as `87282`)
- gamma smoke sample size for `NCN`: `2510` haplotypes
- `SCN` matched gamma uses `5432` haplotypes
- `SCN` clean genome-wide run keeps:
  - demographic family: `Gravel_CHB`
  - `Ne(0)=100000`
  - corrected `SAMPLE_SIZE=5432`

Scripts / jobs

- conversion script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/build_demography_npz_from_smcpp_csv.py`
- clean `SCN` genome-wide launcher:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn5432matched_genomewide_submitter_array.sh`
- invalid phlash-based jobs (do not use for interpretation):
  - `487116`, `487117`, `487118`
- valid `NCN fine_quick` 3-point smoke:
  - `487119`, `487120`, `487121`
- valid `NCN fine_quick` full-grid smoke:
  - chunks: `487197`
  - aggregates: `487198`
  - finalize: `487199`
- dependent `NCN chr1/chr6` repair launcher from ready `g_file`:
  - `487246`
- clean `SCN` matched-gamma genome-wide submitter array:
  - array: `487135`
  - finalizer: `487136`
- `SCN` postprocess / diagnostics from finalized genome-wide run:
  - postprocess: `487247`
  - diagnostics: `487248`
- dependent `SCN` old-vs-matched genome-wide compare:
  - `487250`

Result

## 1. `phlash npz` was explicitly ruled out

User clarified that the prior `npz` demographic history came from phlash and is wrong. Therefore:

- jobs `487116-487118` are invalid for repair inference
- subsequent repair work must rely on pop-specific `SMC++` curves instead

## 2. `NCN_fine_quick` is much more plausible than `NCN_fine`

Three-point smoke at `DAF = 0.05 / 0.07 / 0.10`:

- `NCN_fine` (`present Ne(0)≈137410`)
  - `0.05 -> 1.33483058058943`
  - `0.07 -> 1.37568541005867`
  - `0.10 -> 1.30210756923231`
- `NCN_fine_quick` (`present Ne(0)≈87282`)
  - `0.05 -> 0.628890586555761`
  - `0.07 -> 0.640481508152885`
  - `0.10 -> 0.633003699390556`

Compared with the default `Gravel_CHB@100k` low-frequency head:

- `0.05 -> 0.7103425643116044`
- `0.07 -> 0.6789969557660382`
- `0.10 -> 0.6721468570164826`

Interpretation:

- `NCN_fine` is clearly worse than default `Gravel_CHB@100k` and should not be extended
- `NCN_fine_quick` is directionally better and is the first pop-specific `NCN` demography worth testing as a real repair candidate

## 3. `SCN` clean genome-wide matched-gamma repair is now formally launched

Previous `chrlist` wrapper attempts were messy and partly stalled in shell return / manifest recording. To avoid reusing the contaminated partial output tree, a clean run was launched to a new output root:

- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526`
- submitter-array manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn5432matched_submitter_array_20260526_134055_36191/manifest.tsv`
- jobs:
  - `487135` submitter array
  - `487136` finalizer

This clean run uses the already-validated matched gamma:

- `Gravel_CHB`
- `Ne(0)=100000`
- `SAMPLE_SIZE=5432`

so it is the first end-to-end genome-wide repair run for `SCN` based on a confirmed root cause.

By the time of this log update:

- `487135` had already produced `chr1` through `chr22` `final_job` records under:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn5432matched_submitter_array_20260526_134055_36191/results`
- the clean output root had already started materializing arm/final archives such as:
  - `chr1.archive.csv`
  - `chr2.archive.csv`
  - `chr4.archive.csv`
  together with multiple per-arm compute summaries, confirming that the clean run is not just queued but actively generating chromosome outputs

Later in the same turn, the chromosome-final stage was verified more strongly:

- the `results/chr*.final_job.tsv` table under
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn5432matched_submitter_array_20260526_134055_36191/results`
  already covered `chr1-22`
- explicit `bjobs` checks showed `chr2-22` final jobs as `DONE`
- `chr1.sds.tsv` also existed in the clean output root even though its job id had already disappeared from active `bjobs`

At the file level, the clean output root already contained finalized chromosome outputs such as:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526/SCN/chr1.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526/SCN/chr10.sds.tsv`
- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526/SCN/chr22.sds.tsv`

The submitter finalizer `487136` then completed and submitted:

- `487247` postprocess
- `487248` diagnostics

To avoid another manual handoff after those jobs finish, a dependent comparison job was also submitted:

- script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn5432matched_genomewide_compare.sh`
- compare helper:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/compare_sds_frequency_bins.py`
- dependent job:
  - `487250`

This compare will automatically generate:

- old vs matched diagnostics comparison
- old vs matched `frequency_bins.tsv` delta table

## 4. `NCN chr1/chr6` repair smoke is now chained automatically

To avoid another manual handoff after `NCN_fine_quick` gamma finalize, a dependent launcher was added:

- script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_ncn_smcppquick_chr1_6_submitter_array.sh`
- dependent launcher job:
  - `487246`

This means:

- if `487199` succeeds and writes
  - `/data/home/grp-wangyf/xuyuan/sds/tmp/ncn_smcppquick_smokegrid_20260526/ncn_smcppquick_present87282.g_file.txt`
- then `487246` will automatically submit the next repair stage:
  - `NCN chr1/chr6`
  - same rebuilt input
  - pop-specific `SMC++ quick` gamma
  - fresh output root for direct compare

Conclusion

- `SCN` now has a clean genome-wide repair line running under the matched gamma that already improved `chr1/chr6`.
- `NCN` should not continue along phlash-derived demography or `NCN_fine`; the next valid direction is `NCN_fine_quick`.
- A full-grid `NCN_fine_quick` gamma smoke (`487197-487199`) has been launched and is the gating step before `NCN chr1/chr6` repair compute.

## 5. `SCN` matched-gamma genome-wide run finished and improved the worst bins, but did not fully normalize the family

The clean genome-wide matched-gamma run completed under:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_gravel_chb_ne100k_scn5432matched_genomewide_run2_20260526/SCN`

with downstream compare outputs under:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn5432matched_genomewide_compare_20260526`

Key bin-level improvement versus the old `SCN Gravel_CHB@100k + new input` run:

- `[0.01,0.02)`: mean `4.3923 -> 2.0380`, sd `10.5053 -> 6.5571`
- `[0.02,0.03)`: mean `1.9450 -> 1.4901`, sd `8.6105 -> 7.0815`
- `[0.03,0.04)`: mean `0.2951 -> 0.2528`, sd `3.3952 -> 3.1533`

This confirms that `SCN` sample-size mismatch was a real technical contributor. However, the repaired track still retained substantial low-frequency inflation, so `sample-size` alone does not close the gap to plausible biology.

## 6. `NCN_fine_quick chr1/chr6` is a negative result

The dependent `NCN chr1/chr6` repair smoke launched successfully after fixing the expected `g_file` basename:

- state dir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/ncn_smcppquick_chr1_6_submitter_array_20260526_162647_237562`
- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_ncn_smcppquick_chr1_6_20260526/NCN`
- jobs:
  - `submitter_array = 487252`
  - `finalizer = 487253`
  - `postprocess = 487264`
  - `diagnostics = 487265`

The resulting compare against the old `NCN Gravel_CHB@100k + new input` run did **not** show convergence to plausible biology.

Observed overlap summary:

- `chr1`, `(0.01,0.02)`: mean `|rSDS| 0.2498 -> 0.2724`, `>=5: 76 -> 151`, `>=10: 63 -> 137`
- `chr6`, `(0.01,0.02)`: mean `|rSDS| 0.3042 -> 0.3359`, `>=5: 152 -> 230`, `>=10: 152 -> 230`

Chrom-tail diagnostics were mixed rather than improving cleanly:

- `chr1`: `>=20 359 -> 474`, `>=50 309 -> 271`, `>=100 191 -> 82`
- `chr6`: `>=20 554 -> 690`, `>=50 523 -> 354`, `>=100 303 -> 41`

Interpretation:

- `NCN_fine_quick` may compress the most extreme tail, but it increases the moderate tail and does not pull the low-frequency bins toward the olddefault/plausible regime.
- Therefore `NCN_fine_quick` should not be promoted as the current repair line for `NCN`.

## 7. `SCN olddefault chr1/chr6` strongly favors gamma family over sample-size as the dominant remaining factor

A same-input `SCN olddefault` chr1/chr6 smoke was launched under:

- state dir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_chr1_6_submitter_array_20260526_175110_267968`
- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_chr1_6_20260526/SCN`

This track uses:

- current rebuilt `SCN` input
- historical `olddefault` gamma file
  - `/data/home/grp-wangyf/xuyuan/sds/g_file.txt`

The result is dramatically cleaner than both:

- old `SCN Gravel_CHB@100k + new input`
- `SCN` sample-size-matched `Gravel_CHB@100k`

Tail counts:

- `chr1`:
  - old: `>=20 207`, `>=50 71`, `>=100 36`
  - matched: `>=20 199`, `>=50 40`, `>=100 8`
  - olddefault: `>=20 0`, `>=50 0`, `>=100 0`
- `chr6`:
  - old: `>=20 178`, `>=50 67`, `>=100 45`
  - matched: `>=20 163`, `>=50 45`, `>=100 13`
  - olddefault: `>=20 0`, `>=50 0`, `>=100 0`

Site-overlap compare:

- `chr1`, `(0.01,0.02)`:
  - old mean `5.0558`
  - matched mean `2.5670`
  - olddefault mean `0.3592`
  - `>=5`: `12359 -> 7444 -> 0`
- `chr6`, `(0.01,0.02)`:
  - old mean `5.0234`
  - matched mean `2.6330`
  - olddefault mean `0.3446`
  - `>=5`: `9479 -> 5899 -> 0`

Interpretation:

- fixing `SAMPLE_SIZE` helps `SCN`
- but the remaining dominant issue is still the **gamma family**
- on current rebuilt input, `olddefault` is much closer to plausible biology than the current `Gravel_CHB@100k` family

## 8. `SCN olddefault genome-wide` promoted to the main repair candidate and launched

Because the chr1/chr6 discrimination strongly favored the historical gamma family, a full genome-wide repair run was launched:

- launcher:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_submitter_array.sh`
- manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_163409_455329/manifest.tsv`
- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_20260527`
- jobs:
  - `submitter_array = 487479`
  - `finalizer = 487480`

At launch time the output root and log directory already existed, confirming the new genome-wide repair line is live.

Subsequent queue inspection confirmed:

- `487479` entered active `RUN` state on `smp`
- `487480` is pending on the submitter-array dependency as expected

To make the new line self-closing, comparison infrastructure was also added:

- compare launcher:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_compare.sh`
- wait-and-compare wrapper:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/wait_and_compare_scn_olddefault_genomewide.sh`
- dependent compare job:
  - `487481`

`487481` is submitted with dependency `done(487480)` and then waits until:

- `SCN.frequency_bins.tsv`
- `diagnostics/SCN.gravel_chb_ne100k_submitter_array.chrom_tail_counts.tsv`

exist under the new olddefault genome-wide output root before running the comparison.

## 9. Added a faster `%6` submitter-array relaunch and attached auto-compare

The original `SCN olddefault genome-wide` launcher used `submitter_parallel=2`, which turned out to be too slow because each submitter performs nontrivial synchronous preprocessing before it can emit per-chromosome final jobs.

To accelerate the main repair candidate, the same launcher was reused with higher submitter concurrency:

- manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_164347_11611/manifest.tsv`
- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_fast_20260528`
- jobs:
  - `submitter_array = 487487`
  - `finalizer = 487488`

Queue inspection after launch showed:

- `487487[1-6]` in active `RUN`
- later submitter tasks pending on available `smp` slots
- `487488` pending on `done(487487)` as expected

This explains why the queue now shows many `PEND` jobs: there are currently **two** `SCN olddefault genome-wide` trees alive in parallel:

- slow `%2` run: `487479 / 487480 / 487481`
- fast `%6` run: `487487 / 487488`

The fast run immediately began generating chunk-level preprocessing outputs under:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_fast_20260528/SCN/chunk_submit`

including `chr2/4/5/6` `p.filter.csv` and `p.t.feather` artifacts, confirming that it is advancing rather than stalled.

To avoid losing automatic triage once the fast run finishes, the comparison scripts were parameterized so the olddefault root and compare output directory are no longer hard-coded:

- updated:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_compare.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/wait_and_compare_scn_olddefault_genomewide.sh`

A second dependent compare job was then attached to the fast finalizer:

- compare job:
  - `487494`
- compare output dir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_fast_compare_20260528`

So the current primary observation target is the **fast** `%6` run:

- `487487` submitter array
- `487488` finalizer
- `487494` wait-and-compare

The slower `%2` run remains useful as a fallback, but the faster run is now the main route to the first genome-wide `SCN olddefault + rebuilt input` Manhattan and diagnostics set.

## 10. Re-audit showed the earlier `%2` run was actually ahead; fast relaunch was then retired

After tracing job provenance by command line rather than job-name prefix alone, it became clear that the apparently active `chr1/chr2` chunk arrays belonged to the **earlier** `%2` run, not the later `%6` relaunch.

Key evidence:

- slow-run manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_163409_455329/manifest.tsv`
- corresponding result files:
  - `chr1.final_job.tsv`
  - `chr2.final_job.tsv`
- `chr1.submitter.out` and `chr2.submitter.out` are both non-empty and record:
  - `chr1: P_ARRAY_JOB=487482, Q_ARRAY_JOB=487483, P_MERGE_JOB=487484, Q_MERGE_JOB=487485, FINAL_JOB=487486`
  - `chr2: P_ARRAY_JOB=487489, Q_ARRAY_JOB=487490, P_MERGE_JOB=487491, Q_MERGE_JOB=487492, FINAL_JOB=487493`

Direct `bjobs -l` inspection on these arrays showed their commands point to the slow-run output root:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_20260527/SCN`

By contrast, the fast `%6` relaunch still had zero-byte `results/chr*.submitter.out` files and no `chr*.final_job.tsv`, meaning it had not yet reached the same stage.

Because the fast relaunch had become purely redundant queue pressure, it was retired:

- `487488` (fast finalizer): terminated
- `487494` (fast compare): terminated
- `487487` (fast submitter array): termination requested; completed tasks remained `DONE`/`EXIT` while active tasks were being drained

Current interpretation:

- the **slow** `%2` run is the real primary repair line
- the earlier apparent advantage of the `%6` relaunch was an artifact of mixed job-name prefixes and overlapping queue output
- current main jobs to monitor are now:
  - `487479` submitter array
  - `487480` finalizer
  - `487481` wait-and-compare

At the time of this re-audit:

- `chr1.final_job.tsv` already points to `487486`
- `chr2.final_job.tsv` already points to `487493`
- `487486` and `487493` are correctly pending only because their merge dependencies are not finished yet
- merge jobs `487484/487485/487491/487492` are likewise pending only on their chunk-array completion

So the run is not blocked by a logic failure; it is progressing through the expected dependency graph, with the dominant remaining constraint being queue slot pressure on the large `chr1/chr2` arrays.

## 11. Added a progress-summary helper for the active slow-run repair line

Because the remaining work is now mostly queue-driven, a small helper was added to summarize submitter-array state without repeatedly reconstructing it from multiple `find` and `bjobs` calls:

- script:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/summarize_submitter_array_progress.sh`

It reports:

- total/non-empty `chr*.submitter.out`
- `chr*.final_job.tsv` count
- `chr*.sds.tsv` count
- `chr*.archive.csv` count
- presence of `normalized.tsv`, `frequency_bins.tsv`, Manhattan PNG, and diagnostics

Running it on the active slow-run state directory:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_163409_455329`

gave:

- `submitter_out_total = 6`
- `submitter_out_nonempty = 4`
- `final_job_tsv_count = 4`
- `chr_sds_tsv_count = 0`
- `chr_archive_csv_count = 0`
- `has_normalized = 0`
- `has_frequency_bins = 0`
- `has_manhattan_png = 0`
- `has_diagnostics_chrom_tail = 0`

This matches the current direct inspection:

- `chr1–4` have already advanced to non-empty `submitter.out` and emitted `final_job.tsv`
- `chr5–6` submitters are still live but not yet finalized
- no chromosome-level merged outputs exist yet, so postprocess/diagnostics have not started

Operationally, this means:

- the repair line is still progressing correctly
- the current bottleneck is queue completion of the arm-level chunk arrays and their downstream merge jobs
- once the first `chr*.sds.tsv` appears, the run will have crossed from “submitted correctly” into “material outputs exist”, which is the next meaningful checkpoint for biological plausibility verification

## 12. Split the olddefault genome-wide repair into an original `chr1–6` tree plus a higher-concurrency remainder tree

Further queue audit showed the active `%2` slow-run tree had already advanced to:

- `chr1.final_job.tsv -> 487486`
- `chr2.final_job.tsv -> 487493`
- `chr3.final_job.tsv -> 487514`
- `chr4.final_job.tsv -> 487509`

while the remaining pending work was mostly caused by low submitter concurrency rather than a logic fault.

To accelerate the unfinished tail without discarding the successful early chromosomes:

1. the original slow-run finalizer/compare jobs were retired:
   - `487480` -> `EXIT`
   - `487481` -> `EXIT`
2. pending original submitter-array elements for `chr7–22` were terminated, leaving:
   - `chr1–4`: `DONE`
   - `chr5–6`: allowed to finish on the original tree
3. the launcher was updated to support:
   - `--skip-finalizer`
4. a new remainder-only olddefault submitter tree was launched against the **same** output root:
   - manifest:
     - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_165754_34113/manifest.tsv`
   - chromosomes:
     - `7–22`
   - submitter concurrency:
     - `6`
   - `skip_finalizer = 1`
   - submitter array job:
     - `487515`

At the time of launch:

- original tree:
  - `487479`
  - `chr1–6` only, with `chr5–6` still active and `chr1–4` already emitted `final_job.tsv`
- remainder tree:
  - `487515`
  - `chr7–22`

This avoids recomputing the chromosomes already successfully submitted while materially increasing throughput for the unfinished tail.

## 13. Added a combined finalizer that waits for all 22 chromosome final jobs across both state trees

Because the output root is now being fed by two independent submitter trees, a single-tree finalizer is no longer sufficient.

A dedicated combined wait/finalize script was added:

- `/data/home/grp-wangyf/xuyuan/sds/scripts/wait_and_finalize_scn_olddefault_combined.sh`

It:

- polls multiple state directories until the expected count of `chr*.final_job.tsv` files is reached
- copies all per-chromosome `final_job.tsv` files into a temporary combined directory
- submits postprocess/diagnostics through:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/submit_postprocess_from_final_jobs.sh`
- then waits for:
  - `SCN.frequency_bins.tsv`
  - diagnostics tail counts
- and finally launches the standard old/matched/olddefault compare

This combined finalizer has been launched as:

- `487516` (`SDS_SCN_OLDDEFAULT_COMBINED_FINALIZE`)

and it watches these two state trees:

- original:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_163409_455329`
- remainder:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_165754_34113`

Current interpretation:

- the repair line is no longer bottlenecked by a single low-concurrency submitter array
- `chr1–6` progress is preserved
- `chr7–22` now have a dedicated higher-concurrency path
- postprocess and Manhattan generation are now wired to the **union** of both state trees rather than to a single launcher

## 14. All 22 `final_job.tsv` files are now present; combined postprocess/diag/compare is attached through dependency aggregators

Subsequent progress checks showed:

- original tree:
  - `submitter_out_nonempty = 6`
  - `final_job_tsv_count = 6`
- remainder tree:
  - `submitter_out_nonempty = 16`
  - `final_job_tsv_count = 16`

Together this gives the full expected set of `22` chromosome-level `final_job.tsv` files.

An initial attempt to submit one large finalizer job with all `22` `done(job)` dependencies in a single `-w` expression repeatedly hit an LSF library/socket failure even though ordinary `bsub` and short dependency expressions worked.

To avoid that scheduler-side failure mode, the dependency graph was re-expressed in two stages:

1. four short dependency aggregator jobs:
   - `487618` = `sds_SCN_olddefault_agg_3`
   - `487619` = `sds_SCN_olddefault_agg_2`
   - `487620` = `sds_SCN_olddefault_agg_4`
   - `487621` = `sds_SCN_olddefault_agg_1`
2. one final combined pipeline job:
   - `487623` = `sds_SCN_olddefault_combined_pipeline`

`487623` depends only on:

- `done(487618) && done(487619) && done(487620) && done(487621)`

and then runs, in sequence:

1. `postprocess_sds_results.py`
2. `diagnose_sds_scan.py`
3. `launch_scn_olddefault_genomewide_compare.sh`

against the repaired genome-wide olddefault output root:

- `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_20260527/SCN`

This means the SCN olddefault repair line has now crossed an important structural threshold:

- the chromosome-level submission graph is complete across all `22` chromosomes
- the remaining work is no longer “launch more compute”
- the remaining work is “wait for chromosome final jobs to finish, then let `487623` produce normalized output, diagnostics, and the comparison set needed for biological plausibility verification”

## 15. Added a coarse-chunk olddefault genome-wide acceleration line

Even after the olddefault repair line was structurally complete, direct inspection of manifest sizes showed why the `5000-row` chunking strategy remained a severe scheduler bottleneck:

- `chr1_p.manifest.tsv`: `478` chunks
- `chr2_q.manifest.tsv`: `590` chunks

This is enough to keep the repair line trapped in queueing/merge dependency stages even after the biological direction is already clear.

To attack that bottleneck directly, the submitter-array plumbing was extended so chunking parameters can be passed through the existing launcher stack:

- updated:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/run_submitter_array_entry.sh`
  - `/data/home/grp-wangyf/xuyuan/sds/scripts/launch_scn_olddefault_genomewide_submitter_array.sh`

New launcher parameters now flow through to `submit_sds_compute_chunked_chr.sh`:

- `--chunk-rows`
- `--array-parallel`
- `--chunk-job-slots`

Using this, a fresh **coarse-chunk** `SCN olddefault genome-wide` run was launched:

- manifest:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_submitter_array_20260528_183757_121492/manifest.tsv`
- output root:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/sds_output_scn_olddefault_genomewide_coarse_20260528`
- parameters:
  - `submitter_parallel = 6`
  - `chunk_rows = 20000`
  - `array_parallel = 48`
  - `chunk_job_slots = 1`
- jobs:
  - submitter array: `487624`
  - finalizer: `487625`

At launch time:

- `487624[1-6]` entered active `RUN`
- `487625` was queued behind the submitter-array completion

To keep this acceleration line self-closing, a dependent compare job was also attached:

- `487626` = `SDS_SCN_OLDDEFAULT_COARSE_COMPARE`
- compare output dir:
  - `/data/home/grp-wangyf/xuyuan/sds/data/processed/scn_olddefault_genomewide_coarse_compare_20260528`

Current interpretation:

- the earlier combined olddefault repair line remains valid and preserved
- but the new coarse-chunk line is now the most likely route to the **first usable genome-wide repaired SCN Manhattan**, because it attacks the known scheduler bottleneck without changing the gamma family or rebuilt input contract

Direct manifest-size comparison confirmed that the coarser chunking materially reduced scheduler fragmentation:

- original `5000-row` line:
  - `chr1_p.manifest.tsv = 478`
  - `chr2_q.manifest.tsv = 590`
- coarse `20000-row` line:
  - `chr1_p.manifest.tsv = 120`
  - `chr2_q.manifest.tsv = 148`

So for these representative arms, the coarse line reduced chunk counts by about `4x`, which is exactly the bottleneck it was designed to target.
