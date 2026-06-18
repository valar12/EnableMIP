# EnableMIP - Enable MIP Sensitivity Labels

PowerShell automation toolkit for enabling Microsoft Information Protection (MIP) sensitivity label prerequisites across Commercial and USGov Microsoft 365 tenants.

## Overview

This repository contains the `Enable-MIPSensitivityLabels.ps1` PowerShell command for preparing a tenant for MIP sensitivity labels, including Microsoft 365 Groups, SharePoint Sites, OneDrive, co-authoring/PDF options, and Azure AD label synchronization.

## Features

- ✅ Automated installation for missing or outdated modules (Microsoft Graph, Exchange Online, SharePoint Online)
- ✅ Microsoft Graph directory settings configuration for Groups and Sites
- ✅ SharePoint Online Azure Information Protection (AIP) integration
- ✅ Security & Compliance Center label synchronization
- ✅ Comprehensive error handling and logging
- ✅ Interactive authentication with MFA support
- ✅ Commercial and USGov tenant endpoint support
- ✅ PowerShell 7 SharePoint Online module import compatibility (`-UseWindowsPowerShell`)
- ✅ Optional co-authoring and PDF sensitivity label enablement
- ✅ Idempotent configuration checks (safe to run multiple times without reapplying settings that are already enabled)
- ✅ Detailed progress reporting and success verification

## Scripts Included

### Enable-MIPSensitivityLabels.ps1
The primary script that configures the Microsoft Information Protection tenant prerequisites for sensitivity labels across Microsoft 365 services.

**What it does:**
1. Installs/updates required PowerShell modules
2. Connects to Microsoft Graph with appropriate permissions
3. Enables MIP labels for Microsoft 365 Groups
4. Enables MIP labels for SharePoint Sites
5. Enables SharePoint Online AIP integration
6. Optionally enables co-authoring and PDF sensitivity label support
7. Connects to Security & Compliance Center with Commercial or USGov endpoints
8. Syncs sensitivity labels to Azure AD

## Prerequisites

- PowerShell 5.1 or later
- Global Administrator or Compliance Administrator role
- Microsoft 365 tenant with appropriate licenses
- Internet connection for module installation and Microsoft 365 connectivity

## Required Permissions

- `Directory.ReadWrite.All` (Microsoft Graph)
- Exchange Online administrative access
- SharePoint Online administrative access
- Security & Compliance Center access

## Installation

1. Clone this repository:
   ```powershell
   git clone https://github.com/GarthVDW/EnableMIP.git
   cd EnableMIP
   ```

2. Ensure your execution policy allows running scripts:
   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
   ```

## Usage

### Basic Usage
Run the `Enable-MIPSensitivityLabels.ps1` command with default Commercial settings:
```powershell
.\Enable-MIPSensitivityLabels.ps1 -SharePointAdminUrl https://contoso-admin.sharepoint.com
```

### USGov Tenant
Run against a USGov tenant by selecting the USGov environment and providing your `.us` SharePoint admin center URL:
```powershell
.\Enable-MIPSensitivityLabels.ps1 `
  -TenantEnvironment USGov `
  -UserPrincipalName admin@contoso.onmicrosoft.us `
  -SharePointAdminUrl https://contoso-admin.sharepoint.us
```

For USGov, the script uses Microsoft Graph `USGov`, the compliance PowerShell endpoint `https://ps.compliance.protection.office365.us/powershell-liveid/`, and the authorization endpoint `https://login.microsoftonline.us/organizations`.

### Skip Module Installation
If modules are already installed:
```powershell
.\Enable-MIPSensitivityLabels.ps1 -SharePointAdminUrl https://contoso-admin.sharepoint.com -SkipModuleInstall
```

### Custom Log Path
Specify a custom location for log files:
```powershell
.\Enable-MIPSensitivityLabels.ps1 -SharePointAdminUrl https://contoso-admin.sharepoint.com -LogPath "C:\Logs\M365"
```

### Force Module Reinstall
Force reinstallation of all modules:
```powershell
.\Enable-MIPSensitivityLabels.ps1 -SharePointAdminUrl https://contoso-admin.sharepoint.com -Force
```

### Enable Co-authoring and PDF Support
Enable optional tenant settings for co-authoring with sensitivity labels and PDF sensitivity label support:
```powershell
.\Enable-MIPSensitivityLabels.ps1 `
  -SharePointAdminUrl https://contoso-admin.sharepoint.com `
  -EnableCoauthoring `
  -EnablePdfSupport
```

