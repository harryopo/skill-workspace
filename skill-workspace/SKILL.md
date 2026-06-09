---
name: skill-workspace
description: |
  Skill 全生命周期工作台。当用户提到"skill工作台"、"管理skill"、"搜索skill"、
  "下载skill"、"优化skill"、"部署skill"时触发。开发和审查功能由子技能处理。
  面向所有 Agent（Claude Code、Codex CLI、ChatGPT 等），一站式完成 Skill 的
  搜索、下载、安全审查、开发生成、优化改进、质量测评、部署上线、更新卸载。
argument-hint: "[子命令] [参数]"
context: fork
agent: general-purpose
allowed-tools: Read Write Edit Glob Grep Bash WebFetch
---

# Skill 全生命周期工作台（v3.0）

一站式 Skill 管理平台，覆盖 Skill 从发现到退役的完整生命周期。

## 核心原则

> **深入理解需求，全网搜索最佳方案，分阶段开发，循环审查，完整交付。**

## 子命令路由

| 子命令 | 说明 | 处理方式 |
|--------|------|----------|
| **开发** | 从零创建新 Skill | 加载 dev 子技能 |
| **审查** | 10 维度质量评分 | 加载 review 子技能 |
| **搜索** | 在线搜索可用 Skill | 本文件处理 |
| **下载** | 从 URL/名称/GitHub 安装 Skill | 本文件处理 |
| **安全审查** | 对 Skill 做安全扫描 | 本文件处理 |
| **优化** | 改进现有 Skill 质量 | 本文件处理 |
| **部署** | 安装到全局目录 | 本文件处理 |
| **管理** | 更新、卸载、列表 | 本文件处理 |

**路由规则：**
1. 用户说"开发xxx skill" → 加载 `subskills/dev/SKILL.md`
2. 用户说"审查skill" / "skill评分" → 加载 `subskills/review/SKILL.md`
3. 其他子命令 → 在本文件中处理

## 子技能加载

当路由到子技能时，使用 `Read` 工具加载对应的 SKILL.md：

```
开发 → Read ${CLAUDE_SKILL_DIR}/subskills/dev/SKILL.md
审查 → Read ${CLAUDE_SKILL_DIR}/subskills/review/SKILL.md
```

子技能是完整的包，有自己的 references 和 evals，可独立使用。

---

## 流程一：搜索 Skill

**目标：** 在线查找用户需要的 Skill（面向所有 Agent，不限于 Claude），避免重复造轮子。

**搜索源（按优先级）：**

**Tier 1：Skill 聚合市场（最优先）**

SkillsMP (skillsmp.com) — 1.5M+ skills，兼容所有 Agent（Claude Code、Codex CLI、ChatGPT 等）：
```bash
curl -s "https://skillsmp.com/api/v1/skills/search?q={关键词}&limit=10&sortBy=stars"
```
- 支持匿名访问（有每日速率限制）
- 可用参数：`category`（分类）、`occupation`（职业）、`page`、`limit`
- 返回：skill 名称、描述、GitHub 链接、stars

**Tier 2：Skills.sh 生态（CLI 工具）**
```bash
npx skills find {关键词}
```
- Skills.sh 是开源 Agent Skills 生态的包管理器
- 支持交互式搜索和关键词搜索
- 安装命令：`bash /path/to/skill/scripts/install-skill.sh <owner/repo@skill-name>`
- 检查更新：`npx skills check`
- 批量更新：`npx skills update`

**Tier 3：CocoLoop API**
```bash
curl -s "https://api.cocoloop.com/api/v1/store/skills?page=1&page_size=10&keyword={关键词}&sort=downloads"
```

**Tier 4：GitHub 搜索**
```bash
curl -s "https://api.github.com/search/repositories?q={关键词}+filename:SKILL.md&sort=stars&per_page=5"
```

**Tier 5：clawhub CLI（兜底）**
```bash
npx clawhub@latest search {关键词}
```

**搜索策略：**

```
第一步：SkillsMP API 搜索（覆盖面最广）
  ↓ 无结果或超时
第二步：npx skills find（Skills.sh 生态）
  ↓ 无结果或超时
第三步：CocoLoop API 搜索
  ↓ 无结果或超时
第四步：GitHub API 搜索
  ↓ 无结果或超时
第五步：clawhub CLI 搜索
  ↓ 无结果
第六步：提示用户手动搜索或走「开发」流程
```

