# Skill 全生命周期工作台 v5.0

> 一站式完成 Skill 的 **搜索 → 下载 → 安全审查 → 开发 → 优化 → 测评 → 部署 → 管理**

***

## 📋 项目简介

**Skill 全生命周期工作台** 是一个覆盖 Skill 从发现到退役的完整管理平台。它将 Skill 的开发、审查、部署等分散功能整合为统一入口，通过子技能架构实现模块化管理。

### 核心价值

| 痛点 | 解决方案 |
|------|----------|
| 不知道网上有没有现成的 Skill | 6 层搜索源覆盖：SkillsMP → Skills.sh → CocoLoop → GitHub → clawhub → 深度搜索（16 引擎） |
| 写完 Skill 不知道好不好 | Darwin 9 维度评估体系（100 分制），多评委独立交叉验证 |
| 下载的 Skill 是否安全 | 4 步安全审查协议：元数据→权限→内容→Typosquat |
| 优化容易退步 | 棘轮机制确保质量只升不降，退步自动回滚 |
| 部署容易出错 | 标准化部署流程，Agent 中立化，支持多 Agent 环境 |
| 多个 Skill 触发冲突 | 统一入口 + 子技能路由，互不干扰 |

***

## 🏗️ 架构设计

### 设计理念

采用 **"主入口 + 独立子技能包"** 架构：

```
┌─────────────────────────────────────────────────────────────┐
│                      skill-workspace                        │
│                     （主入口 + 路由）                         │
├─────────────────────────────────────────────────────────────┤
│  搜索 │ 下载 │ 安全审查 │ 优化 │ 部署 │ 管理 │ 合并        │
└─────────────────────────────────────────────────────────────┘
                    │              │
          ┌─────────┴──┐    ┌─────┴────────┐
          │ skill-dev  │    │ skill-review │
          │ (子技能)   │    │ (子技能)     │
          ├────────────┤    ├──────────────┤
          │ references/│    │ references/  │
          │ evals/     │    │ evals/       │
          └────────────┘    └──────────────┘
```

**为什么不用三个独立 Skill？**

| 方案 | 优点 | 缺点 |
|------|------|------|
| **一个大 Skill** | 简单 | SKILL.md 500+ 行，上下文浪费，触发冲突 |
| **三个独立 Skill** | 模块化 | 用户要装 3 个，触发词容易冲突 |
| **主入口 + 子技能** ✅ | 统一入口，模块化 | 一次安装，按需加载 |

***

## 📁 目录结构

```
skill-workspace/
├── SKILL.md                              # 主入口（700+ 行）
├── README.md                             # 本文件
├── references/
│   ├── optimize.md                       # 优化方法论（棘轮机制）
│   ├── deploy.md                         # 部署指南（Agent 中立）
│   ├── search-strategy.md                # 搜索策略
│   ├── requirements.md                   # 需求深挖指南
│   ├── proposal-template.md              # 方案推送模板
│   ├── merge-workflow.md                 # 合并流程（v4.0）
│   ├── context-management.md             # 上下文管理（v4.0）
│   ├── environment-check.md              # 环境检查（v4.0）
│   ├── darwin-rubric.md                  # Darwin 9 维度评估体系（v5.0）
│   ├── ratchet-mechanism.md              # 棘轮机制（v5.0）
│   ├── multi-judge.md                    # 多评委独立评估（v5.0）
│   ├── anti-patterns.md                  # 反例黑名单（v5.0）
│   └── agent-neutral-guide.md            # Agent 中立指南（v5.0）
├── evals/
│   └── evals.json                        # 主技能评测集（18 条）
│
└── subskills/
    ├── dev/                              # 开发子技能（完整包）
    │   ├── SKILL.md                      # 开发流程（500 行）
    │   ├── references/
    │   │   ├── official-spec.md          # 官方规范
    │   │   ├── template.md              # 模板文件
    │   │   ├── example.md               # 示例文件
    │   │   └── methodology.md           # 实战方法论
    │   └── evals/
    │       └── evals.json               # 开发评测集（6 条）
    │
    └── review/                           # 审查子技能（完整包）
        ├── SKILL.md                      # 审查流程（Darwin 9 维度）
        ├── references/
        │   ├── official-spec.md          # 官方规范
        │   ├── scoring-criteria.md      # 10 维度评分标准
        │   └── security.md              # 安全审查规则
        └── evals/
            ├── evals.json               # 审查评测集（5 条）
            └── test-cases.json          # 测试用例
```

