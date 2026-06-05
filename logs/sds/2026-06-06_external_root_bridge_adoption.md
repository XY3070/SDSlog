# 2026-06-06 External Root Bridge Adoption

## Context

截至 2026-06-05，`SDSpipeline` 的路径 contract 已经改成通过本地 `config/paths.env` 解析服务器相关路径，但这台机器上的真实外部资产仍分散在 legacy 位置：

- `ms/`
- `relate/`
- `smcpp_latest.sif`
- `smcpp_latest.sandbox`
- `third_party/smcpp_matplotlib`
- `third_party/singularity_conf`
- `/usr/bin/singularity`

如果本地 `paths.env` 继续直接指向这些分散位置，那么虽然脚本不再写死绝对路径，但“canonical external root” 仍不存在。

## Decision

先不做物理搬迁，而是先把当前服务器切到一个稳定的 canonical external root：

- `SDSworkspace/external/`

具体做法是建立 symlink bridge，使 pipeline 统一只看 `SDSworkspace/external/...`，而 symlink 再指向旧位置的真实资产。

## Implementation

已创建以下 canonical 路径：

- `SDSworkspace/external/ms -> /data/home/grp-wangyf/xuyuan/ms`
- `SDSworkspace/external/relate -> /data/home/grp-wangyf/xuyuan/relate`
- `SDSworkspace/external/smcpp/smcpp_latest.sif -> /data/home/grp-wangyf/xuyuan/smcpp_latest.sif`
- `SDSworkspace/external/smcpp/smcpp_latest.sandbox -> /data/home/grp-wangyf/xuyuan/smcpp_latest.sandbox`
- `SDSworkspace/external/smcpp/smcpp_matplotlib -> /data/home/grp-wangyf/xuyuan/third_party/smcpp_matplotlib`
- `SDSworkspace/external/smcpp/singularity_conf -> /data/home/grp-wangyf/xuyuan/third_party/singularity_conf`
- `SDSworkspace/external/singularity/bin/singularity -> /usr/bin/singularity`

同时：

- 更新了本地未跟踪 `SDSpipeline/config/paths.env`，使 `SDS_MS_ROOT`、`SDS_RELATE_DIR`、`SDS_SMCPP_*` 全部先指向 `SDSworkspace/external/`
- 新增 `SDSworkspace/external/README.md`
- 新增 `SDSworkspace/provenance/datasets/2026-06-06_external_asset_bridge.tsv`

## Consequence

从今天开始，这台服务器上的 canonical 外部资产入口已经不再是 legacy 顶层目录，而是：

- `SDSworkspace/external/ms`
- `SDSworkspace/external/relate`
- `SDSworkspace/external/smcpp/...`

这意味着后续即使真的物理迁移这些大文件，也只需要：

1. 替换 symlink target 或把 symlink 改成真实目录/文件；
2. 保持 `paths.env` 不变或只做局部修正；
3. 在 provenance 里记录这次迁移。

## Verification

- `common_env.sh` 已能解析出新的 `SDS_MS_ROOT`、`SDS_RELATE_DIR`、`SDS_SMCPP_ROOT`、`SDS_SMCPP_IMAGE`、`SDS_SMCPP_SINGULARITY_CONFDIR`、`SDS_SMCPP_SINGULARITY_BIN`
- 所有新建 bridge 路径均通过 `test -e`
- 新迁入的 shell / python helper 继续通过 `bash -n` 与 `py_compile`

## Remaining gap

当前 external root 仍然是“canonical by symlink”，不是“canonical by physical relocation”。

这已经足够解决脚本入口和路径 contract 的问题，但还没有完成最终的数据/依赖搬家。后续如果要彻底收口，需要把外部资产逐步物理迁入 `SDSworkspace/external/`，并保留相应 provenance。
