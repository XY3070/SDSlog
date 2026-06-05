# 2026-06-06 Input Root Bridge Adoption

## Context

在 `SDSworkspace/external/` 建立 canonical bridge 之后，当前服务器的 input 侧仍有一个对称问题：

- `SDS_VCF_ROOT` 还直接指向 legacy `sds/data/vcf`
- freeze sample list 的 canonical 目录存在，但实际文件还没有迁入

这意味着脚本虽然已经支持 `SDSworkspace/` 语义，但当前机器的默认输入入口仍未完全收口到新结构。

## Decision

继续采用和 external root 一样的过渡策略：

- 先建立 canonical input bridge
- 不直接搬动原始大输入目录
- 让 `paths.env` 和 pipeline 默认入口都优先经过 `SDSworkspace/input/...`

## Implementation

已建立以下 canonical input 路径：

- `SDSworkspace/input/raw/vcf -> /data/home/grp-wangyf/xuyuan/sds/data/vcf`
- `SDSworkspace/input/freeze/sample_lists/NCN.txt -> /data/home/grp-wangyf/xuyuan/sds/data/NCN.txt`
- `SDSworkspace/input/freeze/sample_lists/SCN.txt -> /data/home/grp-wangyf/xuyuan/sds/data/SCN.txt`
- `SDSworkspace/input/freeze/sample_lists/MERGED3971.txt -> /data/home/grp-wangyf/xuyuan/sds/data/MERGED3971.txt`
- `SDSworkspace/input/freeze/sample_lists/MERGED3971PL.txt -> /data/home/grp-wangyf/xuyuan/sds/data/MERGED3971PL.txt`

同时：

- 本地未跟踪 `SDSpipeline/config/paths.env` 已将 `SDS_VCF_ROOT` 改回 `${SDS_INPUT_ROOT}/raw/vcf`
- 新增 `SDSworkspace/provenance/datasets/2026-06-06_input_bridge.tsv`
- `SDSworkspace/README.md` 与 `SDSworkspace/input/README.md` 已说明 canonical input path 允许处于 symlink bridge 过渡态

## Consequence

从今天开始，这台服务器上的 canonical input 入口已经变成：

- `SDSworkspace/input/raw/vcf`
- `SDSworkspace/input/freeze/sample_lists/*.txt`

因此后续如果需要真正把 VCF 或 sample list 物理迁走，只需要修改 symlink target 或替换成真实文件，而不需要再改 pipeline 入口。

## Verification

- `SDSworkspace/input/raw/vcf` 与 4 个 sample list bridge 均通过 `test -e`
- `common_env.sh` 继续能在本地 `paths.env` 下解析 canonical input root

## Remaining gap

当前 input root 仍然是“canonical by symlink”，不是“canonical by physical relocation”。

此外，freeze 体系本身还没有完全 backfill 到正式 cohort manifest；目前只是先把最常用 sample list 入口收口到新结构。