***

## 🚀 快速开始

### 安装

```bash
# 从工作区复制到全局目录
# 路径因 Agent 而异：
#   Claude Code: ~/.claude/skills/
#   Codex CLI:   ~/.codex/skills/
#   Cursor:      ~/.cursor/skills/
#   通用:        ~/.skills/
cp -r skill-workspace/ {全局skills目录}/skill-workspace/
```

### 基本使用

```bash
# 搜索 Skill
/skill-workspace 搜索 代码格式化

# 开发 Skill
/skill-workspace 开发 代码审查 skill

# 审查 Skill（Darwin 9 维度）
/skill-workspace 审查 skill-dev

# 优化 Skill（棘轮机制）
/skill-workspace 优化 my-skill

# 部署 Skill（Agent 中立）
/skill-workspace 部署 my-skill

# 合并 Skill
/skill-workspace 合并 skill-a skill-b
```

***

## 🔧 功能详解

### 1. 搜索 Skill

**目标：** 在线查找已有 Skill（面向所有 Agent，不限于 Claude），避免重复造轮子。

**6 层搜索源覆盖：**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| Tier 1 | **SkillsMP** (skillsmp.com) | 1.5M+ skills，兼容所有 Agent，有 REST API |
| Tier 2 | **npx skills find** | Skills.sh 生态，`npx skills find {关键词}` |
| Tier 3 | CocoLoop API | Skill 聚合市场 |
| Tier 4 | GitHub 搜索 | 搜索包含 SKILL.md 的仓库 |
| Tier 5 | clawhub CLI | 命令行搜索工具（兜底） |
| Tier 6 | **深度搜索增强** | deep-research-pro / multi-search-engine（16 引擎） |

**搜索降级策略：**
```
SkillsMP API → npx skills find → CocoLoop → GitHub → clawhub → deep-research-pro → multi-search-engine → 提示用户
```

**使用示例：**
```
用户: 帮我找个代码格式化的 skill

📋 搜索结果（来源：SkillsMP）:
  1. code-formatter (⭐ 15.5k stars)
     📝 支持多语言的代码格式化工具
     🔗 https://github.com/xxx/xxx
  2. prettier-skill (⭐ 8.2k stars)
     📝 基于 Prettier 的格式化 Skill
     🔗 https://github.com/xxx/xxx

是否安装？[1/2/取消]
```

***

### 2. 开发 Skill

**目标：** 引导式创建符合官方规范的 Skill。

**前置判断：**

- 这个任务重复出现过几次？— 至少 3 次才值得
- Agent 每次是否稳定犯同一类错？— 不稳定犯错说明边界不清

**三条路径：**

| 路径 | 适用场景 | 第一步 |
|------|----------|--------|
| A. 找现成的 | 常见场景 | 去 SkillsMP/CocoLoop/GitHub 搜索 |
| B. 改造现成的 | 方向接近但不完全匹配 | 保留核心结构，改 7 个点 |
| C. 从零写 | 没有合适 Skill | 走完整流程 |

**开发流程：**

1. **环境检查**（v4.0） — 检查基础依赖、网络、权限
2. **需求深挖** — 5-8 个结构化问题，理解真实痛点
3. **全网搜索** — 6 层搜索源，含深度搜索（16 引擎）
4. **方案推送** — A/B/C 三方案对比，带评分表
5. **分阶段开发** — P0（核心）→ P1（基础）→ P2（高级）
6. **阶段审查** — 每阶段 Darwin 9 维度评分
7. **循环返工** — 评分 < 80% 返工，直到达标
8. **本地交付** — 先生成在工作区，用户测试后才部署
9. **打包部署** — cp 到全局目录

