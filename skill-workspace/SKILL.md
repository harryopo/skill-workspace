---
name: skill-workspace
description: |
  Skill 全生命周期工作台。当用户提到"skill工作台"、"管理skill"、"搜索skill"、
  "下载skill"、"优化skill"、"部署skill"、"合并skill"时触发。开发和审查功能由子技能处理。
  面向所有 Agent（Claude Code、Codex CLI、ChatGPT 等），一站式完成 Skill 的
  搜索、下载、安全审查、开发生成、优化改进、质量测评、部署上线、合并整合、更新卸载。
  v5.0 新增：Darwin 9 维度评估体系、棘轮优化机制、多评委独立审查、反例黑名单。
argument-hint: "[子命令] [参数]"
context: fork
agent: general-purpose
allowed-tools: Read Write Edit Glob Grep Bash WebFetch
---

# Skill 全生命周期工作台（v5.0）

一站式 Skill 管理平台，覆盖 Skill 从发现到退役的完整生命周期。

## 核心原则

> **先检查环境，深入理解需求，全网搜索最佳方案，分阶段开发，循环审查，完整交付。**

## 子命令路由

| 子命令 | 说明 | 处理方式 |
|--------|------|----------|
| **开发** | 从零创建新 Skill（含需求深挖、全网搜索、方案推送、分阶段开发） | 加载 dev 子技能 |
| **合并** | 将多个相关 Skill 整合为一个更强大的 Skill | 本文件处理（流程七） |
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
3. 用户说"合并skill" / "整合skill" → 在本文件中执行流程七
4. 其他子命令 → 在本文件中处理

## 子技能加载

当路由到子技能时，使用 `Read` 工具加载对应的 SKILL.md：

```
开发 → Read ${SKILL_DIR}/subskills/dev/SKILL.md
审查 → Read ${SKILL_DIR}/subskills/review/SKILL.md
```

子技能是完整的包，有自己的 references 和 evals，可独立使用。

---

## 流程零：环境检查（v4.0 新增）

**目标：** 在任何操作开始之前，确保执行环境正常、依赖完整、网络通畅、权限充足。

**原则：** 先检查，后执行。缺失依赖自动提示并提供安装方案。

**详细指南：** `${SKILL_DIR}/references/environment-check.md`

### 执行流程

```
第一步：检查基础依赖是否安装
  ↓
第二步：检查环境变量是否配置
  ↓
第三步：检查网络是否通畅
  ↓
第四步：检查权限是否充足
  ↓
第五步：检查搜索增强 Skill 是否已安装
  ↓
生成环境检查报告 → 有缺失？→ 询问用户是否自动安装
```

### 第一步：检查基础依赖

```bash
curl --version    # HTTP 客户端
git --version     # 版本控制
python --version  # Python 运行时
node --version    # Node.js 运行时
npm --version     # npm 包管理器
```

### 第二步：检查搜索增强 Skill

| Skill | 用途 | 检查方式 |
|-------|------|----------|
| deep-research-pro | 深度研究（16 搜索引擎） | 检查 SKILL.md 是否存在 |
| multi-search-engine | 多搜索引擎集成（16 引擎） | 检查 SKILL.md 是否存在 |

```bash
ls ~/.skills/deep-research-pro/SKILL.md 2>/dev/null && echo "已安装" || echo "未安装"
ls ~/.skills/multi-search-engine/SKILL.md 2>/dev/null && echo "已安装" || echo "未安装"
```

### 第三步：检查网络连通性

```bash
curl -s -o /dev/null -w "%{http_code}" https://github.com
curl -s -o /dev/null -w "%{http_code}" https://skillsmp.com
curl -s -o /dev/null -w "%{http_code}" https://registry.npmjs.org
```

### 环境检查报告格式

```
========================================
  环境检查报告
========================================

基础依赖:
  ✅ curl    — 已安装 (7.88.1)
  ✅ git     — 已安装 (2.39.2)
  ✅ python  — 已安装 (3.12.0)
  ✅ node    — 已安装 (20.11.0)
  ✅ npm     — 已安装 (10.2.0)

搜索增强 Skill:
  ✅ deep-research-pro   — 已安装
  ❌ multi-search-engine — 未安装

网络连通性:
  ✅ github.com    — HTTP 200
  ✅ skillsmp.com  — HTTP 200
  ❅ registry.npmjs.org — 超时

========================================
  发现 1 个缺失依赖，1 个网络问题
  是否自动安装缺失依赖？(y/n)
========================================
```

### 自动安装机制

- 用户确认 → 根据操作系统选择对应安装命令（详见 `environment-check.md`）
- 用户拒绝 → 输出手动安装指引，继续执行（跳过依赖功能）

**跨平台安装命令：**
- Windows: `winget install` 或 `choco install`
- macOS: `brew install`
- Linux: `apt install` / `yum install` / `dnf install`