### PowerShell 7 SharePoint Online Import
When running in PowerShell 7, the SharePoint Online module must be imported through Windows PowerShell compatibility. The script now does this automatically with the equivalent of:
```powershell
Import-Module Microsoft.Online.SharePoint.PowerShell -UseWindowsPowerShell
```

## Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `LogPath` | String | No | Path for transcript logs (default: current directory) |
| `SkipModuleInstall` | Switch | No | Skip module installation if already installed |
| `Force` | Switch | No | Force reinstall modules even if present |
| `TenantEnvironment` | String | No | Target cloud environment: `Commercial` or `USGov` (default: `Commercial`) |
| `UserPrincipalName` | String | No | Admin account for Exchange Online Protection / Purview PowerShell connection |
| `SharePointAdminUrl` | String | No | SharePoint admin center URL, such as `https://contoso-admin.sharepoint.com` or `https://contoso-admin.sharepoint.us`; required before the SharePoint Online configuration step |
| `EnableCoauthoring` | Switch | No | Runs `Set-PolicyConfig -EnableLabelCoauth $true` |
| `EnablePdfSupport` | Switch | No | Runs `Set-SPOTenant -EnableSensitivityLabelforPDF $true` |

## What Gets Configured

1. **Microsoft Graph Directory Settings**
   - Enables `EnableMIPLabels` setting in Group.Unified template
   - Allows sensitivity labels on Microsoft 365 Groups

2. **SharePoint Online**
   - Enables Azure Information Protection integration
   - Allows sensitivity labels on SharePoint Sites and OneDrive

3. **Co-authoring and PDF Support (Optional)**
   - Enables `EnableLabelCoauth` for co-authoring files with sensitivity labels
   - Enables `EnableSensitivityLabelforPDF` for PDF sensitivity label support in SharePoint and OneDrive

4. **Azure AD Label Sync**
   - Synchronizes sensitivity labels from Microsoft Purview to Azure AD
   - Makes labels available across all Microsoft 365 services


## Idempotent Behavior

The command checks current tenant and local module state before making changes:

- Required PowerShell modules are skipped when the installed version already meets the minimum requirement, unless `-Force` is used.
- Microsoft Graph reconnects only when the existing session targets a different cloud environment.
- `EnableMIPLabels`, SharePoint AIP integration, PDF support, and label co-authoring are only set when their current values are not already enabled.
- Label sync can be run repeatedly; Microsoft 365 handles synchronization as an idempotent refresh.

## Logging

All script executions create detailed transcript logs with timestamps:
- Format: `EnableMIPSensitivityLabels_YYYYMMDD-HHMMSS.log`
- Location: Specified by `-LogPath` parameter or current directory
- Contains: All commands, output, errors, and stack traces

## Post-Configuration Steps

After running the script:

1. **Wait for propagation** - Changes may take up to 24 hours to reflect across all services
2. **Create sensitivity labels** in Microsoft Purview compliance portal
3. **Publish labels** to appropriate users and groups
4. **Test label application** on Teams, Groups, and SharePoint Sites
5. **Verify labels** appear in SharePoint and OneDrive

## Troubleshooting

### "Function capacity exceeded" Error
The script uses targeted Microsoft Graph modules to avoid this issue. If you still encounter it, try:
```powershell
# Close and reopen PowerShell, then run:
.\Enable-MIPSensitivityLabels.ps1 -SharePointAdminUrl https://contoso-admin.sharepoint.com -SkipModuleInstall
```

### Authentication Issues
If you encounter authentication errors:
1. Ensure you have the required admin roles
2. Try closing and reopening PowerShell as Administrator
3. Clear cached credentials: `Disconnect-MgGraph` and `Disconnect-ExchangeOnline`

### Labels Not Appearing
- Wait up to 24 hours for full propagation
- Verify labels are published to your users
- Check that users are licensed appropriately

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

MIT License - feel free to use and modify for your organization's needs.

## Author

Garth van der Woude  
Created: November 7, 2025

## Disclaimer

This script is provided as-is with no warranties. Always test in a non-production environment first. Ensure you have proper backups and understand the changes being made to your Microsoft 365 tenant.

## Support

For issues, questions, or contributions, please open an issue in this repository.
