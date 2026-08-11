```powershell
# ============================================================
# GitHub Personal Account -> Organization Repository Migration
#
# Source:
#   PMSQLDBA
#
# Destination:
#   TechwithPraveenMadupu
#
# Uses:
#   GitHub CLI (gh)
#   Git
#
# Migration method:
#   git clone --mirror
#   git push --mirror
# ============================================================

$SourceOwner = "PMSQLDBA"
$TargetOrg   = "TechwithPraveenMadupu"

$WorkDir = Join-Path $HOME "GitHubRepos\Migration"
$LogFile = Join-Path $WorkDir "migration-log.txt"

# ------------------------------------------------------------
# Create working directory
# ------------------------------------------------------------

New-Item -ItemType Directory -Force -Path $WorkDir | Out-Null

"============================================================" | Out-File $LogFile
"GitHub Repository Migration" | Out-File $LogFile -Append
"Started: $(Get-Date)" | Out-File $LogFile -Append
"Source: $SourceOwner" | Out-File $LogFile -Append
"Target: $TargetOrg" | Out-File $LogFile -Append
"============================================================" | Out-File $LogFile -Append

# ------------------------------------------------------------
# Check required commands
# ------------------------------------------------------------

if (-not (Get-Command git -ErrorAction SilentlyContinue)) {
    Write-Host "ERROR: Git is not installed." -ForegroundColor Red
    exit 1
}

if (-not (Get-Command gh -ErrorAction SilentlyContinue)) {
    Write-Host "ERROR: GitHub CLI (gh) is not installed." -ForegroundColor Red
    exit 1
}

# ------------------------------------------------------------
# Check GitHub authentication
# ------------------------------------------------------------

Write-Host ""
Write-Host "Checking GitHub authentication..." -ForegroundColor Cyan

gh auth status

if ($LASTEXITCODE -ne 0) {
    Write-Host ""
    Write-Host "You are not authenticated with GitHub CLI." -ForegroundColor Red
    Write-Host "Run: gh auth login" -ForegroundColor Yellow
    exit 1
}

# ------------------------------------------------------------
# Verify organization access
# ------------------------------------------------------------

Write-Host ""
Write-Host "Checking organization access..." -ForegroundColor Cyan

gh api "orgs/$TargetOrg" --silent

if ($LASTEXITCODE -ne 0) {
    Write-Host ""
    Write-Host "ERROR: Cannot access organization $TargetOrg" -ForegroundColor Red
    exit 1
}

# ------------------------------------------------------------
# Get all repositories
# ------------------------------------------------------------

Write-Host ""
Write-Host "Getting repositories from $SourceOwner..." -ForegroundColor Cyan

$ReposJson = gh repo list $SourceOwner `
    --limit 1000 `
    --json name,isPrivate,isArchived `
    --no-archived

if ($LASTEXITCODE -ne 0) {
    Write-Host "ERROR: Could not retrieve repositories." -ForegroundColor Red
    exit 1
}

$Repos = $ReposJson | ConvertFrom-Json

if ($Repos.Count -eq 0) {
    Write-Host "No repositories found." -ForegroundColor Yellow
    exit 0
}

Write-Host ""
Write-Host "Found $($Repos.Count) repositories." -ForegroundColor Green

# ------------------------------------------------------------
# Confirm before starting
# ------------------------------------------------------------

Write-Host ""
Write-Host "SOURCE      : $SourceOwner" -ForegroundColor Cyan
Write-Host "DESTINATION : $TargetOrg" -ForegroundColor Cyan
Write-Host "REPOSITORIES: $($Repos.Count)" -ForegroundColor Cyan
Write-Host "WORK DIR    : $WorkDir" -ForegroundColor Cyan
Write-Host ""

$Confirm = Read-Host "Type YES to start migration"

if ($Confirm -ne "YES") {
    Write-Host "Migration cancelled." -ForegroundColor Yellow
    exit 0
}

# ------------------------------------------------------------
# Counters
# ------------------------------------------------------------

$SuccessCount = 0
$FailedCount  = 0
$SkippedCount = 0

$FailedRepos = @()

# ------------------------------------------------------------
# Process repositories
# ------------------------------------------------------------

