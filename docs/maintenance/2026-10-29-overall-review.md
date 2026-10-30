# 主入口、references 与 README 整体复核

日期：2026-10-29。基线 `347fc4c`。本轮只调整技能说明和维护记录，未改脚本、依赖、插件身份或既有备份；未运行比赛训练或 Kaggle 提交。

## 调整结果

| 范围 | 复核前 | 复核后 | 主要变化 |
|---|---:|---:|---|
| `SKILL.md` | 207 行 | 72 行 | 只保留请求范围、模式分流、最短工作循环、按需引用与工具入口 |
| README | 329 行 | 123 行 | 合并重复能力/维护介绍，修正技能子目录，改为持久 checkout 与可更新安装 |
| 13 份 reference | 1,114 行 | 661 行 | 规则归属集中，去掉重复 DAG/完成状态与任意架构配方 |

总计由 1,650 行缩为 856 行；历史记录不计入运行说明。长度是结果，不是删减目标。上游标签暴露、独立确认、真实指标聚合、代理对手边界和提交 writer/action 链等证据要求均保留。

## 修正的冲突和过约束

- **请求范围**：训练、验证、修代码与复盘不自动变成参赛提交。`submission-endgame.md` 统一维护评分请求的完成/停止状态；`stopped` 明确表示预算到达但评分目标未完成，不伪装成功或外部阻塞。
- **验证定义**：CV reference 定义无泄漏 held-out 证据，可使用 OOF 或独立 holdout；按未见实体或已知实体未来预测决定 group/time。全量最终 refit 不强制再做 K 折，stacker 的拟合分数也不自动成为独立验证。
- **架构选择**：单 GPU、代码赛和多模态本身不强制多 producer。固定 producer 数、embedding→模型→后处理升级顺序、文本双轨和全部单模态先成熟等要求改为按证据选择。
- **托管契约**：不能从通用 400/403/ERROR 推断规则未接受或 code-only；空 visible test 按 starter 处理，不统一制造 placeholder/CSV。file、server、model/archive 与 agent 各走实际契约。
- **运行与回执**：consumer 记录运行和制品证据；hosted receipt 在提交并观察结果后由 orchestrator 保存。manifest 在有意义的变化时更新，不要求每次轮询重写。
- **互动任务**：episode 去重与策略谱系追踪分开；支持多玩家、连续 reward、不同 active-slot 数与不可控 seed，不把双人 W/D/L 当成所有仿真的固定输出。
- **来源与分享**：缺正文的纯分析任务记录缺证，不转去训练；公开来源可正常署名。必要非秘密 private refs 可保留在访问控制 manifest，未经授权不公开。

## 验证

三名审查员分别负责执行、验证/模态和互动/来源规则；主节点统一主入口与 README。另以无预设答案的只读前向测试覆盖：

1. 已知门店未来预测，仅训练单 booster 与时间 holdout，不提交。
2. 独立 holdout 已验证后的单 GPU、单 notebook 全量 refit 和模型包。
3. inference server、空 visible test、无 CSV、400 与未来 receipt 的时间顺序。
4. 三玩家连续 reward、不可控 seed、4 个 active slot 和同谱系 replay。
5. 首次评分但约定修复预算耗尽，仍为内部 runtime 失败，无 incumbent。
6. 仅比较指定 writeup，部分正文不可获取；另检查 meta-model 同批拟合/评分的证据边界。

六种情境与 meta-model 检查通过，未发现阻断性冲突。技能结构校验、82 个资源/本地链接、16 个代码块（含 3 个 JSON、8 个 Bash 语法检查）、四个 helper 的 `--help` 与 diff 检查通过。结构检查不能证明未来比赛成绩；本轮没有重新验证未修改的训练算法或平台 API。

本机技能入口继续链接到维护仓库的技能子目录，内容同步；`.DS_Store` 和历史备份不纳入提交。
