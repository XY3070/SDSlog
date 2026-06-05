# 2026-06-05 Cross-Server Portability Contract

## Context

在 `SDSlog/`、`SDSpipeline/`、`SDSworkspace/` 三层结构明确之后，又进一步确认一个现实约束：大文件、外部二进制和历史中间结果在不同服务器上的路径与可用性并不一致。如果继续把这些路径写死进 pipeline repo，那么：

- 同一套脚本无法直接迁移到另一台服务器；
- 一次性补丁脚本会被误吸收到可复用 repo；
- provenance 会再次退化成“记得大概在哪，但无法严格复现”的状态。

## Decision

正式把跨服务器 contract 收紧为：

1. `SDSpipeline/` 只保存可复用脚本、模板、契约和少量示例元数据。
2. 所有服务器特定的绝对路径只允许出现在本地未跟踪文件 `SDSpipeline/config/paths.env`。
3. 所有大输入、大输出、run root、queue log、cache、外部依赖和一次性脚本都留在非 git 的 `SDSworkspace/`。
4. 某台服务器专用的小修补脚本、桥接脚本、恢复脚本，统一放在 `SDSworkspace/oneoff/`，不进入 `SDSpipeline/`。
5. 如果某个关键文件只存在于部分服务器，必须把“该文件在哪台服务器存在、实际路径是什么、缺失时用了什么替代物”写进 `SDSworkspace/provenance/`。

## Implementation completed today

- `SDSpipeline/scripts/common_env.sh` 现在负责加载本地 `config/paths.env`，并统一导出：
  - `SDS_WORKSPACE_ROOT`
  - `SDS_SDS_INPUT_ROOT`
  - `SDS_SDS_OUTPUT_ROOT`
  - `SDS_DEMOGRAPHY_ROOT`
  - `SDS_GAMMA_ROOT`
  - `SDS_MS_*`
- 主入口默认不再指向 legacy `sds/data/...`，而是优先走 `SDSworkspace/` 下的 canonical root：
  - `scripts/run_sds_input.sh`
  - `scripts/run_sds_compute.sh`
  - `scripts/submit_sds_compute_chunked_chr.sh`
  - `scripts/run_raw_joint_chr_inputs.sh`
  - `scripts/submit_sds_compute_chunked_chr_via_smp.sh`
  - `scripts/run_single_snp_gamma_chunk.sh`
  - `scripts/run_smcpp_recent_resolution_workflow.sh`
  - `scripts/submit_smcpp_recent_job.sh`
- `bootstrap_workspace.sh` 改为按 repo 相对位置或 `SDS_WORKSPACE_ROOT` 创建 workspace 骨架，不再写死当前服务器路径。
- `config/paths.env.example` 改成真正的跨服务器模板，不再把当前机器路径伪装成规范。
- 当前服务器也已实际创建本地未跟踪 `SDSpipeline/config/paths.env`，以 `SDSworkspace/` 为 canonical run root，同时用局部 override 指向仍留在 legacy 位置的 `smcpp` image/rootfs、`ms/` 与 `relate/` 资产。
- `README.md`、`WORKSPACE_CONTRACT.md`、`PROJECT_LAYOUT.md` 和新增的 `docs/SERVER_PORTABILITY.md` 都已明确这条规则。
- 后续又把先前仍滞留在 legacy `sds/scripts/` 的 demography helper 迁入 `SDSpipeline/scripts/`：
  - `run_smcpp_benchmark.py`
  - `run_smcpp_benchmark_lsf.sh`
  - `prepare_smcpp_runtime.sh`
  - `run_relate_download_refs.sh`
- 上述 helper 也已改成默认通过 `SDS_DEMOGRAPHY_ROOT`、`SDS_VCF_ROOT`、`SDS_SMCPP_*`、`SDS_RELATE_*` 等本地配置变量解析路径，而不是再假定当前服务器目录结构。

## Consequence

今后把项目迁到另一台服务器时，默认只应需要：

1. 新建或修改该机器的 `SDSpipeline/config/paths.env`；
2. 补少量仅该服务器需要的 `SDSworkspace/oneoff/` 脚本；
3. 在 provenance manifest 里记录大文件与依赖的实际落点。

如果迁移需要修改 tracked pipeline 脚本里的绝对路径，应视为 contract 被破坏，而不是正常操作。

## Remaining gaps

- `smcpp` / `relate` 主依赖脚本已经迁入 `SDSpipeline/`，但它们仍依赖 `SDS_EXTERNAL_ROOT` 下真实存在的 container/runtime/reference 资产，因此“脚本可迁移”不等于“外部大文件已经自动同步”。
- 历史产物仍未完全补 provenance；尤其 olddefault gamma family 仍然是方法学 blocking issue，而不只是路径问题。
- 旧目录中的 reusable 代码还需要继续盘点，确认哪些是真正 canonical、哪些只该留在 legacy 供审计参考。
