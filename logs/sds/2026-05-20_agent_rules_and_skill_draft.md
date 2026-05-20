Date
- 2026-05-20

Topic
- sds

Type
- summary

Question
- What collaboration lessons from the 2026-04-25 onward SDS work should be formalized as reusable rules or a skill?

Input paths
- /data/home/grp-wangyf/xuyuan/SDSlog/logs/sds
- /data/home/grp-wangyf/xuyuan/SDSlog/logs/index.md
- /data/home/grp-wangyf/xuyuan/SDSlog/logs/result_index.md
- /data/home/grp-wangyf/xuyuan/SDSlog/plots/plot_index.md

Output paths
- /data/home/grp-wangyf/xuyuan/AGENT.md
- /data/home/grp-wangyf/xuyuan/skills/sds_workflow/SKILL.md
- /share/home/grp-wangyf/yf_intern/.codex/skills/sds-workflow/SKILL.md

Parameters
- Scope: collaboration rules and reusable workflow patterns derived from SDS work after 2026-04-25

Scripts / commands / job IDs
- documentation only

Key plots
- none

Key result files
- /data/home/grp-wangyf/xuyuan/AGENT.md
- /data/home/grp-wangyf/xuyuan/skills/sds_workflow/SKILL.md
- /share/home/grp-wangyf/yf_intern/.codex/skills/sds-workflow/SKILL.md

Demographic model
- not applicable

Gamma shape source
- not applicable

Present-day Ne(0)
- not applicable

Input lineage
- not applicable

Model role
- audit-only

Conclusion
- Two reusable artifacts were created:
  - `AGENT.md` at repo root with hard SDS working rules
  - a repo-local SDS workflow skill source at `skills/sds_workflow/SKILL.md`
- The SDS workflow skill was also formally installed at:
  - `~/.codex/skills/sds-workflow/SKILL.md`
- The most important lessons formalized were:
  - log and provenance discipline
  - plausibility before biological interpretation
  - strict separation of model vs input lineage
  - conservative cleanup
  - default use of `CMplot` for genome-wide Manhattan plots
  - preference for original joint VCF/BCF over `PLINK` roundtrip as SDS input source

Next step
- Keep the repo-local source and the installed skill synchronized when the workflow rules change.