**镜像源加速：**
```bash
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
npm config set registry https://registry.npmmirror.com
```

---

## 流程一：搜索 Skill

**目标：** 在线查找用户需要的 Skill（面向所有 Agent，不限于特定 Agent），避免重复造轮子。

**详细策略：** `${SKILL_DIR}/references/search-strategy.md`

### 搜索源（按优先级）

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

**Tier 4：GitHub 搜索（含国内访问方案）**
```bash
curl -s "https://api.github.com/search/repositories?q={关键词}+filename:SKILL.md&sort=stars&per_page=5"
```

**GitHub 国内访问降级策略：**
```
GitHub 直连 → 超时？
  ├── 尝试镜像站（github.moeyy.xyz / hub.fastgit.xyz / ghproxy.com）
  │   ├── 成功 → 使用镜像站
  │   └── 失败 ↓
  ├── 尝试 GitHub API（api.github.com，比网页更稳定）
  │   ├── 成功 → 使用 API
  │   └── 失败 ↓
  ├── 尝试代理（检测 $http_proxy / $https_proxy）
  │   ├── 成功 → 使用代理
  │   └── 失败 ↓
  └── 降级到 Tier 1-3 搜索源
```

**Tier 5：clawhub CLI（兜底）**
```bash
npx clawhub@latest search {关键词}
```

**Tier 6：深度搜索引擎（v4.0 新增）**

集成 deep-research-pro 和 multi-search-engine 两个增强搜索 Skill：

| Skill | 引擎数 | 适用场景 |
|-------|--------|----------|
| deep-research-pro | 16 个（含 7 个中文） | 深度调研、技术方案、最佳实践、学术研究 |
| multi-search-engine | 16 个（含抖音/B站等） | 通用搜索、中文内容、视频教程、行业分析 |

**调用方式：**

```
Skill: deep-research-pro
任务：调研 {关键词} 的最佳实践和现有方案

Skill: multi-search-engine
关键词：{关键词}
引擎：全部
```

**multi-search-engine 支持的引擎：**

| 类型 | 引擎 | 特点 |
|------|------|------|
| 国内 | 抖音、B站、知乎、掘金、CSDN、微信公众号、小红书 | 中文内容、视频教程 |
| 国际 | Google、GitHub、Stack Overflow、Reddit、Dev.to 等 | 技术文档、社区讨论 |

### 搜索策略

```
第一步：环境检查（流程零）
  ↓ 环境正常
第二步：SkillsMP API 搜索（覆盖面最广）
  ↓ 无结果或超时
第三步：npx skills find（Skills.sh 生态）
  ↓ 无结果或超时
第四步：CocoLoop API 搜索
  ↓ 无结果或超时
第五步：GitHub API 搜索（国内走降级策略）
  ↓ 无结果或超时
第六步：clawhub CLI 搜索
  ↓ 无结果
第七步：调用 deep-research-pro 深度搜索
  ↓ 无结果
第八步：调用 multi-search-engine 多引擎搜索
  ↓ 无结果
第九步：提示用户手动搜索或走「开发」流程
```

**输出格式：**
```
📋 搜索结果（来源：SkillsMP）:
  1. skill-name (⭐ 15.5k stars)
     📝 描述文本
     🔗 GitHub: https://github.com/xxx/xxx
  2. ...

🔍 深度搜索结果（来源：deep-research-pro）:
  1. 方案名称
     📝 描述文本
     📊 推荐度：⭐⭐⭐⭐⭐
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

1. **环境检查**（v4.0 新增）
   - 调用流程零检查基础依赖
   - 确保 curl/git 可用

2. **获取 Skill 内容**
   - URL → curl 下载
   - 名称 → 按搜索流程找到后下载
   - GitHub → git clone 或 curl 下载 SKILL.md
   - 国内环境 → 按 GitHub 降级策略获取

3. **安全审查**（强制）
   - 调用「安全审查」流程
   - 评级 ≥ B → 继续
   - 评级 ≤ C → 询问用户是否继续

4. **安装到工作区**
   ```bash
   # 安装到当前工作区（开发模式）
   cp -r {skill目录}/ "./{skill名}/"

   # 或安装到全局（使用模式）
   # 通用路径：~/.skills/{skill名}/
   cp -r {skill目录}/ ~/.skills/{skill名}/
   ```

5. **确认安装结果**

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

**v5.0 新增：棘轮优化机制**

优化循环采用棘轮机制，确保质量只升不降：
- 每轮优化后必须通过量化评估（评分 ≥ 上一轮）
- 若本轮评分 < 上一轮，立即自动回滚
- 连续 2 轮 Δ < 2 分时自动停止优化
- 优化后文件大小 ≤ 原始大小 × 1.5

**详细指南：** `${SKILL_DIR}/references/ratchet-mechanism.md`

**v5.0 新增：多评委独立评估**

采用多评委独立评估机制，避免 LLM 自评偏差：
- 每轮启动至少 2 个独立评委 agent
- 评委不复用，下一轮重新 spawn
- 至少 2 个评委共识才有效
- 子 agent 不可用时自动切换到干跑模式

**详细指南：** `${SKILL_DIR}/references/multi-judge.md`

---

## 流程五：部署

**目标：** 将本地测试通过的 Skill 安装到全局目录。

**部署流程：**

1. **环境检查**（v4.0 新增） — 调用流程零，确保目标目录可写
2. **确认来源** — 工作区中的哪个 skill 目录
3. **检查是否通过测评** — 建议先走「审查」流程
4. **复制到全局**
   ```bash
   # 通用路径：~/.skills/
   cp -r "./{skill名}/" ~/.skills/{skill名}/
   ```
5. **验证部署**
   ```bash
   ls ~/.skills/{skill名}/
   ```
6. **测试全局可用** — 使用 Agent CLI 验证，如：
   - `claude -p "使用 {name} 完成 XXX"`
   - `codex "使用 {name} 完成 XXX"`

**⚠️ 永远用 `cp`，不用 `mv`，保留源文件。**

---

## 流程六：管理

**目标：** 管理已安装的 Skill。

### 列出已安装 Skill
```bash
# 通用路径：~/.skills/
ls ~/.skills/
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

