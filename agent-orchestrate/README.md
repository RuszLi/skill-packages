# agent-orchestrate

用于强制主会话只做编排、把实现/修复/验证/审查等执行任务分派给独立上下文 subagent 的技能包。

## 目录结构

```text
agent-orchestrate/
  SKILL.md
  README.md
  agents/
    openai.yaml
```

## 安装 / 复制方式

将仓库中的 `agent-orchestrate/` 整个目录复制到本地 Codex skills 目录下：

```powershell
Copy-Item -Path .\agent-orchestrate -Destination "$env:USERPROFILE\.codex\skills\" -Recurse -Force
```

目标应为：

```text
$HOME/.codex/skills/agent-orchestrate/SKILL.md
```

## 使用方式

在需要实现、修复、验证、审查或执行计划时启用该技能；主会话只负责：

- 读取规则与拆分任务
- 选择模型与推理等级
- 打包自包含任务并分派 subagent
- 审查返回结果与证据核验

具体执行（代码修改、命令验证、诊断排查）必须由 subagent 完成。

## 规则加载策略

该技能默认采用“两层加载”：

- 主 agent：完整读取项目规则，例如 `AGENTS.md`、`SOUL.md`、`USER.md`、`CLAUDE.md`
- subagent：默认只接收主 agent 提炼后的最小必要规则摘要，不重复全量加载整套规则文件

这样做的目的：

- 降低每个 subagent 的前缀 token 消耗
- 提高并行分派时的提示稳定性
- 提高缓存复用概率
- 避免轻量任务被无关规则淹没

默认摘要至少应包含：

- 当前任务的成功标准
- 当前任务的禁止事项
- 与任务直接相关的项目硬约束
- 允许修改和禁止修改的文件范围

仅在以下情况建议把完整规则一并传给 subagent：

- 复杂根因分析
- 架构级或跨模块改动
- 高风险安全 / 交易 / 审计任务
- 主 agent 判断摘要不足以约束边界

轻量搜索、字段核对、局部实现、机械性修改、常规测试建议，默认只传摘要。

## 关键行为

1. **Subagent 编排**  
   每个 subagent 必须收到自包含任务包（目标、范围、约束、验证证据要求），禁止依赖“见上文”。

2. **规则摘要优先**  
   主 agent 默认给 subagent 传规则摘要，而不是重复传完整项目规则；只有高风险、高复杂度任务才附全文。

3. **并行优先**  
   可独立推进且写入范围不冲突的任务默认并行分派；仅在依赖关系、写集冲突或集成前置验收时串行。

4. **同链复用**  
   出现缺口时先判断是否同一任务链；同链优先复用原 agent（`send_input` / `resume_agent` 后继续 `wait_agent`）。

5. **任务链结束再 close**  
   `close_agent` 仅在结果已接受、无需继续上下文且任务链结束后执行，避免过早关闭导致断链。
