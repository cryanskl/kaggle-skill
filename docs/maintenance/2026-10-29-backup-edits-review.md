# 安装备份中七个未提交修改文件的复核

日期：2026-10-29。对照维护仓库基线 `880a4f9`，审查旧安装本地提交 `2bdf79f` 之上的未提交增量。原始补丁 SHA256 为 `8c3fb1ca6f1da296c1070767e3b119233b5c03d62d3dfbd36d09e372692df1e1`。仅迁移有用且尚未覆盖的决策规则，不复制旧版文件覆盖当前重构。

## 逐文件结论

路径相对于 `skills/agentic-kaggle-skill/`。

| 备份修改文件 | 已有覆盖 | 本次处理 |
|---|---|---|
| `SKILL.md` | 预测/互动/分析分流、无合格升级的完成状态、fold 写 `processed/` 均已覆盖 | 保留新增的 baseline/plateau 决策入口，路由到干预层选择与 upstream exposure；不恢复旧流程结构 |
| `agents/openai.yaml` | 备份与当前文件逐字节一致 | 不改动，不重复提交 |
| `references/advanced-notebook-architecture.md` | 当前已规定 GPU 或 hidden-test notebook 不必多阶段，互动 reference 也有表示能力检查 | 为预测路线补 metric-gap / development oracle、可表达性、干预收益与成本、有界 pilot 和截止倒排；保留单 kernel 路线 |
| `references/competition-intel.md` | 当前有公开来源渠道与方法提取，互动 reference 有部分复盘边界 | 补通用证据等级、同谱系非独立证据、正文空壳的许可 fallback，以及跨团队结果不能归因组件收益 |
| `references/cross-validation-and-metrics.md` | 当前有基础 GroupKFold 和折内预处理防泄漏 | 补 upstream 标签暴露、未知训练覆盖、独立确认、pooled metric 重算、独立组不确定性与来源标签约定；补 custom scorer/reader 区分 |
| `references/ensembling-and-reproducibility.md` | 当前有 OOF blending、stacking 和制品追溯 | 补真实提交 runtime 与最终 writer 的同输入复核、容差/字节身份范围，以及失败实验的拒绝与重开证据 |
| `references/submission-endgame.md` | 预算、模式化验收、incumbent 保留、部分格式检查已覆盖 | 补序列化 round trip、受控评分尝试与恢复时间、实际 final-selection 规则、互补性和 hindsight selection regret |

七个文件中没有需要完全废弃的核心思想；需要淘汰的是重复内容、旧结构和过强措辞。UI 已完全覆盖；其余六个文件只补缺失部分。

## 合并时收窄的表述

- Oracle 只是在开发数据和已测试候选集合内估计 hindsight headroom，不是可部署得分或所有方法的上限；组件上限可能相互作用。
- 下游新分折不能抹去 upstream 见过验证标签的事实。未知覆盖应标明，不能把条件诊断包装成端到端 OOF。
- 小面板用于调试和便宜否决；晋升使用可行的独立组确认，不复制固定样本量或某队 gate。
- 复盘按比赛使用 held-out/private 或 tournament 结果，不假定每场都有 private split。
- 提交边界 replay 只证明同输入实现一致；数值容差、ID/schema 与最终 writer 都纳入，不推导隐藏测试泛化。
- 历史研究细节留在本记录；运行技能只加载通用 reference。原始备份、补丁、Git bundle 与旧提交保留在本机扫描目录之外，未放入公共仓库。

## 验证范围

两名独立审查员分别复核策略/intel 与 validation/reproducibility 增量。另用只读前向测试检查 upstream 覆盖未知的小面板 stacking、候选空间受限的平台期、writer/评分与 final-selection 三种情境。结构校验包括技能 frontmatter、资源引用、Markdown 本地链接与 `git diff --check`；脚本和运行依赖未修改。

三种情境均走到相应的证据与行动判断，纯维护请求不会触发训练或提交。测试还发现迁入规则与既有集成段的两处衔接需要澄清：相关性用作诊断，不以固定 0.90 阈值自动淘汰；producer 进入 consumer 实验与最终 blend 验收分开。已作最小修正，最终 blend 仍需与最强单模型比较。

补充复核通过：相关性 0.97 但有独立主指标增益的 blend 不会被直接淘汰；降低方差却令完整 blend 主指标退化的 producer 也不能绕过最终验收。技能结构校验、32 个资源/本地链接与 diff 检查通过；两份旧安装备份各 131 个条目的原始文件哈希未改变，UI 确认已被当前版本完全覆盖。

本次属于技能维护，不启动比赛训练、推理或 Kaggle 提交。本机安装通过符号链接直接使用维护仓库技能目录，提交后无需再复制安装文件。
