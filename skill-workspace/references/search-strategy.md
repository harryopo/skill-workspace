# 搜索策略指南

## 概述

全网搜索是 Skill 开发的第二步，目标是全方位搜索，不遗漏任何有价值的信息源。通过 5 层搜索源并行搜索、智能排序和深度分析，找到最优质的参考实现和技术方案。

## 5 层搜索源

### Tier 1: SkillsMP (skillsmp.com)

**说明：** 1.5M+ skills，兼容所有 Agent（Claude Code、Codex CLI、ChatGPT 等，下同）

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
- "{关键词} agent skill"
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

## 增强搜索能力

### 集成 deep-research-pro

**说明：** 深度研究 agent，整合 16 个搜索引擎（含 7 个中文引擎），自动分解子问题、多源搜索、深度阅读、生成引用报告。

**调用方式：**

```markdown
使用 Skill tool 调用 deep-research-pro
```

**适用场景：**
- 深度调研：需要全面了解某个技术领域的现状
- 技术方案搜索：寻找最佳技术实现方案
- 最佳实践搜索：查找业界公认的最佳实践
- 学术研究：搜索论文、技术文档、RFC 等

**使用示例：**

```markdown
# 深度调研某个技术领域
Skill: deep-research-pro
任务：调研 Agent Skill 开发的最佳实践

# 搜索技术方案
Skill: deep-research-pro
任务：搜索 Python CLI 工具开发的最佳方案
```

**特点：**
- 支持中文/英文智能引擎选择
- 自动分解复杂问题为子问题
- 多源交叉验证，提高信息可靠性
- 生成带引用的结构化报告

### 集成 multi-search-engine

**说明：** 多搜索引擎集成，支持 16 个引擎（7 国内 + 9 国际），无需 API 密钥。

**支持的搜索引擎：**

| 类型 | 引擎 | 特点 |
|------|------|------|
| 国内 | 抖音 | 短视频、教程 |
| 国内 | B站 | 技术视频、教程 |
| 国内 | 知乎 | 问答、深度讨论 |
| 国内 | 掘金 | 技术博客 |
| 国内 | CSDN | 技术文章 |
| 国内 | 微信公众号 | 行业分析 |
| 国内 | 小红书 | 产品体验、评测 |
| 国际 | Google | 综合搜索 |
| 国际 | GitHub | 代码、项目 |
| 国际 | Stack Overflow | 技术问答 |
| 国际 | Reddit | 讨论、经验分享 |
| 国际 | Dev.to | 技术博客 |

**调用方式：**

```markdown
使用 Skill tool 调用 multi-search-engine
```

**适用场景：**
- 通用搜索：需要快速获取多个来源的信息
- 中文内容搜索：搜索国内技术社区内容
- 视频教程搜索：在抖音、B站搜索教程
- 行业分析：搜索微信公众号、小红书等平台

**使用示例：**

```markdown
# 搜索中文技术内容
Skill: multi-search-engine
关键词：Python CLI 工具开发教程
引擎：掘金、知乎、CSDN

# 搜索视频教程
Skill: multi-search-engine
关键词：Agent 使用教程
引擎：B站、抖音

# 全网搜索
Skill: multi-search-engine
关键词：AI Agent 开发框架
引擎：全部
```

**特点：**
- 无需配置 API 密钥
- 支持高级搜索语法
- 支持时间过滤
- 支持站点搜索
- 支持 WolframAlpha 知识查询

## GitHub 国内访问解决方案

### 问题背景

国内访问 GitHub 经常遇到以下问题：
- 访问速度慢
- 克隆仓库超时
- API 请求失败
- 下载文件失败

### 解决方案

#### 方案 1：使用镜像站

**推荐镜像站：**

| 镜像站 | 地址 | 特点 |
|--------|------|------|
| moeyy | github.moeyy.xyz | 稳定、速度快 |
| fastgit | hub.fastgit.xyz | 社区维护 |
| ghproxy | ghproxy.com | 支持 raw 文件 |
| kgithub | kgithub.com | 综合镜像 |

**使用方法：**

```bash
# 克隆仓库（使用 moeyy 镜像）
git clone https://github.moeyy.xyz/owner/repo.git

# 下载 raw 文件
curl -o SKILL.md https://github.moeyy.xyz/owner/repo/raw/main/SKILL.md

# 下载 release 文件
curl -o file.zip https://github.moeyy.xyz/owner/repo/releases/download/v1.0/file.zip
```

#### 方案 2：使用 GitHub API

**说明：** GitHub API 比直接访问网页更稳定，推荐优先使用。

**使用方法：**

```bash
# 搜索仓库
curl -s "https://api.github.com/search/repositories?q={关键词}&sort=stars"

# 获取仓库内容
curl -s "https://api.github.com/repos/owner/repo/contents/SKILL.md"

# 获取 release
curl -s "https://api.github.com/repos/owner/repo/releases/latest"
```