**整合 Anthropic 官方工具：**
```bash
# 初始化 skill（如 skill-creation-guide 已安装）
python {skill-creation-guide路径}/scripts/init_skill.py <skill-name> --path .

# 打包 skill
python {skill-creation-guide路径}/scripts/package_skill.py <skill-folder>/
```

***

### 3. 审查 Skill（Darwin 9 维度）

**目标：** 基于 Darwin 评估体系进行质量评分，输出审查报告。

**理论基础：** 微软 SkillLens 论文 (arXiv:2605.23899) 实证验证

**Darwin 9 维度评估体系（100 分制）：**

| 维度 | 说明 | 满分 |
|------|------|------|
| D1 结构完整性 | frontmatter、目录结构、references | 10 |
| D2 指令具体性 | 触发条件、工作流程、输出格式 | 10 |
| D3 失败模式编码 | 错误处理、边界条件、降级策略 | 10 |
| D4 可执行具体性 | 命令可执行、路径可访问、依赖可安装 | 10 |
| D5 边界条件 | 输入验证、异常处理、超时控制 | 10 |
| D6 用户确认点 | 关键决策点、用户交互、确认机制 | 10 |
| D7 高风险行动黑名单 | 危险操作拦截、安全审查、权限控制 | 10 |
| D8 实测表现 | evals 评测集、实测数据、效果指标 | 20 |
| D9 反例黑名单检查 | 反例完整性、误用防护、常见问题 | 10 |

**综合评级：**

| 总分范围 | 评级 | 综合评价 |
|----------|------|----------|
| 80-100 | A | 强烈推荐 |
| 60-79 | B | 值得一试 |
| 40-59 | C | 仍需改进 |
| <40 | D | 建议重写 |

**多评委独立评估（v5.0）：**

- 每轮启动至少 2 个独立评委 agent
- 评委 A 侧重 D1-D4（结构质量与指令具体性）
- 评委 B 侧重 D5-D9（边界条件与实测效果）
- 分差 ≤1 取平均，=2 取中位数，>2 启动仲裁评委
- 避免 LLM 自评偏差（LLM 自评准确率仅 46.4%）

**安全审查（4 步协议）：**

| 步骤 | 检查内容 |
|------|----------|
| 第 1 步 | 元数据检查（name、version、description、author） |
| 第 2 步 | 权限范围分析（Read/Write/Network/Shell） |
| 第 3 步 | 内容扫描（敏感路径、危险命令、base64 混淆） |
| 第 4 步 | Typosquat 检测（单字符交换、同形异义字符） |

***

### 4. 优化 Skill（棘轮机制）

**目标：** 改进现有 Skill 的质量，确保质量只升不降。

**优化触发信号：**
- 用户说"这个 skill 不好用"
- 测评发现的问题需要修复
- 用户想增加新功能或改进行为

**棘轮优化循环（v5.0）：**

```
┌───────────┐    ┌───────────┐    ┌───────────┐
│ Phase 0   │───→│ Phase 1   │───→│ Phase 2   │
│ 诊断评估   │    │ 方案制定   │    │ 执行改进   │
└───────────┘    └───────────┘    └───────────┘
      ↑                                  │
      │                                  ↓
┌───────────┐                   ┌───────────┐
│ Phase 3   │←──────────────────│ 多评委审查  │
│ 决策与记录 │                   └───────────┘
└───────────┘
      │
      ↓
[通过] → 进入下一轮  |  [退步] → 回滚到上一版本
[触顶] → 停止迭代   |  [违规] → 反例黑名单拦截
```

**棘轮机制规则：**

- 每轮优化后必须通过量化评估（评分 ≥ 上一轮）
- 若本轮评分 < 上一轮，立即自动回滚
- 连续 2 轮 Δ < 2 分时自动触顶停止
- 优化后文件大小 ≤ 原始大小 × 1.5
- 保留可追溯链：每轮记录变更前快照、诊断结果、改进方案、变更后快照、评估分数和通过/回滚决策

