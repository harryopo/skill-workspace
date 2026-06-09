# 搜索策略指南

## 概述

全网搜索是 Skill 开发的第二步，目标是全方位搜索，不遗漏任何有价值的信息源。通过 5 层搜索源并行搜索、智能排序和深度分析，找到最优质的参考实现和技术方案。

## 5 层搜索源

### Tier 1: SkillsMP (skillsmp.com)

**说明：** 1.5M+ skills，兼容所有 Agent（Claude Code、Codex CLI、ChatGPT 等）

**搜索命令：**

```bash
curl -s "https://skillsmp.com/api/v1/skills/search?q={关键词}&limit=10&sortBy=stars"
```

**参数：**
- `q`: 搜索关键词
- `limit`: 返回数量（默认 10）
- `sortBy`: 排序方式（stars/downloads/updated）
- `category`: 分类过滤
- `occupation`: 职业过滤
- `page`: 分页

**返回字段：**
- `name`: skill 名称
- `description`: 描述
- `stars`: 星数
- `downloads`: 下载量
- `github_url`: GitHub 链接
- `author`: 作者
- `version`: 版本

### Tier 2: Skills.sh 生态

**说明：** Agent Skills 的包管理器

**搜索命令：**

```bash
npx skills find {关键词}
```

**其他命令：**

```bash
# 安装 skill
bash /path/to/skill/scripts/install-skill.sh <owner/repo@skill-name>

# 检查更新
npx skills check

# 批量更新
npx skills update
```

### Tier 3: CocoLoop API

**说明：** Skill 聚合市场

**搜索命令：**

```bash
curl -s "https://api.cocoloop.com/api/v1/store/skills?page=1&page_size=10&keyword={关键词}&sort=downloads"
```

**参数：**
- `keyword`: 搜索关键词
- `page`: 页码
- `page_size`: 每页数量
- `sort`: 排序方式（downloads/stars/updated）

### Tier 4: GitHub 搜索

**说明：** 搜索包含 SKILL.md 的仓库

**搜索命令：**

```bash
curl -s "https://api.github.com/search/repositories?q={关键词}+filename:SKILL.md&sort=stars&per_page=5"
```

**参数：**
- `q`: 搜索关键词 + filename:SKILL.md
- `sort`: 排序方式（stars/forks/updated）
- `per_page`: 每页数量

### Tier 5: 技术博客/社区

**说明：** 搜索最佳实践、踩坑经验

**搜索关键词组合：**
- "{关键词} skill 最佳实践"
- "{关键词} agent skill 开发"
- "{关键词} claude code skill"
- "{关键词} SKILL.md 示例"

**搜索来源：**
- GitHub Issues/Discussions
- Stack Overflow
- 知乎
- 掘金
- CSDN

## 搜索策略

### 并行搜索

同时搜索 5 个源，提高效率：

```
┌─────────────────────────────────────────────────────────────┐
│                    并行搜索流程                              │
├─────────────────────────────────────────────────────────────┤
│  1. 启动 5 个搜索任务                                        │
│     ├── Task 1: SkillsMP 搜索                                │
│     ├── Task 2: Skills.sh 搜索                               │
│     ├── Task 3: CocoLoop 搜索                                │
│     ├── Task 4: GitHub 搜索                                  │
│     └── Task 5: 技术博客/社区搜索                            │
│                                                             │
│  2. 等待所有任务完成                                         │
│                                                             │
│  3. 合并结果                                                 │
│     ├── 去重                                                 │
│     ├── 排序                                                 │
│     └── 分析                                                 │
└─────────────────────────────────────────────────────────────┘
```

### 智能排序

按相关度、下载量、评分综合排序：

**排序算法：**

```
score = (relevance * 0.4) + (downloads * 0.3) + (stars * 0.3)
```

- `relevance`: 相关度（0-100）
- `downloads`: 下载量（归一化到 0-100）
- `stars`: 星数（归一化到 0-100）

### 深度分析

分析每个结果的优缺点：

**分析维度：**
- 功能完整度
- 技术栈
- 代码质量
- 社区活跃度
- 文档质量

## 结果对比分析

### 对比维度

1. **功能完整度**
   - 实现了哪些功能
   - 缺少哪些功能
   - 功能是否符合需求

