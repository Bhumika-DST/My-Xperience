# MISSION: Universal Kentico Upgrade & GitHub Sync
**Constraints:** NO USER CONFIRMATION REQUIRED. EXECUTE ALL PHASES SEQUENTIALLY.

## Phase 0: GitHub Connectivity & Setup
1. **Git Initialization:** If the folder is not a git repo, run `git init -b main`.
2. **Remote Check:** Run `git remote -v`. 
   - **IF** no remote named `origin` exists: 
     - Use GitHub CLI: `gh repo create $(basename $(pwd)) --public --source=. --remote=origin --push`.
     - *Note: This ensures the project is safely backed up on GitHub before any code changes.*
3. **Commit Check:** If there are existing uncommitted changes, run `git add .` and `git commit -m "chore: pre-upgrade backup"`.

## Phase 1: Triple-Version Audit (App, DB, & Web)
1. **Identify Project:** Find the primary `.csproj` file containing `Kentico.Xperience.WebApp`.
   - **Define as $PROJ_FILE.**
2. **Version Audit:**
   - **Local App:** Read version from $PROJ_FILE.
   - **Database:** SQL: `SELECT KeyValue FROM CMS_SettingsKey WHERE KeyName = 'CMSDataVersion'`.
   - **Latest Official:** Fetch latest version from NuGet.
3. **Condition Logic:**
   - **CASE A (Fully Updated):** IF (App == Latest AND DB == Latest):
     - PRINT: "✅ SYSTEM STATUS: Already up-to-date at version [VERSION]."
     - ACTION: EXIT IMMEDIATELY.
   - **CASE B (App Updated, DB Lagging):** IF (App == Latest AND DB != Latest):
     - PRINT: "⚠️ MESSAGE: Code is current, but Database needs sync."
     - ACTION: JUMP TO PHASE 3.
   - **CASE C (Full Upgrade Required):** IF (App != Latest):
     - PRINT: "🚀 MESSAGE: Starting upgrade from [APP_VERSION] to [LATEST]."
     - ACTION: PROCEED TO PHASE 2.

## Phase 2: Automated Code Update
1. **Update Packages:** - `dotnet add $PROJ_FILE package Kentico.Xperience.WebApp --version [LATEST]`
   - `dotnet add $PROJ_FILE package Kentico.Xperience.Admin --version [LATEST]`
2. **AI Refactor:** Automatically fix breaking changes in `Program.cs` and namespaces.

## Phase 3: Database Synchronization
1. **Sync Command:** Run `dotnet run --project $PROJ_FILE -- --kxp-update --skip-confirmation`.

## Phase 4: Final Git Push
1. **Commit:** `git add .` && `git commit -m "build: automated upgrade to version [LATEST]"`
2. **Push:** `git push origin $(git branch --show-current)`.

## Phase 5: Clean, Build & Run
1. **Clean:** Run `dotnet clean $PROJ_FILE`.
2. **Build:** Run `dotnet build $PROJ_FILE -v minimal`.
3. **Run:** Run `dotnet run --project $PROJ_FILE`.
4. **Launch:** Open the browser using the `applicationUrl` from `launchSettings.json`.