**反例黑名单（v5.0）：**

- AP-01: 同 context 自评自改
- AP-02: git reset --hard 当回滚
- AP-03: 为凑分增冗余
- AP-04: 跳过 test-prompts 直接评分
- AP-05: 轮内改多个维度
- AP-06: dry_run 比例 > 30%
- AP-07: 静默跳过异常
- AP-08: 忽视维度相关性单独优化

***

### 5. 安全审查

**目标：** 对 Skill 进行安全扫描，识别风险。

**快速检查清单：**

1. **元数据检查**
   - [ ] name 无拼写欺骗
   - [ ] description 与实际行为一致

2. **权限分析**
   - [ ] Read/Write/Bash/network 权限是否合理
   - [ ] 有无危险组合（network + shell）

3. **内容扫描**
   - 🔴 BLOCK：引用敏感路径、使用 curl/wget/nc、base64 混淆
   - 🟡 WARNING：宽泛通配符、sudo 使用
   - ℹ️ INFO：缺少 description/version

**输出：**
```
安全评级：SAFE / WARNING / DANGER / BLOCK
风险标记：{数量}
建议：install / sandbox first / do not install
```

***

### 6. 部署 Skill（Agent 中立）

**目标：** 将本地测试通过的 Skill 安装到全局目录，支持多 Agent 环境。

**Agent 中立部署路径：**

| Agent | 全局路径 |
|-------|----------|
| Claude Code | `~/.claude/skills/` |
| Codex CLI | `~/.codex/skills/` |
| Cursor | `~/.cursor/skills/` |
| 通用 | `~/.skills/` |

**部署流程：**

1. **环境检查**（v4.0） — 检查目标目录可写
2. **确认来源** — 工作区中的哪个 skill 目录
3. **检查是否通过测评** — 建议先走「审查」流程
4. **复制到全局**
   ```bash
   # 路径因 Agent 而异
   cp -r "./{skill名}/" {全局skills目录}/{skill名}/
   ```
5. **验证部署**
   ```bash
   ls {全局skills目录}/{skill名}/
   ```
6. **测试全局可用** — 使用 Agent CLI 测试
   - Claude Code: `claude -p "使用 {name} 完成 XXX"`
   - Codex CLI: `codex "使用 {name} 完成 XXX"`
   - Cursor: 在 Cursor 中调用

**⚠️ 永远用 `cp`，不用 `mv`，保留源文件。**

***

### 7. 管理 Skill

**目标：** 管理已安装的 Skill。

**列出已安装 Skill：**
```bash
# 路径因 Agent 而异
ls {全局skills目录}/
ls ./
```

**更新 Skill：**

```bash
# 单个更新
# 1. 查询最新版本（SkillsMP API、CocoLoop API 或 GitHub）
# 2. 比较本地版本与远程版本
# 3. 有更新 → 备份旧版 → 下载新版 → 安全审查 → 安装

# 批量更新（如 find-skills 已安装）
npx skills update

# 检查更新
npx skills check
```

**卸载 Skill：**
1. 确认 skill 存在
2. 询问用户确认
3. 删除 skill 目录
4. 清理相关配置

***

### 8. 合并 Skill（v4.0）

**目标：** 将多个相关 Skill 整合为一个更强大、更完整的 Skill。

**合并流程：**

```
第一步：审查原 Skill
  ↓ 调用 review 子技能，对每个原 Skill 进行 Darwin 9 维度打分
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

***

## 📊 完整工作流

```
需求 → 环境检查 → 搜索 → 找到？ → 安全审查 → 下载 → 优化（棘轮） → 测评（Darwin） → 部署（Agent 中立）
         ↓ 没找到
       开发（分阶段） → 测评 → 部署
```

***

## 🎯 使用场景

### 场景 1：开发新 Skill

```
用户: 帮我开发一个代码格式化 skill