## 流程七：合并 Skill（v4.0 新增）

**目标：** 将多个相关 Skill 整合为一个更强大、更完整的 Skill。

**详细指南：** `${SKILL_DIR}/references/merge-workflow.md`

### 合并目标

1. **消除冗余** — 减少功能重叠，降低维护成本
2. **增强能力** — 集成多个 Skill 的优点，形成更强功能
3. **优化体验** — 统一触发条件和输出规范，减少用户困惑
4. **解决问题** — 修复原 Skill 的痛点和缺陷

### 合并流程

```
第一步：审查原 Skill
  ↓ 调用 review 子技能，对每个原 Skill 进行 10 维度打分
第二步：理解原 Skill
  ↓ 阅读每个 SKILL.md，理解触发条件、工作流程、输出规范
第三步：询问用户
  ↓ 明确合并方向：保留优点、解决痛点、新增需求、优先级
第四步：网上调研
  ↓ 使用 multi-search-engine 搜索最佳实践和解决方案
第五步：综合优化
  ↓ 设计合并方案：保留优点、解决痛点、集成新功能、简化结构
第六步：正常工作流
  ↓ 按标准流程分阶段开发合并后的 Skill
第七步：用户确认
  → 每个阶段完成后都需要用户确认
```

### 合并确认清单

每个阶段完成后都需用户确认：
- [ ] 审查结果准确
- [ ] 理解正确
- [ ] 需求完整
- [ ] 调研结果可用
- [ ] 合并方案满意
- [ ] 最终成果符合预期

### 合并方案文档模板

```markdown
# Skill 合并方案

## 背景
- 合并原因：[说明]
- 原 Skill 列表：[skill A]、[skill B]、[skill C]

## 审查结果
[各 Skill 10 维度评分对比表]

## 用户需求
- 必须保留：[功能列表]
- 必须解决：[问题列表]
- 新增需求：[需求列表]

## 调研结果
[最佳实践 + 踩坑经验]

## 合并方案
- 触发条件：[统一后的触发条件]
- 工作流程：[合并后的工作流程]
- 输出规范：[统一后的输出规范]
```

---

## 上下文管理（v4.0 新增）

**详细指南：** `${SKILL_DIR}/references/context-management.md`

### 核心原则

- **主动规划优先** — 任何任务开始前，先生成明确的执行计划
- **阶段性记忆** — 将长任务分解为阶段，每个阶段独立保存关键信息
- **智能压缩** — 当上下文过长时，自动压缩而非丢失信息
- **可追溯性** — 任何决策和完成状态都应有据可查

### Plan 机制

**开始前：** 生成结构化任务计划（目标、拆解、依赖、风险、成功标准）

**执行中：** 每个阶段完成时保存阶段记忆（完成状态、关键输出、关键决策、下阶段输入）

**压缩触发条件：**
- 对话轮次 > 20 轮
- 估算 token 数 > 80% 上下文窗口
- 用户明确要求"继续"或"下一步"时检查

**压缩公式：**
```
压缩后上下文 = 
  最近 5 轮完整对话 +
  所有阶段记忆摘要（每个阶段 ≤ 200 字）+
  当前任务计划 +
  待完成任务列表 +
  关键决策索引
```

---

## 参考资料

