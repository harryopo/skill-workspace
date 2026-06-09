# Skill 安全检查标准

## 检查流程

### 1. 元数据检查
- [ ] `name` 匹配预期名称（无拼写欺骗）
- [ ] `description` 清晰且与实际行为一致
- [ ] `version` 遵循 semver（可选但建议）

### 2. 拼写欺骗检测
- 单字符替换（如 `skil1-review`）
- 同形字符（l/1, O/0, a/а）
- 多余连字符（`skill--review`）
- 与已安装 skill 重名或近似

### 3. 权限范围分析

| 权限 | 风险 | 说明 |
|------|------|------|
| Read | 🟢 Low | 几乎总是合理 |
| Write | 🟡 Medium | 必须说明写入哪些文件 |
| Bash | 🔴 Critical | 必须说明执行哪些命令 |
| network | 🔴 Critical | 必须说明访问哪些端点 |

**危险组合：** `network` + `shell` 同时出现 → 数据泄露风险，必须 BLOCK。

### 4. 内容安全扫描

#### 🔴 Critical（直接 BLOCK）
- 引用 `~/.ssh`、`~/.aws`、`~/.env` 等敏感路径
- 使用 `curl`、`wget`、`nc`、`bash -i` 等网络/反弹命令
- `base64` 混淆内容
- 禁用安全机制的指令
- 未知或可疑 URL

#### 🟡 Warning（需要人工审查）
- `/**/*` 等宽泛通配符
- `sudo` 使用
- 潜在的提示注入

#### ℹ️ Info（建议改进）
- 缺少 description/version

### 5. 动态代码加载检查（URL 递归检查）

检查 Skill 是否从网络动态加载可执行代码，最多 2 层递归：

```
Skill 代码（第 0 层）
    ↓ 发现 fetch/import/require 远程 URL
第 1 层：下载并检查该 URL 内容
    ↓ 如包含新的动态加载
第 2 层：继续检查下一层内容
    ↓ 如第 2 层仍有动态加载
   强制标记为 C 级
```

**递归检查规则：**

| 层级 | 结果 | 处理 |
|------|------|------|
| 无动态加载 | 正常 | 正常评级流程 |
| 仅第 1 层 | T1→B级, T2→C级, T3→禁止 | 根据来源分级 |
| 存在第 2 层 | 最高 C 级 | 标记风险 |
| 超过 2 层 | 强制 C 级 | 多层动态加载风险 |

### 6. 安全审查结论

```
安全评级：SAFE / WARNING / DANGER / BLOCK
风险标记：{数量}
权限范围：{Read / Write / Bash / network}
建议：install / sandbox first / do not install
```

**评级规则：**
- SAFE：无风险标记
- WARNING：仅 ℹ️ 级别
- DANGER：有 🟡 级别
- BLOCK：有 🔴 级别（停止，不进入质量评分）