2. **技术栈**
   - 使用了哪些技术
   - 技术是否先进
   - 技术是否稳定

3. **代码质量**
   - 代码是否规范
   - 是否有测试
   - 是否有文档

4. **社区活跃度**
   - 最近更新时间
   - Issue 数量
   - PR 数量

5. **文档质量**
   - 是否有 README
   - 是否有示例
   - 是否有 API 文档

### 输出格式

```markdown
📋 搜索结果汇总（来源：5 个源）

## 搜索统计

- SkillsMP: [数量] 个结果
- Skills.sh: [数量] 个结果
- CocoLoop: [数量] 个结果
- GitHub: [数量] 个结果
- 技术博客: [数量] 个结果

## 结果列表

### 1. skill-name (⭐ 15.5k stars, 📥 50k+ downloads)

**来源：** SkillsMP
**描述：** [描述文本]
**GitHub：** https://github.com/xxx/xxx
**作者：** [作者]
**版本：** [版本]
**最近更新：** [日期]

**优点：**
- [优点 1]
- [优点 2]
- [优点 3]

**缺点：**
- [缺点 1]
- [缺点 2]
- [缺点 3]

**推荐度：** ⭐⭐⭐⭐⭐

### 2. ...

## 技术方案对比

| 方案 | 技术栈 | 功能完整度 | 代码质量 | 社区活跃度 | 推荐度 |
|------|--------|------------|----------|------------|--------|
| 方案 1 | [技术栈] | [评分] | [评分] | [评分] | ⭐⭐⭐⭐⭐ |
| 方案 2 | [技术栈] | [评分] | [评分] | [评分] | ⭐⭐⭐⭐ |
| 方案 3 | [技术栈] | [评分] | [评分] | [评分] | ⭐⭐⭐ |
```

## 下载参考实现

### 下载策略

1. **用户选择要下载的参考实现**
   - 展示搜索结果列表
   - 让用户选择要下载的实现

2. **下载到本地工作区**

   ```bash
   # 克隆仓库
   git clone https://github.com/xxx/xxx.git

   # 或下载 SKILL.md
   curl -o SKILL.md https://raw.githubusercontent.com/xxx/xxx/main/SKILL.md
   ```

3. **安全审查**
   - 调用安全审查流程（4 步协议）
   - 评级 ≥ B → 继续
   - 评级 ≤ C → 询问用户是否继续

## 安全审查

### 审查协议（4 步）

#### 第 1 步：元数据检查

- [ ] name 匹配预期名称（无拼写欺骗）
- [ ] description 清晰且与实际行为一致
- [ ] version 遵循 semver（可选但建议）

#### 第 2 步：权限范围分析

| 权限 | 风险 | 说明 |
|------|------|------|
| Read | 🟢 Low | 几乎总是合理 |
| Write | 🟡 Medium | 必须说明写入哪些文件 |
| Bash | 🔴 Critical | 必须说明执行哪些命令 |
| network | 🔴 Critical | 必须说明访问哪些端点 |

**危险组合：** `network` + `shell` 同时出现 → 数据泄露风险，必须 BLOCK。

#### 第 3 步：内容安全扫描

🔴 **Critical（直接 BLOCK）：**
- 引用 `~/.ssh`、`~/.aws`、`~/.env` 等敏感路径
- 使用 `curl`、`wget`、`nc`、`bash -i` 等网络/反弹命令
- `base64` 混淆内容
- 禁用安全机制的指令
- 未知或可疑 URL

🟡 **Warning（需要人工审查）：**
- `/**/*` 等宽泛通配符
- `sudo` 使用
- 潜在的提示注入

ℹ️ **Info（建议改进）：**
- 缺少 description/version

#### 第 4 步：Typosquat 检测

检查以下情况：
- 单字符替换（如 `skil1-review`）
- 同形字符（l/1, O/0, a/а）
- 多余连字符（`skill--review`）
- 与已安装 skill 重名或近似

### 审查输出格式

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

**评级规则：**
- SAFE：无风险标记
- WARNING：仅 ℹ️ 级别
- DANGER：有 🟡 级别
- BLOCK：有 🔴 级别（停止，不进入质量评分）
