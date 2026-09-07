# Streamflix Folder Structure — Bash & PowerShell

This document combines the Bash and PowerShell scripts for creating the Streamflix folder structure.

---

# Part 1 — Bash

# Streamflix Folder Structure

## Bash Script

Save as `create-streamflix-structure.sh`:

```bash
#!/bin/bash

# ============================================
# Streamflix Folder Structure Generator
# ============================================

set -e

ROOT_DIR="${1:-Streamflix}"

echo "Creating Streamflix folder structure..."
echo "Root directory: $ROOT_DIR"
echo ""

mkdir -p "$ROOT_DIR"/{
    .github/workflows,

    backend/src/config,
    backend/src/db/migrations,

    backend/src/media/cdn,
    backend/src/media/storage,
    backend/src/media/transcoder,

    backend/src/middleware,

    backend/src/modules/admin,
    backend/src/modules/audit,
    backend/src/modules/auth,
    backend/src/modules/catalog,
    backend/src/modules/content,
    backend/src/modules/health,
    backend/src/modules/media,
    backend/src/modules/payments,
    backend/src/modules/playback,
    backend/src/modules/subscriptions/providers,
    backend/src/modules/users,
    backend/src/modules/webhooks,

    backend/src/services/cdn,
    backend/src/services/storage,
    backend/src/services/transcoder,

    backend/src/utils,
    backend/src/worker,

    backend/tests/helpers,
    backend/tests/integration,
    backend/tests/unit,

    docs,

    frontend/public,
    frontend/src/api,
    frontend/src/components,
    frontend/src/context,
    frontend/src/hooks,
    frontend/src/pages/admin,
    frontend/src/styles,

    infra/helm/streamflix/templates,

    infra/terraform/modules/cloudfront,
    infra/terraform/modules/dynamodb,
    infra/terraform/modules/ecr,
    infra/terraform/modules/eks,
    infra/terraform/modules/elasticache,
    infra/terraform/modules/iam,
    infra/terraform/modules/mediaconvert,
    infra/terraform/modules/rds,
    infra/terraform/modules/s3,
    infra/terraform/modules/vpc,

    infra/terragrunt/live/dev,
    infra/terragrunt/live/prod,

    scripts
}

echo "Folder structure created successfully."
echo ""

if command -v tree >/dev/null 2>&1; then
    echo "============================================"
    echo "Created structure:"
    echo "============================================"
    tree "$ROOT_DIR"
else
    echo "Install 'tree' to display the folder structure:"
    echo "  Ubuntu/Debian : sudo apt install tree"
    echo "  RHEL/CentOS   : sudo yum install tree"
    echo "  macOS         : brew install tree"
fi
```

## How to Run

Create the script:

```bash
nano create-streamflix-structure.sh
```

Make it executable:

```bash
chmod +x create-streamflix-structure.sh
```

Run it:

```bash
./create-streamflix-structure.sh
```

This creates the default root:

```text
Streamflix/
```

You can specify another location:

```bash
./create-streamflix-structure.sh /opt/streamflix
```

or:

```bash
./create-streamflix-structure.sh ~/projects/streamflix
```

## Complete Folder Structure

```text
Streamflix/
├── .github/
│   └── workflows/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── db/
│   │   │   └── migrations/
│   │   ├── media/
│   │   │   ├── cdn/
│   │   │   ├── storage/
│   │   │   └── transcoder/
│   │   ├── middleware/
│   │   ├── modules/
│   │   │   ├── admin/
│   │   │   ├── audit/
│   │   │   ├── auth/
│   │   │   ├── catalog/
│   │   │   ├── content/
│   │   │   ├── health/
│   │   │   ├── media/
│   │   │   ├── payments/
│   │   │   ├── playback/
│   │   │   ├── subscriptions/
│   │   │   │   └── providers/
│   │   │   ├── users/
│   │   │   └── webhooks/
│   │   ├── services/
│   │   │   ├── cdn/
│   │   │   ├── storage/
│   │   │   └── transcoder/
│   │   ├── utils/
│   │   └── worker/
│   └── tests/
│       ├── helpers/
│       ├── integration/
│       └── unit/
├── docs/
├── frontend/
│   ├── public/
│   └── src/
│       ├── api/
│       ├── components/
│       ├── context/
│       ├── hooks/
│       ├── pages/
│       │   └── admin/
│       └── styles/
├── infra/
│   ├── helm/
│   │   └── streamflix/
│   │       └── templates/
│   ├── terraform/
│   │   └── modules/
│   │       ├── cloudfront/
│   │       ├── dynamodb/
│   │       ├── ecr/
│   │       ├── eks/
│   │       ├── elasticache/
│   │       ├── iam/
│   │       ├── mediaconvert/
│   │       ├── rds/
│   │       ├── s3/
│   │       └── vpc/
│   └── terragrunt/
│       └── live/
│           ├── dev/
│           └── prod/
└── scripts/
```

## Verify

If `tree` is installed:

```bash
tree Streamflix
```

For Ubuntu/WSL, install it with:

```bash
sudo apt update
sudo apt install tree
```

Then:

```bash
tree Streamflix
```

The structure above follows the attached `streamflix.txt` folder listing. fileciteturn0file0L4-L71


---

# Part 2 — PowerShell

# Streamflix Folder Structure — PowerShell

This document contains the PowerShell script to create the complete Streamflix folder structure from the provided `streamflix.txt` tree listing.

## 1. PowerShell Script

Save the following as:

```text
Create-StreamflixStructure.ps1
```

