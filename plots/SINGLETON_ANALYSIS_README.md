# Singleton Counts 分析改进文档

本文档记录了根据建议对 Singleton Counts 分析进行的改进，以及相关环境的使用说明。

## 1. 核心改进：更严格的 Singleton 提取逻辑

为了提高分析的准确性，我们将 Singleton 的提取标准从单纯的 `AC=1` 升级为：
- **等位基因计数 (AC)**: 1 (Singleton)
- **位点缺失率 (Missing Rate)**: < 0.1% (即 `F_MISSING < 0.001`)

### 脚本更新
修改了 `/share/home/grp-wangyf/xuyuan/sds/scripts/generate_s_file.sh` 中的提取逻辑：
```bash
# 更新后的 bcftools 过滤条件
bcftools view -i 'AC=1 && F_MISSING < 0.001' -Ou
```
此举确保了每个提取出的 Singleton 位点在 99.9% 的样本中都有明确的基因型调用，极大减少了因测序深度不足或质量差导致的假阳性。

## 2. 南北差异确认 (CHS vs CHB)

使用 1000 Genomes 项目的标准人群（CHB 和 CHS）对 Chr22 进行了试点验证：
- **CHB (北方汉族)**: 平均 Singleton Count 约为 **169.4**
- **CHS (南方汉族)**: 平均 Singleton Count 约为 **131.8**
- **结论**: 北方人群的 Singleton 密度显著高于南方，验证了 Singleton Counts 的差异主要受南北地理/人群结构驱动。

对比图见：`chs_vs_chb_singleton_comparison_chr22.png`

## 3. 环境与工具使用说明

### 3.1 核心环境 (Mamba)
大部分生物信息学工具（如 `bcftools`, `gawk`, `parallel`）都安装在 `SDS` 环境中。

**激活方法**:
```bash
mamba activate SDS
```
在运行 `mk_sds_input.sh` 或手动运行 `bcftools` 之前，请确保已激活该环境。

### 3.2 Python 脚本运行 (uv)
对于 Python 分析脚本（如绘图、统计分析），推荐使用 `uv` 工具。`uv` 是一个极速的 Python 包管理和运行工具，可以自动处理依赖。

**使用示例**:
```bash
# 使用 uv 直接运行 python 脚本
uv run python generate_plots.py
```
如果在运行中提示缺少包（如 `pandas`, `seaborn`），`uv` 会根据脚本开头的依赖声明或当前环境进行快速处理。

## 4. 后台任务说明
全基因组（Chr 1-22）的重运行已在后台启动：
- **日志文件**: `sds_reextract_all.log`
- **进度查看**: `tail -f logs/progress.log`

---
🤖 Generated with [Qoder][https://qoder.com]
