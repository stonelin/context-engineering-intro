# 🚀 Claude Code 完全使用指南

使用 Claude Code 构建任何项目所需的一切知识！本指南将带你从安装开始，逐步深入高级上下文工程、子代理、钩子和并行代理工作流。

## 📋 前提条件

- 终端/命令行访问
- 已安装 Node.js（用于安装 Claude Code）
- GitHub 账号（用于 GitHub CLI 集成）
- 文本编辑器（推荐 VS Code）

## 🔧 安装

**macOS/Linux：**
```bash
npm install -g @anthropic-ai/claude-code
```

**Windows（推荐使用 WSL）：**
详细说明请参阅 [install_claude_code_windows.md](./install_claude_code_windows.md)

**验证安装：**
```bash
claude --version
```

---

## ✅ 技巧 1：创建和优化 CLAUDE.md 文件

设置上下文文件，Claude 会自动将其加载到每次对话中，包含项目特定的信息、命令和指南。

```bash
mkdir your-folder-name && cd your-folder-name
claude
```

使用内置命令：
```
/init
```

或者基于本仓库中的模板创建自己的 CLAUDE.md 文件。参见 `CLAUDE.md` 获取 Python 项目的示例结构，包括：
- 项目感知和上下文规则
- 代码结构指南
- 测试要求
- 任务完成工作流
- 风格约定
- 文档标准

### 高级提示技巧

**关键词强化**：Claude 对某些关键词会产生增强的响应行为（信息密度关键词）：
- **IMPORTANT**：强调不应被忽视的关键指令
- **Proactively**：鼓励 Claude 主动提出改进建议
- **Ultra-think**：可触发更深入的分析（谨慎使用）

**核心提示工程技巧**：
- 避免提示"生产就绪"代码——这通常会导致过度工程化
- 提示 Claude 编写脚本来验证其工作："实现后，创建一个验证脚本"
- 除非特别需要，否则避免向后兼容——Claude 倾向于不必要地保留旧代码
- 专注于清晰度和具体需求，而非模糊的质量描述

### 文件放置策略

Claude 会自动从多个位置读取 CLAUDE.md 文件：

```bash
# 仓库根目录（最常见）
./CLAUDE.md              # 提交到 git，与团队共享
./CLAUDE.local.md        # 仅本地使用，添加到 .gitignore

# 父目录（用于 monorepo）
root/CLAUDE.md           # 通用项目信息
root/frontend/CLAUDE.md  # 前端特定上下文
root/backend/CLAUDE.md   # 后端特定上下文

# 引用外部文件以增加灵活性
echo "Follow best practices in: ~/company/engineering-standards.md" > CLAUDE.md
```

**专业提示**：许多团队保持 CLAUDE.md 精简，引用共享标准文档。这样便于：
- 在不同 AI 编码助手之间切换
- 更新标准时无需修改每个项目
- 跨团队共享最佳实践

*注意：虽然 Claude Code 自动读取 CLAUDE.md，其他 AI 编码助手也可以使用类似的上下文文件（如 Cursor 的 .cursorrules）*

---

## ✅ 技巧 2：设置权限管理

配置工具白名单以简化开发流程，同时维护文件操作和系统命令的安全性。

**方法 1：交互式白名单**
当 Claude 请求权限时，为常用操作选择"始终允许"。

**方法 2：使用 /permissions 命令**
```
/permissions
```
然后添加：
- `Edit`（用于文件编辑）
- `Bash(git commit:*)`（用于 git 提交）
- `Bash(npm:*)`（用于 npm 命令）
- `Read`（用于读取文件）
- `Write`（用于创建文件）

**方法 3：创建项目设置文件**
创建 `.claude/settings.local.json`：
```json
{
  "allowedTools": [
    "Edit",
    "Read",
    "Write",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(npm:*)",
    "Bash(python:*)",
    "Bash(pytest:*)"
  ]
}
```

**安全最佳实践**：
- 永远不要允许 `Bash(rm -rf:*)` 或类似的破坏性命令
- 使用特定的命令模式，而非 `Bash(*)`
- 定期审查权限
- 为不同项目使用不同的权限集

*注意：所有 AI 编码助手都有权限管理——有些内置，有些需要对每个操作手动批准。*

---

## ✅ 技巧 3：掌握自定义斜杠命令

斜杠命令是将你自己的工作流添加到 Claude Code 的关键。它们位于 `.claude/commands/` 中，使你能够创建可重用的参数化工作流。