```powershell
# ============================================
# Streamflix Folder Structure Generator
# ============================================

param(
    [string]$TreeFile = ".\streamflix.txt",
    [string]$OutputPath = ".\Streamflix"
)

if (-not (Test-Path $TreeFile)) {
    Write-Host "ERROR: Tree file not found: $TreeFile" -ForegroundColor Red
    exit 1
}

# Create root directory
if (-not (Test-Path $OutputPath)) {
    New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null
}

# Read the tree file
$lines = Get-Content -Path $TreeFile -Encoding Default

# Stores the full path for each tree level
$pathStack = @{}

foreach ($line in $lines) {

    # Skip empty lines
    if ([string]::IsNullOrWhiteSpace($line)) {
        continue
    }

    # Skip tree header
    if ($line -match 'Volume serial number') {
        continue
    }

    # Only process directory entries
    if ($line -notmatch '\+---') {
        continue
    }

    # Everything before +--- represents indentation
    $plusIndex = $line.IndexOf('+---')

    if ($plusIndex -lt 0) {
        continue
    }

    $prefix = $line.Substring(0, $plusIndex)

    # Each tree indentation level is approximately 4 characters
    $level = [math]::Floor($prefix.Length / 4)

    # Extract directory name
    $folderName = $line.Substring($plusIndex + 4).Trim()

    if ([string]::IsNullOrWhiteSpace($folderName)) {
        continue
    }

    # Determine parent directory
    if ($level -eq 0) {
        $parentPath = $OutputPath
    }
    elseif ($pathStack.ContainsKey($level - 1)) {
        $parentPath = $pathStack[$level - 1]
    }
    else {
        $parentPath = $OutputPath
    }

    # Build full path
    $fullPath = Join-Path $parentPath $folderName

    # Create directory
    if (-not (Test-Path $fullPath)) {
        New-Item -ItemType Directory -Path $fullPath -Force | Out-Null
        Write-Host "Created: $fullPath" -ForegroundColor Green
    }
    else {
        Write-Host "Exists:  $fullPath" -ForegroundColor Yellow
    }

    # Store current level path
    $pathStack[$level] = $fullPath

    # Remove deeper paths
    foreach ($key in @($pathStack.Keys)) {
        if ($key -gt $level) {
            $pathStack.Remove($key)
        }
    }
}

Write-Host ""
Write-Host "============================================" -ForegroundColor Cyan
Write-Host "Streamflix folder structure created!" -ForegroundColor Cyan
Write-Host "============================================"
Write-Host "Location: $((Resolve-Path $OutputPath).Path)" -ForegroundColor Cyan
```

## 2. Files Required

Place these files in the same directory:

```text
streamflix.txt
Create-StreamflixStructure.ps1
```

The `streamflix.txt` file contains the source tree structure. The PowerShell script reads that file and creates the corresponding directories. The source listing includes `.github`, `backend`, `frontend`, `infra`, `docs`, and `scripts` along with their nested folders. fileciteturn0file0L4-L71

## 3. Run the Script

Open PowerShell in the directory containing the files.

If PowerShell blocks script execution, allow it only for the current PowerShell process:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then run:

```powershell
.\Create-StreamflixStructure.ps1
```

The default output directory is:

```text
Streamflix
```

## 4. Specify a Different Output Directory

You can provide a custom output location:

```powershell
.\Create-StreamflixStructure.ps1 -OutputPath "C:\Projects\Streamflix"
```

You can also specify a different tree input file:

```powershell
.\Create-StreamflixStructure.ps1 `
    -TreeFile ".\my-structure.txt" `
    -OutputPath "C:\Projects\Streamflix"
```

## 5. Complete Folder Structure

```text
Streamflix/
├── .github/
│   └── workflows/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── db/
│   │   │   └── migrations/
│   │   ├── media/
│   │   │   ├── cdn/
│   │   │   ├── storage/
│   │   │   └── transcoder/
│   │   ├── middleware/
│   │   ├── modules/
│   │   │   ├── admin/
│   │   │   ├── audit/
│   │   │   ├── auth/
│   │   │   ├── catalog/
│   │   │   ├── content/
│   │   │   ├── health/
│   │   │   ├── media/
│   │   │   ├── payments/
│   │   │   ├── playback/
│   │   │   ├── subscriptions/
│   │   │   │   └── providers/
│   │   │   ├── users/
│   │   │   └── webhooks/
│   │   ├── services/
│   │   │   ├── cdn/
│   │   │   ├── storage/
│   │   │   └── transcoder/
│   │   ├── utils/
│   │   └── worker/
│   └── tests/
│       ├── helpers/
│       ├── integration/
│       └── unit/
├── docs/
├── frontend/
│   ├── public/
│   └── src/
│       ├── api/
│       ├── components/
│       ├── context/
│       ├── hooks/
│       ├── pages/
│       │   └── admin/
│       └── styles/
├── infra/
│   ├── helm/
│   │   └── streamflix/
│   │       └── templates/
│   ├── terraform/
│   │   └── modules/
│   │       ├── cloudfront/
│   │       ├── dynamodb/
│   │       ├── ecr/
│   │       ├── eks/
│   │       ├── elasticache/
│   │       ├── iam/
│   │       ├── mediaconvert/
│   │       ├── rds/
│   │       ├── s3/
│   │       └── vpc/
│   └── terragrunt/
│       └── live/
│           ├── dev/
│           └── prod/
└── scripts/
```

## 6. Verify the Result

After running the script, you can inspect the structure from PowerShell:

```powershell
Get-ChildItem -Path .\Streamflix -Recurse -Directory
```

If you have the Windows `tree` command available:

```powershell
tree .\Streamflix /A
```

## 7. Important Notes

- The script uses `New-Item -ItemType Directory -Force`.
- Existing directories are not deleted.
- Running the script multiple times is safe.
- The script reads the tree file rather than requiring every folder to be manually entered.
- The default input file is `streamflix.txt`.
- The default output directory is `Streamflix`.