**注意事项：**
- 未认证请求限制：60 次/小时
- 认证后限制：5000 次/小时
- 使用 `jq` 解析 JSON 响应

#### 方案 3：使用代理

**说明：** 如果用户配置了代理，优先使用代理访问。

**配置方法：**

```bash
# 设置 Git 代理
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

# 设置环境变量
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 临时使用代理
curl -x http://127.0.0.1:7890 https://github.com
```

**检测代理：**

```bash
# 检查是否配置了代理
echo $http_proxy
echo $https_proxy

# 检查 Git 代理配置
git config --global --get http.proxy
```

#### 方案 4：降级到其他搜索源

**说明：** 当 GitHub 完全无法访问时，降级到其他搜索源。

**降级策略：**

```
GitHub 不可用
    ├── 尝试镜像站
    │   ├── 成功 → 使用镜像站
    │   └── 失败 → 继续降级
    ├── 尝试 GitHub API
    │   ├── 成功 → 使用 API
    │   └── 失败 → 继续降级
    ├── 尝试代理
    │   ├── 成功 → 使用代理
    │   └── 失败 → 继续降级
    └── 降级到其他源
        ├── SkillsMP
        ├── Skills.sh
        ├── CocoLoop
        └── 技术博客/社区
```

**降级代码示例：**

```bash
# 尝试直接访问
if curl -s --connect-timeout 5 "https://github.com" > /dev/null; then
    echo "GitHub 可直接访问"
    SOURCE="github"
# 尝试镜像站
elif curl -s --connect-timeout 5 "https://github.moeyy.xyz" > /dev/null; then
    echo "使用 moeyy 镜像"
    SOURCE="moeyy"
# 尝试 API
elif curl -s --connect-timeout 5 "https://api.github.com" > /dev/null; then
    echo "使用 GitHub API"
    SOURCE="api"
else
    echo "GitHub 不可用，降级到其他源"
    SOURCE="fallback"
fi
```

## 调研报告生成模板

### 模板说明

调研报告用于记录搜索过程、分析结果和推荐方案，便于后续参考和团队协作。

### 完整模板

```markdown
# 调研报告：{调研主题}

**调研时间：** {日期}
**调研人：** {姓名}
**调研目标：** {简述目标}

---

## 1. 搜索统计

### 搜索源使用情况

| 搜索源 | 搜索关键词 | 结果数量 | 有效结果 | 备注 |
|--------|-----------|---------|---------|------|
| SkillsMP | {关键词} | {数量} | {数量} | {备注} |
| Skills.sh | {关键词} | {数量} | {数量} | {备注} |
| CocoLoop | {关键词} | {数量} | {数量} | {备注} |
| GitHub | {关键词} | {数量} | {数量} | {备注} |
| deep-research-pro | {关键词} | {数量} | {数量} | {备注} |
| multi-search-engine | {关键词} | {数量} | {数量} | {备注} |

### 搜索关键词汇总

- 主关键词：{关键词 1}
- 扩展关键词：{关键词 2}、{关键词 3}
- 长尾关键词：{关键词 4}

### 搜索统计

- 总搜索次数：{数量}
- 总结果数量：{数量}
- 有效结果数量：{数量}
- 有效率：{百分比}

---

## 2. 调研发现

### 2.1 现有方案

#### 方案 1：{方案名称}

- **来源：** {来源}
- **地址：** {链接}
- **描述：** {描述}
- **技术栈：** {技术栈}
- **优点：**
  - {优点 1}
  - {优点 2}
- **缺点：**
  - {缺点 1}
  - {缺点 2}
- **适用场景：** {场景}

#### 方案 2：{方案名称}

（同上格式）

### 2.2 最佳实践

#### 实践 1：{实践名称}

- **来源：** {来源}
- **描述：** {描述}
- **关键点：**
  - {关键点 1}
  - {关键点 2}
- **参考代码：**
  ```{language}
  {代码示例}
  ```

#### 实践 2：{实践名称}

（同上格式）

### 2.3 踩坑经验

#### 坑 1：{问题描述}

- **现象：** {现象}
- **原因：** {原因}
- **解决方案：** {方案}
- **预防措施：** {措施}

#### 坑 2：{问题描述}

（同上格式）

---

## 3. 推荐方案

### 推荐方案：{方案名称}

**推荐理由：**

1. **技术可行性：** {理由}
2. **社区活跃度：** {理由}
3. **文档完整性：** {理由}
4. **学习成本：** {理由}

**实施方案：**

1. **第一步：** {步骤}
2. **第二步：** {步骤}
3. **第三步：** {步骤}

**风险评估：**

| 风险 | 影响 | 概率 | 应对措施 |
|------|------|------|---------|
| {风险 1} | {影响} | {概率} | {措施} |
| {风险 2} | {影响} | {概率} | {措施} |

**备选方案：**

- 备选 1：{方案名称}（当 {条件} 时考虑）
- 备选 2：{方案名称}（当 {条件} 时考虑）

---

## 4. 参考资料

1. [{标题}]({链接}) - {说明}
2. [{标题}]({链接}) - {说明}
3. [{标题}]({链接}) - {说明}

---

## 5. 附录

### 附录 A：搜索结果详情

（详细搜索结果列表）

### 附录 B：代码示例

（相关代码示例）

### 附录 C：配置文件

（相关配置文件）
```