### 内置命令
- `/init` - 生成初始 CLAUDE.md
- `/permissions` - 管理工具权限
- `/clear` - 在任务之间清除上下文
- `/agents` - 管理子代理
- `/help` - 获取 Claude Code 帮助

### 自定义命令示例

**仓库分析**：
```
/primer
```
执行全面的仓库分析，让 Claude Code 了解你的代码库，以便你可以开始实现修复或新功能，且 Claude 拥有所有必要的上下文。

### 创建你自己的命令

1. 在 `.claude/commands/` 中创建 markdown 文件：
```markdown
# Command: analyze-performance

分析 $ARGUMENTS 中指定文件的性能。

## 步骤：
1. 读取路径为 $ARGUMENTS 的文件
2. 识别性能瓶颈
3. 建议优化方案
4. 创建基准测试脚本
```

2. 使用该命令：
```
/analyze-performance src/heavy-computation.js
```

命令可以使用 `$ARGUMENTS` 接收参数，并可以调用 Claude 的任何工具。

*注意：其他 AI 编码助手可以将这些命令作为常规提示使用——只需复制命令内容并粘贴你的参数。*

---

## ✅ 技巧 4：集成 MCP 服务器

将 Claude Code 连接到模型上下文协议（MCP）服务器以增强功能。在 [MCP 文档](https://docs.anthropic.com/en/docs/claude-code/mcp) 中了解更多。

**添加 Serena MCP 服务器** - 最强大的编码工具包：

确保先 [安装 uvx](https://docs.astral.sh/uv/getting-started/installation/#standalone-installer)。以下是在 Windows WSL 中的安装方式：
```bash
sudo snap install astral-uv --classic
```

然后使用以下命令添加 Serena：
```bash
# 安装 Serena 用于语义代码分析和编辑
claude mcp add serena -- uvx --from git+https://github.com/oraios/serena serena start-mcp-server --context ide-assistant --project $(pwd)
```

[Serena](https://github.com/oraios/serena) 将 Claude Code 转变为功能齐全的编码代理：
- 语义代码检索和分析
- 使用语言服务器协议（LSP）的高级编辑功能
- 支持 Python、TypeScript/JavaScript、PHP、Go、Rust、C/C++、Java
- 免费开源，替代订阅制编码助手

**管理 MCP 服务器：**
```bash
# 列出所有已配置的服务器
claude mcp list

# 获取特定服务器的详情
claude mcp get serena

# 移除服务器
claude mcp remove serena
```

**即将推出**：Archon V2（重大升级）- 面向 AI 编码助手的全面知识和任务管理骨架——首次实现真正的人机协作编码。

*注意：MCP 已集成到每个主流 AI 编码助手中，服务器的管理方式非常相似。*

---

## ✅ 技巧 5：使用示例进行上下文工程

将你的开发工作流从简单的提示转变为全面的上下文工程——为 AI 提供端到端实现所需的所有信息。

### 快速入门

PRP（产品需求提示）框架是一种简单的三步上下文工程策略：

```bash
# 1. 用示例和上下文定义你的需求
# 编辑 INITIAL.md 以包含示例代码和模式

# 2. 生成全面的 PRP
/generate-prp INITIAL.md

# 3. 执行 PRP 以实现你的功能
/execute-prp PRPs/your-feature-name.md
```

### 定义你的需求

你的 INITIAL.md 应始终包含：

```markdown
## FEATURE（功能）
构建用户认证系统

## EXAMPLES（示例）
- 认证流程：`examples/auth-flow.js`
- 类似的 API 端点：`src/api/users.js`
- 数据库模式模式：`src/models/base-model.js`
- 验证方式：`src/validators/user-validator.js`

## DOCUMENTATION（文档）
- JWT 库文档：https://github.com/auth0/node-jsonwebtoken
- 我们的 API 标准：`docs/api-guidelines.md`

## OTHER CONSIDERATIONS（其他考虑事项）
- 使用现有的错误处理模式
- 遵循我们的标准响应格式
- 包含速率限制
```

### 关键 PRP 策略

**示例**：最强大的工具——提供代码片段、类似功能和要遵循的模式

**验证门槛**：确保全面测试，反复迭代直到所有测试通过

**拒绝随意编码**：在执行 PRP 之前验证它们，执行后验证代码！

你提供的示例越具体，Claude 就越能匹配你现有的模式和风格。

*注意：上下文工程适用于任何 AI 编码助手——PRP 框架和示例驱动方法是通用原则。*

---

## ✅ 技巧 6：利用子代理处理专业任务

子代理是在独立上下文窗口中运作的专业 AI 助手，具有专注的专业知识。它们使 Claude 能够将特定任务委托给专家，提高质量和效率。

### 理解子代理

每个子代理：
- 拥有自己的上下文窗口（不会污染主对话）
- 使用专业的系统提示运作
- 可以限制使用特定工具
- 自主处理委托的任务

### 本仓库中的示例子代理

**文档管理器**（`.claude/agents/documentation-manager.md`）：
- 当代码更改时自动更新文档
- 确保 README 的准确性
- 维护 API 文档
- 创建迁移指南

**验证门槛**（`.claude/agents/validation-gates.md`）：
- 在更改后运行所有测试
- 反复修复直到测试通过
- 执行代码质量标准
- 测试未通过时永不标记任务为完成

### 创建你自己的子代理

1. 使用 `/agents` 命令或在 `.claude/agents/` 中创建文件：

```markdown
---
name: security-auditor
description: "安全专家。主动审查代码中的漏洞并提出改进建议。"
tools: Read, Grep, Glob
---

你是一个专注于识别和预防安全漏洞的安全审计专家...

## 核心职责
1. 审查代码中的 OWASP Top 10 漏洞
2. 检查暴露的密钥或凭据
3. 验证输入清理
4. 确保正确的认证/授权
...
```

### 子代理最佳实践

**1. 专注的专业知识**：每个子代理应有一个明确的专业领域

**2. 主动描述**：在描述中使用 "proactively" 以实现自动调用：
```yaml
description: "代码审查员。主动审查所有代码更改以确保质量。"
```

**3. 工具限制**：只给子代理它们需要的工具：
```yaml
tools: Read, Grep  # 仅审查的代理不需要写入权限
```

**4. 信息流设计**：理解信息如何从主代理 → 子代理 → 主代理流动。子代理的描述至关重要，因为它告诉你的主 Claude Code 代理何时以及如何使用它。在描述中包含清晰的指示，说明主代理应如何提示此子代理。

**5. 单次上下文**：子代理没有完整的对话历史——它们从主代理接收单个提示。在设计子代理时要考虑这一限制。

在 [子代理文档](https://docs.anthropic.com/en/docs/claude-code/sub-agents) 中了解更多。

*注意：虽然其他 AI 助手没有正式的子代理，但你可以通过创建专业提示和在不同对话上下文之间切换来实现类似的效果。*

---

## ✅ 技巧 7：使用钩子实现自动化

钩子通过用户定义的 Shell 命令提供对 Claude Code 行为的确定性控制，这些命令在预定义的生命周期事件中执行。

### 可用的钩子事件

Claude Code 提供了几个可以挂钩的预定义动作：
- **PreToolUse**：工具执行前（可以阻止操作）
- **PostToolUse**：工具成功完成后
- **UserPromptSubmit**：用户提交提示时
- **SubagentStop**：子代理完成任务时
- **Stop**：主代理完成响应时
- **SessionStart**：会话初始化时
- **PreCompact**：上下文压缩前
- **Notification**：系统通知期间

在 [钩子文档](https://docs.anthropic.com/en/docs/claude-code/hooks) 中了解更多。

### 示例钩子：工具使用日志

本仓库在 `.claude/hooks/` 中包含了一个简单的钩子示例：

**log-tool-usage.sh** - 记录所有工具使用情况以便追踪和调试：
```bash
#!/bin/bash
# 带时间戳的工具使用日志
# 创建 .claude/logs/tool-usage.log
# 无需外部依赖
```

### 设置钩子

1. **创建钩子脚本** 在 `.claude/hooks/` 中
2. **使其可执行**：`chmod +x your-hook.sh`
3. **添加到设置** 在 `.claude/settings.local.json` 中：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/log-tool-usage.sh"
          }
        ]
      }
    ]
  }
}
```

钩子确保某些动作始终发生，而不是依赖 AI 去记住——非常适合日志记录、安全验证和构建触发器。

*注意：其他 AI 助手没有钩子（虽然 Kiro 有！），我几乎可以保证其他助手很快也会支持。*

---

## ✅ 技巧 8：GitHub CLI 集成

设置 GitHub CLI 使 Claude 能够与 GitHub 交互，处理 Issues、Pull Requests 和仓库管理。

```bash
# 安装 GitHub CLI
# 访问：https://github.com/cli/cli#installation

