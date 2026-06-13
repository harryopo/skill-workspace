# Agent Skill 官方规范

> 来源：Agent Skill 规范文档
> 适用于所有支持 SKILL.md 的 Agent（Claude Code、Codex CLI、ChatGPT 等）

---

## 一、Skill 本质

Skill 是一个**目录**，核心是 `SKILL.md` 文件。

```
my-skill/
├── SKILL.md          # 必须
├── references/       # 可选——参考资料，被引用时按需加载
├── assets/           # 可选——模板、图片等
└── scripts/          # 可选——可执行脚本
```

---

## 二、三层渐进式加载

| 层级 | 何时加载 | 上下文占用 |
|------|---------|-----------|
| Level 1：元数据 | 会话启动时 | name + description，~100 tokens/skill |
| Level 2：SKILL.md 正文 | 被调用时 | 全文注入 |
| Level 3：目录资源 | 正文引用时 | 仅读取的文件 |

**关键：** description 始终在上下文里，正文和目录资源只在调用时才加载。

---

## 三、Frontmatter 完整字段（12 字段）

### 标识
```yaml
name: my-skill              # 全小写+数字+连字符，≤64 字符
description: |              # ≤1024 字符（截断为 250 字符）
  做什么 + 什么时候触发
argument-hint: "[参数]"     # 自动补全提示
```

### 触发控制
```yaml
disable-model-invocation: false  # true = 只能手动触发
user-invocable: true              # false = 用户不可见
paths: "*.md, src/**/*.ts"        # 限定文件范围
```

### 执行模式
```yaml
context: fork                     # fork = 独立子 Agent
agent: "general-purpose"          # Explore | Plan | general-purpose
allowed-tools: Read Write Edit    # 免询问工具
model: sonnet                     # 覆盖模型
effort: medium                    # low | medium | high | max
```

### 生命周期
```yaml
hooks:                            # Skill 级钩子
  PostToolUse:
    - matcher: "Write"
      hooks:
        - type: command
          command: "echo 'done'"
shell: bash                       # bash | powershell
```

---

## 四、调用控制矩阵

| 配置 | 用户可调用 | Agent 可自动调用 | description 在上下文 |
|------|-----------|------------------|-------------------|
| 默认 | ✅ | ✅ | ✅ |
| disable-model-invocation: true | ✅ | ❌ | ❌ |
| user-invocable: false | ❌ | ✅ | ✅ |

---

## 五、context: fork vs inline

| | fork 模式 | inline 模式（默认） |
|--|----------|-------------------|
| 对话历史 | 不继承 | 继承完整历史 |
| 上下文消耗 | 独立窗口，不占主对话 | 占用主对话 token |
| 执行 Agent | 由 `agent` 字段指定 | 当前会话 |
| 适用场景 | 独立任务 | 需要对话上下文 |

**判断法则：** 指令自带全部信息用 fork，依赖对话上下文用 inline

---

## 六、字符串替换变量

| 变量 | 说明 |
|------|------|
| `$ARGUMENTS` | 用户传入的所有参数 |
| `$ARGUMENTS[N]` | 按索引访问第 N 个参数 |
| `$N` | `$ARGUMENTS[N]` 的简写 |
| `${SKILL_DIR}` | SKILL.md 所在目录绝对路径 |
| `${SESSION_ID}` | 当前会话 ID |

---

## 七、Shell 预处理

```markdown
单行：!`git status --short`
多行：
```!
find . -name "*.md" | head -20
```
```

---

## 八、存放位置

| 优先级 | 路径 | 范围 |
|--------|------|------|
| 1 | `~/.skills/<name>/SKILL.md` | 用户全局（通用） |
| 2 | `.skills/<name>/SKILL.md` | 当前项目 |

**注意：** 不同 Agent 的路径可能不同：
- Claude Code: `~/.claude/skills/`
- Codex CLI: `~/.codex/skills/`
- Cursor: `~/.cursor/skills/`
- 通用: `~/.skills/`

---

## 九、Description 预算

```
所有 Skill 的 description 共享上下文预算：
- 默认 = 上下文窗口的 1%，fallback 为 8000 字符
- 每个 Skill 的 description 在列表中截断为 250 字符
```

---

## 十、最佳实践

- description 前 250 字符包含触发词
- 正文 ≤500 行
- 参考资料放 references/
- 用 ${SKILL_DIR} 引用文件
- 复杂任务用 context: fork
