Run **automatically every 2 hours**, a `.bat` file is a good fit.

The script below will:

* Check every repository under `PMSQLDBA`
* Create the corresponding repository under `TechwithPraveenMadupu` if it doesn't exist
* Keep destination repositories **private**
* Initially clone each repository as a mirror
* On subsequent runs, update the existing mirror instead of cloning again
* `git push --mirror` to synchronize branches/tags/history
* Continue if one repository fails
* Write a log file with success/failure information

One important point: this is a **one-way sync**:

`PMSQLDBA → TechwithPraveenMadupu`

Changes made directly in the organization repositories will be overwritten by the next mirror sync.

### 1. Create the batch file

Create:

```text
C:\Users\DELL\GitHubRepos\sync-github-repos.bat
```

Use this:

```bat
@echo off
setlocal EnableExtensions EnableDelayedExpansion

REM ============================================================
REM GitHub Personal -> Organization Automatic Sync
REM
REM SOURCE : PMSQLDBA
REM TARGET : TechwithPraveenMadupu
REM
REM Run frequency: Windows Task Scheduler (every 2 hours)
REM ============================================================

set "SOURCE=PMSQLDBA"
set "TARGET=TechwithPraveenMadupu"

set "BASE=C:\Users\DELL\GitHubRepos\AutoSync"
set "LOG=%BASE%\sync.log"
set "REPOS=%BASE%\repos"

REM ------------------------------------------------------------
REM Create working directories
REM ------------------------------------------------------------

if not exist "%BASE%" mkdir "%BASE%"
if not exist "%REPOS%" mkdir "%REPOS%"

echo.>> "%LOG%"
echo ============================================================>> "%LOG%"
echo Sync started: %date% %time%>> "%LOG%"
echo Source: %SOURCE%>> "%LOG%"
echo Target: %TARGET%>> "%LOG%"
echo ============================================================>> "%LOG%"

echo.
echo ============================================================
echo GitHub Repository Sync
echo %SOURCE%  -->  %TARGET%
echo ============================================================
echo.

REM ------------------------------------------------------------
REM Check Git
REM ------------------------------------------------------------

where git >nul 2>&1

if errorlevel 1 (
    echo ERROR: Git was not found.
    echo ERROR: Git was not found.>> "%LOG%"
    exit /b 1
)

REM ------------------------------------------------------------
REM Check GitHub CLI
REM ------------------------------------------------------------

where gh >nul 2>&1

if errorlevel 1 (
    echo ERROR: GitHub CLI was not found.
    echo ERROR: GitHub CLI was not found.>> "%LOG%"
    exit /b 1
)

REM ------------------------------------------------------------
REM Check GitHub authentication
REM ------------------------------------------------------------

echo Checking GitHub authentication...

gh auth status >nul 2>&1

if errorlevel 1 (
    echo ERROR: GitHub CLI is not authenticated.
    echo ERROR: GitHub CLI is not authenticated.>> "%LOG%"
    exit /b 1
)

REM ------------------------------------------------------------
REM Get repositories from personal account
REM ------------------------------------------------------------

echo Getting repositories from %SOURCE%...

powershell.exe -NoProfile -ExecutionPolicy Bypass -Command ^
 "$r = gh repo list '%SOURCE%' --limit 1000 --json name,isArchived | ConvertFrom-Json; $r | Where-Object { -not $_.isArchived } | ForEach-Object { $_.name }" > "%BASE%\repo-list.txt"

if errorlevel 1 (
    echo ERROR: Could not retrieve repository list.
    echo ERROR: Could not retrieve repository list.>> "%LOG%"
    exit /b 1
)

REM ------------------------------------------------------------
REM Counters
REM ------------------------------------------------------------

set /a TOTAL=0
set /a SUCCESS=0
set /a FAILED=0
set /a CREATED=0
set /a UPDATED=0

REM ------------------------------------------------------------
REM Process each repository
REM ------------------------------------------------------------

for /f "usebackq delims=" %%R in ("%BASE%\repo-list.txt") do (

    set "REPO=%%R"
    set /a TOTAL+=1

    echo.
    echo ============================================================
    echo Processing: !REPO!
    echo ============================================================

    echo.>> "%LOG%"
    echo ------------------------------------------------------------>> "%LOG%"
    echo Repository: !REPO!>> "%LOG%"
    echo Started: %date% %time%>> "%LOG%"

    REM --------------------------------------------------------
    REM Check destination repository
    REM --------------------------------------------------------

    gh repo view "%TARGET%/!REPO!" --json name >nul 2>&1

    if errorlevel 1 (

        echo Destination does not exist. Creating...

        echo Creating destination repository...>> "%LOG%"

        gh repo create "%TARGET%/!REPO!" ^
            --private ^
            --description "Automatically synchronized from %SOURCE%/!REPO!"

        if errorlevel 1 (
            echo FAILED: Could not create !REPO!
            echo FAILED: Could not create destination repository.>> "%LOG%"
            set /a FAILED+=1
            goto :NEXT_REPO
        )

        echo Created: !REPO!
        echo Created destination repository.>> "%LOG%"
        set /a CREATED+=1
    )

    REM --------------------------------------------------------
    REM Local mirror path
    REM --------------------------------------------------------

    set "LOCAL=%REPOS%\!REPO!.git"

    REM --------------------------------------------------------
    REM First run: clone mirror
    REM --------------------------------------------------------

    if not exist "!LOCAL!\HEAD" (

        echo Creating local mirror...

        git clone --mirror ^
            "https://github.com/%SOURCE%/!REPO!.git" ^
            "!LOCAL!"

        if errorlevel 1 (
            echo FAILED: Clone failed for !REPO!
            echo FAILED: Clone failed.>> "%LOG%"
            set /a FAILED+=1
            goto :NEXT_REPO
        )

        echo Initial mirror created.
        echo Initial mirror created.>> "%LOG%"

    ) else (

        REM ----------------------------------------------------
        REM Existing mirror: update from personal repository
        REM ----------------------------------------------------

        echo Updating existing local mirror...

        pushd "!LOCAL!"

        git remote update --prune

        if errorlevel 1 (
            echo FAILED: Could not update local mirror for !REPO!
            echo FAILED: git remote update failed.>> "%LOG%"
            popd
            set /a FAILED+=1
            goto :NEXT_REPO
        )

        popd
    )

    REM --------------------------------------------------------
    REM Push mirror to organization
    REM --------------------------------------------------------

    echo Synchronizing to organization...

    pushd "!LOCAL!"

    git push --mirror ^
        "https://github.com/%TARGET%/!REPO!.git"

    if errorlevel 1 (
        echo FAILED: Push failed for !REPO!
        echo FAILED: git push --mirror failed.>> "%LOG%"
        popd
        set /a FAILED+=1
        goto :NEXT_REPO
    )

    popd

    echo SUCCESS: !REPO!
    echo SUCCESS: Repository synchronized.>> "%LOG%"
    echo Completed: %date% %time%>> "%LOG%"

    set /a SUCCESS+=1
    set /a UPDATED+=1

    :NEXT_REPO
)

REM ------------------------------------------------------------
REM Final summary
REM ------------------------------------------------------------

echo.
echo ============================================================
echo SYNC COMPLETE
echo ============================================================
echo Total repositories : %TOTAL%
echo Successful         : %SUCCESS%
echo Failed             : %FAILED%
echo Created            : %CREATED%
echo Updated            : %UPDATED%
echo.
echo Log:
echo %LOG%
echo ============================================================

echo.>> "%LOG%"
echo ============================================================>> "%LOG%"
echo Sync completed: %date% %time%>> "%LOG%"
echo Total: %TOTAL%>> "%LOG%"
echo Successful: %SUCCESS%>> "%LOG%"
echo Failed: %FAILED%>> "%LOG%"
echo Created: %CREATED%>> "%LOG%"
echo Updated: %UPDATED%>> "%LOG%"
echo ============================================================>> "%LOG%"

exit /b 0
```

