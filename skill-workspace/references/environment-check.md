# 环境和依赖检查指南

## 概述

环境检查是 Skill 开发和执行的第零步，目标是在任何操作开始之前，确保执行环境正常、依赖完整、网络通畅、权限充足。通过系统化的检查流程，提前发现并解决环境问题，避免在开发或部署过程中因环境问题导致失败。

**核心原则：**
- 先检查，后执行
- 缺失依赖自动提示并提供安装方案
- 网络和权限问题提前诊断
- 跨平台兼容（Windows / macOS / Linux）

## 检查流程

### 第一步：检查依赖是否安装

验证所有必需工具是否已安装并可执行。

```bash
# 基础依赖检查
curl --version    # HTTP 客户端
git --version     # 版本控制
python --version  # Python 运行时（或 python3 --version）
node --version    # Node.js 运行时
npm --version     # npm 包管理器
```

**检查逻辑：**
- 命令存在 → 记录版本号，继续
- 命令不存在 → 标记为缺失，进入自动安装流程

### 第二步：检查环境是否配置

验证关键环境变量和配置是否正确设置。

```bash
# 网络代理配置
echo $HTTP_PROXY
echo $HTTPS_PROXY

# PATH 是否包含关键路径
echo $PATH

# Python pip 源
pip config list

# npm 镜像源
npm config get registry
```

**检查项：**
- 代理配置（如在企业网络环境中）
- PATH 环境变量完整性
- 包管理器镜像源配置

### 第三步：检查网络是否正常

验证外部服务的可达性。

```bash
# GitHub 连通性
curl -s -o /dev/null -w "%{http_code}" https://github.com

# SkillsMP 连通性
curl -s -o /dev/null -w "%{http_code}" https://skillsmp.com

# npm registry 连通性
curl -s -o /dev/null -w "%{http_code}" https://registry.npmjs.org

# PyPI 连通性
curl -s -o /dev/null -w "%{http_code}" https://pypi.org
```

**检查逻辑：**
- HTTP 200 → 网络正常
- 超时或非 200 → 进入网络诊断流程

### 第四步：检查权限是否足够

验证对关键目录的读写权限和命令执行权限。

```bash
# 检查当前目录写权限
touch .permission_test && rm .permission_test && echo "写权限正常"

# 检查全局 skills 目录
ls {全局skills目录}/ 2>/dev/null || echo "skills 目录不存在或无权限"

# 检查命令执行权限
which curl && which git && which python && which node
```

**检查项：**
- 当前工作目录读写权限
- 全局 skills 目录访问权限
- 临时目录写入权限
- 命令执行权限

## 依赖检查清单

### 基础依赖

| 依赖 | 最低版本 | 用途 | 检查命令 |
|------|----------|------|----------|
| curl | 7.x | HTTP 请求、API 调用 | `curl --version` |
| git | 2.x | 版本控制、仓库克隆 | `git --version` |
| python | 3.8+ | 脚本执行、工具运行 | `python --version` |
| node | 16.x+ | npm 包、JS 工具 | `node --version` |
| npm | 8.x+ | 包管理 | `npm --version` |

### 搜索依赖

| 依赖 | 用途 | 检查方式 |
|------|------|----------|
| deep-research-pro | 深度研究搜索（16 引擎） | Skill 已安装检查 |
| multi-search-engine | 多搜索引擎集成 | Skill 已安装检查 |

**检查 Skill 是否已安装：**

```bash
# Agent
ls {全局skills目录}/deep-research-pro/SKILL.md 2>/dev/null && echo "已安装" || echo "未安装"
ls {全局skills目录}/multi-search-engine/SKILL.md 2>/dev/null && echo "已安装" || echo "未安装"
```

### 安全依赖

| 依赖 | 用途 | 检查方式 |
|------|------|----------|
| skill-vetter | Skill 安全审查 | Skill 已安装检查 |

```bash
ls {全局skills目录}/skill-vetter/SKILL.md 2>/dev/null && echo "已安装" || echo "未安装"
```

## 自动安装机制

### 流程概览

```
检测缺失依赖 → 生成报告 → 提示用户 → 询问是否自动安装 → 执行安装 → 验证安装结果
```

