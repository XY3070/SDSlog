# Relate chr19 cleanup audit and active rerun — 2026-08-23

## Scope

Audited working directory for core dumps, failed Relate/LSF artifacts, partial runs, and redundant scripts. Active job `505938` was excluded from cleanup.

## Current run

- Job: `505938`, queue `smp`, status `RUN`, host `32*s001` (fat node)
- Resources: 32 slots; LSF reservation `3,000,000 MB` (`MEMLIMIT` about 2.8 TB); walltime 96 h
- Relate: full chr19; `RELATE_THREADS=32`; `RELATE_MEMORY_GB=64`
- Map: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/refs/maps/chr19_CHB_pyrho.relate.map`
- Workdir: `/data/home/grp-wangyf/xuyuan/SDSworkspace/results/production/orthogonal/relate_clues2/ALL/run_work_chr19_chb_pyrho/`
- Input symlinks verified: phased ALL haps/sample, CHB pyrho Relate map, constant scaffold coal.

## Failed attempts

- `505935`: owner-terminated after initial resource correction; discarded after audit.
- `505936`: exit 1 because `UKBQC_ALL_chr19` already existed.
- `505937`: owner-terminated while correcting `--memory`; discarded after audit.

## Preserved failure evidence

Kept old `run_work_chr19/` `.anc/.mut/.bin` and old Relate failure logs. They document mut-format incompatibility, `FinalizePopulationSize` assertion, LSF hostname/resource failure, and OOM/abort behavior. These are provenance, not redundant outputs.

## Cleanup performed

After this log was written, removed:

1. `run_work_chr19_v7/core.82055` (32G ELF core from obsolete `phase_common_static`; no active job referenced it).
2. Empty/near-empty `UKBQC_ALL_chr19.failed_505936/` and `UKBQC_ALL_chr19.failed_505937/` scratch directories.
3. `lsf.505935.{out,err}`, `lsf.505936.{out,err}`, `lsf.505937.{out,err}`.

No clearly redundant scripts were found safe to remove. Prepared inputs, CHB map, coal reference, active run directory, and old failure evidence were retained.

## Completion update (2026-08-26)

- `bmod -W 168:00 505938` succeeded; LSF `RUNLIMIT` changed from `5760` to `10080` minutes.
- Job completed normally at `2026-08-26 18:42:45 CST`: status `DONE`, exit code `0`; no walltime or OOM termination.
- Completed Painting, mutation propagation, Combining AncesTrees in Sections, and Finalizing.
- Finalizing reported `192570` SNPs not mapped and `935` flipped SNPs; downstream QC required before interpretation.
- Final outputs: `UKBQC_ALL_chr19.anc` (~18.1 GB) and `UKBQC_ALL_chr19.mut` (~122 MB).
- Peak memory ~1.9 TB, below `2.8 TB MEMLIMIT`.
