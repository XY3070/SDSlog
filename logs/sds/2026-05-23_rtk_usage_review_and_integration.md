Date
- 2026-05-23

Topic
- sds

Type
- summary

Question
- How should the local `rtk` command be used in this workspace, and how should it be integrated into `AGENT.md` and the SDS workflow skill?

Input paths
- /data/home/grp-wangyf/xuyuan/AGENT.md
- /data/home/grp-wangyf/xuyuan/skills/sds_workflow/SKILL.md
- /share/home/grp-wangyf/yf_intern/.local/bin/rtk

Output paths
- /data/home/grp-wangyf/xuyuan/AGENT.md
- /data/home/grp-wangyf/xuyuan/skills/sds_workflow/SKILL.md
- /share/home/grp-wangyf/yf_intern/.codex/skills/sds-workflow/SKILL.md

Parameters
- `rtk` subcommands reviewed:
  - `read`
  - `grep`
  - `find`
  - `summary`
  - `err`
  - `test`
  - `rewrite`
  - `init`
  - `trust`
  - `proxy`
  - `run`
  - `config`

Scripts / commands / job IDs
- `which rtk`
- `rtk --help`
- `rtk help read`
- `rtk help grep`
- `rtk help find`
- `rtk help summary`
- `rtk help err`
- `rtk help test`
- `rtk help rewrite`
- `rtk help init`
- `rtk help trust`
- `rtk help proxy`
- `rtk help run`
- `rtk help config`

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
- `rtk` is best treated as a token-saving inspection layer for large outputs, not as a provenance system.
- The most useful commands for this workspace are:
  - `rtk read`
  - `rtk grep`
  - `rtk find`
  - `rtk summary`
  - `rtk err`
  - `rtk test`
  - `rtk rewrite`
- `AGENT.md` and the SDS workflow skill were updated to prefer `rtk` for large-output triage while still requiring conclusions to point back to original files and `SDSlog`.

Next step
- Keep the installed skill at `~/.codex/skills/sds-workflow/SKILL.md` synchronized with the repo-local source.
