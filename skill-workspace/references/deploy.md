# Skill 部署指南

> 面向所有 Agent（Claude Code、Codex CLI、ChatGPT 等）

## 部署位置

不同 Agent 的全局 skills 目录：

| Agent | 全局路径 | 项目路径 |
|-------|----------|----------|
| Claude Code | `~/.claude/skills/<name>/` | `.claude/skills/<name>/` |
| Codex CLI | `~/.codex/skills/<name>/` | `.codex/skills/<name>/` |
| ChatGPT 等 | `~/.openai/skills/<name>/` | `.openai/skills/<name>/` |
| 通用 | `~/.skills/<name>/` | `.skills/<name>/` |

**本指南使用 `{全局skills目录}` 作为占位符，请替换为你的 Agent 对应的路径。**

## 部署流程

### 前提条件
- 本地测试通过
- 建议先走「测评」流程，评级 ≥ C

### 部署步骤

1. **确认来源**
   ```bash
   # 检查当前目录中的 skill 目录
   ls ./{skill名}/
   ```

2. **复制到全局**
   ```bash
   # ⚠️ 永远用 cp，不用 mv
   cp -r ./{skill名}/ {全局skills目录}/{skill名}/
   ```

3. **验证部署**
   ```bash
   # 检查文件
   ls {全局skills目录}/{skill名}/

   # 确认 SKILL.md 存在
   cat {全局skills目录}/{skill名}/SKILL.md | head -5
   ```

4. **测试全局可用**
   ```bash
   # 使用你的 Agent CLI 测试
   # Claude Code: claude -p "使用 {name} 完成 XXX 任务"
   # Codex CLI:   codex "使用 {name} 完成 XXX 任务"
   # ChatGPT 等: {agent-cli} -p "使用 {name} 完成 XXX 任务"
   ```

## 版本管理建议

- **工作区** = 源码仓库（开发、测试、迭代）
- **全局目录** = 部署位置（生产环境）
- 修改时先改工作区，测试通过后再部署
- 保留工作区副本，方便回滚

## 更新已部署的 Skill

1. 在工作区修改并测试
2. 用 cp 覆盖全局目录
3. 验证更新生效

## 回滚

如果部署后发现问题：
```bash
# 从工作区重新部署旧版本
cp -r ./{skill名}/ {全局skills目录}/{skill名}/
```

## ⚠️ 重要规则

- **永远用 `cp`，不用 `mv`** — 保留源文件
- **部署前先测试** — 不要直接推到全局
- **保留工作区副本** — 方便回滚和迭代