**输出格式：**
```
📋 搜索结果（来源：SkillsMP）:
  1. skill-name (⭐ 15.5k stars)
     📝 描述文本
     🔗 GitHub: https://github.com/xxx/xxx
  2. ...
```

**搜索后：**
- 展示结果列表，询问用户是否安装
- 如果都没找到 → 建议用户走「开发」流程

---

## 流程二：下载/安装 Skill

**目标：** 从各种来源安全安装 Skill。

**支持的来源：**

| 来源 | 格式 | 处理方式 |
|------|------|----------|
| URL | `https://.../*.skill` | 直接下载 |
| 名称 | `skill-name` | 搜索 → 确认 → 安装 |
| GitHub | `owner/repo` | 克隆 → 检查 → 安装 |

**安装流程：**

1. **获取 Skill 内容**
   - URL → curl 下载
   - 名称 → 按搜索流程找到后下载
   - GitHub → git clone 或 curl 下载 SKILL.md

2. **安全审查**（强制）
   - 调用「安全审查」流程
   - 评级 ≥ B → 继续
   - 评级 ≤ C → 询问用户是否继续

3. **安装到工作区**
   ```bash
   # 安装到当前工作区（开发模式）
   cp -r {skill目录}/ "./{skill名}/"

   # 或安装到全局（使用模式）
   # 路径因 Agent 而异：
   #   Claude Code: ~/.claude/skills/{skill名}/
   #   Codex CLI:   ~/.codex/skills/{skill名}/
   #   通用:        ~/.skills/{skill名}/
   cp -r {skill目录}/ {全局skills目录}/{skill名}/
   ```

4. **确认安装结果**

---

## 流程三：安全审查

**目标：** 对 Skill 进行安全扫描，识别风险。整合 skill-vetter 的 4 步审查协议。

**审查协议（4 步）：**

### 第 1 步：元数据检查
- [ ] `name` 与预期 skill 名称匹配（无 typosquatting）
- [ ] `version` 遵循语义化版本号
- [ ] `description` 清晰且与实际行为一致
- [ ] `author` 可识别

### 第 2 步：权限范围分析

| 权限 | 风险等级 | 说明 |
|------|----------|------|
| Read | Low | 几乎总是合法的 |
| Write | Medium | 必须说明写入哪些文件 |
| Network | High | 必须说明访问哪些端点 |
| Shell/Bash | Critical | 必须说明执行哪些命令 |

**⚠️ 危险组合：** `network` + `shell` 同时出现 → 可能导致数据泄露

### 第 3 步：内容扫描

**🔴 BLOCK（阻止安装）：**
- 引用 `~/.ssh`、`~/.aws`、`~/.env` 等敏感路径
- 使用 `curl`、`wget`、`nc`、`bash -i` 等命令
- `base64` 混淆内容
- 禁用安全机制
- 未知或可疑 URL

**⚠️ WARNING（需要审查）：**
- `/**/*` 等宽泛通配符
- `sudo` 使用
- 潜在的提示注入

**ℹ️ INFO（信息）：**
- 缺少 description/version/author

### 第 4 步：Typosquat 检测
- 检查单字符交换（如 `skil` vs `skill`）
- 检查同形异义字符（如 `l/1`、`O/0`）
- 检查多余连字符（如 `skill--name`）

**输出格式：**
```
安全审查报告
============
Skill: {name}
安全评级: SAFE / WARNING / DANGER / BLOCK
风险标记: {数量}
建议: install / sandbox first / do not install

详细发现:
- 元数据: ✅ 正常 / ⚠️ 问题描述
- 权限: ✅ 最小权限 / ⚠️ 过度权限
- 内容: ✅ 无风险 / ⚠️ 风险项列表
- Typosquat: ✅ 无 / ⚠️ 可疑命名
```

**参考：** 如果 skill-vetter 已安装，可参考其详细审查协议

---

## 流程四：优化 Skill

**目标：** 改进现有 Skill 的质量，解决具体问题。

**优化触发信号：**
- 用户说"这个 skill 不好用"
- 测评发现的问题需要修复
- 用户想增加新功能或改进行为

**优化流程：**

1. **诊断** — 先用「审查」流程找出问题
2. **制定方案** — 按优先级排列改进项
3. **执行改进**
   - description 不准 → 改触发词，补正例/反例
   - workflow 有漏洞 → 补步骤，加检查点
   - 输出不合格 → 改格式，补样例
   - 误触发 → 收窄触发条件
4. **验证** — 用同一个 case 跑一遍，确认改善
5. **回归** — 用 evals 重跑，确保没引入新问题

