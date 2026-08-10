# Clump-Based Signal Catalog — Adopted Method + Registry

Date: 2026-08-08
Status: REGISTRY COMPLETE (chr4 pending ALL phasing) — 15 strict signals registered
Role: mainline — adopted signal-decomposition method
Input lineage: ALL joint-call v2; phased ALL (4383); normalized ALL
Demographic model: SDS-MODEL-001 (unchanged); Gamma unchanged
Sample policy: ALL (NCN 1255 + SCN 2716 = 4383); LD panel = ALL phased VCF (chr4 pending)

## 1. Adopted method (user decision)

**greedy clump + cM window (0.5 cM) + r² ≥ 0.1 + lead-lead 直接 LD 后合并 (post-merge)**

- Greedy clump: most significant unclumped SNP = lead; absorb unclumped SNPs within 0.5 cM (pyrho CHB map) with r² ≥ 0.1 vs lead; next lead; repeat
- Post-step: merge clumps whose LEADS have direct r² ≥ 0.1 (fixpoint, lead-level edges only — no chaining through arbitrary intermediates; unlike connected components)
- Signal rule: strict — clump must contain ≥1 Bonferroni-significant SNP (p < 1.0999e-08)
- Config label: `r2=0.1_win=0.5cM_ldmerge`

## 2. Implementation

- `SDSpipeline/scripts/postprocess/clump_signals.py`: `--win-cm-primary`, `--post-merge-leads`, `--members-out`
- `SDSpipeline/scripts/postprocess/build_clump_catalog.py`: signal_id assignment + annotation carry-over
- Outputs: `SDSworkspace/results/production/sds_output/ALL/signals_clump/` (parallel to `signals/`)
  - `all_clump_signals.tsv` (all 20 sweep configs), `cc_reference_signals.tsv`, `clump_vs_cc.tsv`
  - `chaining_audit.tsv` (298 pairs — ALL chaining class, 0 distance: lead-merge removed window-boundary splits)
  - `clump_vs_production.tsv`, `config_summary.tsv`, `clump_members.tsv` (53 clumps / 3,455 members)
  - **`all_signals_catalog.tsv`** — the registry (15 strict signals, chr4 pending)

## 3. Registered signals (15, chr4 excluded)

| signal_id | lead | p | n_snps | sig | EHH/iHS |
|---|---|---|---|---|---|
| chr11_48235kb_sig1 | chr11:48235462:G:A | 7.1e-09 | 190 | 1 | carried |
| chr12_111181kb_sig1 | chr12:111803962:G:A | 9.1e-23 | 1079 | 95 | carried (ALDH2) |
| chr19_10690kb_sig1..4 | 10378293 / 10515885 / 10555015 / 10765735 | 3.1e-14..1.9e-20 | 5-62 | 5-12 | sig4 carried (DNM2) |
| chr19_4885kb_sig1..8 | 4768268 / 4838039 / 4867638 / 4867689 / 4912313 / 4917007 / 4951434 / 5023601 | 1.5e-25..9.7e-13 | 2-171 | 1-7 | sig1/4/7 carried |
| chr22_41498kb_sig1 | chr22:41504089:G:A | 1.2e-09 | 48 | 7 | carried |

- 8/15 leads carry rsID + EHH + iHS annotation from production catalog (exact lead match); 7 new leads pending annotation (VEP/eQTL/EHH/iHS) — to be filled with chr4 in one pass
- signal_id convention unchanged: `chr{X}_{floor(mid_kb)}kb_sig{N}` per region by lead pos

## 4. Key differences vs production catalog (15 signals, connected components, 9k LD)

- **chr12: 5 production signals → 1**: TRPV4/ATP2A2/ALDH2/NAA25/OAS3 leads chain-merged via lead-lead direct LD (0.5 cM window + merge). Merged clump: 1079 SNPs, 95 sig, lead = ALDH2 (111803962, same as production chr12_111181kb_sig1)
- **chr19 4885kb: 6 → 8 strict** (new independent LD blocks: 4838039 p=4.6e-17, 4867638, 4912313, 4917007, 5023601 171-SNP block)
- **chr19 10690kb (DNM2): 1 → 4** independent LD blocks
- chr11 / chr22: unchanged
- Production minor signals (4710802 p=4.5e-8, 4903524 p=1.0e-6, 4889675 p=6.3e-5) NOT strict → excluded from registry (available in all_clump_signals.tsv as minor clumps)
- Production n_snps de-chained: e.g. production chr19_10690kb_sig1 n=820 → chr12 clump 1079 is MERGED (not de-chained); per-block n_snps 5-62

## 5. Validation

- Refinement invariant (clump ⊆ cc): 0 merge/new/absent across all 20 configs
- Sensitivity (all with _ldmerge): strict 13–30, parity 31–108; monotonicity holds
- chaining_audit: 298 pairs all `chaining` (r²(leads) < 0.1, median 0.007) — residual cc splits are pure chaining; distance-class pairs eliminated by lead-merge
- Determinism: byte-identical across independent runs (verified earlier)

## 6. Pending

- **chr4 (ADH1B)**: ALL phasing still running (job 504164, since Aug 6). When done: run clump_signals.py chr4 (ALL LD) + build_clump_catalog.py --all → complete catalog (rsID/EHH/iHS/VEP/eQTL for new leads in one pass)
- Annotation of 7 new leads (VEP, 1KCP eQTL, EHH at new leads, iHS at new leads)
- Decision: replace `signals/all_signals_catalog.tsv` as paper-facing catalog once chr4 + annotations complete
