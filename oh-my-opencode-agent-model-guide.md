# Oh My OpenCode Agent 模型分配指南

> 本文档详细介绍 OpenCode Go 各模型的能力特点，以及在不同工作场景下如何为 oh-my-opencode 的 Agent 分配最优模型。
>
> **适用版本**：oh-my-opencode v4.0.0+
> **目标读者**：希望根据工作类型（通用开发 / 深度代码分析 / 架构设计）优化 Agent 模型配置的用户

---

## 目录

1. [OpenCode Go 模型概览](#1-opencode-go-模型概览)
2. [模型能力分档](#2-模型能力分档)
3. [Agent 角色详解](#3-agent-角色详解)
4. [场景化配置策略](#4-场景化配置策略)
   - 4.1 [通用开发场景](#41-通用开发场景)
   - 4.2 [代码分析与修改场景](#42-代码分析与修改场景)
   - 4.3 [架构设计与规划场景](#43-架构设计与规划场景)
5. [完整配置示例](#5-完整配置示例)
6. [配置验证](#6-配置验证)
7. [模型选择速查表](#7-模型选择速查表)

---

## 1. OpenCode Go 模型概览

运行以下命令查看你当前可用的所有模型：

```bash
opencode models
```

**OpenCode Go 提供的模型：**

```
opencode-go/deepseek-v4-flash    # 轻量版 DeepSeek，快速响应
opencode-go/deepseek-v4-pro      # DeepSeek 旗舰版，代码能力最强
opencode-go/glm-5                # GLM 基础版
opencode-go/glm-5.1              # GLM 增强版，复杂逻辑推理最强
opencode-go/kimi-k2.5            # Kimi 上一代
opencode-go/kimi-k2.6            # Kimi 旗舰版，综合能力最强
opencode-go/mimo-v2.5            # Mimo 基础版，轻量快速
opencode-go/mimo-v2.5-pro        # Mimo 增强版
opencode-go/minimax-m2.5         # MiniMax 基础版
opencode-go/minimax-m2.7         # MiniMax 多模态版
opencode-go/qwen3.5-plus         # 通义千问上一代
opencode-go/qwen3.6-plus         # 通义千问最新版，代码均衡型
```

> **注意**：模型列表会随 OpenCode Go 更新而变化，请以 `opencode models` 实际输出为准。

---

## 2. 模型能力分档

根据实际使用经验和 oh-my-opencode 内置回退链设计，OpenCode Go 模型可分为三档：

### 第一梯队：最强推理（旗舰级）

| 模型 | 核心优势 | 适用场景 |
|------|----------|----------|
| **`opencode-go/kimi-k2.6`** | 综合能力最强，全局理解、判断、调度一流 | 主调度器、规划师、审查员 |
| **`opencode-go/glm-5.1`** | 复杂逻辑推理最强，抽象思维、数学、架构分析 | 架构咨询、复杂 Debug、逻辑分析 |
| **`opencode-go/deepseek-v4-pro`** | 代码理解最深，长上下文、端到端代码执行最强 | 代码分析、重构、编写、深度执行 |

**这三者是 oh-my-opencode 内置回退链中优先级最高的模型。**

### 第二梯队：强推理（均衡型）

| 模型 | 核心优势 | 适用场景 |
|------|----------|----------|
| **`opencode-go/qwen3.6-plus`** | 代码+推理均衡，性价比最优 | 中等复杂度编程、代码检索、快速执行 |
| **`opencode-go/minimax-m2.7`** | 多模态强模型，跨模态推理 | 需要理解图片/设计稿的代码任务 |
| **`opencode-go/mimo-v2.5-pro`** | 快速但可靠，响应速度优先 | 对延迟敏感的推理任务 |

### 第三梯队：轻量/快速（效率型）

| 模型 | 核心优势 | 适用场景 |
|------|----------|----------|
| `opencode-go/deepseek-v4-flash` | DeepSeek 轻量版，速度快 | 简单代码任务、快速响应 |
| `opencode-go/mimo-v2.5` | 极速响应，成本最低 | 工具调用、搜索、简单问答 |
| `opencode-go/qwen3.5-plus` | 上一代均衡模型 | 普通编程任务、向后兼容 |

> **成本与质量的权衡**：第一梯队模型能力强但调用成本/延迟较高；第三梯队便宜快速但深度不足。建议**核心 Agent 用第一梯队，辅助 Agent 用第二/三梯队**。

---

## 3. Agent 角色详解

oh-my-opencode 内置 11 个可覆盖模型的 Agent，每个角色职责不同，对模型的能力需求也不同：

### 核心调度层

| Agent | 职责 | 对模型的核心要求 |
|-------|------|-----------------|
| **Sisyphus** | 主调度器，分配任务、跟踪进度、协调 Agent | 全局理解力、判断决策力、多任务调度 |
| **Atlas** | 快速执行者，处理中等复杂度任务 | 综合能力均衡、响应速度 |
| **Sisyphus-Junior** | 轻量执行者，单文件修改、简单任务 | 快速响应、准确执行 |

### 规划层

| Agent | 职责 | 对模型的核心要求 |
|-------|------|-----------------|
| **Prometheus** | 规划师，访谈需求、制定执行计划 | 需求理解、步骤拆解、风险评估 |
| **Metis** | 预规划顾问，识别模糊点、架构风险 | 批判性思维、风险识别、抽象分析 |
| **Momus** | 计划审查员，审查计划可执行性 | 全局视角、快速判断、不钻细节 |

### 执行层

| Agent | 职责 | 对模型的核心要求 |
|-------|------|-----------------|
| **Hephaestus** | 深度工作者，端到端独立执行复杂任务 | 长上下文、代码能力、自主探索 |

### 专家咨询层

| Agent | 职责 | 对模型的核心要求 |
|-------|------|-----------------|
| **Oracle** | 架构咨询、复杂 Debug、高难度问题分析 | 深度推理、根因分析、架构思维 |
| **Multimodal-Looker** | 分析图片、PDF、图表等多媒体文件 | 多模态理解、视觉分析 |

### 信息检索层

| Agent | 职责 | 对模型的核心要求 |
|-------|------|-----------------|
| **Librarian** | 检索文档、代码示例、最佳实践 | 信息检索、示例生成、知识调用 |
| **Explore** | 代码库搜索、模式发现、快速 grep | 工具调用为主，模型能力要求低 |

### 不可覆盖的 Agent

| Agent | 说明 |
|-------|------|
| **Build** | oh-my-opencode 执行工具调用的默认 Agent，模型由 OpenCode 本身控制 |

---

## 4. 场景化配置策略

根据你的主要工作类型，选择对应的配置方案。

### 4.1 通用开发场景

适合：日常开发、混合任务（写文档+写代码+查资料）、不确定任务类型时

**配置逻辑**：
- 调度/规划用 **kimi-k2.6**（综合能力最强，不会偏科）
- 执行用 **deepseek-v4-pro**（代码兜底）
- 其他默认即可

```jsonc
{
  "agents": {
    "sisyphus": { "model": "opencode-go/kimi-k2.6" },
    "prometheus": { "model": "opencode-go/kimi-k2.6" },
    "hephaestus": { "model": "opencode-go/deepseek-v4-pro" },
    "momus": { "model": "opencode-go/kimi-k2.6" }
  }
}
```

### 4.2 代码分析与修改场景（推荐）

适合：重构项目、审计代码、复杂 Bug 修复、大规模代码迁移

**配置逻辑**：

| Agent | 推荐模型 | 理由 |
|-------|----------|------|
| **Sisyphus** | `kimi-k2.6` | 调度需要全局视角，不只是代码 |
| **Prometheus** | `deepseek-v4-pro` | 规划师要分析代码结构，deepseek 代码理解最强 |
| **Hephaestus** | `deepseek-v4-pro` | 代码执行主力，必须用最强代码模型 |
| **Momus** | `kimi-k2.6` | 审查计划是否可行，需要全局判断而非代码细节 |
| **Metis** | `glm-5.1` | 预规划要识别架构风险，glm-5.1 逻辑推理最强 |
| **Oracle** | `glm-5.1` | 复杂 Debug 和架构咨询需要最强抽象推理 |
| **Librarian** | `qwen3.6-plus` | 检索代码示例，qwen 代码均衡且成本更低 |
| **Explore** | `qwen3.5-plus` | 工具调用为主，轻量模型足够 |
| **Atlas** | `qwen3.6-plus` | 中等任务执行，性价比最优 |
| **Sisyphus-Junior** | `qwen3.6-plus` | 简单代码修改，快速响应 |

**完整配置：**

```jsonc
{
  "agents": {
    // 核心调度层
    "sisyphus": { "model": "opencode-go/kimi-k2.6" },

    // 规划层
    "prometheus": { "model": "opencode-go/deepseek-v4-pro" },
    "metis": { "model": "opencode-go/glm-5.1" },
    "momus": { "model": "opencode-go/kimi-k2.6" },

    // 执行层
    "hephaestus": { "model": "opencode-go/deepseek-v4-pro" },
    "atlas": { "model": "opencode-go/qwen3.6-plus" },
    "sisyphus-junior": { "model": "opencode-go/qwen3.6-plus" },

    // 专家咨询层
    "oracle": { "model": "opencode-go/glm-5.1" },
    "multimodal-looker": { "model": "opencode-go/kimi-k2.6" },

    // 信息检索层
    "librarian": { "model": "opencode-go/qwen3.6-plus" },
    "explore": { "model": "opencode-go/qwen3.5-plus" }
  },

  "background_agents": { "max_concurrent": 5 },
  "mcps": {
    "websearch": { "enabled": true },
    "context7": { "enabled": true },
    "grep_app": { "enabled": true }
  },
  "lsp": { "enabled": true },
  "experimental": {
    "aggressive_truncation": false,
    "auto_resume": true
  }
}
```

### 4.3 架构设计与规划场景

适合：从零设计系统、技术选型、制定团队规范、复杂系统设计

**配置逻辑**：
- 架构相关 Agent 全部用 **glm-5.1**（逻辑推理最强）
- 调度仍用 **kimi-k2.6**（综合判断不偏科）
- 执行用 **deepseek-v4-pro**（代码实现兜底）

```jsonc
{
  "agents": {
    "sisyphus": { "model": "opencode-go/kimi-k2.6" },
    "prometheus": { "model": "opencode-go/glm-5.1" },
    "metis": { "model": "opencode-go/glm-5.1" },
    "momus": { "model": "opencode-go/glm-5.1" },
    "oracle": { "model": "opencode-go/glm-5.1" },
    "multimodal-looker": { "model": "opencode-go/kimi-k2.6" },
    "hephaestus": { "model": "opencode-go/deepseek-v4-pro" },
    "atlas": { "model": "opencode-go/qwen3.6-plus" },
    "librarian": { "model": "opencode-go/qwen3.6-plus" },
    "explore": { "model": "opencode-go/qwen3.5-plus" }
  }
}
```

---

## 5. 完整配置示例

将以下配置保存到 `~/.config/opencode/oh-my-openagent.jsonc`：

```jsonc
{
  // Oh My OpenAgent 插件配置 - 代码分析与修改场景优化版
  // 优化逻辑：
  // - 调度/审查：kimi-k2.6（全局视角、综合判断）
  // - 代码分析/执行：deepseek-v4-pro（代码理解、端到端执行）
  // - 架构/复杂逻辑：glm-5.1（抽象推理、根因分析）
  // - 检索/轻量任务：qwen3.6-plus / qwen3.5-plus（快速响应、节省成本）

  "agents": {
    // ===== 核心调度层 =====
    "sisyphus": {
      "model": "opencode-go/kimi-k2.6"
      // 主调度器：需要最强综合能力，理解业务逻辑+代码依赖+合理分配任务
    },

    // ===== 规划层 =====
    "prometheus": {
      "model": "opencode-go/deepseek-v4-pro"
      // 规划师：分析代码结构、识别修改范围、制定重构计划
    },
    "metis": {
      "model": "opencode-go/glm-5.1"
      // 预规划顾问：识别需求模糊点、架构风险
    },
    "momus": {
      "model": "opencode-go/kimi-k2.6"
      // 计划审查：检查代码修改计划是否可执行，需要全局视角
    },

    // ===== 执行层 =====
    "hephaestus": {
      "model": "opencode-go/deepseek-v4-pro"
      // 深度代码工作者：写代码、改代码、端到端执行
    },
    "atlas": {
      "model": "opencode-go/qwen3.6-plus"
      // 快速执行者：处理中等复杂度任务
    },
    "sisyphus-junior": {
      "model": "opencode-go/qwen3.6-plus"
      // 单文件修改、简单修错字等轻量代码任务
    },

    // ===== 专家咨询层 =====
    "oracle": {
      "model": "opencode-go/glm-5.1"
      // 架构咨询 + 复杂调试
    },

    // ===== 信息检索层 =====
    "librarian": {
      "model": "opencode-go/qwen3.6-plus"
      // 检索代码示例、API 文档
    },
    "explore": {
      "model": "opencode-go/qwen3.5-plus"
      // 快速 grep 代码库、找模式（工具调用为主）
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

> **注意**：以上配置中未包含 `$schema` 字段。如果你想启用 JSON Schema 验证，可以手动添加指向正确路径的 `$schema`，但这不是必需的。

---

## 6. 配置验证

### 6.1 运行诊断

```bash
npx oh-my-opencode doctor
```

**期望输出：**

```
 oMoMoMoMo Doctor

 ✓ System OK (opencode 1.14.20 · oh-my-openagent unknown)
```

### 6.2 如果看到警告

| 警告 | 原因 | 解决 |
|------|------|------|
| `Configured models rely on compatibility fallback` | 模型不在 oh-my-opencode 内置数据库中 | 不影响使用，只是缺少 guardrails 优化。可改为内置支持的模型消除警告 |
| `Model override uses unavailable provider` | 模型格式错误或模型已被移除 | 检查 `opencode models` 列表，使用存在的模型 |
| `GitHub CLI missing` | gh 未安装或未在 PATH | 参考安装指南安装 gh CLI |

### 6.3 验证模型是否生效

修改配置后，**需要重启 opencode** 才能使新配置生效：

```bash
# 如果已经在 opencode 中，先退出
exit

# 重新启动
opencode .
```

启动后，你可以通过以下方式验证模型配置是否生效：

1. **运行 doctor 再次检查**：`npx oh-my-opencode doctor` 应显示 `System OK`
2. **在对话中询问**："你当前使用的是什么模型？" Agent 会回答其配置
3. **观察行为差异**：如果之前配置的是轻量模型，现在换成了旗舰模型，你会感觉回答质量明显提升

---

## 7. 模型选择速查表

### 按任务类型选模型

| 你想做什么 | 选什么模型 | 分配给哪个 Agent |
|------------|-----------|-----------------|
| 调度整个团队 | `kimi-k2.6` | Sisyphus |
| 写代码/改代码 | `deepseek-v4-pro` | Hephaestus |
| 分析代码结构并制定计划 | `deepseek-v4-pro` | Prometheus |
| 审查计划是否靠谱 | `kimi-k2.6` | Momus |
| 解决复杂 Bug / 根因分析 | `glm-5.1` | Oracle |
| 做技术选型 / 架构设计 | `glm-5.1` | Metis / Oracle |
| 找代码示例 / 查 API 用法 | `qwen3.6-plus` | Librarian |
| 搜索代码库中的模式 | `qwen3.5-plus` | Explore |
| 快速改个文件 | `qwen3.6-plus` | Atlas / Sisyphus-Junior |

### 按能力需求选模型

| 你需要的能力 | 首选 | 备选 |
|-------------|------|------|
| 最强综合能力 | `kimi-k2.6` | `glm-5.1` |
| 最强代码能力 | `deepseek-v4-pro` | `qwen3.6-plus` |
| 最强逻辑推理 | `glm-5.1` | `kimi-k2.6` |
| 最快响应 | `mimo-v2.5` | `deepseek-v4-flash` |
| 最佳性价比 | `qwen3.6-plus` | `qwen3.5-plus` |
| 多模态（图片理解） | `minimax-m2.7` | `kimi-k2.6` |

---

## 附录：常见问题

### Q1: 修改配置后需要重启 opencode 吗？

**是的**，需要重启。

oh-my-opencode 的配置在启动时加载，修改 `~/.config/opencode/oh-my-openagent.jsonc` 后：

```bash
# 退出当前 opencode 会话
exit

# 重新启动
opencode .
```

### Q2: 如何恢复默认配置？

如果配置出错想恢复默认，有两种方式：

**方式一：清空 agents 配置（推荐）**

编辑 `~/.config/opencode/oh-my-openagent.jsonc`，将 `agents` 部分留空：

```jsonc
{
  "agents": {},
  "mcps": { ... },
  ...
}
```

这样所有 Agent 都会使用 oh-my-opencode 内置的默认回退链。

**方式二：备份并恢复**

```bash
# 如果你有备份
cp ~/.config/opencode/oh-my-openagent.jsonc.bak ~/.config/opencode/oh-my-openagent.jsonc

# 或者删除配置文件，oh-my-opencode 会使用完全默认配置
rm ~/.config/opencode/oh-my-openagent.jsonc
```

### Q3: 为什么配置了模型但感觉没变化？

可能原因：
1. **没有重启 opencode** — 修改配置后必须重启
2. **模型名称拼写错误** — 运行 `opencode models` 确认名称正确
3. **该 Agent 未被触发** — 某些 Agent（如 Oracle、Metis）只在特定场景下被调用
4. **后台任务模型不可覆盖** — 部分后台 Agent 使用固定模型

### Q4: 可以同时配置多个场景吗？

不能同时生效，但你可以维护多个配置文件按需切换：

```bash
# 创建不同场景的配置
cp ~/.config/opencode/oh-my-openagent.jsonc ~/.config/opencode/oh-my-openagent-code.jsonc
cp ~/.config/opencode/oh-my-openagent.jsonc ~/.config/opencode/oh-my-openagent-arch.jsonc

# 需要切换时
cp ~/.config/opencode/oh-my-openagent-code.jsonc ~/.config/opencode/oh-my-openagent.jsonc
# 然后重启 opencode
```

### Q5: Multimodal-Looker 是做什么的？什么时候用？

**Multimodal-Looker** 负责分析图片、PDF、图表等非文本文件。当你：
- 上传了一张架构图让 AI 分析
- 给了一个设计稿让 AI 实现
- 需要理解 PDF 文档中的图表

此时 Sisyphus 会调度 Multimodal-Looker。默认配置使用 `kimi-k2.6`（多模态能力强）。如果你主要处理文本，可以分配给轻量模型节省成本。

---

> **维护者**：FengWatermelon  
> **基于**：oh-my-opencode v4.0.0 + OpenCode Go v1.14.20  
> **日期**：2026-05-08