流程:
1. 环境检查 — 检查基础依赖、网络、权限
2. 需求深挖 — 5-8 个结构化问题
3. 搜索已有 — 6 层搜索源，含深度搜索
4. 方案推送 — A/B/C 三方案对比
5. 分阶段开发 — P0→P1→P2
6. 阶段审查 — Darwin 9 维度评分
7. 循环返工 — 评分 < 80% 返工
8. 本地交付 — 先生成在工作区
9. 部署上线 — cp 到全局目录
```

### 场景 2：审查现有 Skill（Darwin 9 维度）

```
用户: 审查一下这个 skill 的质量

流程:
1. 安全审查 — 4 步协议：元数据→权限→内容→Typosquat
2. Darwin 9 维度评分 — 100 分制
3. 多评委独立评估 — 至少 2 个评委交叉验证
4. 生成报告 — 评分表格 + 亮点 + 问题与建议
5. 综合评级 — A/B/C/D
```

### 场景 3：优化现有 Skill（棘轮机制）

```
用户: 这个 skill 触发不太准，帮我优化

流程:
1. 诊断 — 用审查流程找出问题
2. 方案 — 按优先级排列改进项
3. 改进 — 改 description，补正例/反例
4. 评估 — Darwin 9 维度评分
5. 决策 — 评分 ≥ 上轮？通过 : 回滚
6. 验证 — 用同一个 case 跑一遍
7. 回归 — 用 evals 重跑
8. 触顶检查 — 连续 2 轮 Δ < 2 分？停止 : 继续
```

### 场景 4：搜索并安装

```
用户: 帮我找个代码格式化的 skill

流程:
1. 环境检查 — 检查网络连通性
2. 搜索 — 6 层搜索源，逐级降级
3. 展示结果 — 列表显示，询问用户选择
4. 安全审查 — 4 步协议
5. 安装 — 复制到工作区或全局
```

### 场景 5：合并 Skill

```
用户: 把这几个 skill 合并成一个

流程:
1. 环境检查 — 检查基础依赖
2. 审查原 Skill — Darwin 9 维度逐个评分
3. 理解原 Skill — 阅读每个 SKILL.md
4. 询问用户 — 明确合并方向
5. 网上调研 — 深度搜索最佳实践
6. 综合优化 — 设计合并方案
7. 分阶段开发 — P0→P1→P2
8. 用户确认 — 每个阶段都确认
```

***

## 🔧 单独使用子技能

如果只需要开发或审查功能，可以单独部署子技能：

```bash
# 只部署开发子技能
# 路径因 Agent 而异
cp -r skill-workspace/subskills/dev/ {全局skills目录}/skill-dev/

# 只部署审查子技能
cp -r skill-workspace/subskills/review/ {全局skills目录}/skill-review/
```

**开发子技能：**

- 由主入口路由：用户说"开发xxx skill" → 加载 dev 子技能
- 也可独立部署使用

**审查子技能：**

- 由主入口路由：用户说"审查skill" / "skill评分" → 加载 review 子技能
- 也可独立部署使用

***

## 📚 参考资料

### 主技能

| 文件 | 说明 |
|------|------|
| `references/search-strategy.md` | 搜索策略（6 层搜索源） |
| `references/requirements.md` | 需求深挖指南 |
| `references/proposal-template.md` | 方案推送模板 |
| `references/optimize.md` | 优化方法论（棘轮机制） |
| `references/deploy.md` | 部署指南（Agent 中立） |
| `references/merge-workflow.md` | 合并流程（v4.0） |
| `references/context-management.md` | 上下文管理（v4.0） |
| `references/environment-check.md` | 环境检查（v4.0） |
| `references/darwin-rubric.md` | Darwin 9 维度评估体系（v5.0） |
| `references/ratchet-mechanism.md` | 棘轮机制（v5.0） |
| `references/multi-judge.md` | 多评委独立评估（v5.0） |
| `references/anti-patterns.md` | 反例黑名单（v5.0） |
| `references/agent-neutral-guide.md` | Agent 中立指南（v5.0） |

### 开发子技能

| 文件 | 说明 |
|------|------|
| `subskills/dev/references/official-spec.md` | Agent Skill 规范 |
| `subskills/dev/references/template.md` | SKILL.md 模板文件 |
| `subskills/dev/references/example.md` | 好结果、坏结果、边界样例 |
| `subskills/dev/references/methodology.md` | 社区实战经验总结 |

### 审查子技能

| 文件 | 说明 |
|------|------|
| `subskills/review/references/official-spec.md` | Agent Skill 规范 |
| `subskills/review/references/scoring-criteria.md` | 10 维度评分标准（120 分制） |
| `subskills/review/references/security.md` | 安全审查规则 |

***

## ⚠️ 注意事项

### 必须遵守

- 执行前先做环境检查（流程零）
- 深入理解需求，不要只问几个表面问题
- 全网搜索（含深度搜索 16 引擎），不要自己造轮子
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

***

## 🧪 测试

### 运行评测

```bash
# 主技能评测（18 条）
cat evals/evals.json | jq .

