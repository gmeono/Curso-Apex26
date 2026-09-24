# Claude Code Setup Runbook — Oracle APEX 26.1 + APEXlang + SQLcl Projects + Git

> **Purpose:** This file is an execution runbook for the **Claude Code extension for VS Code**, running on Windows.
>
> Claude Code should use this document to configure a local Oracle APEX 26.1 development environment that supports:
>
> - VS Code + the Claude Code extension (already installed)
> - Git source control
> - Oracle SQLcl 26.1+
> - SQLcl Projects
> - Oracle APEX 26.1 APEXlang
> - Oracle APEX and Database agent skills
> - SQLcl MCP access from Claude Code
> - Editing and compiling PL/SQL/database objects
> - Editing, validating, and importing APEXlang applications
> - Building release artifacts for later TEST/PROD deployment

## Known starting environment

This setup is for **Windows only**.

Assume the following are already installed unless inspection proves otherwise:

- VS Code
- **The Claude Code extension for VS Code** — already installed and signed in. Do not attempt to install it, trigger its onboarding flow, or sign it in. Treat it exactly like the other already-installed tools below: verify it is active, don't reinstall or reconfigure it from scratch.
- Oracle SQL Developer for VS Code extension
- Java
- Oracle SQLcl

Java and SQLcl may be older versions and may need to be upgraded.

**Do not reinstall Java, SQLcl, or the Claude Code extension just because they already exist.** First determine:

1. The currently active version of each tool.
2. The exact executable Windows resolves from `PATH` (for Java and SQLcl).
3. Whether another version is already installed elsewhere.
4. Whether `JAVA_HOME` points to the same Java installation used by `java.exe`.
5. Whether VS Code needs to be reloaded (**Developer: Reload Window**) or restarted after any PATH, environment, or configuration change.

Prefer upgrading/repointing the existing Windows setup over creating unnecessary parallel installations.

### A note on the Claude Code extension vs. the standalone CLI

The Claude Code VS Code extension bundles its own private copy of the CLI for its chat panel only. **It does not add `claude` to your system PATH.** Several steps in this runbook use the VS Code integrated terminal for things the chat panel doesn't do directly (running `sql`, `git`, `npx`, or `claude mcp add`). Those steps need a **separate, standalone Claude Code CLI install**.

- Do not install the standalone CLI up front "just in case."
- Only install it when a specific terminal step in this runbook needs it and it isn't already present (see Phase 1).
- Installing the standalone CLI does not affect, reinstall, or reconfigure the VS Code extension — they are independent installs that share the same configuration files (`CLAUDE.md`, `.claude/settings.json`, `.mcp.json`, `~/.claude.json`).

## Known repository

This runbook does not hardcode a specific repository. Perform the complete setup inside whatever local Git repository is currently open in VS Code.

Unlike a from-scratch template, **do not assume the repository is empty**. It may already contain commits and files from other work. Phase 0 inspects the repository and adapts accordingly.

Values to discover (see Phase 0 for exact commands):

```text
PROJECT_ROOT
PROJECT_NAME
GITHUB_OWNER
GITHUB_REPOSITORY      (owner/repo)
GITHUB_REMOTE
GITHUB_REMOTE_URL
GIT_BRANCH_DEV          (permanent branch mapped to DEV — typically `desarrollo`)
GIT_BRANCH_TEST         (permanent branch mapped to PRUEBAS — typically `pruebas`)
GIT_BRANCH_PROD         (permanent branch mapped to PRODUCCIÓN — typically `main`)
```

If you already know these values, fill them in here before starting so Claude Code doesn't need to (re-)discover them:

```text
GITHUB_REPOSITORY=<owner>/<repo>
GITHUB_REMOTE=origin
GITHUB_REMOTE_URL=<https://github.com/... or git@github.com:...>
GIT_BRANCH_DEV=desarrollo
GIT_BRANCH_TEST=pruebas
GIT_BRANCH_PROD=main
```

This project uses **three permanent branches, mapped 1:1 to the three environments** — see `01-estrategia-git-sqlcl-projects.md` for the full team Git strategy this runbook assumes. In short: `desarrollo` = DEV, `pruebas` = PRUEBAS, `main` = PRODUCCIÓN. All work branches (`feature/*`, `bugfix/*`) are created from `desarrollo`, never from `pruebas` or `main` — including urgent fixes, which follow the cherry-pick exception documented in that file rather than being committed directly to `pruebas` or `main`.

Therefore:

- Do not clone another copy of anything.
- Do not create a nested repository.
- Do not run `git init` when the current directory already contains correct `.git` metadata.
- Use the open repository root as `PROJECT_ROOT`.
- Initialize SQLcl Projects directly in that repository root, using `PROJECT_NAME` (the repository's folder/repo name) unless the user selects another name.
- Add all generated project configuration and source files under this repository.
- Do not push or create commits unless the user explicitly requests it.

---

# 1. Instructions to Claude Code

You are responsible for performing as much of this setup as possible from the current repository and local machine, using the Claude Code extension's chat panel. Where a step needs the integrated terminal (see the note above), open it with `` Ctrl+` `` and confirm the standalone CLI is available first.

This machine runs **Windows**. Use **PowerShell** and Windows paths/commands throughout this runbook unless a specific tool has its own interactive command language. Do not provide macOS/Linux installation steps or shell commands such as `brew`, `apt`, `bash`, `export`, or Unix-only path conventions.

## 1.1 Do not blindly reinstall or overwrite

Before making changes:

1. Inspect the current environment.
2. Detect tools and configuration that already exist.
3. Reuse valid installations and configuration where possible.
4. Do not overwrite existing configuration without inspecting it first.
5. Do not delete or replace an existing Git repository, SQLcl Project, `.mcp.json`, `CLAUDE.md`, `.claude/` directory, or `.dbtools/` configuration without explicit approval.
6. Do not assume the Claude Code extension needs installing, configuring, or signing in — it is already installed and authenticated. Do not attempt to install it via the Extensions view or trigger its onboarding flow.

## 1.2 Manual checkpoint rule

When a step requires something the user must do manually, **stop at that step and ask the user to perform it**.

Examples include:

- Administrator/elevated Windows permissions are required.
- A GUI action in VS Code, the Claude Code panel, or Oracle APEX is required and cannot be performed safely from the chat panel or terminal.
- Database credentials are required.
- A password must be entered.
- A wallet must be downloaded or installed.
- A VPN must be connected.
- A firewall or network rule must be changed.
- The database host/service information is unknown.
- A VS Code extension must be installed manually because automated installation failed (this applies to the Oracle SQL Developer extension, not the already-installed Claude Code extension).
- VS Code Workspace Trust must be granted before the Claude Code extension activates, or before project-scoped `.claude/` configuration loads.
- A new MCP server entry in `.mcp.json` needs its one-time approval the first time Claude Code sees it.
- Claude Code or VS Code must be reloaded (**Developer: Reload Window**) or restarted.
- A destructive database operation is necessary.
- A production/test deployment is requested.

When stopping:

1. Explain exactly what is required.
2. Give the minimum steps the user must perform.
3. Tell the user what evidence or value to return.
4. Resume from the same checkpoint after the user responds.

Do not ask the user to perform tasks Claude Code can safely perform itself.

## 1.3 Credentials and secrets

Never:

- Put a database password in the repository.
- Put a password in `CLAUDE.md`.
- Put a password in `.mcp.json` or `.claude/settings.json`.
- Put passwords, wallets, API keys, or secrets in Git.
- Print passwords back to the user.
- Add a PROD password or unrestricted production connection for agent use.

When SQLcl asks for a password interactively, let the user enter it manually.

## 1.4 Database safety

The default agent-accessible database must be **DEV only**.

Before executing any of the following, ask for explicit approval:

- `DROP`
- `TRUNCATE`
- mass `DELETE`
- schema recreation
- destructive migration
- destructive `ALTER`
- deployment to TEST
- deployment to PROD

Never deploy to PROD as part of initial setup.

---

# 2. Target Architecture

The target environment is:

```text
VS Code
│
├── Claude Code extension (already installed)
│   ├── CLAUDE.md
│   ├── .mcp.json
│   └── .claude/
│       ├── settings.json
│       └── skills/
│
├── Oracle SQL Developer for VS Code
│   ├── database connections
│   ├── object browser
│   └── SQL worksheets
│
├── Git repository
│
├── SQLcl Project
│   ├── .dbtools/
│   ├── src/database/
│   ├── dist/
│   └── artifact/
│
├── APEXlang source
│   └── *.apx
│
└── SQLcl 26.1+
    ├── APEXlang compiler
    ├── SQLcl Projects
    └── MCP server
           │
           ▼
      Oracle DEV database
          │
          └── APEX 26.1
```

Claude Code should be able to:

```text
Natural language request
        │
        ▼
   Claude Code
      /   \
     /     \
 APEXlang  PL/SQL / SQL
    │          │
    ▼          ▼
apex validate  SQLcl compile
    │          │
    ▼          ▼
apex import   USER_ERRORS
      \        /
       \      /
        DEV DB
```

---

# 3. Values Claude Code Must Discover or Request

Do not ask for these immediately if they can be discovered.

Determine the following values during setup:

```text
PROJECT_ROOT
PROJECT_NAME
GITHUB_OWNER
GITHUB_REPOSITORY
GITHUB_REMOTE
GITHUB_REMOTE_URL
GIT_BRANCH_DEV
GIT_BRANCH_TEST
GIT_BRANCH_PROD
DEV_SCHEMA
DEV_CONNECTION_NAME
APEX_APPLICATION_ID
APEX_WORKSPACE
SQLCL_PATH
JAVA_PATH
TNS_ADMIN                # only if applicable
CLAUDE_CLI_INSTALLED     # whether the standalone CLI is present, for terminal steps
```

Suggested defaults — confirm against the actual repository and connection, do not assume:

```text
DEV_CONNECTION_NAME=apex-dev
GIT_BRANCH_DEV=desarrollo    # confirm; do not rename an existing branch that already plays this role
GIT_BRANCH_TEST=pruebas
GIT_BRANCH_PROD=main
```

Claude Code must still inspect the local checkout and confirm these values before modifying it.

## Required user information

If Claude Code cannot discover them, ask for:

1. DEV parsing schema.
2. Database connection information.
3. APEX application ID if configuring an existing application.
4. APEX workspace name if required.
5. Wallet/TNS location if using Autonomous Database or a wallet-based connection.
6. Repository identity, if it can't be discovered from the open folder (GitHub owner/repo, remote URL).

Do not request PROD credentials.

---

# 4. Phase 0 — Verify the Existing Checkout

The complete setup must be performed in whatever repository is currently open in VS Code. This runbook does not hardcode a repository name — discover it here.

## 4.1 Confirm the open folder

Open the integrated terminal (`` Ctrl+` ``) — this requires the standalone CLI only if you also intend to run `claude` commands there; plain `git`/PowerShell commands work regardless. Run:

```powershell
Get-Location
Get-ChildItem -Force
git rev-parse --show-toplevel
git remote -v
git branch --show-current
git branch -a
```

Set:

```text
PROJECT_ROOT   = repository root from `git rev-parse --show-toplevel`
PROJECT_NAME   = folder name of PROJECT_ROOT
GITHUB_REMOTE  = usually `origin`
GITHUB_REMOTE_URL, GITHUB_OWNER, GITHUB_REPOSITORY = parsed from the remote URL (HTTPS or SSH form both acceptable)
GIT_BRANCH_DEV / GIT_BRANCH_TEST / GIT_BRANCH_PROD = confirm whether `desarrollo`, `pruebas`, and `main` already exist locally/remotely (`git branch -a`); if the repository has no commits yet, these need to be created (see 4.3)
```

If the current folder is not the root of the intended checkout, move to the directory `git rev-parse --show-toplevel` reports before continuing.

If `git rev-parse` fails, stop and ask the user to confirm whether VS Code opened the actual cloned repository directory, and whether a repository needs to be cloned or initialized first.

Do not run `git init` until the mismatch is understood.

## 4.2 Verify Git repository and remote

Confirm a remote exists and note it. This runbook does not assume a specific owner or repository name, so if `origin` is missing entirely, or if there's uncertainty about whether this is the correct repository for the project, stop and ask the user to confirm rather than guessing.

Confirm:

```text
Repository: <GITHUB_OWNER>/<GITHUB_REPOSITORY>
Branches: <GIT_BRANCH_DEV> / <GIT_BRANCH_TEST> / <GIT_BRANCH_PROD>
Remote name: <GITHUB_REMOTE>
```

Do not automatically replace an existing nonmatching remote without explicit approval.

## 4.3 The repository's state is unknown — inspect before assuming

Unlike a guaranteed-empty template repository, this repository may already contain commits, files, or its own conventions. Run:

```powershell
git log --oneline -5
git status
```

- **If there are no commits and no tracked files**, treat setup like a fresh, empty-repository case: no existing conventions to conflict with.
- **If files already exist**, inspect them before adding anything. Look specifically for an existing `.gitignore`, `CLAUDE.md`, `.mcp.json`, `.claude/`, `.dbtools/`, `src/`, `dist/`, `artifact/`, and `scripts/`. Merge with what's already there rather than overwriting it (see §1.1).
- **If the repository has substantial unrelated history or content**, stop and confirm with the user that this is in fact the intended repository for this Oracle APEX project before adding any APEX/SQLcl scaffolding.

If the repository has no commits yet, this is the point to name the local branch `desarrollo` — the first of the project's three permanent branches (`desarrollo`, `pruebas`, `main`; see `01-estrategia-git-sqlcl-projects.md`):

```powershell
git branch -M desarrollo
```

Only run this when no existing branch history would be affected. `pruebas` and `main` are created from `desarrollo` once it has an initial commit — see §8.2.

If `desarrollo`, `pruebas`, and `main` already exist (or the team uses different names for the same three roles), do not rename or restructure them just to match this convention — confirm with the user instead.

Do not create the initial commit or push to the remote unless the user explicitly asks for that action.

## 4.4 Check for existing setup files

Check whether these already exist:

```text
.git/
.gitignore
CLAUDE.md
.mcp.json
.claude/
.dbtools/
src/
dist/
artifact/
scripts/
CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md
```

If any exist, inspect them before continuing.

Do not overwrite existing configuration without reviewing it first.

## 4.5 Repository-root rule

All project components must be created relative to `PROJECT_ROOT`.

Expected high-level result:

```text
<repo-root>/
├── .git/
├── .gitignore
├── CLAUDE.md
├── CLAUDE_CODE_ORACLE_APEX_26_1_SETUP.md
├── .mcp.json
├── .claude/
│   ├── settings.json
│   └── skills/
├── .dbtools/
├── src/
│   └── database/
├── dist/
├── artifact/
└── scripts/
```

Do not create a nested layout such as:

```text
<repo-root>/
└── <repo-root-again>/
    └── .git/
```

or an unrelated subproject layout such as:

```text
<repo-root>/
└── some-other-project-name/
    └── .dbtools/
```

unless the user explicitly decides to make this repository a multi-project monorepo.

For the initial setup, the SQLcl Project belongs at the repository root.

---

# 5. Phase 1 — Windows Preflight and Version Discovery

The user already has Java, SQLcl, VS Code, and the Claude Code extension installed. **Inspect them before deciding whether any update is required.**

Run all operating-system commands from **PowerShell**.

## 5.1 Basic tool versions

Run:

```powershell
code --version
code --list-extensions
git --version
java -version
sql -version
node --version
npm --version
npx --version
```

Confirm the Claude Code extension is present and active:

```powershell
code --list-extensions | Select-String -Pattern "anthropic.claude-code"
```

If it does not appear, **stop — do not install it.** This runbook assumes it's already installed per "Known starting environment." Ask the user to confirm the extension is enabled (it may be disabled rather than absent) before proceeding.

Also determine whether the **standalone CLI** is available, since some steps below (SQLcl checks from the terminal, `claude mcp add`, `npx skills add`) use the integrated terminal rather than the chat panel:

```powershell
claude --version
```

The extension does not put `claude` on PATH, so this is expected to fail on a machine where only the extension is installed. That is not a problem by itself — install the standalone CLI only when a specific later step needs the terminal and this check fails (see 5.1.1).

A command that is missing is not automatically an installation failure. First check whether the executable exists but is not currently on `PATH`.

### 5.1.1 Installing the standalone CLI, only if a terminal step needs it

If a later phase's terminal command fails because `claude` is not found, and that step cannot be done through the chat panel or `/mcp` dialog instead:

```powershell
irm https://claude.ai/install.ps1 | iex
```

This installs the standalone CLI only. It does not reinstall, reconfigure, or otherwise affect the VS Code extension — the two share the same configuration files (`CLAUDE.md`, `.mcp.json`, `.claude/settings.json`, `~/.claude.json`). After installing, open a **new** integrated terminal and re-run `claude --version`.

## 5.2 Discover exactly which Java Windows is using

Run:

```powershell
Get-Command java -ErrorAction SilentlyContinue | Format-List *
where.exe java
$env:JAVA_HOME
```

If `JAVA_HOME` is populated, inspect it:

```powershell
$env:JAVA_HOME
Test-Path $env:JAVA_HOME
Get-ChildItem $env:JAVA_HOME -ErrorAction SilentlyContinue
```

Also inspect user and machine environment variables without modifying them:

```powershell
[Environment]::GetEnvironmentVariable("JAVA_HOME", "User")
[Environment]::GetEnvironmentVariable("JAVA_HOME", "Machine")
[Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::GetEnvironmentVariable("Path", "Machine")
```

Determine:

```text
ACTIVE_JAVA_VERSION
ACTIVE_JAVA_EXE
JAVA_HOME
```

Do not assume the first Java installation found on disk is the one SQLcl/VS Code is actually using.

## 5.3 Discover exactly which SQLcl Windows is using

Run:

```powershell
Get-Command sql -ErrorAction SilentlyContinue | Format-List *
where.exe sql
sql -version
```

Determine:

```text
ACTIVE_SQLCL_VERSION
SQLCL_PATH
SQLCL_BIN
```

The final MCP configuration must use the **absolute path to the intended `sql.exe`**, not merely rely on whatever happens to resolve from PATH.

If several SQLcl installations are found, identify which one is active and which one should remain after the upgrade.

## 5.4 Expected versions

### Oracle SQLcl

Use **SQLcl 26.1 or later** for this APEX 26.1 environment.

APEXlang support requires SQLcl 26.1+ when working with APEX 26.1.

If the installed SQLcl version is already 26.1 or newer and functions correctly, **do not upgrade it solely for this setup**.

If it is older than 26.1, upgrade it.

### Java

For this runbook, use a Java version supported by the installed SQLcl 26.1 release.

Prefer **Java 21** for a new or upgraded JDK unless the currently installed supported JDK is intentionally being retained.

If Java is already supported and current enough for SQLcl, do not replace it without reason.

### Git

Any current Git release is acceptable.

## 5.5 Windows restart/reload rule

After changing any of the following:

```text
PATH
JAVA_HOME
SQLcl installation location
Java installation location
.mcp.json MCP server executable path
```

remember that existing VS Code windows, the Claude Code extension's chat panel, and PowerShell processes may still have the old environment.

If verification from the current process cannot see the updated values, create a manual checkpoint and ask the user to:

1. Close/reopen the integrated terminal, or
2. Run **Developer: Reload Window** from the VS Code Command Palette, or restart VS Code entirely, as appropriate.

Then re-run the discovery commands before continuing.

---

# 6. Phase 2 — Update or Repair Windows Tooling

The user already has Java and SQLcl installed. The default action is therefore:

```text
inspect → compare versions → reuse or upgrade
```

not:

```text
reinstall everything
```

## 6.1 Git

If Git is available and reasonably current, leave it unchanged.

If Git is missing or broken:

1. Check whether Windows Package Manager is available:

```powershell
winget --version
```

2. Ask the user for permission before performing a system-level installation or upgrade.
3. Use a trusted official package source.
4. Verify:

```powershell
git --version
Get-Command git
```

If a terminal or VS Code restart is required, create a manual checkpoint.

---

## 6.2 Java — verify first, upgrade only if needed

Assume Java is already installed.

First run:

```powershell
java -version
Get-Command java | Format-List Source,Version,Path
where.exe java
$env:JAVA_HOME
```

### If Java is already suitable

If the installed Java version is supported by SQLcl 26.1 and `JAVA_HOME`/PATH resolve consistently, keep the existing installation.

Record:

```text
JAVA_PATH
JAVA_HOME
JAVA_VERSION
```

### If Java needs to be upgraded

Prefer installing/upgrading to **Java 21**.

Before a system-level installation or uninstall:

1. Tell the user what Java version is currently active.
2. Explain why an upgrade is required.
3. Ask for permission if administrator access or a system-wide installation is needed.

Do not remove an older JDK automatically. Other development tools may depend on it.

After installing the new JDK, configure the Windows environment so the intended Java is used.

Prefer **user-scoped** environment changes unless system scope is necessary.

Example inspection:

```powershell
[Environment]::GetEnvironmentVariable("JAVA_HOME", "User")
[Environment]::GetEnvironmentVariable("Path", "User")
```

When appropriate, set user `JAVA_HOME` with the actual JDK path:

```powershell
[Environment]::SetEnvironmentVariable(
    "JAVA_HOME",
    "C:\Program Files\Java\<JDK_DIRECTORY>",
    "User"
)
```

Do not copy this placeholder path literally. Discover the actual installed path first.

If PATH needs to reference Java, prefer `%JAVA_HOME%\bin` or the actual JDK `bin` path rather than accumulating multiple stale Java entries.

After the change, a new process may be required. Create a manual checkpoint if necessary and then verify in a **new PowerShell session**:

```powershell
java -version
Get-Command java | Format-List Source,Path
where.exe java
$env:JAVA_HOME
```

Do not continue until the active Java version and path are understood.

---

## 6.3 SQLcl — verify first, upgrade only if older than 26.1

Assume SQLcl is already installed.

First run:

```powershell
sql -version
Get-Command sql | Format-List Source,Version,Path
where.exe sql
```

### If SQLcl is already 26.1+

If the active SQLcl is 26.1 or later:

1. Keep it.
2. Record its absolute executable path.
3. Verify it starts normally:

```powershell
sql /nolog
```

Exit SQLcl after verification.

### If SQLcl is older than 26.1

Upgrade SQLcl.

Do **not** overwrite the currently working SQLcl directory until the new version has been extracted and verified.

Preferred Windows strategy:

1. Identify the existing SQLcl installation and active PATH entry.
2. Download the current Oracle SQLcl release from Oracle.
3. Extract the new version to a stable directory, for example:

```text
C:\tools\sqlcl
```

or a versioned directory such as:

```text
C:\tools\sqlcl-26.1
```

4. Verify the new executable **using its absolute path before changing PATH**:

```powershell
& "C:\tools\sqlcl\bin\sql.exe" -version
```

5. Verify Java compatibility:

```powershell
java -version
& "C:\tools\sqlcl\bin\sql.exe" -version
```

6. Only after verification, update the **user PATH** or the project MCP configuration to point to the intended SQLcl installation.

Do not modify machine-wide PATH without explicit approval.

Do not uninstall/delete the previous SQLcl installation until the new installation works from:

- PowerShell,
- VS Code,
- and the Claude Code SQLcl MCP server.

### PATH verification

After a PATH change, use a new PowerShell process and run:

```powershell
Get-Command sql | Format-List Source,Path
where.exe sql
sql -version
```

If `where.exe sql` returns multiple paths, ensure the intended 26.1+ installation appears first, or configure the SQLcl MCP entry in `.mcp.json` with the explicit absolute path.

The MCP configuration should always use the absolute path even if PATH is correct.

Oracle's official SQLcl installation/download documentation should be used when downloading or upgrading SQLcl.

If automated download, extraction, environment modification, or installation is blocked, stop and ask the user to perform only that manual step.

---

## 6.4 Node/npm/npx

Node is only required here when using npm/npx-based skill installation.

If Node/npm/npx are already available:

```powershell
node --version
npm --version
npx --version
```

leave them unchanged unless an actual compatibility problem occurs.

Do not upgrade Node merely because a newer version exists.

If Node is missing and the Oracle skill installation path selected later requires `npx`, explain why it is needed and ask permission before installing it.

---

## 6.5 Verify the final Windows toolchain

Before leaving this phase, run:

```powershell
git --version
java -version
sql -version
code --version
```

And capture resolved executables:

```powershell
Get-Command git | Select-Object Source
Get-Command java | Select-Object Source
Get-Command sql | Select-Object Source
Get-Command code | Select-Object Source
```

Record the final:

```text
JAVA_VERSION
JAVA_PATH
JAVA_HOME
SQLCL_VERSION
SQLCL_PATH
```

The `SQLCL_PATH` recorded here is the path that must later be used in `.mcp.json`.

---

# 7. Phase 3 — Verify and Use Oracle SQL Developer for VS Code

The user already has **Oracle SQL Developer for VS Code** installed and plans to use it as part of the development environment.

Treat it as a supported part of the workflow.

It complements Claude Code and SQLcl MCP; it does not replace them.

## 7.1 Intended responsibilities

Use Oracle SQL Developer for VS Code for developer-facing database work such as:

- browsing schemas and database objects,
- reviewing tables, views, packages, triggers, sequences, and other objects,
- opening SQL worksheets,
- manually running or testing SQL/PLSQL when useful,
- managing or inspecting Oracle database connections,
- reviewing compilation errors and object state interactively,
- inspecting database metadata during development,
- manually troubleshooting database connectivity,
- launching or interacting with SQLcl where supported by the extension.

Use **Claude Code + SQLcl MCP** for agent-driven execution such as:

- schema discovery requested by Claude Code,
- executing SQL/PLSQL,
- compiling repository package files,
- querying `USER_ERRORS`,
- running repository SQL scripts,
- validating database changes,
- APEXlang/SQLcl automation,
- repeatable development workflows.

The intended relationship is:

```text
VS Code
│
├── Claude Code extension
│   └── SQLcl MCP
│         └── agent-driven database operations
│
└── Oracle SQL Developer for VS Code
      └── human database browser / worksheet / connection tooling
```

Both may connect to the same DEV database, but Claude Code must continue to use the named DEV SQLcl connection and SQLcl MCP for automated database operations.

## 7.2 Verify the extension is installed

Run:

```powershell
code --list-extensions
```

Identify the installed **official Oracle SQL Developer for VS Code** extension.

Do not replace it with another Oracle/database extension.

If the extension is installed but disabled, ask the user before changing extension state.

If the command-line extension list cannot conclusively identify it, Claude Code may ask the user to confirm that Oracle SQL Developer for VS Code is visible and enabled in VS Code.

## 7.3 Reuse existing connections where practical

Before creating a duplicate DEV database connection:

1. Inspect whether the user already has the DEV database configured in Oracle SQL Developer for VS Code.
2. Determine whether the connection information can be reused for SQLcl.
3. Keep the connection name consistent where practical.

Preferred logical connection name:

```text
apex-dev
```

However, do not rename or delete an existing working connection merely to match this convention.

If the SQL Developer extension already has the correct DEV connection, use its connection details as the basis for the SQLcl named connection.

Passwords must still follow the credential rules in this runbook.

Do not extract, display, copy into repository files, or expose saved passwords from the VS Code extension.

## 7.4 Connection verification

The same DEV target should be identifiable from both:

```text
Oracle SQL Developer for VS Code
SQLcl / SQLcl MCP
```

Verify through SQLcl/MCP:

```sql
select user,
       sys_context('USERENV','DB_NAME') db_name,
       sys_context('USERENV','SERVICE_NAME') service_name
  from dual;
```

The user may also verify the connection manually in the SQL Developer extension.

If SQL Developer and SQLcl appear to connect to different databases or schemas, stop and resolve that discrepancy before continuing.

## 7.5 Do not make the extension a deployment dependency

The repository and deployment process must remain usable through:

```text
Git
SQLcl
SQLcl Projects
APEXlang
Claude Code + SQLcl MCP
```

Oracle SQL Developer for VS Code is an important development tool, but CI/CD or scripted deployments should not depend on a human opening the extension or clicking UI actions.

## 7.6 Human fallback

If a database operation cannot be performed safely or reliably through Claude Code/SQLcl MCP, the runbook may instruct the user to perform a specific verification or troubleshooting step using Oracle SQL Developer for VS Code.

When doing so:

1. State exactly which connection to use.
2. Give the SQL or navigation action required.
3. Ask the user to return the result.
4. Resume from the same checkpoint.

---

# 8. Phase 4 — Configure the Existing Git Repository

Do not run `git clone` and do not create a nested repository.

## 8.1 Verify repository state again

Run:

```powershell
git rev-parse --show-toplevel
git status
git remote -v
git branch --show-current
```

The repository root must match `PROJECT_ROOT`.

The expected remote is the one confirmed in Phase 0:

```text
<GITHUB_REMOTE>  <GITHUB_REMOTE_URL>
```

An equivalent SSH origin is acceptable.

## 8.2 Branch handling

If the repository has no commits yet, the local branch may be unborn. Ensure the intended branch name matches `GIT_BRANCH_DEV` (typically `desarrollo`):

```powershell
git branch -M desarrollo
```

Only do this if safe — i.e., no existing branch history would be affected. Do not rename an established branch just to standardize the name.

Do not create a commit merely to create the branch unless the user requests the initial commit.

Once `desarrollo` has at least one commit (see §8.4), create the other two permanent branches from it:

```powershell
git checkout -b pruebas
git push -u origin pruebas
git checkout -b main
git push -u origin main
git checkout desarrollo
```

These three branches are permanent and map 1:1 to the three environments (`desarrollo` → DEV, `pruebas` → PRUEBAS, `main` → PRODUCCIÓN) — see `01-estrategia-git-sqlcl-projects.md` for the full team Git strategy this runbook assumes. From this point on:

- `feature/*` and `bugfix/*` branches are always created from `desarrollo`, never from `pruebas` or `main`.
- `pruebas` and `main` only ever receive merges via Pull Request from the branch immediately below them — never a direct commit, not even for urgent fixes (those follow the cherry-pick exception documented in the Git strategy file).
- If the user has admin access to the repository, ask before configuring GitHub branch protection rules on `pruebas` and `main` (require PR review, restrict direct pushes) — this is a one-time GitHub settings change, not something to do silently.

If `desarrollo`, `pruebas`, and `main` already exist in the repository, do not recreate or restructure them — just confirm they match these roles.

## 8.3 Configure `.gitignore`

Inspect or create:

```text
.gitignore
```

At minimum, ensure local secrets and temporary files are excluded.

Recommended entries:

```gitignore
# OS / editor
.DS_Store
Thumbs.db

# Logs
*.log

# Local environment
.env
.env.*
!.env.example

# Oracle wallets / secrets
*.p12
*.sso
cwallet.sso
ewallet.p12
tnsnames.local.ora

# Local generated temporary files
tmp/
.temp/

# Claude Code personal/local files (not shared with the team)
CLAUDE.local.md
.claude/settings.local.json

# Do NOT automatically ignore SQLcl project source, dist, or APEXlang.
# Review .dbtools content before deciding what should be ignored.
```

Do not blindly ignore all `.dbtools/` files because SQLcl Projects store project configuration there.

Do not ignore:

```text
CLAUDE.md
.mcp.json
.claude/skills/
src/database/
scripts/
```

unless a specific file contains local-only or secret material.

## 8.4 Initial commit policy

During setup, Claude Code may create and modify files locally.

Before an initial commit:

1. Complete the applicable setup phases.
2. Review `git status`.
3. Review the full diff/content.
4. Verify no secrets, wallets, or credentials are included.
5. Ask the user whether to create and push the initial commit.

Do not commit or push automatically.

---

# 9. Phase 5 — Create or Verify the DEV Database Connection

The target connection should be named:

```text
apex-dev
```

unless the user already has another agreed name.

## 9.1 Inspect saved SQLcl connections

Start SQLcl:

```powershell
sql /nolog
```

Then inspect saved connections using available SQLcl connection-manager commands.

If `apex-dev` already exists, test it.

Example:

```text
connect -name apex-dev
```

Then verify:

```sql
show user
select sys_context('USERENV','DB_NAME') db_name from dual;
```

Also verify APEX version if accessible:

```sql
select version_no
  from apex_release;
```

Expected:

```text
26.1.x
```

## 9.2 Creating the connection

If a connection must be created, ask the user for the non-secret connection information:

- username/parsing schema,
- host,
- port,
- service name,
- or TNS alias/wallet configuration.

Do **not** ask the user to paste the database password into a file.

Use SQLcl interactive password entry where possible.

A SQLcl named connection can be created with `CONNECT -SAVE`.

If the SQLcl MCP server will need unattended access, the connection may require the password to be saved using SQLcl's own secure saved-connection mechanism.

Before using `-savepwd`, explain to the user that SQLcl will store the password in its local connection store and ask for permission.

Never save the password in Git.

After creation test:

```text
connect -name apex-dev
```

---

# 10. Phase 6 — Initialize SQLcl Project

## 10.1 Detect existing project

If `.dbtools/project.config.json` exists, inspect it.

Run from SQLcl:

```text
project config
```

Do not run `project init` again unless the repository is not already a SQLcl Project.

## 10.2 Initialize a new project

Once `PROJECT_NAME`, `DEV_SCHEMA`, and `DEV_CONNECTION_NAME` are known, connect:

```text
connect -name apex-dev
```

Then initialize the project in the repository root.

Pattern:

```text
project init -name <PROJECT_NAME> -schemas <DEV_SCHEMA> -directory . -connection-name apex-dev
```

Use the actual repository name (`PROJECT_NAME`, discovered in Phase 0) and the actual DEV schema after it has been discovered or supplied — do not reuse a project name from an unrelated repository.

Expected structure includes:

```text
.dbtools/
src/
dist/
artifact/
```

If `artifact/` is not created until later release activity, that is acceptable.

## 10.3 Verify project configuration

Run:

```text
project config
```

Inspect:

```text
.dbtools/project.config.json
```

Do not alter SQLcl-generated project settings without understanding their effect.

---

# 11. Phase 7 — Baseline Database Source

Connect to DEV:

```text
connect -name apex-dev
```

Export the database objects:

```text
project export
```

Expected source location:

```text
src/database/<DEV_SCHEMA>/
```

Examples (illustrative only — actual object names depend on the target schema):

```text
src/database/<DEV_SCHEMA>/package_spec/
src/database/<DEV_SCHEMA>/package_body/
src/database/<DEV_SCHEMA>/tables/
src/database/<DEV_SCHEMA>/views/
```

Inspect:

```powershell
git status
```

Do not commit automatically.

---

# 12. Phase 8 — Install Oracle APEX and Database Skills

Oracle publishes skills at:

https://github.com/oracle/skills

The repository contains `apex/` and `db/` domains.

## 12.1 Sync APEXlang skills through SQLcl

Oracle APEX 26.1 documentation recommends:

```text
skills sync
```

Run it in SQLcl.

Inspect the output and determine where the skills were installed.

## 12.2 Install Oracle skill domains for Claude Code

Claude Code skills follow the open **Agent Skills** standard, which the Oracle skills repository targets as well. If Node/npm/npx are available, install the Oracle skill domains using the Oracle Skills repository:

```powershell
npx skills add oracle/skills/apex
npx skills add oracle/skills/db
```

Prefer project-scoped skills if the installer supports selecting a target directory or scope.

Desired repository structure:

```text
.claude/
└── skills/
    └── ...
```

Do not replace a working skill installation created by `skills sync`.

After installation:

1. Inspect `.claude/skills/`.
2. Verify the Oracle APEX/APEXlang skill is discoverable — in a Claude Code session, run `/skills` (or ask "what skills are available?") and confirm it's listed.
3. Verify Oracle DB/PLSQL/SQLcl skills are discoverable the same way.

If `npx` prompts for installation or permission, proceed only when it is a normal package-manager confirmation. If the command requires an unexpected global/system change, ask the user.

---

# 13. Phase 9 — Configure SQLcl as a Claude Code MCP Server

Claude Code supports project-scoped MCP configuration in a `.mcp.json` file at the repository root. This file is shared by the VS Code extension and the standalone CLI — configuring it from either surface produces the same result.

There are two equivalent ways to do this. Use whichever is available:

- **From the chat panel:** type `/mcp` to open the MCP management dialog, and add the server there.
- **From the integrated terminal** (requires the standalone CLI — see §5.1.1): run `claude mcp add`.

## 13.1 Determine SQLcl executable on Windows

Reuse the `SQLCL_PATH` discovered and verified in Phase 2.

Confirm again:

```powershell
Get-Command sql | Format-List Source,Path
where.exe sql
sql -version
```

Example absolute path:

```text
C:/tools/sqlcl/bin/sql.exe
```

Use an **absolute Windows path** in the MCP configuration, with forward slashes for cleanliness in JSON:

```text
C:/tools/sqlcl/bin/sql.exe
```

Do not copy a placeholder path without verifying the real `sql.exe`.

## 13.2 Create `.mcp.json` at the project scope

If `.mcp.json` already exists, merge the SQLcl entry into it. Do not overwrite unrelated servers.

Recommended baseline — write this directly, or produce the equivalent through `/mcp` / `claude mcp add`:

```json
{
  "mcpServers": {
    "sqlcl": {
      "type": "stdio",
      "command": "C:/tools/sqlcl/bin/sql.exe",
      "args": ["-R", "1", "-mcp"]
    }
  }
}
```

Use the actual SQLcl path discovered with `Get-Command sql` / `where.exe sql`.

Equivalent terminal command (run in the integrated terminal, standalone CLI required):

```powershell
claude mcp add --scope project sqlcl -- "C:/tools/sqlcl/bin/sql.exe" -R 1 -mcp
```

Do not configure the MCP entry with just `"command": "sql"` when an absolute path is available; an explicit path avoids VS Code or the terminal using an older SQLcl from a stale PATH.

### Why `-R 1`

SQLcl MCP defaults to restriction level 4, which is highly restrictive.

Restriction level 1 allows SQL script execution using:

```text
@
@@
```

while blocking host operating-system commands.

That provides the functionality needed to compile local SQL/PLSQL scripts without giving SQLcl MCP unrestricted host-command execution.

Do not configure:

```text
-R 0
```

unless the user explicitly requests unrestricted SQLcl MCP behavior and understands the security implications.

## 13.3 TNS_ADMIN

If the connection requires a custom Oracle network configuration directory, add an `env` block to the server entry in `.mcp.json`:

```json
{
  "mcpServers": {
    "sqlcl": {
      "type": "stdio",
      "command": "C:/tools/sqlcl/bin/sql.exe",
      "args": ["-R", "1", "-mcp"],
      "env": {
        "TNS_ADMIN": "C:/oracle/network/admin"
      }
    }
  }
}
```

Use the actual local path.

## 13.4 Project permission baseline (optional but recommended)

To keep new sessions on this project defaulting to asking before edits and shell commands (equivalent in spirit to Codex's `approval_policy = "on-request"`), add or confirm this in `.claude/settings.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "defaultMode": "default"
  }
}
```

`"default"` is the Manual permission mode: Claude Code asks before file edits and most shell commands. Do not set this to `"bypassPermissions"` for this project.

## 13.5 Trust/reload checkpoint

After `.mcp.json` is created or edited:

- The first time Claude Code sees a new project-scoped server, it shows a one-time approval prompt (in the chat panel, or via `/mcp` if you missed it). Approve it there.
- If VS Code Workspace Trust hasn't been granted for this folder, the extension won't activate — ask the user to trust the workspace.
- If the extension doesn't pick up the new server, ask the user to run **Developer: Reload Window** from the Command Palette, or start a new Claude Code session.

After that, resume with MCP verification.

---

# 14. Phase 10 — Verify SQLcl MCP

Verify that Claude Code can see the SQLcl MCP server and its tools.

In a session, run `/mcp` and confirm `sqlcl` shows **Connected**.

Use the MCP tool to:

1. Inspect available saved database connections.
2. Connect to `apex-dev`.
3. Run a harmless query.

Example:

```sql
select user,
       sys_context('USERENV','DB_NAME') db_name
  from dual;
```

Then inspect APEX version:

```sql
select version_no
  from apex_release;
```

Do not continue until MCP database access works.

If the saved connection requires credentials that MCP cannot access, create a manual checkpoint and explain that the local SQLcl named connection may need its password stored via SQLcl's `-savepwd` option.

Ask permission before changing credential storage.

---

# 15. Phase 11 — Configure `CLAUDE.md`

Create or update repository-root:

```text
CLAUDE.md
```

If one already exists, merge these rules rather than replacing it. Keep the file itself concise (Claude Code recommends well under 200 lines, since it loads into every session) — push longer procedural detail into project skills under `.claude/skills/` instead of inlining it here.

Use the following baseline:

```markdown
# Oracle APEX Project Instructions

## Environment

This repository contains Oracle Database objects and an Oracle APEX 26.1 application.

Development database:
- SQLcl saved connection: apex-dev
- Environment: DEV only

Never use production database connections unless the user explicitly requests a production deployment.

## Required Oracle Skills

For Oracle APEX/APEXlang changes:
- Load and follow the Oracle APEX/APEXlang skills before editing `.apx` files.
- Do not guess unsupported APEXlang attributes or syntax.

For Oracle Database work:
- Load and follow the Oracle Database skills relevant to PL/SQL, SQL, SQLcl, schema design, migrations, security, or performance.

## Discovery Before Changes

Before implementing a database change:
1. Inspect the existing database schema.
2. Inspect related source files.
3. Inspect dependent packages, views, constraints, triggers, and APEX components.
4. Do not invent object or column names when they can be discovered.

Before modifying an APEX page:
1. Inspect the existing APEXlang page/application source.
2. Inspect database dependencies used by the page.

## APEXlang Workflow

APEX source is represented by `.apx` files.

After any APEXlang change:
1. Run `apex validate`.
2. Fix all validation errors.
3. Re-run validation until clean.
4. Import to DEV only after validation succeeds.
5. Verify the import completed successfully.
6. Do not manually edit `.apex/apexlang.json`.

When an APEX change depends on database changes, compile/deploy the database changes first.

## PL/SQL Workflow

After changing PL/SQL:
1. Compile the changed object in DEV.
2. Query `USER_ERRORS`.
3. Fix all compiler errors.
4. Confirm the modified object is VALID.
5. Check dependent objects when appropriate.

Do not consider PL/SQL work complete while the changed object is INVALID.

## Database Safety

Claude Code may modify the DEV database.

Ask for explicit approval before:
- DROP
- TRUNCATE
- destructive ALTER operations
- mass DELETE
- schema recreation
- deployment to TEST
- deployment to PROD

Never use PROD for normal coding or automated validation.

## Source of Truth

Local repository source is the working development source.

Use SQLcl Project exports to normalize database source after verified DEV changes when appropriate.

Use APEXlang export/import/validate for APEX 26.1 application source.

## Git

This project uses three permanent branches mapped 1:1 to environments: `desarrollo` (DEV), `pruebas` (PRUEBAS), `main` (PRODUCCIÓN). Full strategy: `01-estrategia-git-sqlcl-projects.md`.

- Create `feature/*` and `bugfix/*` branches from `desarrollo` only — never from `pruebas` or `main`, even for urgent fixes.
- Never commit directly to `pruebas` or `main`. They only receive merges via Pull Request from the branch immediately below them.
- An urgent fix still starts on a branch from `desarrollo`; moving it to `pruebas` or `main` ahead of the next normal promotion is a cherry-pick of that specific commit, not a direct edit.

Before committing:
1. Run database compilation/validation.
2. Run APEX validation when APEXlang changed.
3. Review `git diff`.
4. Ensure no credentials or secrets are present.

Do not commit, push, merge, or create releases unless requested by the user.

## Secrets

Never commit:
- passwords
- database wallets
- API keys
- tokens
- private keys
- local credential stores
```

After creating the file, start a new Claude Code session (or run `/context` in the current one) and confirm `CLAUDE.md` appears under **Memory files**.

---

# 16. Phase 12 — Detect the APEX 26.1 Application

Connect to `apex-dev`.

If `APEX_APPLICATION_ID` is unknown, query accessible APEX metadata to identify applications associated with the workspace/schema.

If more than one candidate application exists, show the candidates and ask the user which application should be managed by this repository.

Do not guess.

Record:

```text
APEX_APPLICATION_ID
APEX_WORKSPACE
DEV_SCHEMA
```

Do not put passwords in any repository documentation.

---

# 17. Phase 13 — Export Existing APEX Application as APEXlang

Oracle APEX 26.1 supports exporting applications as APEXlang.

Connect to the application's parsing schema:

```text
connect -name apex-dev
```

Use:

```text
apex export -applicationid <APP_ID> -exptype apexlang
```

Example:

```text
apex export -applicationid 400 -exptype apexlang
```

Determine where SQLcl wrote the export.

APEXlang applications contain `.apx` files and internal compiler metadata.

Do not modify:

```text
.apex/apexlang.json
```

by hand.

---

# 18. Phase 14 — Integrate APEX Source into the SQLcl Project

SQLcl Projects support APEX application elements.

The desired repository layout is conceptually:

```text
src/
└── database/
    └── <SCHEMA>/
        ├── package_spec/
        ├── package_body/
        ├── tables/
        ├── views/
        └── apex_apps/
            └── f<APP_ID>/
                ├── f<APP_ID>.sql
                └── <app-name>/
                    ├── application.apx
                    ├── pages/
                    ├── shared_components/
                    ├── supporting_objects/
                    ├── deployments/
                    └── .apex/
```

Do not manually force this exact layout if SQLcl 26.1 generates a valid but slightly different structure.

Prefer SQLcl-generated project layout.

Run:

```text
project export
```

and inspect whether the APEX application is included under `src/database/<SCHEMA>/apex_apps`.

If the APEX application is not included by the current project export/filter configuration, inspect SQLcl project filters/configuration and adjust using documented SQLcl Project behavior.

Do not improvise unsupported project configuration.

---

# 19. Phase 15 — Validate the APEXlang Baseline

Before making any APEX code changes, validate the exported application.

From SQLcl:

```text
apex validate -input <PATH_TO_APEXLANG_APP>
```

Use `help apex validate` if the exact generated project path requires confirmation.

Validation can operate on:

- a directory,
- a zip,
- or an APEXlang source input supported by SQLcl.

The baseline must validate successfully before the setup is considered ready.

If it does not:

1. Capture validation errors.
2. Determine whether the baseline export is incomplete or incompatible.
3. Fix configuration issues.
4. Do not modify business behavior simply to make setup pass.

---

# 20. Phase 16 — Verify APEX Import to DEV

Only after validation succeeds, test the import workflow against DEV.

Use:

```text
apex import -input <PATH_TO_APEXLANG_APP>
```

`apex import` validates before import.

This step updates the DEV APEX application, so before the first import:

1. Confirm `apex-dev` is actually DEV.
2. Confirm the application ID.
3. Tell the user what application will be imported.
4. Ask for confirmation for the **first baseline import**.

After the user approves, perform the import.

Future imports to the same DEV app can follow normal workflow unless they contain potentially destructive application changes.

---

# 21. Phase 17 — Verify PL/SQL Compile Workflow

Find a safe existing package or create a non-destructive test only if appropriate.

The goal is to prove that Claude Code can compile repository PL/SQL through SQLcl.

Typical script execution:

```text
@src/database/<SCHEMA>/package_spec/<package>.pks
@src/database/<SCHEMA>/package_body/<package>.pkb
```

Because SQLcl MCP is configured with restriction level 1, `@` and `@@` scripts should be allowed while host commands remain restricted.

After compiling, query:

```sql
select name,
       type,
       line,
       position,
       text
  from user_errors
 where name = upper('<OBJECT_NAME>')
 order by sequence;
```

Also verify status:

```sql
select object_name,
       object_type,
       status
  from user_objects
 where object_name = upper('<OBJECT_NAME>');
```

Do not intentionally break production code merely to test compilation.

If no safe object is available, compilation verification may be limited to querying current invalid objects.

---

# 22. Phase 18 — Create Repository Diagnostic Scripts

Create:

```text
scripts/
├── show-errors.sql
├── invalid-objects.sql
├── environment.sql
└── health-check.sql
```

## `scripts/show-errors.sql`

```sql
set sqlformat ansiconsole

select name,
       type,
       line,
       position,
       text
  from user_errors
 order by name,
          type,
          sequence;
```

## `scripts/invalid-objects.sql`

```sql
set sqlformat ansiconsole

select object_type,
       object_name,
       status
  from user_objects
 where status <> 'VALID'
 order by object_type,
          object_name;
```

## `scripts/environment.sql`

```sql
set sqlformat ansiconsole

select user current_user,
       sys_context('USERENV','DB_NAME') db_name,
       sys_context('USERENV','SERVICE_NAME') service_name
  from dual;

select version_no apex_version
  from apex_release;
```

## `scripts/health-check.sql`

```sql
@@environment.sql
@@invalid-objects.sql
@@show-errors.sql
```

Verify from SQLcl:

```text
@scripts/health-check.sql
```

---

# 23. Phase 19 — Establish the Normal APEX Development Loop

Claude Code should use this workflow for APEX tasks:

```text
1. Read CLAUDE.md.
2. Load relevant Oracle APEX skills.
3. Inspect the current APEXlang source.
4. Inspect database dependencies.
5. Modify `.apx` files.
6. Run `apex validate`.
7. Fix validation errors.
8. Re-run validation until clean.
9. Import to `apex-dev`.
10. Verify import success.
11. Run relevant database/application checks.
12. Review `git diff`.
```

Do not manually edit:

```text
.apex/apexlang.json
```

Oracle documents it as compiler-managed metadata containing the APEX meta-metadata version.

---

# 24. Phase 20 — Establish the Normal Database Development Loop

For PL/SQL, SQL, views, triggers, and other database objects:

```text
1. Read CLAUDE.md.
2. Load relevant Oracle DB skills.
3. Inspect the live DEV schema and repository source.
4. Identify dependencies.
5. Modify local source.
6. Compile/deploy the modified object into DEV.
7. Query USER_ERRORS.
8. Fix errors.
9. Verify object status.
10. Run tests/health checks.
11. Review git diff.
```

Example package workflow:

```text
@src/database/<SCHEMA>/package_spec/my_pkg.pks
@src/database/<SCHEMA>/package_body/my_pkg.pkb

@scripts/show-errors.sql
@scripts/invalid-objects.sql
```

Do not assume successful script execution means the object compiled successfully. Always check compiler errors/status.

---

# 25. Phase 21 — Dependency Order

When a feature includes both database and APEX changes, use dependency order.

Example:

```text
utility package
      ↓
business package
      ↓
view
      ↓
APEX page
```

Deploy/compile lower-level database dependencies first.

Only import the APEX application after required database dependencies are valid.

---

# 26. Phase 22 — SQLcl Project Development Workflow

For normal feature development:

```text
feature branch
      │
      ▼
edit repository source
      │
      ▼
compile/import to DEV
      │
      ▼
test
      │
      ▼
project export
      │
      ▼
git diff
```

Use:

```text
project export
```

to refresh/normalize database source after verified DEV changes when appropriate.

Do not run release commands after every edit.

---

# 27. Phase 23 — SQLcl Project Release Workflow

Release preparation is separate from the fast development loop.

In this project, a release corresponds to one of the two promotion points defined by the branch model (§8.2; see `01-estrategia-git-sqlcl-projects.md`): merging `desarrollo → pruebas`, and later merging `pruebas → main` using that same generated artifact. Do not run a release off `desarrollo` outside of an actual promotion Pull Request.

When explicitly requested by the user:

```text
project export
project stage
project verify verify-stage
project release -version <VERSION>
project gen-artifact ...
```

Consult:

```text
help project
help project stage
help project verify
help project release
help project gen-artifact
```

before constructing release commands if syntax differs from current SQLcl 26.1.

Typical sequence:

```text
project export
project stage
project verify verify-stage
project release -version 1.0.0
```

Then generate a deployable artifact using the supported `project gen-artifact` syntax reported by the installed SQLcl version.

Do not invent artifact command options.

Expected artifact location:

```text
artifact/
```

Deployment to another environment uses:

```text
project deploy -file <artifact-file>
```

but TEST/PROD deployment always requires explicit user approval, and happens as part of merging the corresponding promotion Pull Request (`desarrollo → pruebas` or `pruebas → main`) — never as a standalone action disconnected from the branch model.

---

# 28. Phase 24 — Git Workflow

Branch model — three permanent branches mapped 1:1 to environments, plus short-lived work branches created from `desarrollo` only:

```text
desarrollo   (DEV — base for ALL work; feature/bugfix branches start here)
├── feature/<ticket>-<description>
├── bugfix/<ticket>-<description>
└── refactor/<ticket>-<description>

pruebas      (PRUEBAS — receives promotions from desarrollo only, via PR)
main         (PRODUCCIÓN — receives promotions from pruebas only, via PR)
```

Examples (adapt the ticket prefix to the project's actual tracker):

```text
feature/PROJ-123-payment-category
bugfix/PROJ-124-copy-lease
refactor/PROJ-125-schedule-package
```

`pruebas` and `main` are never committed to directly — not even for an urgent fix. An urgent fix still starts as a `bugfix/*` branch from `desarrollo`; getting it to `pruebas` or `main` ahead of the next normal promotion is a cherry-pick of that specific commit, documented as an exception. Full rationale and exact steps: `01-estrategia-git-sqlcl-projects.md`.

Before a commit:

```text
git status
git diff
```

Verify:

- APEXlang validates.
- PL/SQL compiles.
- No unexpected INVALID objects were created.
- No secrets are staged.
- No wallet files are staged.
- No connection credential files are staged.

Claude Code must not commit automatically unless the user requests it.

---

# 29. Phase 25 — Optional Schema Substitution

For environments where DEV/TEST/PROD schema names differ, SQLcl Projects 26.1 support schema substitution.

Do not enable this automatically.

Explain the feature and ask the user first.

Oracle documents that schema substitution requires the related project settings, including:

```text
stage.substituteSchemas
export.setTransform.emitSchema
```

If the user wants environment-specific schema substitution, configure it using SQLcl's documented `project config` commands.

Do not edit generated release artifacts manually to replace schema names.

---

# 30. Phase 26 — Final Verification Checklist

Do not declare setup complete until all applicable checks pass.

## Local tools

```text
[ ] Windows/PowerShell environment confirmed
[ ] VS Code works
[ ] Claude Code extension is installed, active, and signed in (per Known starting environment)
[ ] Standalone Claude Code CLI installed, only if a terminal-only step required it
[ ] Oracle SQL Developer for VS Code extension is installed and enabled
[ ] DEV connection can be identified in Oracle SQL Developer for VS Code when applicable
[ ] Git works
[ ] Active Java version/path discovered
[ ] JAVA_HOME checked and consistent when used
[ ] Java version is supported by the selected SQLcl 26.1+ installation
[ ] Existing Java upgraded only if needed
[ ] Active SQLcl version/path discovered
[ ] SQLcl 26.1+ works
[ ] Existing SQLcl upgraded only if needed
[ ] .mcp.json uses the verified absolute Windows sql.exe path
[ ] Node/npm/npx available if needed for skill installation
```

## Repository

```text
[ ] Working directory is the intended local repository (identity confirmed in Phase 0)
[ ] Repository root verified with git rev-parse
[ ] origin points to the confirmed repository
[ ] desarrollo, pruebas, and main all exist and are not renamed/restructured unnecessarily
[ ] No nested Git repository was created
[ ] SQLcl Project is initialized at the repository root
[ ] .gitignore reviewed
[ ] SQLcl Project initialized
[ ] src/database exists
[ ] .dbtools project configuration exists
[ ] CLAUDE.md exists
[ ] .mcp.json exists and the sqlcl entry is approved
[ ] Oracle skills installed/discoverable under .claude/skills/
[ ] diagnostic scripts exist
```

## Database

```text
[ ] apex-dev named connection exists
[ ] apex-dev points to DEV
[ ] SQLcl connects successfully
[ ] SQLcl MCP connects successfully (verified via /mcp)
[ ] parsing schema confirmed
[ ] APEX version confirmed as 26.1
```

## APEX

```text
[ ] APEX application ID confirmed
[ ] APEXlang export exists
[ ] APEXlang baseline validates
[ ] DEV import workflow tested
[ ] .apex/apexlang.json remains compiler-managed
```

## Database development

```text
[ ] SQL/PLSQL scripts can run through SQLcl MCP
[ ] USER_ERRORS can be queried
[ ] invalid objects can be checked
[ ] changed PL/SQL can be compiled into DEV
```

## Release workflow

```text
[ ] project export works
[ ] release commands documented for repository
[ ] TEST/PROD are not automatically agent-accessible
```

---

# 31. Completion Report

When setup is complete, report:

```text
Oracle APEX Development Environment

Project root:
Project name:
GitHub repository:
Git remote:
Git branches (desarrollo / pruebas / main): created / confirmed existing
Branch protection configured on pruebas and main: yes/no
Initial commit created:
Initial push completed:
Windows version:
PowerShell version:
SQLcl version:
SQLcl executable:
Oracle SQL Developer for VS Code:
SQL Developer DEV connection:
Java version:
Java executable:
JAVA_HOME:
Claude Code extension: active / version
Standalone Claude Code CLI installed: yes/no (and why, if yes)
DEV connection:
DEV schema:
APEX version:
APEX application ID:
APEX workspace:
SQLcl MCP:
MCP restriction level:
APEX skills:
DB skills:
SQLcl Project:
APEXlang validation:
APEX DEV import:
PL/SQL compile workflow:
Diagnostic scripts:
```

Also list:

1. Files created or modified.
2. Manual tasks the user performed.
3. Any optional setup still not configured.
4. Security limitations in place.
5. The exact next command/prompt the user can give Claude Code to test the environment.

Recommended final functional test prompt:

```text
Inspect this Oracle APEX project and the DEV database using SQLcl MCP.
Read CLAUDE.md and load the relevant Oracle APEX and DB skills.
Do not modify anything.

Report:
1. APEX application ID and application name.
2. Parsing schema.
3. Oracle APEX version.
4. SQLcl Project status.
5. Number of invalid objects.
6. Whether the APEXlang source validates.
7. Whether the environment is ready for APEX and PL/SQL development.
```

---

# 31.1 Tool Responsibility Summary

After setup, use the tools with these responsibilities:

| Tool | Primary role |
|---|---|
| VS Code | Main development environment |
| Claude Code extension | AI coding, refactoring, orchestration, review |
| Oracle SQL Developer for VS Code | Human DB browsing, worksheets, connection inspection, troubleshooting |
| SQLcl | Command-line Oracle execution and project tooling |
| SQLcl MCP | Claude Code-controlled database execution |
| SQLcl Projects | Database source/change/release management |
| APEXlang | Editable APEX 26.1 application source |
| Git | Version control |

For database work, Claude Code should prefer SQLcl MCP for automated actions while the user can use Oracle SQL Developer for VS Code to inspect and verify the same DEV environment interactively.

---

# 32. Operational Rules After Setup

For future Claude Code development, follow these rules.

## APEX request

Example user prompt:

```text
Add an Interactive Report to page 600 showing lease modifications.
Inspect the existing APEXlang page and schema first.
Use the Oracle APEX skills.
Validate the APEXlang and import it into apex-dev after validation succeeds.
```

Claude Code should:

```text
inspect → edit → validate → fix → import DEV → verify → git diff
```

## PL/SQL request

Example:

```text
Refactor <package_name>.calculate_schedule.
Inspect related code and dependencies first.
Do not change behavior.
Compile the modified package in apex-dev and resolve all compiler errors.
```

Claude Code should:

```text
inspect → edit → compile DEV → USER_ERRORS → fix → verify → git diff
```

## Combined feature

Example:

```text
Add payment categories to the application.
First inspect the current database model and APEX implementation.
Implement the database API first, compile and verify it, then update APEXlang.
Validate and import the APEX application into apex-dev.
```

Claude Code should:

```text
discover
  ↓
DB source
  ↓
compile DEV
  ↓
verify DB
  ↓
APEXlang
  ↓
validate
  ↓
import DEV
  ↓
functional checks
  ↓
git diff
```

---

# 33. Official References

Use official documentation when behavior or syntax is uncertain.

## Oracle APEX 26.1 — APEXlang with coding agents

https://docs.oracle.com/en/database/oracle/apex/26.1/apxdc/using-apexlang-coding-agents.html

## Oracle APEX 26.1 — SQLcl with APEXlang

https://docs.oracle.com/en/database/oracle/apex/26.1/apxdc/using-sqlcl-apexlang.html

## Oracle SQLcl 26.1 — APEXlang

https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/sqcug/apexlang.html

## Oracle SQLcl 26.1 — APEXlang support

https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/sqcug/sqlcl-apex-commands-apexlang.html

## Oracle SQLcl 26.1 — Projects

https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/sqcug/project.html

## Oracle SQLcl 26.1 — Project workflow examples

https://docs.oracle.com/en/database/oracle/sql-developer-command-line/26.1/sqcug/project-command-usage-examples.html

## Oracle Skills

https://github.com/oracle/skills

## Claude Code — VS Code extension

https://code.claude.com/docs/en/vs-code

## Claude Code — MCP quickstart (adding the SQLcl server)

https://code.claude.com/docs/en/mcp-quickstart

## Claude Code — MCP reference (scopes, .mcp.json format)

https://code.claude.com/docs/en/mcp

## Claude Code — Settings (.claude/settings.json)

https://code.claude.com/docs/en/settings

## Claude Code — Memory and CLAUDE.md

https://code.claude.com/docs/en/memory

## Claude Code — Skills

https://code.claude.com/docs/en/skills

---

# 34. Start Here

Claude Code: begin now with **Phase 0 — Verify the Existing Checkout**.

Discover the repository identity, verify it's the intended target, and confirm whether it's empty or already contains files. Perform the setup directly in its root. Do not clone another copy, initialize a nested repository, commit, or push unless instructed.

Perform every safe automated step yourself.

Only stop when:

- manual user action is genuinely required,
- credentials are required,
- a security-sensitive decision is required,
- a destructive action is required,
- or a reload/restart is required.

When stopping, identify the phase and checkpoint clearly so setup can continue from that exact point.