### 检测缺失依赖

```bash
#!/bin/bash
MISSING=()

command -v curl >/dev/null 2>&1 || MISSING+=("curl")
command -v git >/dev/null 2>&1 || MISSING+=("git")
command -v python3 >/dev/null 2>&1 || MISSING+=("python3")
command -v node >/dev/null 2>&1 || MISSING+=("node")
command -v npm >/dev/null 2>&1 || MISSING+=("npm")

if [ ${#MISSING[@]} -eq 0 ]; then
    echo "所有基础依赖已安装"
else
    echo "缺失依赖: ${MISSING[*]}"
fi
```

### 提示用户

输出格式示例：

```
========================================
  环境检查报告
========================================

✅ curl    — 已安装 (7.88.1)
✅ git     — 已安装 (2.39.2)
❌ python3 — 未安装
❌ node    — 未安装
✅ npm     — 已安装 (9.6.7)

缺失 2 个依赖: python3, node

是否自动安装？(y/n)
```

### 询问是否自动安装

- 用户确认 → 根据操作系统选择对应安装命令
- 用户拒绝 → 输出手动安装指引，继续执行（跳过依赖功能）

### 安装后验证

```bash
# 安装后重新检查
for cmd in curl git python3 node npm; do
    if command -v $cmd >/dev/null 2>&1; then
        echo "✅ $cmd — $($cmd --version | head -1)"
    else
        echo "❌ $cmd — 安装失败"
    fi
done
```

## 环境诊断流程

### 网络问题诊断

**GitHub 访问问题：**

```bash
# 测试 DNS 解析
nslookup github.com

# 测试 HTTPS 连接
curl -v https://github.com 2>&1 | head -20

# 测试 Git 协议
git ls-remote https://github.com/octocat/Hello-World.git HEAD
```

**常见原因与解决方案：**

| 症状 | 可能原因 | 解决方案 |
|------|----------|----------|
| 连接超时 | 网络不通或被墙 | 配置代理或镜像 |
| SSL 错误 | 证书问题 | 更新 CA 证书 |
| 403 禁止 | API 限流 | 配置 GitHub Token |
| DNS 解析失败 | DNS 配置错误 | 更换 DNS 服务器 |

**API 调用问题：**

```bash
# 测试 SkillsMP API
curl -s "https://skillsmp.com/api/v1/skills/search?q=test&limit=1" | head -100

# 测试 CocoLoop API
curl -s "https://api.cocoloop.com/api/v1/store/skills?page=1&page_size=1" | head -100
```

### 权限问题诊断

**目录读写问题：**

```bash
# 检查目录权限（Linux/macOS）
ls -la {全局skills目录}/

# Windows 检查
icacls "{全局skills目录}"
```

**命令执行问题：**

```bash
# 检查命令路径
which curl
which git
which python3
which node

# Windows 使用 where
where curl
where git
where python
where node
```

**常见权限问题：**

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| Permission denied | 目录权限不足 | `chmod 755 <dir>` 或以管理员运行 |
| Command not found | PATH 未配置 | 将工具路径加入 PATH |
| EACCES (npm) | npm 全局目录权限 | 配置 npm prefix 或使用 nvm |

### 依赖问题诊断

**版本不兼容：**

```bash
# 检查 Python 版本
python3 -c "import sys; print(sys.version)"

# 检查 Node 版本
node -e "console.log(process.version)"

# 检查 npm 全局包
npm list -g --depth=0
```

**缺失模块：**

```bash
# Python 模块检查
python3 -c "import json; print('json OK')"
python3 -c "import requests; print('requests OK')" 2>/dev/null || echo "requests 未安装"

# Node 模块检查
node -e "try { require('axios'); console.log('axios OK') } catch(e) { console.log('axios 未安装') }"
```

**常见依赖问题：**

| 问题 | 诊断命令 | 解决方案 |
|------|----------|----------|
| Python 模块缺失 | `pip list` | `pip install <module>` |
| Node 包缺失 | `npm list -g` | `npm install -g <package>` |
| 版本过低 | `<cmd> --version` | 升级对应工具 |
| 多版本冲突 | `which python3` | 使用版本管理器 |

