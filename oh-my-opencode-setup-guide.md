# Oh My OpenCode 安装与配置指南（OpenCode Go 版）

> 本文档基于实际安装记录整理，适用于在 OpenCode Go 环境中安装和配置 oh-my-opencode 插件。
> 
> **目标读者**：已有 Node.js/npm 环境，希望在 OpenCode Go 中使用 oh-my-opencode 的用户。

---

## 目录

1. [前提条件](#1-前提条件)
2. [安装 OpenCode Go](#2-安装-opencode-go)
3. [安装 oh-my-opencode](#3-安装-oh-my-opencode)
4. [配置为 OpenCode 插件](#4-配置为-opencode-插件)
5. [安装辅助工具](#5-安装辅助工具)
   - 5.1 [安装 GitHub CLI](#51-安装-github-cli)
   - 5.2 [安装 LSP 服务器](#52-安装-lsp-服务器)
6. [配置模型](#6-配置模型)
7. [验证安装](#7-验证安装)
8. [核心功能与使用](#8-核心功能与使用)
9. [任务执行检查机制](#9-任务执行检查机制)
10. [常见问题与故障排除](#10-常见问题与故障排除)

---

## 1. 前提条件

在开始之前，请确保你的系统已满足以下条件：

### 1.1 必需环境

| 依赖 | 最低版本 | 检查命令 |
|------|----------|----------|
| Node.js | v18+ | `node --version` |
| npm | v9+ | `npm --version` |

```bash
# 检查 Node.js 和 npm
node --version
npm --version
```

如果未安装，请参考 [Node.js 官网](https://nodejs.org/) 或使用包管理器安装：

```bash
# Arch Linux
sudo pacman -S nodejs npm

# Ubuntu/Debian
sudo apt update && sudo apt install -y nodejs npm

# macOS
brew install node
```

### 1.2 可选但推荐的工具

| 工具 | 用途 | 检查命令 |
|------|------|----------|
| git | 版本控制、GitHub 集成 | `git --version` |
| curl | 下载工具 | `curl --version` |

### 1.3 网络要求

- 能够访问 npm registry（`registry.npmjs.org`）
- 能够访问 GitHub（`github.com`）

---

## 2. 安装 OpenCode Go

> ⚠️ **重要**：oh-my-opencode 是 OpenCode 的**插件**，必须先安装 OpenCode 本体。

### 2.1 安装 OpenCode CLI

参考 [OpenCode 官方安装文档](https://docs.opencode.ai/installation) 进行安装。

常见安装方式：

```bash
# 方式一：通过 npm 全局安装
npm install -g @opencode-ai/cli

# 方式二：通过官方安装脚本
curl -sSL https://get.opencode.ai | bash

# 方式三：下载预编译二进制
# 访问 https://github.com/opencode-ai/opencode/releases 下载对应平台的二进制文件
```

### 2.2 登录 OpenCode Go

安装完成后，需要配置 API Key 才能使用模型：

```bash
# 查看 OpenCode 帮助
opencode --help

# 配置 API Key（OpenCode Go 需要）
opencode auth login

# 或手动设置（具体命令可能随版本变化，请以 opencode --help 为准）
# opencode config set api_key <your-api-key>
```

**登录成功后验证：**

```bash
opencode --version
# 应输出类似：1.14.20

opencode models
# 应显示可用模型列表，包含 opencode-go/* 系列
```

> **💡 提示**：如果 `opencode models` 能正常显示模型列表，说明 OpenCode Go 已正确配置并可以访问模型 API。

---

## 3. 安装 oh-my-opencode

### 3.1 选择安装位置

**推荐做法**：在用户主目录下创建专用目录进行安装，避免污染项目目录。

```bash
# 创建 oh-my-opencode 专用目录
mkdir -p ~/tools/oh-my-opencode
cd ~/tools/oh-my-opencode

# 初始化 npm 项目
npm init -y
```

> **为什么不直接在 `~` 目录安装？**
> 
> 在 `~` 目录直接 `npm init -y` 会创建一个全局 `node_modules`，可能导致：
> - 依赖冲突（与其他项目的 npm 包版本冲突）
> - 难以管理（无法确定哪些包是 oh-my-opencode 专用的）
> - 安全风险（全局安装的包对所有目录可见）

### 3.2 安装 oh-my-opencode

```bash
cd ~/tools/oh-my-opencode
npm install oh-my-opencode --save
```

**安装过程说明：**
- 安装包较大（包含多个平台二进制文件），可能需要 **5–15 分钟**
- 安装过程中会下载可选依赖（如 `oh-my-opencode-linux-x64`）
- 如果卡住，请检查网络连接

### 3.3 验证安装

```bash
cd ~/tools/oh-my-opencode
npx oh-my-opencode --version
```

应输出类似：`4.0.0`

---

## 4. 配置为 OpenCode 插件

### 4.1 创建 OpenCode 配置目录

```bash
mkdir -p ~/.config/opencode
```

### 4.2 创建 OpenCode 主配置

编辑或创建 `~/.config/opencode/opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["oh-my-openagent"]
}
```

> **注意**：插件在 npm 中的包名为 `oh-my-opencode`，但在 OpenCode 配置中使用兼容性名称 `oh-my-openagent`。两者为同一项目，过渡期同时支持。

### 4.3 创建插件配置文件

创建 `~/.config/opencode/oh-my-openagent.jsonc`（JSONC 格式支持注释）：

```jsonc
{
  // Oh My OpenAgent 插件配置
  // 详细文档: https://github.com/code-yeongyu/oh-my-openagent/blob/dev/docs/reference/configuration.md

  // 注意：$schema 是可选的，如果路径不正确可以删除
  // "$schema": "~/tools/oh-my-opencode/node_modules/oh-my-opencode/dist/oh-my-opencode.schema.json",

  // 后台任务并发配置
  "background_agents": {
    "max_concurrent": 5
  },

  // 内置 MCP 服务器
  "mcps": {
    "websearch": {
      "enabled": true
    },
    "context7": {
      "enabled": true
    },
    "grep_app": {
      "enabled": true
    }
  },

  // LSP 配置
  "lsp": {
    "enabled": true
  },

  // 实验性功能
  "experimental": {
    "aggressive_truncation": false,
    "auto_resume": true
  }
}
```

> **⚠️ 关于 `$schema`**：
> 
> 配置文件位于 `~/.config/opencode/`，而 oh-my-opencode 安装在 `~/tools/oh-my-opencode/`。
> 两个路径不同，使用相对路径 `./node_modules/...` 会导致 schema 验证失败。
> 
> **解决方案**：
> 1. 使用绝对路径：`"$HOME/tools/oh-my-opencode/node_modules/oh-my-opencode/dist/oh-my-opencode.schema.json"`（部分编辑器支持环境变量）
> 2. 或者**直接删除 `$schema` 字段**（它是可选的，不影响功能）

---

## 5. 安装辅助工具

oh-my-opencode 依赖以下工具提供完整功能。

### 5.1 安装 GitHub CLI

#### 5.1.1 检查是否已安装

```bash
which gh
```

#### 5.1.2 安装方式

**有 sudo 权限时（推荐）：**

```bash
# Arch Linux
sudo pacman -S github-cli

# Ubuntu/Debian
sudo apt update && sudo apt install -y gh

# macOS
brew install gh
```

**无 sudo 权限时（手动安装）：**

```bash
# 创建本地 bin 目录
mkdir -p ~/.local/bin

# 获取最新版本号（自动，不硬编码）
LATEST_VERSION=$(curl -sL https://api.github.com/repos/cli/cli/releases/latest | grep '"tag_name"' | cut -d'"' -f4)
echo "Latest gh version: $LATEST_VERSION"

# 下载对应平台的最新版本（以 linux-amd64 为例）
curl -sL "https://github.com/cli/cli/releases/download/${LATEST_VERSION}/gh_${LATEST_VERSION#v}_linux_amd64.tar.gz" | tar -xz -C /tmp

# 复制到本地 bin
cp "/tmp/gh_${LATEST_VERSION#v}_linux_amd64/bin/gh" ~/.local/bin/

# 验证
~/.local/bin/gh --version
```

#### 5.1.3 将 ~/.local/bin 加入 PATH

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

> **⚠️ 重要**：修改 `.bashrc` 后，**必须执行以下操作之一**才能使 PATH 生效：
> 1. 重新打开一个新的终端窗口/标签页
> 2. 或运行 `source ~/.bashrc`

**验证 PATH 是否生效：**

```bash
source ~/.bashrc
which gh
# 应输出：/home/<username>/.local/bin/gh
```

#### 5.1.4 登录 GitHub

```bash
gh auth login
```

按提示操作：
1. **Where do you use GitHub?** → 选择 `GitHub.com`（**不要选 Other**）
2. **Preferred protocol?** → 选择 `SSH`
3. **Generate SSH key?** → 选择 `Yes`（如果没有现有密钥）
4. **Passphrase** → 
   - 个人开发机：直接回车（留空，最方便）
   - 共享环境/工作电脑：**建议设置密码**，并使用 `ssh-agent` 管理
5. **Title for SSH key** → 回车使用默认 `GitHub CLI`
6. **Upload SSH public key?** → 选择你的公钥文件（如 `~/.ssh/id_ed25519.pub`）
7. **Authenticate GitHub CLI?** → 选择 `Login with a web browser`
8. 复制终端显示的 8 位验证码，浏览器打开 `https://github.com/login/device`，粘贴完成

#### 5.1.5 验证登录

```bash
gh auth status
```

**常见问题：登录失败**

- **错误：`Post "https://fengwatermelon/login/device/code": EOF`**
  - 原因：选择了 `Other` 并填写了错误的主机名
  - 解决：重新运行 `gh auth login`，选择 `GitHub.com`

---

### 5.2 安装 LSP 服务器

LSP（Language Server Protocol）为 AI Agent 提供 IDE 级别的代码操作能力。

#### 5.2.1 安装常用 LSP 服务器

在 oh-my-opencode 安装目录中执行：

```bash
cd ~/tools/oh-my-opencode
npm install --save-dev \
  typescript-language-server \
  vscode-langservers-extracted \
  yaml-language-server \
  bash-language-server \
  pyright
```

> **安装耗时**：约 1–5 分钟，取决于网络速度。

#### 5.2.2 已安装的 LSP 工具列表

安装后，以下命令可在 `~/tools/oh-my-opencode/node_modules/.bin/` 中使用：

| 语言 | 命令 |
|------|------|
| TypeScript/JavaScript | `typescript-language-server` |
| Python | `pyright-langserver` |
| YAML | `yaml-language-server` |
| Bash | `bash-language-server` |
| JSON | `vscode-json-language-server` |
| HTML | `vscode-html-language-server` |
| CSS | `vscode-css-language-server` |
| ESLint | `vscode-eslint-language-server` |

---

## 6. 配置模型

### 6.1 查看可用的 OpenCode Go 模型

```bash
opencode models
```

**可用模型示例：**

```
opencode-go/deepseek-v4-flash
opencode-go/deepseek-v4-pro
opencode-go/glm-5
opencode-go/glm-5.1
opencode-go/kimi-k2.5
opencode-go/kimi-k2.6
opencode-go/mimo-v2.5
opencode-go/mimo-v2.5-pro
opencode-go/minimax-m2.5
opencode-go/minimax-m2.7
opencode-go/qwen3.5-plus
opencode-go/qwen3.6-plus
```

### 6.2 配置 Agent 模型

编辑 `~/.config/opencode/oh-my-openagent.jsonc`，在 `agents` 部分指定模型：

```jsonc
{
  // ... 其他配置 ...

  "agents": {
    // Sisyphus: 主调度器（推荐用最强模型）
    "sisyphus": {
      "model": "opencode-go/kimi-k2.6"
    },
    // Prometheus: 规划师
    "prometheus": {
      "model": "opencode-go/kimi-k2.6"
    },
    // Hephaestus: 深度工作者
    "hephaestus": {
      "model": "opencode-go/deepseek-v4-pro"
    },
    // Momus: 计划审查专家
    "momus": {
      "model": "opencode-go/deepseek-v4-pro"
    }
  }
}
```

#### Agent 角色说明

| Agent | 角色 | 推荐模型类型 |
|-------|------|-------------|
| **Sisyphus** | 主调度器，分配任务给专家团队 | 最强通用模型 |
| **Prometheus** | 规划师，先访谈再规划 | 强推理模型 |
| **Hephaestus** | 深度工作者，独立端到端执行 | 长上下文/代码模型 |
| **Momus** | 计划审查专家，审查可执行性 | 强推理模型 |
| **Oracle** | 架构咨询、复杂调试 | 默认（可覆盖） |
| **Librarian** | 检索文档、代码示例 | 默认（可覆盖） |
| **Explore** | 代码库搜索、模式发现 | 默认（可覆盖） |

#### 模型选择建议

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| `opencode-go/kimi-k2.6` | 综合能力最强 | 主调度器、规划师 |
| `opencode-go/glm-5.1` | 复杂逻辑、架构 | ultrabrain 类别任务 |
| `opencode-go/deepseek-v4-pro` | 深度工作、长上下文 | Hephaestus、Momus |
| `opencode-go/mimo-v2.5-pro` | 快速响应 | 简单任务 |
| `opencode-go/qwen3.6-plus` | 代码生成 | 编程任务 |

> **注意**：模型格式必须为 `"provider/model"`，如 `"opencode-go/kimi-k2.6"`。仅写 `"kimi-k2.6"` 会报错。

#### 未配置 Agent 的默认行为

如果某个 Agent 没有在 `agents` 中显式配置模型，oh-my-opencode 会使用**内置默认回退链**。例如 Momus 的默认回退链：
1. GPT-5.5 (xhigh)
2. Claude Opus 4.7 (max)
3. Gemini 3.1 Pro (high)
4. GLM-5.1 (opencode-go)

如果你只使用 `opencode-go`，未覆盖的 Agent 会自动回退到 `opencode-go/glm-5.1`。

---

## 7. 验证安装

### 7.1 运行诊断命令

```bash
cd ~/tools/oh-my-opencode
npx oh-my-opencode doctor
```

**期望输出：**

```
 oMoMoMoMo Doctor

 ✓ System OK (opencode 1.14.20 · oh-my-openagent unknown)
```

### 7.2 如果仍有警告

| 警告 | 原因 | 解决 |
|------|------|------|
| GitHub CLI missing | gh 未安装或不在 PATH | 安装 gh 并确保 `source ~/.bashrc` 后 `which gh` 能找到 |
| GitHub CLI not authenticated | gh 已安装但未登录 | 运行 `gh auth login` |
| No LSP servers detected | LSP 未安装 | 在 `~/tools/oh-my-opencode` 目录运行 `npm install` 安装依赖 |
| Model override uses unavailable provider | 模型名称格式错误 | 使用 `"opencode-go/model-name"` 格式 |

---

## 8. 核心功能与使用

### 8.1 启动 OpenCode（自动加载插件）

```bash
# 进入项目目录
opencode .
```

进入后，oh-my-opencode 的所有功能已可用。

### 8.2 核心命令

| 命令 | 作用 |
|------|------|
| `ultrawork` 或 `ulw` | 一键触发所有智能体，任务完成前绝不停止 |
| `/start-work` | 召唤 Prometheus 规划师，先访谈、再规划、最后执行 |
| `/init-deep` | 自动生成项目级 `AGENTS.md` 上下文文件 |
| `/ulw-loop` | 启动 Ralph Loop，自我引用闭环直到 100% 完成 |
| `skill(name='review-work')` | 启动 5 智能体并行审查已完成的工作 |

### 8.3 调用 Skill 的方式

在 opencode 对话中，你可以通过以下方式调用内置 skill：

```
# 方式一：直接输入 skill 名称
skill(name='review-work')

# 方式二：使用命令形式（如果 skill 注册了命令）
/review-work

# 方式三：让 Sisyphus 自动检测（在特定关键词下自动触发）
# 例如说 "review my work" 会自动触发 review-work skill
```

**内置 Skills：**

| Skill | 触发方式 | 作用 |
|-------|----------|------|
| `playwright` | `skill(name='playwright')` | 浏览器自动化 |
| `git-master` | `skill(name='git-master')` | 原子级 Git 提交与 rebase |
| `frontend-ui-ux` | `skill(name='frontend-ui-ux')` | UI/UX 实现 |
| `review-work` | `skill(name='review-work')` | 5 智能体验收审查 |
| `ai-slop-remover` | `skill(name='ai-slop-remover')` | 移除 AI 代码坏味道 |
| `team-mode` | `skill(name='team-mode')` | 团队模式调度 |

### 8.4 类别路由（自动选择模型）

Sisyphus 会根据任务类型自动路由到最适合的模型：

- `visual-engineering` → UI/UX、前端、设计
- `ultrabrain` → 复杂逻辑、架构决策
- `deep` → 深度调研与执行
- `quick` → 单文件修改、修错字

**无需手动选择，框架自动匹配。**

### 8.5 内置 MCP（已默认启用）

- **websearch** — 网络搜索（Exa）
- **context7** — 官方文档检索
- **grep_app** — GitHub 代码搜索

### 8.6 Team Mode（团队模式）

oh-my-opencode 支持多智能体并行协作的团队模式。在复杂任务中，Sisyphus 会同时调度多个专家智能体并行工作，显著提高处理效率。

启用方式：在对话中提到需要多个专家协作，或在配置中启用相关设置。

### 8.7 实用工作流示例

#### 示例 1：开发新功能（推荐先规划）

```bash
opencode .
# 在 opencode 中输入：
/start-work
# Prometheus 会访谈你，了解需求后制定详细计划
```

#### 示例 2：直接开干

```bash
opencode .
# 在 opencode 中输入：
ultrawork 给这个 React 项目添加用户认证功能，包括登录、注册和 JWT token 管理
```

#### 示例 3：初始化项目上下文

```bash
cd /path/to/your/project
opencode .
# 在 opencode 中输入：
/init-deep
```

#### 示例 4：完成后审查

```bash
opencode .
# 在 opencode 中输入：
skill(name='review-work')
# 启动 5 智能体验收审查
```

---

## 9. 任务执行检查机制

oh-my-opencode 设计了多层任务执行检查防线，确保任务真正完成：

### 9.1 事前审查：Momus（计划审查专家）

- **职责**：在动手写代码**之前**，审查 Prometheus 生成的计划是否可执行
- **检查重点**：
  - 引用的文件是否真实存在
  - 任务描述是否足够清晰
  - 是否存在会阻塞工作的漏洞
- **原则**：有 80% 清晰度就通过，不过度追求完美

### 9.2 事中强制：Todo Continuation Enforcer（Hook）

- **机制**：系统级 Hook，在每个会话结束时自动触发
- **行为**：
  - 如果还有未完成的 todo，**强制要求 AI 继续执行**，不能结束会话
  - 最大停滞次数：`3`
  - 最大连续失败次数：`5`
- **作用**：防止 AI "摸鱼"或半途而废

### 9.3 事中监控：Sisyphus（主调度器）

- **Boulder State**：管理当前活跃的工作计划状态
- **Task Sessions**：为每个顶层任务维护可复用的子智能体会话
- **自动恢复**：从会话错误、上下文窗口超限、API 失败中自动恢复

### 9.4 事后验收：Review-Work Skill（5 智能体并行审查）

完成重要工作后，调用 `skill(name='review-work')` 会启动 **5 个并行子智能体**：

| # | 智能体 | 角色 | 审查重点 |
|---|--------|------|----------|
| 1 | Goal Verifier (Oracle) | 目标验证 | 我们是否做了用户要求的事？ |
| 2 | QA Executor | 动手 QA | 代码真的能跑吗？ |
| 3 | Code Reviewer (Oracle) | 代码质量 | 代码写得好吗？ |
| 4 | Security Auditor (Oracle) | 安全审计 | 有安全隐患吗？ |
| 5 | Context Miner | 上下文挖掘 | 是否遗漏了重要信息？ |

**规则**：5 个全部通过才算通过，**只要有一个失败，整个审查就失败**。

### 9.5 无限循环：Ralph Loop / `ulw-loop`

- **机制**：自我引用的开发闭环
- **行为**：AI 审查自己的工作 → 发现不足 → 继续改进 → 再审查 → 再改进……
- **终止条件**：达到 100% 完成度才停止

---

## 10. 常见问题与故障排除

### Q1: `opencode: command not found`

**原因**：OpenCode 未安装或未加入 PATH。  
**解决**：

```bash
# 检查是否安装
which opencode

# 如果未安装，参考第 2 节安装 OpenCode Go
npm install -g @opencode-ai/cli

# 或检查 PATH
echo $PATH | grep opencode
```

### Q2: `gh: command not found`

**原因**：`~/.local/bin` 不在 PATH 中，或 `.bashrc` 修改后未生效。  
**解决**：

```bash
# 确认 .bashrc 已修改
grep '.local/bin' ~/.bashrc

# 使 PATH 生效
source ~/.bashrc

# 验证
which gh
```

### Q3: `gh auth login` 报错 `Post "https://<错误主机名>/login/device/code": EOF`

**原因**：选择了 `Other` 并填写了错误的主机名。  
**解决**：重新运行，选择 `GitHub.com`：

```bash
gh auth login
# 选择 GitHub.com，而非 Other
```

### Q4: `npx oh-my-opencode doctor` 提示模型配置错误

**原因**：模型名称格式不正确，缺少 provider 前缀。  
**解决**：使用 `"provider/model"` 格式：

```jsonc
// ❌ 错误
"model": "kimi-k2.6"

// ✅ 正确
"model": "opencode-go/kimi-k2.6"
```

### Q5: Git 推送失败（`Host key verification failed` 或认证问题）

**原因**：SSH 密钥未配置，或 HTTPS 认证失败。  
**解决方案**：

**方案 A：使用 gh CLI 推送（推荐）**

```bash
# 使用 gh 的 git-credential 助手
git config --global credential.https://github.com.helper '!gh auth git-credential'
git push -u origin main
```

**方案 B：使用 SSH（如果已配置）**

```bash
# 确保 SSH key 已添加到 GitHub
gh ssh-key list

# 测试连接
ssh -T git@github.com

# 使用 SSH remote
git remote set-url origin git@github.com:<username>/<repo>.git
git push -u origin main
```

**方案 C：使用 GitHub API 直接上传（无需 git）**

```bash
# 获取 GitHub Token
export GITHUB_TOKEN=$(gh auth token)

# 上传文件
curl -X PUT \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{
    "message": "docs: update guide",
    "content": "'"$(base64 -w 0 your-file.md)"'"
  }' \
  https://api.github.com/repos/<username>/<repo>/contents/your-file.md
```

### Q6: 如何卸载 oh-my-opencode？

```bash
# 1. 从 OpenCode 配置中移除插件
# 编辑 ~/.config/opencode/opencode.json，删除 "oh-my-openagent"

# 2. 删除插件配置文件
rm -f ~/.config/opencode/oh-my-openagent.jsonc

# 3. 卸载 npm 包
cd ~/tools/oh-my-opencode
npm uninstall oh-my-opencode

# 4. （可选）删除整个安装目录
rm -rf ~/tools/oh-my-opencode

# 5. （可选）删除全局配置中的残留
rm -rf ~/.config/opencode/node_modules
```

### Q7: 如何关闭匿名遥测？

oh-my-opencode 默认开启匿名遥测（统计活跃安装数），不会收集敏感信息。如需关闭：

```bash
# 方式一：环境变量（临时，当前终端有效）
export OMO_SEND_ANONYMOUS_TELEMETRY=0

# 方式二：环境变量（永久，写入 .bashrc）
echo 'export OMO_SEND_ANONYMOUS_TELEMETRY=0' >> ~/.bashrc
```

### Q8: 如何查看当前各 Agent 使用的模型？

查看配置文件 `~/.config/opencode/oh-my-openagent.jsonc` 中的 `agents` 部分。未显式配置的 Agent 使用内置默认回退链。

### Q9: npm install oh-my-opencode 卡住或超时

**原因**：安装包较大（含多平台二进制），网络慢时容易超时。  
**解决**：

```bash
# 使用国内镜像加速
npm install oh-my-opencode --save --registry=https://registry.npmmirror.com

# 或增加超时时间
npm install oh-my-opencode --save --fetch-timeout=300000
```

---

## 附录：完整配置示例

`~/.config/opencode/oh-my-openagent.jsonc`：

```jsonc
{
  // 注意：$schema 是可选的，如果路径不正确请删除
  // "$schema": "~/tools/oh-my-opencode/node_modules/oh-my-opencode/dist/oh-my-opencode.schema.json",

  "agents": {
    "sisyphus": {
      "model": "opencode-go/kimi-k2.6"
    },
    "prometheus": {
      "model": "opencode-go/kimi-k2.6"
    },
    "hephaestus": {
      "model": "opencode-go/deepseek-v4-pro"
    },
    "momus": {
      "model": "opencode-go/deepseek-v4-pro"
    }
  },

  "background_agents": {
    "max_concurrent": 5
  },

  "mcps": {
    "websearch": { "enabled": true },
    "context7": { "enabled": true },
    "grep_app": { "enabled": true }
  },

  "lsp": {
    "enabled": true
  },

  "experimental": {
    "aggressive_truncation": false,
    "auto_resume": true
  }
}
```

---

## 附录：目录结构参考

安装完成后，你的目录结构应类似：

```
~
├── .config/
│   └── opencode/
│       ├── opencode.json          # OpenCode 主配置
│       └── oh-my-openagent.jsonc  # 插件配置
├── .local/
│   └── bin/
│       └── gh                     # GitHub CLI
├── tools/
│   └── oh-my-opencode/            # oh-my-opencode 安装目录
│       ├── node_modules/
│       │   ├── oh-my-opencode/    # 主包
│       │   ├── typescript-language-server/
│       │   ├── pyright/
│       │   └── ...               # 其他 LSP 服务器
│       └── package.json
└── Projects/
    └── oh-my-opencode-guide/      # （可选）本文档仓库
```

---

> **维护者**：FengWatermelon  
> **基于**：oh-my-opencode v4.0.0 + OpenCode Go v1.14.20  
> **日期**：2026-05-08
> 
> **使用 review-work skill 审查**：本文档已通过 5-Agent 并行 review。