---

## 流程五：部署

**目标：** 将本地测试通过的 Skill 安装到全局目录。

**部署流程：**

1. **确认来源** — 工作区中的哪个 skill 目录
2. **检查是否通过测评** — 建议先走「审查」流程
3. **复制到全局**
   ```bash
   # 路径因 Agent 而异：
   #   Claude Code: ~/.claude/skills/
   #   Codex CLI:   ~/.codex/skills/
   #   通用:        ~/.skills/
   cp -r "./{skill名}/" {全局skills目录}/{skill名}/
   ```
4. **验证部署**
   ```bash
   ls {全局skills目录}/{skill名}/
   ```
5. **测试全局可用** — 使用 Agent CLI 测试，如：
   - Claude Code: `claude -p "使用 {name} 完成 XXX"`
   - Codex CLI: `codex "使用 {name} 完成 XXX"`

**⚠️ 永远用 `cp`，不用 `mv`，保留源文件。**

---

## 流程六：管理

**目标：** 管理已安装的 Skill。

### 列出已安装 Skill
```bash
# 路径因 Agent 而异：
#   Claude Code: ~/.claude/skills/
#   Codex CLI:   ~/.codex/skills/
#   通用:        ~/.skills/
ls {全局skills目录}/
ls ./
```

### 更新 Skill

**单个更新：**
1. 查询最新版本（SkillsMP API、CocoLoop API 或 GitHub）
2. 比较本地版本与远程版本
3. 有更新 → 备份旧版 → 下载新版 → 安全审查 → 安装

**批量更新（如 find-skills 已安装）：**
```bash
npx skills update
```
- 检查所有已安装 skills 的更新
- 自动更新到最新版本

**检查更新：**
```bash
npx skills check
```

### 卸载 Skill
1. 确认 skill 存在
2. 询问用户确认
3. 删除 skill 目录
4. 清理相关配置

---

## 参考资料

**本工作台：**
- 需求深挖指南：`${CLAUDE_SKILL_DIR}/references/requirements.md`
- 搜索策略指南：`${CLAUDE_SKILL_DIR}/references/search-strategy.md`
- 方案推送模板：`${CLAUDE_SKILL_DIR}/references/proposal-template.md`
- 优化方法：`${CLAUDE_SKILL_DIR}/references/optimize.md`
- 部署指南：`${CLAUDE_SKILL_DIR}/references/deploy.md`

**子技能：**
- 开发子技能：`${CLAUDE_SKILL_DIR}/subskills/dev/SKILL.md`
- 审查子技能：`${CLAUDE_SKILL_DIR}/subskills/review/SKILL.md`

**全局技能（如已安装）：**
- find-skills — Skills.sh 生态搜索
- skill-creation-guide — Anthropic 官方创建指南（含 init_skill.py、package_skill.py）
- skill-vetter — 安全审查协议
- cocoloop — CocoLoop Skill 管理器

**注意：** 全局技能的安装路径因 Agent 而异：
- Claude Code: `~/.claude/skills/`
- Codex CLI: `~/.codex/skills/`
- 通用: `~/.skills/`

## 注意事项

### 必须遵守
- 深入理解需求，不要只问几个表面问题
- 全网搜索，不要自己造轮子
- 分阶段开发，每个阶段都审查打分
- 循环返工，分数低就返工，直到达标
- 完整交付，生成完整 skill 包
- 下载前必须做安全审查
- 部署前建议做质量测评
- 永远用 cp 不用 mv
- 用 AskUserQuestion 与用户交互

### 禁止行为
- 跳过需求深挖直接开发
- 跳过全网搜索直接开发
- 跳过审查打分直接交付
- 跳过安全审查直接部署
- 未测试就部署到全局
- 覆盖用户未确认的文件
- 静默执行危险操作

## 示例

### 搜索并安装
```
用户: 帮我找个代码格式化的 skill
→ 搜索流程 → 展示结果 → 用户选择 → 安全审查 → 安装
```

### 从零开发（v3.0 流程）
```
用户: 帮我开发一个代码审查 skill
→ 需求深挖 → 全网搜索 → 方案推送 → 分阶段开发 → 完整交付
```

### 审查评分
```
用户: 帮我审查一下这个 skill 的质量
→ 加载 review 子技能 → 安全审查 → 10 维度评分 → 输出报告
```

### 优化现有
```
用户: 这个 skill 触发不太准，帮我优化
→ 诊断（审查）→ 找出问题 → 改 description → 验证
```