### 2. Test it manually first

Open PowerShell or CMD and run:

```bat
C:\Users\DELL\GitHubRepos\AutoSync\sync-github-repos.bat
```

For the first run, it will create local mirrors such as:

```text
C:\Users\DELL\GitHubRepos\AutoSync\repos\
    Azure-MI.git
    Repository2.git
    Repository3.git
    ...
```

The next run every two hours will **update those mirrors** rather than downloading the entire repositories again.

The log will be here:

```text
C:\Users\DELL\GitHubRepos\AutoSync\sync.log
```

---

## 3. Configure Windows Task Scheduler

Open:

```text
Task Scheduler
```

Select:

**Create Task...**

### General

Set:

**Name:**

```text
GitHub Personal to Organization Sync
```

Select:

```text
Run whether user is logged on or not
```

I recommend using your **DELL Windows account**, not `SYSTEM`, because GitHub CLI authentication is normally associated with your Windows user profile.

---

### Triggers

Click **New...**

Set:

```text
Begin the task: On a schedule
Daily
Start: whatever time you want
```

Then select:

```text
Repeat task every: 2 hours
for a duration of: Indefinitely
```

Make sure:

```text
Enabled
```

is checked.

---

### Actions

Click **New...**

**Action:**

```text
Start a program
```

**Program/script:**

```text
C:\Users\DELL\GitHubRepos\AutoSync\sync-github-repos.bat
```

Alternatively, for maximum reliability:

**Program/script:**

```text
C:\Windows\System32\cmd.exe
```

**Add arguments:**

```text
/c "C:\Users\DELL\GitHubRepos\AutoSync\sync-github-repos.bat"
```

**Start in:**

```text
C:\Users\DELL\GitHubRepos\AutoSync
```

---

## 4. Important GitHub authentication issue

This is the part I want you to test **before enabling the 2-hour schedule**.

Run:

```powershell
gh auth status
```

and make sure the account shown has access to:

```text
PMSQLDBA
```

and can create/write repositories in:

```text
TechwithPraveenMadupu
```

Then test:

```powershell
gh repo view PMSQLDBA/Azure-MI
```

and:

```powershell
gh repo view TechwithPraveenMadupu/Azure-MI
```

Both should work.

### Why this matters

A scheduled task can run under a different environment than your interactive PowerShell session.

If Task Scheduler runs as another Windows account or as `SYSTEM`, it may not see the GitHub CLI authentication you've already configured.

**Use the same Windows account (`DELL`) that you used when your successful `git push --mirror` worked.**

---

## One more important consideration

Because you're migrating about **150 repositories**, I would **not** blindly run this every two hours until we verify the first automated run.

The safest sequence is:

```text
PMSQLDBA
   │
   │  every 2 hours
   ▼
Windows Task Scheduler
   │
   ▼
sync-github-repos.bat
   │
   ├── Create missing repo
   │
   ├── Update local mirror
   │
   └── git push --mirror
   │
   ▼
TechwithPraveenMadupu
```

Once the first complete run succeeds, Task Scheduler can maintain the organization copies automatically.

**One caution:** `git push --mirror` makes the organization Git refs match the personal repository. If someone creates a branch/tag directly in the organization that doesn't exist in the personal repository, the next sync can remove it. So this should be treated as a **personal → organization replication process**, not a two-way sync.
