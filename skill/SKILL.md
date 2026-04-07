---
name: ue-editormcp-setup
description: Use when configuring UEEditorMCP plugin for a new Unreal Engine project with OpenCode or other MCP-compatible AI agents. Triggers when user wants to enable AI-assisted blueprint manipulation for their UE project.
---

# UEEditorMCP Setup

## Overview

UEEditorMCP is an MCP (Model Context Protocol) plugin for Unreal Engine 5.7+ that enables AI-assisted blueprint manipulation. This skill guides configuration for new UE projects.

**Core principle:** Follow steps in order. Pause for manual operations. Verify each stage before proceeding.

**Acknowledgment:** UEEditorMCP is based on [yangskin/UEEditorMCP](https://github.com/yangskin/UEEditorMCP) (MIT License).

## Prerequisites

Before starting, confirm:

| Requirement | How to Check |
|-------------|--------------|
| UE 5.7+ | Check engine version |
| VS 2022 | `cl` compiler available |
| MCP client | OpenCode installed |

Ask user for:
- **Engine directory** (e.g., `D:\ue\ue574r`)
- **Project directory** (e.g., `D:\ue\MyProject`)
- **Project .uproject file name**

## Configuration Workflow

```
Start
  │
  ├─► [0] Get UEEditorMCP Source ────────────► Auto or Manual
  │
  ├─► [1] Install Project-Level Skill ───────► Auto
  │
  ├─► [2] Learn MCP Tools ───────────────────► Auto (read README)
  │
  ├─► [3] Copy Plugin ──────────────────────► Auto
  │
  ├─► [4] Setup Python Environment ─────────► Auto
  │
  ├─► [5] Generate Project Files ───────────► Auto
  │
  ├─► [6] Compile Plugin ───────────────────► Auto (may take 5-10 min)
  │
  ├─► [7] Create Startup Scripts ───────────► Auto
  │
  ├─► [8] Configure MCP Client ─────────────► Auto
  │
  └─► [9] Test Connection ──────────────────► MANUAL: User starts UE Editor
                                              │
                                              └─► Agent tests ue_ping
```

### Step 0: Get UEEditorMCP Source

**Ask user which option they prefer:**

#### Option A: Git Clone (Recommended)

Agent can do this automatically:

```bash
git clone https://github.com/yangskin/UEEditorMCP.git {{TARGET_PATH}}
```

**Ask user:** "Where should I clone UEEditorMCP? Suggested: `{{PROJECT_PATH}}\Plugins\UEEditorMCP` or a shared location."

**Or clone directly to Plugins folder:**
```bash
git clone https://github.com/yangskin/UEEditorMCP.git {{PROJECT_PATH}}\Plugins\UEEditorMCP
```

If cloning directly to Plugins folder, **skip Step 3** (Copy Plugin).

#### Option B: Use Existing Copy

Ask user: "Do you have an existing UEEditorMCP source? If yes, provide the path."

If user has existing source, note the path as `{{UEEDITORMCP_SOURCE}}`.

#### Option C: Download ZIP

User downloads manually from:
- GitHub: `https://github.com/yangskin/UEEditorMCP`
- Releases: `https://github.com/yangskin/UEEditorMCP/releases`

**Verify:** `{{UEEDITORMCP_SOURCE}}\UEEditorMCP.uplugin` exists.

### Step 1: Install Project-Level Skill

Install the configuration skill to the project's skill folder so it's available in this project context:

```powershell
# Create project skill directory
$skillDir = "{{PROJECT_PATH}}\.opencode\skills\ue-editormcp-setup"
if (-not (Test-Path $skillDir)) { 
    New-Item -ItemType Directory -Path $skillDir -Force 
}

# Copy the skill file
Copy-Item -Path "{{UEEDITORMCP_SOURCE}}\UEEditorMCPWithOpenCode\skill\SKILL.md" -Destination "$skillDir\SKILL.md" -Force
```

**Verify:** `{{PROJECT_PATH}}\.opencode\skills\ue-editormcp-setup\SKILL.md` exists.

**Note:** This makes the skill discoverable by OpenCode within the project context.

### Step 2: Learn MCP Tools

**CRITICAL:** Read the UEEditorMCP README to understand the MCP tools and workflow:

```
Read: {{UEEDITORMCP_SOURCE}}\README.md
```

Focus on these sections:
- **MCP 工具（7 个固定）** - The 7 fixed MCP tools
- **AI 工作流** - Fast path and discovery path workflows
- **动作域** - Available action domains (blueprint, material, widget, etc.)

**Key knowledge for the agent:**

| Tool | Purpose |
|------|---------|
| `ue_ping` | Test connection |
| `ue_actions_search` | Find actions by keyword/tag |
| `ue_actions_schema` | Get action parameters |
| `ue_actions_run` | Execute single action |
| `ue_batch` | Execute multiple actions in one TCP round-trip |
| `ue_resources_read` | Read embedded docs |
| `ue_logs_tail` | Tail recent logs |

**Workflow:**
1. Discovery path: `ue_actions_search` → `ue_actions_schema` → `ue_actions_run`
2. Fast path (when action ID known): Direct `ue_batch` or `ue_actions_run`

### Step 3: Copy Plugin

Skip this step if UEEditorMCP was cloned directly to `Plugins\UEEditorMCP`.

Copy UEEditorMCP to the project's Plugins folder:

```powershell
# Create Plugins folder if not exists
$pluginsDir = "{{PROJECT_PATH}}\Plugins"
if (-not (Test-Path $pluginsDir)) { New-Item -ItemType Directory -Path $pluginsDir }

# Copy plugin
Copy-Item -Path "{{UEEDITORMCP_SOURCE}}" -Destination "$pluginsDir\UEEditorMCP" -Recurse -Force
```

**Verify:** `{{PROJECT_PATH}}\Plugins\UEEditorMCP\UEEditorMCP.uplugin` exists.

### Step 4: Setup Python Environment

Run the setup script using UE's built-in Python:

```powershell
cd {{PROJECT_PATH}}\Plugins\UEEditorMCP
.\setup_mcp.ps1 -EngineRoot "{{ENGINE_PATH}}"
```

**Expected output:**
- Python venv created at `Python\.venv`
- MCP package installed
- `.vscode\mcp.json` generated (can be ignored for OpenCode)

**Verify:** `{{PROJECT_PATH}}\Plugins\UEEditorMCP\Python\.venv\Scripts\python.exe` exists.

### Step 5: Generate Project Files

Generate UE project files to recognize the new plugin:

```batch
{{ENGINE_PATH}}\Engine\Build\BatchFiles\GenerateProjectFiles.bat -Project="{{PROJECT_PATH}}\{{PROJECT_NAME}}.uproject" -Game -Engine
```

**Expected output:** `Result: Succeeded`

**Verify:** Solution files updated.

### Step 6: Compile Plugin

Compile the Editor target (determine correct target name first):

```batch
# Check Source/*.Target.cs for target name
# Common patterns: ProjectEditor, LyraEditor, etc.

{{ENGINE_PATH}}\Engine\Build\BatchFiles\Build.bat {{EDITOR_TARGET}} Win64 Development "{{PROJECT_PATH}}\{{PROJECT_NAME}}.uproject" -waitmutex
```

**To find editor target:**
1. Check `{{PROJECT_PATH}}\Source\` for `*Editor.Target.cs`
2. The target name is the filename without `.Target.cs`

**Expected output:**
- `Compiling...` (many files)
- `Result: Succeeded`
- DLL at `Plugins\UEEditorMCP\Binaries\Win64\UnrealEditor-UEEditorMCP.dll`

**This step may take 5-10 minutes for first compile.**

### Step 7: Create Startup Scripts

Create two batch files for MCP server startup:

**run_mcp.bat:**
```batch
@echo off
set PYTHONPATH={{PROJECT_PATH}}\Plugins\UEEditorMCP\Python
{{PROJECT_PATH}}\Plugins\UEEditorMCP\Python\.venv\Scripts\python.exe -m ue_editor_mcp.server_unified
```

**run_mcp_logs.bat:**
```batch
@echo off
set PYTHONPATH={{PROJECT_PATH}}\Plugins\UEEditorMCP\Python
{{PROJECT_PATH}}\Plugins\UEEditorMCP\Python\.venv\Scripts\python.exe -m ue_editor_mcp.server_unreal_logs
```

**Verify:** Both files exist at `{{PROJECT_PATH}}\Plugins\UEEditorMCP\`.

### Step 8: Configure MCP Client

#### For OpenCode

Add to `~/.config/opencode/opencode.jsonc` in the `mcp` object:

```json
"ue-editor-mcp": {
  "type": "local",
  "command": [
    "cmd",
    "/c",
    "{{PROJECT_PATH}}\\Plugins\\UEEditorMCP\\run_mcp.bat"
  ],
  "enabled": true
},
"ue-editor-mcp-logs": {
  "type": "local",
  "command": [
    "cmd",
    "/c",
    "{{PROJECT_PATH}}\\Plugins\\UEEditorMCP\\run_mcp_logs.bat"
  ],
  "enabled": true
}
```

**Important:** Use double backslashes (`\\`) in JSON paths on Windows.

#### For VS Code Copilot

Create `.vscode/mcp.json` (already generated by setup script).

### Step 9: Test Connection

**MANUAL STEP - Tell user (in Chinese):**

> "配置完成！请启动 Unreal Editor（打开 `{{PROJECT_NAME}}.uproject`）。编辑器完全加载后，告诉我 'UE Editor 已启动'，我将测试 MCP 连接。"

**After user confirms:**

Test connection using `ue_ping`:
- Expected: `{"success": true, "pong": true}`
- If failed: Check UE Editor is running, TCP port 55558 not blocked

**On success, test action discovery:**
```
ue_actions_search(query="blueprint")
```
- Expected: List of blueprint-related actions

## Quick Reference

| Step | Command | Duration | Auto/Manual |
|------|---------|----------|-------------|
| 0. Get Source | `git clone` or existing | ~1 min | Auto or Manual |
| 1. Install Skill | Copy SKILL.md | ~5s | Auto |
| 2. Learn MCPs | Read README | ~10s | Auto |
| 3. Copy | `Copy-Item -Recurse` | ~10s | Auto |
| 4. Setup | `.\setup_mcp.ps1` | ~30s | Auto |
| 5. Generate | `GenerateProjectFiles.bat` | ~15s | Auto |
| 6. Compile | `Build.bat` | 5-10 min | Auto |
| 7. Scripts | Create 2 batch files | ~5s | Auto |
| 8. Configure | Edit opencode.jsonc | ~30s | Auto |
| 9. Test | Start UE + ue_ping | Manual | Manual |

## MCP Tools Reference

After configuration, these tools are available:

| Tool | Purpose | Example |
|------|---------|---------|
| `ue_ping` | Test connection | `ue_ping()` |
| `ue_actions_search` | Find actions | `ue_actions_search(query="blueprint")` |
| `ue_actions_schema` | Get parameters | `ue_actions_schema(action_id="blueprint.create")` |
| `ue_actions_run` | Execute action | `ue_actions_run(action_id="...", params={...})` |
| `ue_batch` | Batch execute | `ue_batch(actions=[...])` |
| `ue_resources_read` | Read docs | `ue_resources_read(name="conventions.md")` |
| `ue_logs_tail` | Tail logs | `ue_logs_tail(source="editor")` |

## Troubleshooting

### Build Fails: "Couldn't find target rules file"

**Cause:** Project files not generated or wrong target name.

**Fix:**
1. Run `GenerateProjectFiles.bat` first
2. Check `Source\*Editor.Target.cs` for correct target name

### MCP Connection Failed (pong: false)

**Cause:** UE Editor not running or TCP port blocked.

**Fix:**
1. Confirm UE Editor is fully loaded (not just splash screen)
2. Check firewall allows localhost port 55558
3. Verify `UnrealEditor-UEEditorMCP.dll` exists in `Binaries\Win64\`

### Python venv Creation Failed

**Cause:** Engine Python not found.

**Fix:**
1. Verify `{{ENGINE_PATH}}\Engine\Binaries\ThirdParty\Python3\Win64\python.exe` exists
2. Run with explicit engine root: `.\setup_mcp.ps1 -EngineRoot "path\to\engine"`

### Git Clone Failed

**Cause:** Network issue or git not installed.

**Fix:**
1. Check git is installed: `git --version`
2. Try manual download from `https://github.com/yangskin/UEEditorMCP`
3. Use existing copy if available

### Skill Not Found in OpenCode

**Cause:** Skill not installed to project-level folder.

**Fix:**
1. Verify `{{PROJECT_PATH}}\.opencode\skills\ue-editormcp-setup\SKILL.md` exists
2. Restart OpenCode session

## Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong target name | Build fails | Check `*Editor.Target.cs` |
| Paths not escaped | JSON parse error | Use `\\` in JSON strings |
| Starting test too early | pong: false | Wait for UE Editor fully loaded |
| Missing venv | MCP server won't start | Re-run setup_mcp.ps1 |
| Git not installed | Clone fails | Install git or download ZIP |
| Skill not installed | Agent doesn't know MCP tools | Run Step 1 |

## Red Flags - STOP and Ask User

- Engine path doesn't contain `Engine\Binaries\ThirdParty\Python3`
- No `.uproject` file found in project directory
- No `*Editor.Target.cs` in Source folder
- Build fails after multiple attempts
- MCP connection fails after UE Editor fully loaded

**These indicate missing prerequisites or configuration issues that need user intervention.**