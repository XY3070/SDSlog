# 项目结构解耦：SDSlog repo、SDSpipeline repo、SDSworkspace 非 git workspace

Date: 2026-06-05
Topic: sds
Type: summary
Question: 当前项目里代码、结果、日志、一次性脚本混在同一工作区，导致 provenance 缺失时既无法回退，也无法系统追错。应该如何从目录和 repo 边界上先把项目拆干净？

## Input paths

- `/data/home/grp-wangyf/xuyuan/PROJECT_LAYOUT.md`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/README.md`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/WORKSPACE_CONTRACT.md`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/MIGRATION_MAP.md`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/manifests/canonical_scripts.tsv`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/templates/run_manifest.template.yaml`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline/templates/dataset_manifest.template.yaml`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace/README.md`
- `/data/home/grp-wangyf/xuyuan/sds/SDS_SCRIPT_PLAN.md`
- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-05_first_principles_rebuild_workflow_compare.md`

## Output paths

- `/data/home/grp-wangyf/xuyuan/SDSlog/logs/sds/2026-06-05_project_structure_decoupling.md`
- `/data/home/grp-wangyf/xuyuan/PROJECT_LAYOUT.md`
- `/data/home/grp-wangyf/xuyuan/SDSpipeline`
- `/data/home/grp-wangyf/xuyuan/SDSworkspace`

## Parameters

- `new_pipeline_repo=/data/home/grp-wangyf/xuyuan/SDSpipeline`
- `new_workspace_root=/data/home/grp-wangyf/xuyuan/SDSworkspace`
- `reporting_repo=/data/home/grp-wangyf/xuyuan/SDSlog`
- `canonical_script_snapshot_count=41 tracked files in SDSpipeline root depth<=2; scripts manifest registers core input/compute/postprocess/demography/gamma helpers`
- `workspace_mode=non-git`

## Scripts / commands / job IDs

- `git init /data/home/grp-wangyf/xuyuan/SDSpipeline`
- copied reusable scripts from:
  - `/data/home/grp-wangyf/xuyuan/sds/scripts`
  - `/data/home/grp-wangyf/xuyuan/benchmark/demography`
- added structure docs/manifests:
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/REPO_LAYOUT.md`
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/WORKSPACE_CONTRACT.md`
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/MIGRATION_MAP.md`
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/manifests/canonical_scripts.tsv`
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/manifests/legacy_migration_map.tsv`
  - `/data/home/grp-wangyf/xuyuan/SDSpipeline/scripts/admin/bootstrap_workspace.sh`
- `job_ids=None`

## Key plots

- `None archived`

## Key result files

- `PROJECT_LAYOUT`: `/data/home/grp-wangyf/xuyuan/PROJECT_LAYOUT.md`
- `PIPELINE-README`: `/data/home/grp-wangyf/xuyuan/SDSpipeline/README.md`
- `PIPELINE-CONTRACT`: `/data/home/grp-wangyf/xuyuan/SDSpipeline/docs/WORKSPACE_CONTRACT.md`
- `PIPELINE-SCRIPTS`: `/data/home/grp-wangyf/xuyuan/SDSpipeline/manifests/canonical_scripts.tsv`
- `RUN-TEMPLATE`: `/data/home/grp-wangyf/xuyuan/SDSpipeline/templates/run_manifest.template.yaml`
- `DATASET-TEMPLATE`: `/data/home/grp-wangyf/xuyuan/SDSpipeline/templates/dataset_manifest.template.yaml`
- `WORKSPACE-README`: `/data/home/grp-wangyf/xuyuan/SDSworkspace/README.md`

## Conclusion

- 现在项目边界被正式拆成三层：`SDSlog/` 负责汇报与写作，`SDSpipeline/` 负责可复用 pipeline 代码，`SDSworkspace/` 负责非 git 的大文件、run root、一次性脚本和 provenance snapshot。
- `SDSpipeline/` 已经初始化为独立 git repo，并从当前 `sds/scripts` 与 `benchmark/demography` 中复制了一批可视作 canonical snapshot 的关键脚本；同时新增了路径契约、迁移映射和 manifest 模板。
- `SDSworkspace/` 已建好 `input/`、`results/`、`runs/`、`oneoff/`、`provenance/` 等目录。后续只要强制每次 run 在 `provenance/` 写 manifest，就能避免再出现 `olddefault gamma` 这种“文件存在但来源无法确定”的情况。

## Next step

- 以后对 reusable 脚本的修改优先落到 `SDSpipeline/`，而不是继续直接改 legacy `sds/scripts/`。
- 新的 compute / audit / demography run 优先把输入输出根目录切到 `SDSworkspace/`，并配套写 `run_manifest.yaml`。
- 第二阶段再逐步把历史结果、历史 input lineage 和真正的一次性脚本从旧目录迁移到 `SDSworkspace/legacy` 与 `SDSworkspace/oneoff`。

