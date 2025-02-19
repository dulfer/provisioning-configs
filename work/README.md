# Work host installation

Documentation around preparing a freshly installed Windows host for work usage.

> 🛡️ the installer will request to run as administrator, expect a prompt.

## Prerequisites 🛡️

Winget package manager and latest PowerShell version are required.

```pwsh
# Setup winget package manager
Install-Module Microsoft.WinGet.Client
Repair-WinGetPackageManager -Force -Latest

# Microsoft PowerShell 🛡️
winget install --id=Microsoft.PowerShell -e

# Allow local script execution
Set-ExecutionPolicy -ExecutionPolicy Remotesigned

# Create Symlink for PowerShell Modules (storing locally instead of OneDrive) 🛡️
New-Item -ItemType Directory -Force -Path "c:\Users\d.dulfer\.pwsh\modules"
New-Item -Path "c:\Users\d.dulfer\OneDrive - Fugro\Documents\PowerShell\Modules" -ItemType SymbolicLink -Value "c:\Users\d.dulfer\.pwsh\modules" -Force
```

## Setup OneDrive and remap folder locations

```pwsh
# Define the new path for the Documents folder
$onedrivePath = "$env:USERPROFILE\OneDrive - Fugro\"

# Update the registry to point to the new location
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" -Name "Personal" -Value "$onedrivePath\Documents"
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -Name "Personal" -Value "$onedrivePath\Documents"

Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" -Name "My Pictures" -Value "$onedrivePath\Pictures"
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" -Name "My Pictures" -Value "$onedrivePath\Pictures"

# Notify the system of the change
[System.Runtime.InteropServices.Marshal]::ReleaseComObject([System.Runtime.InteropServices.Marshal]::GetActiveObject("Shell.Application"))

```

## Packages

### Core

```pwsh

# Windows Terminal (Preview)
winget install --id=Microsoft.WindowsTerminal.Preview -e

# Azure CLI 🛡️
winget install --id=Microsoft.AzureCLI -e

```

### Development

```pwsh

# Git
winget install --id=Git.Git -e

# Add git path to environment variables
$gitPath = "$env:USERPROFILE\AppData\Local\Programs\Git\cmd"
$currentPath = [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::User)
if ($currentPath -notlike "*$gitPath*") {
    $newPath = "$currentPath;$gitPath"
    [System.Environment]::SetEnvironmentVariable("Path", $newPath, [System.EnvironmentVariableTarget]::User)
    Write-Output "Git path added to the environment PATH."
} else {
    Write-Output "Git path is already in the environment PATH."
}
# reload path
$Env:Path = [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::Machine) + ";" + [System.Environment]::GetEnvironmentVariable("Path", [System.EnvironmentVariableTarget]::User)

# Configure Git
git config --global user.name "dulfer"
git config --global user.email "30404904+dulfer@users.noreply.github.com"
git config --global core.editor "code --wait"  # Sets Visual Studio Code as the default editor
git config --global init.defaultBranch main # set default branch name to 'main'
git config --global credential.helper cache # enable Credential Caching

# Bicep 
winget install --id=Microsoft.Bicep -e

# Terraform
winget install --id=HashiCorp.Terraform

# Visual Studio Code
winget install --id=Microsoft.VisualStudioCode -e

# Code Extensions
@codeExtensions = @("azps-tools.azps-tools",
    "azure-automation.vscode-azureautomation",
    "davidanson.vscode-markdownlint",
    "bierner.markdown-mermaid",
    "esbenp.prettier-vscode",
    "github.copilot",
    "github.copilot-chat",
    "github.remotehub",
    "github.vscode-pull-request-github",
    "grapecity.gc-excelviewer",
    "hashicorp.terraform",
    "mechatroner.rainbow-csv",
    "ms-azuretools.azure-dev",
    "ms-azuretools.vscode-azure-github-copilot",
    "ms-azuretools.vscode-azureappservice",
    "ms-azuretools.vscode-azurecontainerapps",
    "ms-azuretools.vscode-azurefunctions",
    "ms-azuretools.vscode-azureresourcegroups",
    "ms-azuretools.vscode-azurestaticwebapps",
    "ms-azuretools.vscode-azurestorage",
    "ms-azuretools.vscode-azurevirtualmachines",
    "ms-azuretools.vscode-bicep",
    "ms-azuretools.vscode-cosmosdb",
    "ms-azuretools.vscode-docker",
    "ms-dotnettools.vscode-dotnet-runtime",
    "ms-python.debugpy",
    "ms-python.python",
    "ms-python.vscode-pylance",
    "ms-vscode-remote.remote-containers",
    "ms-vscode-remote.remote-ssh",
    "ms-vscode-remote.remote-ssh-edit",
    "ms-vscode-remote.remote-wsl",
    "ms-vscode-remote.vscode-remote-extensionpack",
    "ms-vscode.azure-repos",
    "ms-vscode.powershell",
    "ms-vscode.remote-explorer",
    "ms-vscode.remote-repositories",
    "ms-vscode.remote-server",
    "ms-vscode.vscode-node-azure-pack",
    "ms-vsliveshare.vsliveshare",
    "redhat.vscode-xml",
    "redhat.vscode-yaml",
    "redhat.ansible",
    "vscode-icons-team.vscode-icons",
    "yzhang.markdown-all-in-one")

# Install each extension
foreach ($extension in $codeExtensions) {
    code --install-extension $extension
}

```