**本工作台：**
- 需求深挖指南：`${SKILL_DIR}/references/requirements.md`
- 搜索策略指南：`${SKILL_DIR}/references/search-strategy.md`
- 方案推送模板：`${SKILL_DIR}/references/proposal-template.md`
- 优化方法：`${SKILL_DIR}/references/optimize.md`
- 部署指南：`${SKILL_DIR}/references/deploy.md`
- 合并流程（v4.0 新增）：`${SKILL_DIR}/references/merge-workflow.md`
- 上下文管理（v4.0 新增）：`${SKILL_DIR}/references/context-management.md`
- 环境检查（v4.0 新增）：`${SKILL_DIR}/references/environment-check.md`
- Darwin 评估体系（v5.0 新增）：`${SKILL_DIR}/references/darwin-rubric.md`
- 棘轮机制（v5.0 新增）：`${SKILL_DIR}/references/ratchet-mechanism.md`
- 多评委评估（v5.0 新增）：`${SKILL_DIR}/references/multi-judge.md`
- 反例黑名单（v5.0 新增）：`${SKILL_DIR}/references/anti-patterns.md`
- Agent 中立指南（v5.0 新增）：`${SKILL_DIR}/references/agent-neutral-guide.md`

**子技能：**
- 开发子技能：`${SKILL_DIR}/subskills/dev/SKILL.md`
- 审查子技能：`${SKILL_DIR}/subskills/review/SKILL.md`

**全局技能（如已安装）：**
- find-skills — Skills.sh 生态搜索
- skill-creation-guide — Anthropic 官方创建指南（含 init_skill.py、package_skill.py）
- skill-vetter — 安全审查协议
- cocoloop — CocoLoop Skill 管理器
- deep-research-pro — 深度研究（16 搜索引擎，v4.0 集成）
- multi-search-engine — 多搜索引擎集成（16 引擎，v4.0 集成）

**注意：** 全局技能的安装路径统一为 `~/.skills/`。

## 注意事项

### 必须遵守
- 执行前先做环境检查（流程零）
- 深入理解需求，不要只问几个表面问题
- 全网搜索（含 deep-research-pro 和 multi-search-engine），不要自己造轮子
- 分阶段开发，每个阶段都审查打分
- 循环返工，分数低就返工，直到达标
- 完整交付，生成完整 skill 包
- 下载前必须做安全审查
- 部署前建议做质量测评
- 永远用 cp 不用 mv
- 用 AskUserQuestion 与用户交互
- 长对话时主动触发上下文压缩，降低幻觉
- 合并 Skill 时每个阶段都要用户确认
- 优化时采用棘轮机制，确保质量只升不降
- 评估时采用多评委独立审查，避免自评偏差
- 对照反例黑名单检查，避免常见反模式

### 禁止行为
- 跳过环境检查直接执行
- 跳过需求深挖直接开发
- 跳过全网搜索直接开发
- 跳过审查打分直接交付
- 跳过安全审查直接部署
- 未测试就部署到全局
- 覆盖用户未确认的文件
- 静默执行危险操作
- 合并时假设用户意图而不确认
- 同 context 自评自改（AP-01）
- git reset --hard 当回滚（AP-02）
- 为凑分增冗余（AP-03）
- 跳过 test-prompts 直接评分（AP-04）
- 轮内改多个维度（AP-05）
- dry_run 比例 > 30%（AP-06）
- 静默跳过异常（AP-07）
- 忽视维度相关性单独优化（AP-08）

## 示例

### 搜索并安装
```
用户: 帮我找个代码格式化的 skill
→ 环境检查 → 搜索流程 → 展示结果 → 用户选择 → 安全审查 → 安装
```

### 从零开发
```
用户: 帮我开发一个代码审查 skill
→ 环境检查 → 需求深挖 → 全网搜索（含深度搜索）→ 方案推送 → 分阶段开发 → 完整交付
```

### 审查评分
```
用户: 帮我审查一下这个 skill 的质量
→ 加载 review 子技能 → 安全审查 → Darwin 9 维度评分 → 输出报告
```

### 优化现有
```
用户: 这个 skill 触发不太准，帮我优化
→ 诊断（审查）→ 找出问题 → 改 description → 棘轮验证 → 多评委评估
```

### 合并 Skill（v4.0 新增）
```
用户: 把这几个 skill 合并成一个
→ 环境检查 → 审查原 Skill → 理解原 Skill → 询问用户 → 网上调研 → 综合优化 → 分阶段开发 → 用户确认
```

## 版本历史

| 版本 | 日期 | 更新内容 |
|------|------|----------|
| v5.0.0 | 2026-06-13 | 集成 Darwin 评估体系、棘轮机制、多评委评估、反例黑名单；Agent 中立化；统一部署路径 |
| v4.0.0 | 2026-06-13 | 新增合并流程、环境检查、深度搜索集成、上下文管理 |
| v3.0 | — | 初始版本，基础生命周期管理 |