# 开发子技能评测
cat subskills/dev/evals/evals.json | jq .

# 审查子技能评测
cat subskills/review/evals/evals.json | jq .
```

### 手动测试

```bash
# 测试搜索功能（使用你的 Agent CLI）
# Claude Code: claude -p "使用 skill-workspace 搜索 代码格式化"
# Codex CLI:   codex "使用 skill-workspace 搜索 代码格式化"
# Cursor:      在 Cursor 中调用

# 测试开发功能
# Claude Code: claude -p "使用 skill-workspace 开发 代码格式化 skill"

# 测试审查功能（Darwin 9 维度）
# Claude Code: claude -p "使用 skill-workspace 审查 skill-dev"

# 测试优化功能（棘轮机制）
# Claude Code: claude -p "使用 skill-workspace 优化 my-skill"
```

***

## 📈 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| v5.0.0 | 2026-06-13 | 集成 Darwin 9 维度评估体系、棘轮优化机制、多评委独立审查、反例黑名单；Agent 中立化改造；统一部署路径 |
| v4.0.0 | 2026-06-13 | 新增合并流程、环境检查、深度搜索集成（16 引擎）、上下文管理 |
| v3.0.0 | — | 初始版本，基础生命周期管理 |
| v2.3.0 | 2026-06-09 | 面向所有 Agent，移除 Claude 特定路径，支持通用 skills 目录 |
| v2.2.0 | 2026-06-09 | 5 层搜索源、4 步安全审查协议、批量更新支持 |
| v2.1.0 | 2026-06-09 | 整合 find-skills、skill-creation-guide、skill-vetter、cocoloop |
| v2.0.0 | 2026-06-09 | 整合 skill-workspace + skill-dev + skill-review |

***

## 📄 许可证

MIT License

***

## 🔗 相关链接

**官方资源：**
- [Agent Skill 规范](https://docs.anthropic.com/claude-code/skills) — 适用于所有支持 SKILL.md 的 Agent
- [Skills.sh — Agent Skills 生态](https://skills.sh) — `npx skills find`
- [SkillsMP — Agent Skills Marketplace](https://skillsmp.com) — 1.5M+ skills，兼容所有 Agent

**社区资源：**
- [CocoLoop Skill 市场](https://cocoloop.com)
- [TRAE 官方社区](https://forum.trae.cn)

**已整合的全局技能（如已安装）：**
- `find-skills` — Skills.sh 生态搜索
- `skill-creation-guide` — Anthropic 官方创建指南（含 init_skill.py、package_skill.py）
- `skill-vetter` — 安全审查协议
- `cocoloop` — CocoLoop Skill 管理器
- `deep-research-pro` — 深度研究（16 搜索引擎，v4.0 集成）
- `multi-search-engine` — 多搜索引擎集成（16 引擎，v4.0 集成）