## 安装命令模板

### Windows

```powershell
# 使用 winget（推荐，Windows 10/11 自带）
winget install Git.Git
winget install Python.Python.3.12
winget install OpenJS.NodeJS.LTS
winget install cURL.cURL

# 使用 chocolatey
choco install git -y
choco install python -y
choco install nodejs-lts -y
choco install curl -y

# pip 安装 Python 包
pip install <package_name>
pip install -r requirements.txt

# npm 安装 Node 包
npm install -g <package_name>
npm install <package_name>
```

### macOS

```bash
# 使用 Homebrew（推荐）
brew install git
brew install python
brew install node
brew install curl

# 版本管理（推荐）
brew install pyenv        # Python 版本管理
brew install nvm          # Node 版本管理

# pip 安装 Python 包
pip3 install <package_name>
pip3 install -r requirements.txt

# npm 安装 Node 包
npm install -g <package_name>
npm install <package_name>
```

### Linux

```bash
# Debian / Ubuntu
sudo apt update
sudo apt install -y git python3 python3-pip nodejs npm curl

# CentOS / RHEL / Fedora
sudo yum install -y git python3 python3-pip nodejs npm curl
# 或使用 dnf（Fedora）
sudo dnf install -y git python3 python3-pip nodejs npm curl

# Arch Linux
sudo pacman -S git python python-pip nodejs npm curl

# pip 安装 Python 包
pip3 install <package_name>
pip3 install -r requirements.txt

# npm 安装 Node 包
npm install -g <package_name>
npm install <package_name>
```

### 配置镜像源（加速下载）

```bash
# pip 使用清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# npm 使用淘宝源
npm config set registry https://registry.npmmirror.com

# 恢复默认源
pip config unset global.index-url
npm config set registry https://registry.npmjs.org
```

## 快速检查脚本

将以下脚本保存为 `env-check.sh`（Linux/macOS）或 `env-check.ps1`（Windows），一键完成所有检查：

### Bash 版本

```bash
#!/bin/bash
echo "========================================="
echo "  环境和依赖检查"
echo "========================================="

# 基础依赖
for cmd in curl git python3 node npm; do
    if command -v $cmd >/dev/null 2>&1; then
        VERSION=$($cmd --version 2>&1 | head -1)
        echo "✅ $cmd — $VERSION"
    else
        echo "❌ $cmd — 未安装"
    fi
done

echo ""
echo "========================================="
echo "  网络检查"
echo "========================================="

for url in https://github.com https://skillsmp.com https://registry.npmjs.org; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" --connect-timeout 5 $url)
    if [ "$STATUS" = "200" ]; then
        echo "✅ $url — HTTP $STATUS"
    else
        echo "❌ $url — HTTP $STATUS"
    fi
done

echo ""
echo "========================================="
echo "  权限检查"
echo "========================================="

if touch .permission_test 2>/dev/null && rm .permission_test 2>/dev/null; then
    echo "✅ 当前目录 — 可读写"
else
    echo "❌ 当前目录 — 无写权限"
fi

echo ""
echo "检查完成"
```

### PowerShell 版本

```powershell
Write-Host "========================================="
Write-Host "  环境和依赖检查"
Write-Host "========================================="

$commands = @("curl", "git", "python", "node", "npm")
foreach ($cmd in $commands) {
    $path = Get-Command $cmd -ErrorAction SilentlyContinue
    if ($path) {
        $version = & $cmd --version 2>&1 | Select-Object -First 1
        Write-Host "✅ $cmd — $version"
    } else {
        Write-Host "❌ $cmd — 未安装"
    }
}

Write-Host ""
Write-Host "========================================="
Write-Host "  网络检查"
Write-Host "========================================="

$urls = @("https://github.com", "https://skillsmp.com", "https://registry.npmjs.org")
foreach ($url in $urls) {
    try {
        $response = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 5 -ErrorAction Stop
        Write-Host "✅ $url — HTTP $($response.StatusCode)"
    } catch {
        Write-Host "❌ $url — 连接失败"
    }
}

Write-Host ""
Write-Host "检查完成"
```