### Productivity

```pwsh

# oh-my-posh
winget install --id=JanDeDobbeleer.OhMyPosh -e
Install-Module Terminal-Icons -Scope CurrentUser # install nice icons

# Logseq
winget install --id=Logseq.Logseq -e

# PowerToys
winget install --id=Microsoft.PowerToys -e

# Windows App
winget install --id=9N1F85V9T8BN -e

# Keeper Security Desktop (optional)
winget install --id=KeeperSecurity.KeeperDesktop -e

# Paint.Net 🛡️
winget install --id=dotPDN.PaintDotNet -e
```

### Vanity

```pwsh
# Install Cascadia Code fonts
# NF (Nerd Font) variant adds symbols to terminal window
# 🔗 https://github.com/microsoft/cascadia-code

$repository = "microsoft/cascadia-code"
$gh_releases = "https://api.github.com/repos/$repo/releases"
$filename = "CascadiaCode-<version>.zip"

Write-Host "🔎 Lookup latest 'Cascadia Code' release"
$tag = (Invoke-WebRequest $gh_releases | ConvertFrom-Json)[0].tag_name
# project team names release archives without the 'v' in the Tag name
$filename = $filename.Replace("<version>", $tag.Substring(1))
$tempFolder = (Join-Path $env:USERPROFILE "AppData\Local\Temp")

$download = "https://github.com/$repository/releases/download/$tag/$filename"
$dir = "$name-$tag"

Write-Host "🌍 Dowloading latest release"
Invoke-WebRequest $download -Out (Join-Path $tempFolder $filename)

Write-Host "📤 Extracting release archive"
Expand-Archive (Join-Path $env:TEMP $filename) -Force -Destination (Join-Path $tempFolder $tag)

$FontList = Get-ChildItem -Path (Join-Path $tempFolder $tag) -Include ('*.fon','*.otf','*.ttc','*.ttf') -Recurse
foreach ($font in $FontList) {
    Write-Host '🪛 Installing font: ' $Font.BaseName
    Copy-Item $font (Join-Path $env:USERPROFILE "AppData\Local\Microsoft\Windows\Fonts") -Force -ErrorAction SilentlyContinue # copy to user profile font directory
    New-ItemProperty -Name $font.BaseName -Path "HKCU:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Fonts" -PropertyType string -Value $Font.name -ErrorAction SilentlyContinue # register font for current user 
}

# Cleanup font release files and folders
Remove-Item (Join-Path $tempFolder $tag) -Recurse -Force -ErrorAction SilentlyContinue 
Remove-Item (Join-Path $tempFolder $filename) -Force -ErrorAction SilentlyContinue


```