foreach ($Repo in $Repos) {

    $RepoName = $Repo.name

    Write-Host ""
    Write-Host "============================================================" -ForegroundColor DarkGray
    Write-Host "Migrating: $RepoName" -ForegroundColor Cyan
    Write-Host "============================================================" -ForegroundColor DarkGray

    "------------------------------------------------------------" | Out-File $LogFile -Append
    "Repository: $RepoName" | Out-File $LogFile -Append
    "Started: $(Get-Date)" | Out-File $LogFile -Append

    $BareRepoPath = Join-Path $WorkDir "$RepoName.git"

    # --------------------------------------------------------
    # Check whether destination repository already exists
    # --------------------------------------------------------

    gh repo view "$TargetOrg/$RepoName" --json name --silent 2>$null

    if ($LASTEXITCODE -eq 0) {

        Write-Host "Destination repository already exists." -ForegroundColor Yellow

        # We will still push the mirror into it.
        # This allows the script to be safely re-run.

    }
    else {

        Write-Host "Creating destination repository..." -ForegroundColor Yellow

        # Preserve private visibility.
        gh repo create "$TargetOrg/$RepoName" `
            --private `
            --description "Migrated from $SourceOwner/$RepoName"

        if ($LASTEXITCODE -ne 0) {

            Write-Host "FAILED: Could not create repository." -ForegroundColor Red

            "FAILED - Could not create destination repository" |
                Out-File $LogFile -Append

            $FailedCount++
            $FailedRepos += $RepoName

            continue
        }

        Write-Host "Destination repository created." -ForegroundColor Green
    }

    # --------------------------------------------------------
    # Remove old local mirror if present
    # --------------------------------------------------------

    if (Test-Path $BareRepoPath) {

        Write-Host "Removing previous local mirror..." -ForegroundColor Yellow

        Remove-Item -Recurse -Force $BareRepoPath
    }

    # --------------------------------------------------------
    # Clone source repository as mirror
    # --------------------------------------------------------

    Write-Host "Cloning source repository..." -ForegroundColor Yellow

    git clone --mirror `
        "https://github.com/$SourceOwner/$RepoName.git" `
        $BareRepoPath

    if ($LASTEXITCODE -ne 0) {

        Write-Host "FAILED: Clone failed." -ForegroundColor Red

        "FAILED - Clone failed" | Out-File $LogFile -Append

        $FailedCount++
        $FailedRepos += $RepoName

        continue
    }

    Write-Host "Clone successful." -ForegroundColor Green

    # --------------------------------------------------------
    # Push mirror to organization
    # --------------------------------------------------------

    Write-Host "Pushing mirror to organization..." -ForegroundColor Yellow

    Push-Location $BareRepoPath

    git push --mirror `
        "https://github.com/$TargetOrg/$RepoName.git"

    $PushExitCode = $LASTEXITCODE

    Pop-Location

    if ($PushExitCode -ne 0) {

        Write-Host "FAILED: Push failed." -ForegroundColor Red

        "FAILED - Push failed" | Out-File $LogFile -Append

        $FailedCount++
        $FailedRepos += $RepoName

        continue
    }

    # --------------------------------------------------------
    # Success
    # --------------------------------------------------------

    Write-Host "SUCCESS: $RepoName" -ForegroundColor Green

    "SUCCESS" | Out-File $LogFile -Append
    "Completed: $(Get-Date)" | Out-File $LogFile -Append

    $SuccessCount++
}

# ------------------------------------------------------------
# Final summary
# ------------------------------------------------------------

Write-Host ""
Write-Host "============================================================" -ForegroundColor Cyan
Write-Host "MIGRATION COMPLETE" -ForegroundColor Cyan
Write-Host "============================================================" -ForegroundColor Cyan

Write-Host ""
Write-Host "Total repositories : $($Repos.Count)"
Write-Host "Successful         : $SuccessCount" -ForegroundColor Green
Write-Host "Failed             : $FailedCount" -ForegroundColor Red
Write-Host ""

if ($FailedRepos.Count -gt 0) {

    Write-Host "Failed repositories:" -ForegroundColor Red

    foreach ($FailedRepo in $FailedRepos) {
        Write-Host "  - $FailedRepo" -ForegroundColor Red
    }

    Write-Host ""
    Write-Host "Review the log:" -ForegroundColor Yellow
    Write-Host $LogFile
}

"============================================================" | Out-File $LogFile -Append
"Migration finished: $(Get-Date)" | Out-File $LogFile -Append
"Total: $($Repos.Count)" | Out-File $LogFile -Append
"Successful: $SuccessCount" | Out-File $LogFile -Append
"Failed: $FailedCount" | Out-File $LogFile -Append

Write-Host ""
Write-Host "Log file:" -ForegroundColor Cyan
Write-Host $LogFile
```
