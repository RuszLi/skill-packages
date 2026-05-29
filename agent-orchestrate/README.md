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

## 关键行为

1. **Subagent 编排**  
   每个 subagent 必须收到自包含任务包（目标、范围、约束、验证证据要求），禁止依赖“见上文”。

2. **并行优先**  
   可独立推进且写入范围不冲突的任务默认并行分派；仅在依赖关系、写集冲突或集成前置验收时串行。

3. **同链复用**  
   出现缺口时先判断是否同一任务链；同链优先复用原 agent（`send_input` / `resume_agent` 后继续 `wait_agent`）。

4. **任务链结束再 close**  
   `close_agent` 仅在结果已接受、无需继续上下文且任务链结束后执行，避免过早关闭导致断链。