### 简化模板

```markdown
# 调研报告：{调研主题}

**时间：** {日期}

## 搜索统计

- 搜索源：{数量} 个
- 关键词：{关键词列表}
- 结果数量：{数量}

## 调研发现

### 现有方案

1. {方案 1} - {简述}
2. {方案 2} - {简述}

### 最佳实践

- {实践 1}
- {实践 2}

### 踩坑经验

- {坑 1}：{解决方案}
- {坑 2}：{解决方案}

## 推荐方案

**推荐：** {方案名称}

**理由：**
- {理由 1}
- {理由 2}

**实施步骤：**
1. {步骤 1}
2. {步骤 2}
```

### 使用示例

```markdown
# 调研报告：Python CLI 工具开发框架

**调研时间：** 2026-06-13
**调研目标：** 选择合适的 Python CLI 开发框架

---

## 1. 搜索统计

### 搜索源使用情况

| 搜索源 | 搜索关键词 | 结果数量 | 有效结果 |
|--------|-----------|---------|---------|
| GitHub | Python CLI framework | 156 | 23 |
| deep-research-pro | Python CLI 最佳实践 | 45 | 12 |
| multi-search-engine | Python CLI 开发教程 | 89 | 18 |

### 搜索统计

- 总搜索次数：3
- 总结果数量：290
- 有效结果数量：53
- 有效率：18.3%

---

## 2. 调研发现

### 2.1 现有方案

#### 方案 1：Click

- **来源：** GitHub
- **地址：** https://github.com/pallets/click
- **描述：** Python 包装器，用于创建美观的命令行界面
- **技术栈：** Python
- **优点：**
  - 文档完善
  - 社区活跃
  - 易于学习
- **缺点：**
  - 功能相对简单
  - 不支持复杂参数验证
- **适用场景：** 简单 CLI 工具

#### 方案 2：Typer

- **来源：** GitHub
- **地址：** https://github.com/tiangolo/typer
- **描述：** 基于 Click 的现代 CLI 框架
- **技术栈：** Python、Click
- **优点：**
  - 类型提示支持
  - 自动生成帮助文档
  - 与 FastAPI 风格一致
- **缺点：**
  - 相对较新
  - 依赖 Click
- **适用场景：** 现代 Python CLI 工具

### 2.2 最佳实践

#### 实践 1：使用类型提示

- **来源：** Python 官方文档
- **描述：** 使用类型提示提高代码可读性和可维护性
- **关键点：**
  - 使用 `typing` 模块
  - 为所有参数添加类型注解
  - 使用 `Optional` 表示可选参数

### 2.3 踩坑经验

#### 坑 1：Click 参数顺序问题

- **现象：** 参数顺序影响命令解析
- **原因：** Click 按定义顺序解析参数
- **解决方案：** 使用装饰器定义参数顺序
- **预防措施：** 查阅官方文档了解参数顺序规则

---

## 3. 推荐方案

### 推荐方案：Typer

**推荐理由：**

1. **技术可行性：** 基于 Click，成熟稳定
2. **社区活跃度：** FastAPI 作者维护，社区活跃
3. **文档完整性：** 文档完善，示例丰富
4. **学习成本：** 低，与 FastAPI 风格一致

**实施方案：**

1. **第一步：** 安装 Typer
2. **第二步：** 创建基本 CLI 结构
3. **第三步：** 实现核心功能

**风险评估：**

| 风险 | 影响 | 概率 | 应对措施 |
|------|------|------|---------|
| 依赖 Click | 低 | 高 | 关注 Click 更新 |

**备选方案：**

- 备选 1：Click（当需要更底层控制时考虑）
- 备选 2：Argparse（当不希望引入外部依赖时考虑）

---

## 4. 参考资料

1. [Click 官方文档](https://click.palletsprojects.com/) - Click 框架文档
2. [Typer 官方文档](https://typer.tiangolo.com/) - Typer 框架文档
3. [Python CLI 最佳实践](https://realpython.com/command-line-interfaces-python-argparse/) - Real Python 教程
```

## 更新记录

| 日期 | 更新内容 | 更新人 |
|------|---------|--------|
| 2026-06-13 | 初始版本 | - |
| 2026-06-13 | 添加 deep-research-pro、multi-search-engine 集成 | - |
| 2026-06-13 | 添加 GitHub 国内访问解决方案 | - |
| 2026-06-13 | 添加调研报告生成模板 | - |
