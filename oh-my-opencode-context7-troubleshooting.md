# Oh My OpenCode Context7 MCP 错误排查与解决指南

> 本文档记录 oh-my-opencode 内置 MCP 服务器 **context7** 常见的连接错误及其解决方案。
>
> **适用版本**：oh-my-opencode v4.0.0+
> **涉及工具**：Context7 MCP、GitHub CLI、curl

---

## 目录

1. [错误现象](#1-错误现象)
2. [根本原因](#2-根本原因)
3. [解决方案](#3-解决方案)
   - 3.1 [获取 Context7 API Key](#31-获取-context7-api-key)
   - 3.2 [配置环境变量](#32-配置环境变量)
   - 3.3 [验证配置](#33-验证配置)
4. [替代方案](#4-替代方案)
5. [验证 Context7 是否正常工作](#5-验证-context7-是否正常工作)
6. [常见问题](#6-常见问题)

---

## 1. 错误现象

在使用 oh-my-opencode 时，你可能会在终端或日志中看到以下错误：

```
context7 SSE error: Non-200 status code (405)
```

或在运行 `npx oh-my-opencode doctor` 时看到 Context7 相关的连接警告。

### 进一步确认

运行以下命令测试 Context7 服务端的响应：

```bash
curl -I https://mcp.context7.com/mcp
```

**未认证时的返回：**

```
HTTP/1.1 405 Method Not Allowed
...
WWW-Authenticate: Bearer resource_metadata="https://mcp.context7.com/.well-known/oauth-protected-resource"
Allow: GET, POST, DELETE
```

关键信息：
- `405 Method Not Allowed` — 请求被拒绝
- `WWW-Authenticate: Bearer ...` — **服务现在要求 Bearer Token 认证**

---

## 2. 根本原因

Context7 已从**公开访问**升级为**需要 API Key 认证**。

oh-my-opencode 内置的 context7 MCP 配置逻辑如下（来自源码）：

```javascript
headers: process.env.CONTEXT7_API_KEY 
  ? { Authorization: `Bearer ${process.env.CONTEXT7_API_KEY}` } 
  : undefined
```

**结论**：
- 如果环境变量 `CONTEXT7_API_KEY` **未设置** → 请求不带认证头 → 服务端返回 405
- 如果环境变量 `CONTEXT7_API_KEY` **已设置** → 请求带 `Authorization: Bearer <token>` → 正常访问

---

## 3. 解决方案

### 3.1 获取 Context7 API Key

1. 访问 Context7 官网：https://context7.com
2. 注册/登录你的账号
3. 进入设置或 API 管理页面
4. 生成一个新的 API Key

**API Key 格式示例：**

```
ctx7sk-fcfce1f1-5da3-41fc-9b0b-d24c6ebb2a0d
```

> **安全提示**：API Key 相当于你的账号密码，不要分享给他人，不要提交到 Git 仓库。

---

### 3.2 配置环境变量

oh-my-opencode **内置支持**通过 `CONTEXT7_API_KEY` 环境变量自动传递认证信息。

#### 方式一：永久生效（推荐）

将环境变量写入 shell 配置文件：

```bash
echo 'export CONTEXT7_API_KEY="你的-api-key"' >> ~/.bashrc
source ~/.bashrc
```

**验证是否生效：**

```bash
echo $CONTEXT7_API_KEY
# 应输出你的 API Key
```

#### 方式二：临时生效（当前终端）

```bash
export CONTEXT7_API_KEY="你的-api-key"
```

此方式仅在当前终端窗口有效，关闭后失效。

#### 方式三：配置文件中直接写 headers（不推荐）

编辑 `~/.config/opencode/oh-my-openagent.jsonc`：

```jsonc
"mcps": {
  "context7": {
    "enabled": true,
    "headers": {
      "Authorization": "Bearer 你的-api-key"
    }
  }
}
```

> ⚠️ **不推荐**：配置文件中直接写 API Key 有泄露风险（如果误提交到 Git）。环境变量方式更安全。

---

### 3.3 验证配置

设置环境变量后，测试 Context7 连接：

```bash
# 测试认证是否通过
curl -H "Authorization: Bearer $CONTEXT7_API_KEY" https://mcp.context7.com/mcp
```

**成功时的返回（示例）：**

```json
{"jsonrpc":"2.0","error":{"code":-32000,"message":"Server does not support GET requests"},"id":null}
```

**注意**：`Server does not support GET requests` 是**正常的** —— MCP 协议使用 POST/SSE，不是 GET。看到这条说明**认证已通过**，405 错误已解决。

然后重启 opencode 使配置生效：

```bash
# 如果已在 opencode 中，先退出
exit

# 重新启动
opencode .
```

---

## 4. 替代方案

如果你**不需要**检索官方文档，可以直接禁用 Context7，不影响其他功能。

### 禁用 Context7

编辑 `~/.config/opencode/oh-my-openagent.jsonc`：

```jsonc
"mcps": {
  "websearch": {
    "enabled": true
  },
  "context7": {
    "enabled": false
  },
  "grep_app": {
    "enabled": true
  }
}
```

### oh-my-opencode 内置 MCP 对比

| MCP | 功能 | 是否需要认证 | 替代方案 |
|-----|------|-------------|----------|
| **websearch** | 网络搜索（Exa） | 否 | 无 |
| **context7** | 官方文档检索 | **是（API Key）** | websearch + grep_app |
| **grep_app** | GitHub 代码搜索 | 否 | 无 |

**建议**：
- 如果你经常查 npm/pip 包的官方文档 → **配置 Context7 API Key**
- 如果你主要用搜索引擎和 GitHub 代码 → **禁用 Context7** 即可

---

## 5. 验证 Context7 是否正常工作

配置完成后，通过以下方式验证：

### 5.1 运行 doctor

```bash
npx oh-my-opencode doctor
```

如果 Context7 正常工作，不应出现与 context7 相关的错误或警告。

### 5.2 在 opencode 中测试

启动 opencode 后，询问一个需要查文档的问题：

```
React useEffect 的 cleanup 函数最佳实践是什么？
```

如果 Context7 正常工作，Agent 会尝试调用 `context7_query-docs` 检索 React 官方文档。

### 5.3 查看 MCP 调用日志

在 opencode 对话中，观察是否出现 Context7 的工具调用：

```
→ context7_resolve-library-id("react")
→ context7_query-docs(libraryId: "...", query: "useEffect cleanup")
```

---

## 6. 常见问题

### Q1: 设置了 `CONTEXT7_API_KEY` 但还是报错？

**检查清单：**

1. **是否重启了 opencode？**
   ```bash
   exit
   opencode .
   ```

2. **环境变量是否生效？**
   ```bash
   echo $CONTEXT7_API_KEY
   # 应输出你的 API Key，不是空值
   ```

3. **Key 是否正确？**
   - 检查是否有额外空格或换行
   - 确认 Key 未过期或被撤销

4. **配置文件是否覆盖了 headers？**
   - 检查 `~/.config/opencode/oh-my-openagent.jsonc` 中是否手动设置了 `context7.headers`
   - 手动设置的 headers 可能与环境变量冲突

### Q2: 我没有 Context7 账号，必须注册吗？

**不是必须的。**

Context7 只是 oh-my-opencode 的**可选** MCP 服务器。如果你不需要检索官方文档，直接禁用它即可：

```jsonc
"context7": { "enabled": false }
```

`websearch`（网络搜索）和 `grep_app`（GitHub 代码搜索）已经能满足大部分信息检索需求。

### Q3: Context7 API Key 收费吗？

具体收费政策请参考 [Context7 官网](https://context7.com)。通常：
- 个人使用有一定免费额度
- 超出额度后按调用次数或 Token 计费

如果你担心费用，可以先禁用 Context7，需要时再启用。

### Q4: 如何查看当前启用了哪些 MCP？

查看你的 oh-my-opencode 配置文件：

```bash
cat ~/.config/opencode/oh-my-openagent.jsonc | grep -A5 '"mcps"'
```

或在 opencode 对话中询问：

```
当前启用了哪些 MCP 服务器？
```

### Q5: 如何彻底移除 Context7？

如果你不想看到任何与 Context7 相关的提示：

```bash
# 1. 禁用 MCP
# 编辑 ~/.config/opencode/oh-my-openagent.jsonc
# 将 context7 设置为 enabled: false

# 2. 删除环境变量（如果设置了）
sed -i '/CONTEXT7_API_KEY/d' ~/.bashrc
unset CONTEXT7_API_KEY
```

---

> **维护者**：FengWatermelon  
> **基于**：oh-my-opencode v4.0.0 + OpenCode Go v1.14.20  
> **日期**：2026-05-08
