# Agent 中立指南

## 概述

Skill 必须能在任何支持 SKILL.md 的 Agent 中正常运行。这意味着：
- Skill 的核心逻辑和功能不应依赖于特定 Agent 的专有特性
- 所有 Agent 特定的路径、命令或配置都应通过可替换的占位符或配置文件处理
- 确保 Skill 在不同 Agent 环境中具有一致的行为和用户体验

## 部署路径

不同 Agent 的全局 Skill 目录位置如下：

### Claude Code
- **Windows**: `%USERPROFILE%\.claude\skills\`
- **macOS/Linux**: `~/.claude/skills/`

### Codex CLI
- **全局配置目录**: `~/.codex/skills/`
- **项目级目录**: `./skills/`

### ChatGPT
- **全局配置目录**: `~/.chatgpt/skills/`
- **插件目录**: `~/.chatgpt/plugins/`

### Cursor
- **全局配置目录**: `~/.cursor/skills/`
- **项目级目录**: `./.cursor/skills/`

### OpenClaw
- **全局配置目录**: `~/.openclaw/skills/`
- **工作空间目录**: `./openclaw-skills/`

### Trae
- **全局配置目录**: `~/.trae/skills/`
- **项目级目录**: `./trae-skills/`

### 通用路径
- **环境变量**: `${SKILLS_HOME}` 或 `${AGENT_SKILLS_DIR}`
- **默认全局**: `~/.{agent-name}/skills/`
- **默认项目**: `./skills/` 或 `./.{agent-name}/skills/`

## 替换规则

### 必须替换的内容

在将 Skill 从一个 Agent 迁移到另一个 Agent 时，以下内容必须替换：

1. **路径占位符**:
   - `~/.claude/skills/` → `{全局skills目录}/`
   - `%USERPROFILE%\.claude\skills\` → `{全局skills目录}\` (Windows)
   - `~/.{agent-name}/skills/` → `{全局skills目录}/`

2. **Agent 名称引用**:
   - `Claude Code` → `Agent` (通用术语)
   - `claude-code` → `{agent-name}` (命令行标识)
   - `Claude` → `Agent` (上下文提及)

3. **配置文件路径**:
   - `.claude` → `.{agent-name}`
   - `claude.config` → `{agent-name}.config`

### 保留原样的内容

以下内容应保持原样，不做替换：

1. **技术实现细节**:
   - 具体的命令行参数和选项
   - API 调用和响应格式
   - 文件格式和编码规范
   - 算法逻辑和数据处理流程

2. **命令示例**:
   - 具体的 shell 命令示例
   - 脚本代码片段
   - 配置文件示例

3. **平台特定信息**:
   - 操作系统特定的路径分隔符
   - 环境变量语法
   - 权限和访问控制

## 部署命令模板

### 通用部署命令

```bash
# 创建全局 Skill 目录（如果不存在）
mkdir -p {全局skills目录}

# 复制 Skill 文件到全局目录
cp -r ./skill-name {全局skills目录}/

# 设置执行权限（如果需要）
chmod +x {全局skills目录}/skill-name/scripts/*.sh

# 验证部署
ls -la {全局skills目录}/skill-name/
```

### Windows PowerShell 部署命令

```powershell
# 创建全局 Skill 目录（如果不存在）
if (!(Test-Path -Path "{全局skills目录}")) {
    New-Item -ItemType Directory -Path "{全局skills目录}" -Force
}

# 复制 Skill 文件到全局目录
Copy-Item -Path ".\skill-name" -Destination "{全局skills目录}\" -Recurse -Force

# 验证部署
Get-ChildItem -Path "{全局skills目录}\skill-name"
```

### Agent 特定部署命令

```bash
# Claude Code 部署
cp -r ./my-skill ~/.claude/skills/

# Codex CLI 部署
cp -r ./my-skill ~/.codex/skills/

# ChatGPT 部署
cp -r ./my-skill ~/.chatgpt/skills/

# Cursor 部署
cp -r ./my-skill ~/.cursor/skills/

# OpenClaw 部署
cp -r ./my-skill ~/.openclaw/skills/

# Trae 部署
cp -r ./my-skill ~/.trae/skills/
```

## 测试命令模板

### 通用测试命令

```bash
# 检查 Skill 是否正确安装
ls {全局skills目录}/skill-name/SKILL.md

# 验证 Skill 文件完整性
find {全局skills目录}/skill-name -type f | wc -l

# 检查文件权限
ls -la {全局skills目录}/skill-name/scripts/

# 运行 Skill 的测试脚本（如果存在）
if [ -f "{全局skills目录}/skill-name/tests/test.sh" ]; then
    bash {全局skills目录}/skill-name/tests/test.sh
fi
```

### Agent 特定测试命令

```bash
# Claude Code 测试
claude skill list | grep "skill-name"

# Codex CLI 测试
codex skill test skill-name

# ChatGPT 测试
chatgpt skill validate skill-name

# Cursor 测试
cursor skill check skill-name

# OpenClaw 测试
openclaw skill verify skill-name

# Trae 测试
trae skill test skill-name
```

### 环境验证测试

```bash
# 检查 Agent 是否支持 SKILL.md
if [ -f "{全局skills目录}/skill-name/SKILL.md" ]; then
    echo "SKILL.md 存在"
    cat {全局skills目录}/skill-name/SKILL.md | head -5
else
    echo "SKILL.md 不存在"
    exit 1
fi

# 验证 SKILL.md 格式
if grep -q "name:" {全局skills目录}/skill-name/SKILL.md; then
    echo "SKILL.md 格式正确"
else
    echo "SKILL.md 格式可能有问题"
fi
```

## 最佳实践

1. **使用配置文件**: 将 Agent 特定的配置放在单独的配置文件中，而不是硬编码在代码里
2. **环境变量**: 使用环境变量来存储路径和配置，便于不同环境切换
3. **文档更新**: 在迁移 Skill 时，更新所有相关文档中的 Agent 特定引用
4. **测试覆盖**: 确保在目标 Agent 环境中进行充分测试
5. **版本控制**: 使用版本控制来跟踪不同 Agent 环境的配置差异

## 常见问题

### Q: 如何确定当前使用的是哪个 Agent？
A: 检查环境变量或配置文件中的 Agent 标识符。大多数 Agent 会设置 `AGENT_NAME` 或类似环境变量。

### Q: 如果 Skill 依赖于特定 Agent 的 API 怎么办？
A: 创建适配器层来抽象 Agent 特定的 API 调用，使用策略模式或依赖注入来切换实现。

### Q: 如何处理不同 Agent 的权限模型差异？
A: 在 SKILL.md 中明确声明所需的权限，并在部署脚本中检查和设置适当的权限。

### Q: 是否需要为每个 Agent 维护单独的 Skill 版本？
A: 理想情况下不需要。使用本文档中的替换规则，可以维护一个通用版本，通过配置适配不同 Agent。

---

*本指南旨在帮助开发者创建和维护 Agent 中立的 Skill，确保在不同 Agent 环境中的一致性和可移植性。*