# 认证
gh auth login

# 验证设置
gh repo list
```

### 自定义 GitHub 命令

使用 `/fix-github-issue` 命令进行自动修复：

```
/fix-github-issue 123
```

这将：
1. 获取 Issue 详情
2. 分析问题
3. 搜索相关代码
4. 实现修复
5. 运行测试
6. 创建 PR

*注意：GitHub CLI 适用于任何 AI 编码助手——只需安装它，AI 就可以使用 `gh` 命令与你的仓库交互。*

---

## ✅ 技巧 9：使用开发容器的安全 YOLO 模式

允许 Claude Code 执行任何操作，同时通过容器化维护安全性。这使得在不对主机造成破坏性行为的情况下实现快速开发。

**前提条件：**
- 安装 [Docker](https://www.docker.com/)
- VS Code（或兼容的编辑器）

**安全特性：**
- 带白名单的网络隔离
- 无法访问主机文件系统
- 受限的出站连接
- 安全的实验环境

**设置过程：**

1. **在 VS Code 中打开** 并按 `F1`
2. **选择** "Dev Containers: Reopen in Container"（在容器中重新打开）
3. **等待** 容器构建
4. **打开终端**（`Ctrl+J`）
5. **在容器中认证** Claude Code
6. **以 YOLO 模式运行**：
   ```bash
   claude --dangerously-skip-permissions
   ```

**为什么使用开发容器？**
- 安全地测试危险操作
- 实验系统更改
- 快速原型开发
- 一致的开发环境
- 不用担心损坏你的系统

---

## ✅ 技巧 10：使用 Git Worktree 进行并行开发

使用 Git worktree 使多个 Claude 实例同时处理独立任务，或自动化并行实现同一功能。

### 手动 Worktree 设置

```bash
# 为不同功能创建 worktree
git worktree add ../project-auth feature/auth
git worktree add ../project-api feature/api

