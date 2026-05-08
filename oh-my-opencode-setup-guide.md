# Oh My OpenCode 安装与配置指南（OpenCode Go 版）

> 本文档基于实际安装记录整理，适用于在 OpenCode Go 环境中安装和配置 oh-my-opencode 插件。

---

## 目录

1. [环境准备](#1-环境准备)
2. [安装 oh-my-opencode](#2-安装-oh-my-opencode)
3. [配置为 OpenCode 插件](#3-配置为-opencode-插件)
4. [安装辅助工具](#4-安装辅助工具)
   - 4.1 [安装 GitHub CLI](#41-安装-github-cli)
   - 4.2 [安装 LSP 服务器](#42-安装-lsp-服务器)
5. [配置模型](#5-配置模型)
6. [验证安装](#6-验证安装)
7. [核心功能与使用](#7-核心功能与使用)
8. [任务执行检查机制](#8-任务执行检查机制)
9. [常见问题](#9-常见问题)

---

## 1. 环境准备

### 检查现有环境

```bash
# 检查 OpenCode 是否已安装
opencode --version

# 检查 Node.js 和 npm
node --version
npm --version
```

**本文档环境示例：**
- OpenCode: `1.14.20`
- Node.js: `v25.9.0`
- npm: `11.13.0`
- 操作系统: Arch Linux

---

## 2. 安装 oh-my-opencode

### 2.1 初始化 npm 项目（如需要）

如果在用户主目录下没有 `package.json`，先初始化：

```bash
npm init -y
```

### 2.2 安装 oh-my-opencode

```bash
npm install oh-my-opencode --save
```

**安装位置：** `~/node_modules/oh-my-opencode/`

### 2.3 验证安装

```bash
npx oh-my-opencode --version
```

应输出类似：`4.0.0`

---

## 3. 配置为 OpenCode 插件

### 3.1 创建 OpenCode 主配置

编辑或创建 `~/.config/opencode/opencode.json`：

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["oh-my-openagent"]
}
```

> **注意**：插件在 npm 中的包名为 `oh-my-opencode`，但在 OpenCode 配置中使用兼容性名称 `oh-my-openagent`。两者为同一项目，过渡期同时支持。

### 3.2 创建插件配置文件

创建 `~/.config/opencode/oh-my-openagent.jsonc`（JSONC 格式支持注释）：

```jsonc
{
  // Oh My OpenAgent 插件配置
  // 详细文档: https://github.com/code-yeongyu/oh-my-openagent/blob/dev/docs/reference/configuration.md

  "$schema": "./node_modules/oh-my-opencode/dist/oh-my-opencode.schema.json",

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

---

## 4. 安装辅助工具

oh-my-opencode 依赖以下工具提供完整功能：

### 4.1 安装 GitHub CLI

#### 检查是否已安装

```bash
which gh
```

#### 安装方式（无 sudo 权限时）

如果无法使用系统包管理器（如 `pacman`、`apt`），可以手动安装到用户目录：

```bash
# 创建本地 bin 目录
mkdir -p ~/.local/bin

# 下载并安装 gh CLI
curl -sL https://github.com/cli/cli/releases/download/v2.72.0/gh_2.72.0_linux_amd64.tar.gz | tar -xz -C /tmp
cp /tmp/gh_2.72.0_linux_amd64/bin/gh ~/.local/bin/

# 验证
~/.local/bin/gh --version
```

#### 将 ~/.local/bin 加入 PATH

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

#### 登录 GitHub

```bash
gh auth login
```

按提示操作：
1. **Where do you use GitHub?** → 选择 `GitHub.com`
2. **Preferred protocol?** → 选择 `SSH`
3. **Generate SSH key?** → 选择 `Yes`
4. **Passphrase** → 直接回车（可选）
5. **Title for SSH key** → 回车使用默认 `GitHub CLI`
6. **Upload SSH public key?** → 选择你的公钥文件（如 `~/.ssh/id_ed25519.pub`）
7. **Authenticate GitHub CLI?** → 选择 `Login with a web browser`
8. 复制显示的验证码，浏览器中打开 `https://github.com/login/device`，粘贴验证码完成登录

#### 验证登录

```bash
gh auth status
```

### 4.2 安装 LSP 服务器

LSP（Language Server Protocol）为 AI Agent 提供 IDE 级别的代码操作能力。

#### 安装常用 LSP 服务器

```bash
npm install --save-dev \
  typescript-language-server \
  vscode-langservers-extracted \
  yaml-language-server \
  bash-language-server \
  pyright
```

#### 已安装的 LSP 工具列表

安装后，以下命令可在 `~/node_modules/.bin/` 中使用：

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

## 5. 配置模型

### 5.1 查看可用的 OpenCode Go 模型

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

### 5.2 配置 Agent 模型

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

---

## 6. 验证安装

### 6.1 运行诊断命令

```bash
npx oh-my-opencode doctor
```

**期望输出：**

```
 oMoMoMoMo Doctor

 ✓ System OK (opencode 1.14.20 · oh-my-openagent unknown)
```

### 6.2 如果仍有警告

- **GitHub CLI not authenticated** → 运行 `gh auth login`
- **No LSP servers detected** → 检查 LSP 是否安装，或运行 `npm install` 安装依赖
- **Model override uses unavailable provider** → 检查模型名称格式是否为 `"provider/model"`

---

## 7. 核心功能与使用

### 7.1 启动 OpenCode（自动加载插件）

```bash
# 进入项目目录
opencode .
```

### 7.2 核心命令

| 命令 | 作用 |
|------|------|
| `ultrawork` 或 `ulw` | 一键触发所有智能体，任务完成前绝不停止 |
| `/start-work` | 召唤 Prometheus 规划师，先访谈、再规划、最后执行 |
| `/init-deep` | 自动生成项目级 `AGENTS.md` 上下文文件 |
| `/ulw-loop` | 启动 Ralph Loop，自我引用闭环直到 100% 完成 |

### 7.3 类别路由（自动选择模型）

Sisyphus 会根据任务类型自动路由：

- `visual-engineering` → UI/UX、前端、设计
- `ultrabrain` → 复杂逻辑、架构决策
- `deep` → 深度调研与执行
- `quick` → 单文件修改、修错字

**无需手动选择，框架自动匹配。**

### 7.4 内置 MCP（已默认启用）

- **websearch** — 网络搜索（Exa）
- **context7** — 官方文档检索
- **grep_app** — GitHub 代码搜索

### 7.5 实用工作流示例

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

---

## 8. 任务执行检查机制

oh-my-opencode 设计了多层任务执行检查防线：

### 8.1 事前审查：Momus（计划审查专家）

- **职责**：在动手写代码**之前**，审查 Prometheus 生成的计划是否可执行
- **检查重点**：
  - 引用的文件是否真实存在
  - 任务描述是否足够清晰
  - 是否存在会阻塞工作的漏洞
- **原则**：有 80% 清晰度就通过，不过度追求完美

### 8.2 事中强制：Todo Continuation Enforcer（Hook）

- **机制**：系统级 Hook，在每个会话结束时自动触发
- **行为**：
  - 如果还有未完成的 todo，**强制要求 AI 继续执行**，不能结束会话
  - 最大停滞次数：`3`
  - 最大连续失败次数：`5`
- **作用**：防止 AI "摸鱼"或半途而废

### 8.3 事中监控：Sisyphus（主调度器）

- **Boulder State**：管理当前活跃的工作计划状态
- **Task Sessions**：为每个顶层任务维护可复用的子智能体会话
- **自动恢复**：从会话错误、上下文窗口超限、API 失败中自动恢复

### 8.4 事后验收：Review-Work Skill（5 智能体并行审查）

完成重要工作后，调用 `skill(name='review-work')` 会启动 **5 个并行子智能体**：

| # | 智能体 | 角色 | 审查重点 |
|---|--------|------|----------|
| 1 | Goal Verifier (Oracle) | 目标验证 | 我们是否做了用户要求的事？ |
| 2 | QA Executor | 动手 QA | 代码真的能跑吗？ |
| 3 | Code Reviewer (Oracle) | 代码质量 | 代码写得好吗？ |
| 4 | Security Auditor (Oracle) | 安全审计 | 有安全隐患吗？ |
| 5 | Context Miner | 上下文挖掘 | 是否遗漏了重要信息？ |

**规则**：5 个全部通过才算通过，**只要有一个失败，整个审查就失败**。

### 8.5 无限循环：Ralph Loop / `ulw-loop`

- **机制**：自我引用的开发闭环
- **行为**：AI 审查自己的工作 → 发现不足 → 继续改进 → 再审查 → 再改进……
- **终止条件**：达到 100% 完成度才停止

---

## 9. 常见问题

### Q1: `gh: command not found`

**原因**：`~/.local/bin` 不在 PATH 中。  
**解决**：

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### Q2: `gh auth login` 报错 `Post "https://fengwatermelon/login/device/code": EOF`

**原因**：选择了 `Other` 并填写了错误的主机名。  
**解决**：重新运行，选择 `GitHub.com`：

```bash
gh auth login
# 选择 GitHub.com，而非 Other
```

### Q3: `npx oh-my-opencode doctor` 提示模型配置错误

**原因**：模型名称格式不正确，缺少 provider 前缀。  
**解决**：使用 `"provider/model"` 格式：

```jsonc
// ❌ 错误
"model": "kimi-k2.6"

// ✅ 正确
"model": "opencode-go/kimi-k2.6"
```

### Q4: 如何卸载 oh-my-opencode？

```bash
# 1. 从 OpenCode 配置中移除插件
# 编辑 ~/.config/opencode/opencode.json，删除 "oh-my-openagent"

# 2. 删除插件配置文件
rm -f ~/.config/opencode/oh-my-openagent.jsonc

# 3. 卸载 npm 包
npm uninstall oh-my-opencode
```

### Q5: 如何查看当前各 Agent 使用的模型？

查看配置文件 `~/.config/opencode/oh-my-openagent.jsonc` 中的 `agents` 部分。未显式配置的 Agent 使用内置默认回退链。

Momus 的默认回退链（未覆盖时）：
1. GPT-5.5 (xhigh)
2. Claude Opus 4.7 (max)
3. Gemini 3.1 Pro (high)
4. GLM-5.1 (opencode-go)

---

## 附录：完整配置示例

`~/.config/opencode/oh-my-openagent.jsonc`：

```jsonc
{
  "$schema": "./node_modules/oh-my-opencode/dist/oh-my-opencode.schema.json",

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

> **维护者**：FengWatermelon  
> **基于**：oh-my-opencode v4.0.0 + OpenCode Go v1.14.20  
> **日期**：2026-05-08
