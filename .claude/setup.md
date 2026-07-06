# Claude Config — Setup Guide

## What This Is

Version-controlled Claude Code configuration for NBS development. Contains all agents, skills, codebase docs, and MCP setup. Stored in ADO as a standalone repo — pull it on any machine to get the full setup instantly.

## ADO Repo

**Repo:** `https://dev.azure.com/nbsdev/Services/_git/claude-config`

> First-time only: create this repo in ADO under the `Services` project, then push from the machine where the config was originally built. See "Initial Push" below.

---

## New Machine Setup

Run this once after cloning your dev repos:

```powershell
# From any directory
Invoke-Expression (Invoke-WebRequest -Uri "https://dev.azure.com/nbsdev/Services/_git/claude-config/raw/main/bootstrap-claude.ps1" -UseBasicParsing).Content
```

Or clone the repo and run locally:

```powershell
git clone https://nbsdev@dev.azure.com/nbsdev/Services/_git/claude-config
cd claude-config
.\bootstrap-claude.ps1
```

The script will:
1. Ask where your repos root is (default: `C:\neldevsrc\repos`)
2. Ask for your ADO PAT
3. Copy all config files to the right location
4. Inject the PAT into `.mcp.json`
5. Verify the MCP connection

---

## Initial Push (first time only)

From `C:\neldevsrc\repos` after all config files are in place:

```powershell
# Init a git repo scoped to just the Claude config files
cd "C:\neldevsrc\repos"
git init claude-config-tmp
cd claude-config-tmp

# Copy config into temp repo
Copy-Item ..\CLAUDE.md .
Copy-Item ..\bootstrap-claude.ps1 .
Copy-Item ..\mcp-template.json .
Copy-Item -Recurse ..\.claude .

# Commit
git add .
git commit -m "feat: initial NBS Claude Code configuration"

# Push to ADO
git remote add origin https://nbsdev@dev.azure.com/nbsdev/Services/_git/claude-config
git push -u origin main

cd ..
Remove-Item -Recurse -Force claude-config-tmp
```

---

## Keeping It Updated

After changing any agent, skill, or doc file:

```powershell
cd "C:\neldevsrc\repos"
.\update-claude-config.ps1
```

This stages all `.claude/` changes + `CLAUDE.md` + `bootstrap-claude.ps1`, commits, and pushes.

---

## What Is NOT in the Repo

| Excluded | Why |
|----------|-----|
| `.mcp.json` (with real PAT) | Contains credentials — use `mcp-template.json` instead |
| `.claude/stories/` | Per-session generated files, not shared config |
| `.claude/story-summary/` | Per-session generated files, not shared config |
| `.claude/settings.local.json` | Machine-local permission overrides |

---

## Updating Your PAT

If your PAT expires, re-run:

```powershell
.\bootstrap-claude.ps1 -PatOnly
```

This only updates the PAT in `.mcp.json` without copying files again.