# 在每个 worktree 中启动 Claude
cd ../project-auth && claude  # 终端 1
cd ../project-api && claude   # 终端 2
```

### 自动化并行代理

AI 编码助手是非确定性的。运行多次尝试可以增加成功概率并提供实现选项。

**设置并行 worktree：**
```bash
/prep-parallel user-system 3
```

**执行并行实现：**
1. 创建计划文件（`plan.md`）
2. 运行并行执行：

```bash
/execute-parallel user-system plan.md 3
```

**选择最佳实现：**
```bash
# 审查结果
cat trees/user-system-*/RESULTS.md

# 测试每个实现
cd trees/user-system-1 && npm test

# 合并最佳方案
git checkout main
git merge user-system-2
```

### 优势

- **无冲突**：每个实例在隔离环境中工作
- **多种方案**：比较不同的实现
- **质量门槛**：只考虑测试通过的实现
- **轻松集成**：合并最佳方案

---

## 🎯 快速命令参考

| 命令 | 用途 |
|------|------|
| `/init` | 生成初始 CLAUDE.md |
| `/permissions` | 管理工具权限 |
| `/clear` | 在任务之间清除上下文 |
| `/agents` | 创建和管理子代理 |
| `/primer` | 分析仓库结构 |
| `ESC` | 中断 Claude |
| `Shift+Tab` | 进入规划模式 |
| `/generate-prp INITIAL.md` | 创建实现蓝图 |
| `/execute-prp PRPs/feature.md` | 根据蓝图实现 |
| `/prep-parallel [feature] [count]` | 设置并行 worktree |
| `/execute-parallel [feature] [plan] [count]` | 运行并行实现 |
| `/fix-github-issue [number]` | 自动修复 GitHub Issue |

---

## 📚 附加资源

- [Claude Code 文档](https://docs.anthropic.com/en/docs/claude-code)
- [Claude Code 最佳实践](https://www.anthropic.com/engineering/claude-code-best-practices)
- [MCP 服务器库](https://github.com/modelcontextprotocol)

---

## 🚀 下一步

1. **从简单开始**：设置 CLAUDE.md 和基本权限
2. **添加斜杠命令**：为你的工作流创建自定义命令
3. **安装 MCP 服务器**：添加 Serena 以增强编码能力
4. **实现子代理**：为你的技术栈添加专业助手
5. **配置钩子**：自动化重复性任务
6. **尝试并行开发**：实验多种方案

记住：当你提供清晰的上下文、具体的示例和全面的验证时，Claude Code 最为强大。编码愉快！
