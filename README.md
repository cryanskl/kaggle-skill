# Agentic Kaggle Skill / 智能体驱动的 Kaggle 工作流

Reusable workflows for predictive ML, interactive simulation agents, and evidence-based competition reviews.
用于预测建模、互动仿真智能体和竞赛复盘的可复用工作流。

Maintained here: [cryanskl/kaggle-skill](https://github.com/cryanskl/kaggle-skill).
统一技能名称为 `agentic-kaggle-skill`，入口位于 [skills/agentic-kaggle-skill/SKILL.md](skills/agentic-kaggle-skill/SKILL.md)。

## Scope / 适用范围

| Request / 请求 | Workflow / 工作方式 |
| --- | --- |
| Predictive ML / 预测建模 | Match validation to the test distribution, build a viable baseline, and compare models using leakage-safe held-out evidence. 按泛化目标设计验证，使用无泄漏留出证据比较模型。 |
| Interactive agents / 互动智能体 | Validate replay semantics, executable policies, and complete games against representative opponents. 核对回放与动作语义，以完整互动对局评估策略。 |
| Training or validation only / 限定阶段 | Deliver the requested model, evaluation, or artifact without adding a submission request. 按授权阶段交付，不隐含参赛提交。 |
| Review or skill maintenance / 复盘与技能维护 | Audit primary sources and actual artifacts, then make supported changes. 不启动比赛训练或提交。 |

Local execution, a single Kaggle kernel, and staged producer/consumer pipelines are options chosen by evidence and budget. Submission formats and completion states follow the competition contract and the user's requested phase.
本地、单 kernel 和多阶段流水线按证据与预算选择；提交格式和完成条件遵循比赛契约与用户要求的阶段。

## Install And Update / 安装与更新

Use this repository as the update source. Keep a persistent checkout and back up an existing installation before replacing or relinking it.
统一从本仓库更新。使用持久目录保存源码；已有安装先备份，再切换入口。

### Codex

For a new installation / 首次安装：

```bash
mkdir -p "$HOME/.local/share" "$HOME/.agents/skills"
git clone https://github.com/cryanskl/kaggle-skill.git "$HOME/.local/share/kaggle-skill"
ln -s "$HOME/.local/share/kaggle-skill/skills/agentic-kaggle-skill" "$HOME/.agents/skills/agentic-kaggle-skill"
```

If a checkout already exists, reuse its actual path instead of cloning another copy. If the skill path exists, preserve its changes before switching the link. The linked installation follows changes in that checkout.
已有 checkout 时复用实际路径；已有技能目录时先保留改动，再切换链接。链接安装直接使用源码目录的内容。

Update the checkout / 更新源码：

```bash
git -C "$HOME/.local/share/kaggle-skill" pull --ff-only origin main
```

Use the path of your existing checkout when it differs. A copied installation needs an explicit resync after updating; a linked installation does not.
若现有 checkout 位于其他路径，使用该路径。复制安装需要另行同步，链接安装无需再次复制。

Invoke / 调用：

```text
Use $agentic-kaggle-skill to help with this Kaggle request.
```

For repository-scoped use, link the same skill subdirectory under that repository's `.agents/skills/`.
仓库级使用时，将同一技能子目录链接到项目的 `.agents/skills/`。

### Claude Code

```text
/plugin marketplace add cryanskl/kaggle-skill
/plugin install kaggle-skill@kaggle-skill
```

Invoke with `/kaggle-skill:agentic-kaggle-skill`. The plugin uses the same skill instructions.
插件使用同一套技能说明。

### Hermes And Other Hosts / 其他宿主

Install the complete `skills/agentic-kaggle-skill/` directory using the host's skill mechanism, including its references and scripts. Preserve local changes before refreshing a copied installation.
按宿主的技能机制安装完整子目录，包含 references 与 scripts；更新复制安装前保留本地改动。

### Helper Dependencies / 辅助工具依赖

The helpers are optional if the project already has equivalent tools. From the repository root / 从仓库根目录：

```bash
python3 -m pip install -r skills/agentic-kaggle-skill/requirements.txt
python3 skills/agentic-kaggle-skill/scripts/prepare_kaggle_kernel.py --help
```

Most helpers use the Python standard library; `make_folds.py` uses pandas/numpy with optional scikit-learn splitters.

## Layout And Rule Ownership / 目录与规则归属

```text
kaggle-skill/
├── .claude-plugin/                    # plugin metadata
├── docs/                              # historical cases and maintenance records
└── skills/agentic-kaggle-skill/
    ├── SKILL.md                       # request scope, workflow, resource routing
    ├── agents/openai.yaml             # Codex UI metadata
    ├── references/                    # details loaded when relevant
    ├── scripts/                       # optional repeatable helpers
    └── requirements.txt
```

| Canonical reference / 维护位置 | Responsibility / 职责 |
| --- | --- |
| [Validation](skills/agentic-kaggle-skill/references/cross-validation-and-metrics.md) | Splits, held-out/OOF definitions, upstream exposure, metric aggregation and confirmation. |
| [Simulation agents](skills/agentic-kaggle-skill/references/simulation-game-agents.md) | Episode data, action semantics, interactive evaluation and BC/RL choices. |
| [Pipeline](skills/agentic-kaggle-skill/references/kaggle-pipeline.md) | Submission contracts, inference servers and reusable artifact handoffs. |
| [Compute](skills/agentic-kaggle-skill/references/kaggle-offload.md) | Remote execution, run logs and output retrieval. |
| [Endgame](skills/agentic-kaggle-skill/references/submission-endgame.md) | Authorized scoring, completion/stop states, receipts and final selection. |
| [Debugging](skills/agentic-kaggle-skill/references/code-competition-debugging.md) | Failure diagnosis and budgeted retries. |

The entry point links the remaining modality, ensembling, architecture, intel and sharing references. Historical `docs/` are not required by the installed skill.
其余模态、集成、架构、情报与分享规则由主入口按需引用；安装后的技能不依赖历史 `docs/`。

## Maintain / 维护

- Keep detailed rules in their owning reference and link to them elsewhere. 规则集中维护，其他位置引用。
- Treat defaults and examples as choices; reserve hard requirements for contracts, evidence integrity and actual constraints. 区分建议、示例与真实约束。
- Check skill metadata, resource links and realistic request behavior before publishing. 发布前校验结构与真实请求下的决策。
- Follow [sharing and licensing guidance](skills/agentic-kaggle-skill/references/information-sharing-policy.md) before publishing competition-derived material.

[Historical examples](docs/examples/) · [Research notes](docs/research/) · [Maintenance records](docs/maintenance/)

## Attribution And License / 归属与许可

Based on the original [Frank S (IntelLab) skill](https://github.com/FrankS-IntelLab/agentic-kaggle-skill), with maintenance and extensions in this repository. Preserve third-party attribution when adapting public material.
基于原始技能维护与扩展；使用外部材料时保留适用的第三方归属。

MIT. See [LICENSE](LICENSE).
