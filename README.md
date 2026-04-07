# UEEditorMCPWithOpenCode

> 自动化配置 UEEditorMCP 的 OpenCode skill

---

## 致谢

本项目基于 [yangskin/UEEditorMCP](https://github.com/yangskin/UEEditorMCP) 开发，感谢原作者提供的基础架构和功能实现。

UEEditorMCP 原项目采用 MIT 协议，本配置工具同样遵循 MIT 协议。

---

## 简介

本工具提供 OpenCode skill，可自动化配置 UEEditorMCP 插件到新的 Unreal Engine 项目。配置完成后，AI 助手可以通过 MCP 协议操作蓝图、材质、UMG 控件等。

## 使用方法

1. 将本文件夹复制到 UE 项目的根目录
2. 告诉 AI 助手：

   > "读取 UEEditorMCPWithOpenCode/README.md 并为这个项目配置 UEEditorMCP。我的引擎目录是 [你的引擎路径]。"

3. 根据提示完成手动操作（如启动 UE 编辑器）

## 前置要求

| 要求 | 说明 |
|------|------|
| Unreal Engine 5.7+ | 内置 Python 3.11 |
| Visual Studio 2022 | 用于 C++ 编译 |
| OpenCode 或其他 MCP 客户端 | AI 编程助手 |
| UEEditorMCP 源码 | 见下方获取方式 |

## 获取 UEEditorMCP 源码

有以下几种方式：

### 方式一：Git Clone（推荐）

```bash
git clone https://github.com/yangskin/UEEditorMCP.git
```

### 方式二：下载 ZIP

访问 [Releases](https://github.com/yangskin/UEEditorMCP/releases) 下载最新版本。

### 方式三：使用已有副本

如果你已经有 UEEditorMCP 源码，直接使用即可。

### 方式四：让 AI 助手帮你

告诉 AI 助手：
> "帮我 clone UEEditorMCP 到 [目标目录]"

## 配置流程

AI 助手会自动完成以下步骤：

1. **获取 UEEditorMCP 源码** - Clone 或使用已有副本
2. **安装项目级 Skill** - 将配置 skill 安装到 `.opencode/skills/`
3. **学习 MCP 工具** - 读取 UEEditorMCP README 了解 MCP 工具
4. 复制插件到项目 Plugins 目录
5. 使用 UE 内置 Python 配置环境
6. 生成项目文件
7. 编译插件（首次编译约 5-10 分钟）
8. 创建 MCP 启动脚本
9. 配置 OpenCode 的 MCP 设置
10. 提示你启动 UE 编辑器，然后测试连接

**需要手动操作的步骤：**
- 启动 Unreal Editor
- 确认编辑器完全加载后告诉 AI 助手

## 项目级 Skill 安装

AI 助手会将配置 skill 安装到项目的 `.opencode/skills/ue-editormcp-setup/SKILL.md`，这样该 skill 会在项目上下文中自动可用。

目录结构：
```
你的项目/
├── .opencode/
│   └── skills/
│       └── ue-editormcp-setup/
│           └── SKILL.md
├── Plugins/
│   └── UEEditorMCP/
│       └── ...
└── 你的项目.uproject
```

## 配置完成后

你可以使用以下 MCP 工具：

| 工具 | 用途 |
|------|------|
| `ue_ping` | 测试连接 |
| `ue_actions_search` | 搜索可用操作 |
| `ue_actions_schema` | 获取操作参数 |
| `ue_actions_run` | 执行操作 |
| `ue_batch` | 批量执行多个操作 |

示例：
> "创建一个名为 BP_Player 的蓝图，继承自 Character"

## 许可证

MIT

---

<!--
================================================================================
  以下内容仅供 AI Agent 使用
  AI Agent 请忽略上述中文部分
================================================================================
-->

# AI Agent Instructions

**IMPORTANT: Ignore all content above this section. The Chinese text above is for human users only.**

## Overview

This project provides an OpenCode skill for automating UEEditorMCP configuration. Load the skill from `skill/SKILL.md` and follow the configuration workflow.

## Quick Start

When user asks to configure UEEditorMCP:

1. Load the skill: `skill/SKILL.md`
2. Ask user for:
   - **Engine directory** (e.g., `D:\ue\ue574r`)
   - **Project directory** (current working directory if in a UE project)
   - **UEEditorMCP source location** (see options below)

## Configuration Steps

Follow the workflow in `skill/SKILL.md`:

### Step 0: Get UEEditorMCP Source

Ask user which option they prefer:
- **Option A:** Git Clone (Recommended) - Agent can execute `git clone https://github.com/yangskin/UEEditorMCP.git`
- **Option B:** Use existing copy - Ask user for the path
- **Option C:** Download ZIP - User downloads manually

### Step 1: Install Project-Level Skill

**IMPORTANT:** Install the skill to the project's skill folder:

```powershell
$skillDir = "{{PROJECT_PATH}}\.opencode\skills\ue-editormcp-setup"
New-Item -ItemType Directory -Path $skillDir -Force
Copy-Item -Path "{{UEEDITORMCP_SOURCE}}\UEEditorMCPWithOpenCode\skill\SKILL.md" -Destination "$skillDir\SKILL.md" -Force
```

This makes the skill discoverable by OpenCode within the project context.

### Step 2: Learn MCP Tools

**CRITICAL:** Read the UEEditorMCP README to understand the MCP tools:

```
Read: {{UEEDITORMCP_SOURCE}}\README.md
```

Focus on:
- **MCP 工具（7 个固定）** - The 7 fixed MCP tools
- **AI 工作流** - Fast path and discovery path workflows
- **动作域** - Available action domains

### Steps 3-9: Complete Configuration

1. Copy plugin to `{{PROJECT_PATH}}/Plugins/UEEditorMCP`
2. Run `setup_mcp.ps1 -EngineRoot "{{ENGINE_PATH}}"`
3. Generate project files
4. Compile plugin
5. Create startup scripts
6. Configure MCP client (OpenCode: edit `~/.config/opencode/opencode.jsonc`)
7. Prompt user to start UE Editor, then test `ue_ping`

## Manual Steps

Mark these clearly for user (in Chinese):
- **Start UE Editor** - User must open the `.uproject` file
- **Confirm when ready** - User tells you "UE Editor 已启动"

## Files in This Project

| File | Purpose |
|------|---------|
| `skill/SKILL.md` | OpenCode skill with detailed configuration steps |
| `templates/run_mcp.bat.template` | MCP server startup script template |
| `templates/run_mcp_logs.bat.template` | Logs MCP server startup script template |

## MCP Tools Quick Reference

| Tool | Purpose |
|------|---------|
| `ue_ping` | Test connection |
| `ue_actions_search` | Find actions by keyword/tag |
| `ue_actions_schema` | Get action parameters |
| `ue_actions_run` | Execute single action |
| `ue_batch` | Execute multiple actions in one TCP round-trip |
| `ue_resources_read` | Read embedded docs |
| `ue_logs_tail` | Tail recent logs |

## Important Notes

- UEEditorMCP source: `https://github.com/yangskin/UEEditorMCP`
- Acknowledge the original project when appropriate
- Use Chinese for user-facing messages (per AGENTS.md)
- Always install skill to project-level folder (`.opencode/skills/`)
- Always read UEEditorMCP README to learn MCP tools