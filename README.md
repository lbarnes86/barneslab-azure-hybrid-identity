# BarnesLab — Azure AD Connect + Hybrid Identity Lab

![Status](https://img.shields.io/badge/Status-Active-brightgreen)
![Platform](https://img.shields.io/badge/Platform-VMware%20Workstation-blue)
![Azure](https://img.shields.io/badge/Azure-Entra%20ID-0078D4?logo=microsoftazure)
![OS](https://img.shields.io/badge/OS-Windows%20Server%202022-0078D4?logo=windows)

## Overview

BarnesLab Hybrid Identity is Phase 3 of the BarnesLab home lab series. This project extends the existing on-premises Active Directory environment by synchronizing it with Microsoft Azure Entra ID (formerly Azure AD) using Azure AD Connect — creating a hybrid identity architecture that mirrors what enterprises use in real-world Microsoft 365 and Azure deployments.

This lab demonstrates hands-on proficiency in hybrid identity configuration, Azure AD Connect sync, Conditional Access policy enforcement, Multi-Factor Authentication (MFA), and Entra ID user and group management — skills directly aligned with Systems Administrator, Cloud Administrator, and Cloud Security Engineer roles.

📺 **Demo Video:** [Watch the BarnesLab Hybrid Identity Walkthrough on YouTube](#) *(link coming soon)*

➡️ **Prerequisite:** [BarnesLab Active Directory Home Lab](https://github.com/lbarnes86/barneslab-active-directory-lab) 

---

## Lab Architecture

```
+-----------------------------+          +-----------------------------+
|      ON-PREMISES            |          |       MICROSOFT AZURE       |
|                             |          |                             |
|   BARNESLAB-DC              |  Azure   |   Entra ID Tenant           |
|   Windows Server 2022       |  AD      |   (yourdomain.onmicro       |
|   barneslab.local           |  Connect |    soft.com)                |
|   IP: 192.168.10.10         +--------->+                             |
|                             |  Sync    |   Synced Users & Groups     |
|   On-Prem Users:            |          |   Cloud-Only Users          |
|   jsmith, mjohnson, rbrown  |          |   Conditional Access        |
|                             |          |   MFA Enforcement           |
+-----------------------------+          +-----------------------------+
           |
    VMware NAT Network
    Subnet: 192.168.10.0/24
```

---

## Environment Specifications

| Component | Details |
|---|---|
| Host Machine | Lenovo ThinkCentre M73 Tiny |
| Virtualization Platform | VMware Workstation Pro 17 |
| On-Premises Domain | barneslab.local |
| Domain Controller | BARNESLAB-DC (Windows Server 2022) |
| Azure Tenant | Free Azure Account (portal.azure.com) |
| Azure Tenant Domain | yourdomain.onmicrosoft.com |
| Sync Tool | Azure AD Connect (Microsoft Entra Connect) |
| Identity Platform | Microsoft Entra ID (Azure AD) |

---

## Virtual Machines

| VM Name | OS | Role | IP Address |
|---|---|---|---|
| BARNESLAB-DC | Windows Server 2022 | Domain Controller / Azure AD Connect Host | 192.168.10.10 |
| WKSTN1 | Windows 10 Enterprise | Domain-Joined Test Workstation | 192.168.10.20 |

---

## What I Built

### Azure Free Tenant Setup
- Created a free Microsoft Azure account at portal.azure.com
- Configured a new Entra ID tenant with a default domain (onmicrosoft.com)
- Created cloud-only test users and assigned Microsoft 365 trial licenses
- Reviewed tenant settings including security defaults and directory roles

### Azure AD Connect Deployment
- Downloaded and installed Microsoft Entra Connect on BARNESLAB-DC
- Selected Express Settings for lab simplicity
- Authenticated with Azure Global Administrator credentials
- Authenticated with on-premises barneslab.local domain administrator credentials
- Initiated first directory synchronization — all on-prem users, groups, and OUs synced to Entra ID
- Verified synced objects in Azure portal showing "Synced from on-premises" status

### Directory Synchronization Validation
- Confirmed all on-premises AD users (jsmith, mjohnson, rbrown) appeared in Entra ID after sync
- Verified UPN (User Principal Name) mapping between on-prem and cloud identities
- Ran manual sync using PowerShell: `Start-ADSyncSyncCycle -PolicyType Delta`
- Tested user attribute updates — modified on-prem display name, confirmed it propagated to Entra ID within sync cycle

### Conditional Access Policies
- Navigated to Entra ID > Security > Conditional Access
- Created the following policies:

| Policy Name | Users | Condition | Control |
|---|---|---|---|
| Require MFA — All Users | All synced users | Any cloud app | Require MFA |
| Block Legacy Authentication | All users | Legacy auth clients | Block access |
| Require Compliant Device | IT-Admins group | Azure portal access | Require compliant device |

- Tested MFA prompt by signing in to portal.azure.com as jsmith — MFA registration was enforced
- Validated legacy auth block by attempting basic auth connection — access was denied as expected

### Multi-Factor Authentication (MFA)
- Registered Microsoft Authenticator app for test user accounts
- Walked through MFA registration flow for jsmith
- Verified MFA prompt appears on subsequent logins
- Reviewed MFA registration status in Entra ID > Users > Authentication Methods

### Role-Based Access Control (RBAC) in Azure
- Assigned the following Entra ID directory roles to test users:

| User | Role Assigned | Justification |
|---|---|---|
| jsmith | Security Reader | IT admin with read-only security visibility |
| mjohnson | User Administrator | HR user lifecycle management |
| Administrator | Global Administrator | Full tenant control |

- Verified role assignments under Entra ID > Roles and Administrators

### Self-Service Password Reset (SSPR)
- Enabled SSPR for all users in the tenant
- Configured authentication methods: email + security questions
- Tested SSPR flow from the login page using a test account
- Verified password reset completed successfully without admin intervention

---

## Key Commands Reference

### Azure AD Connect — PowerShell Sync Management
```powershell
# Import the ADSync module
Import-Module ADSync

# Run a delta sync (syncs changes since last sync)
Start-ADSyncSyncCycle -PolicyType Delta

# Run a full sync (syncs all objects)
Start-ADSyncSyncCycle -PolicyType Initial

# Check sync status
Get-ADSyncConnectorRunStatus

# View sync errors
Get-ADSyncRunStepResult | Where-Object {$_.Result -ne "success"}
```

### Verify Synced Users in Azure via PowerShell
```powershell
# Install Azure AD module if needed
Install-Module AzureAD

# Connect to Azure AD
Connect-AzureAD

# List all synced users
Get-AzureADUser | Where-Object {$_.DirSyncEnabled -eq $true} | Select DisplayName, UserPrincipalName, DirSyncEnabled

# List cloud-only users
Get-AzureADUser | Where-Object {$_.DirSyncEnabled -ne $true} | Select DisplayName, UserPrincipalName
```

### On-Premises AD — User Attribute Updates for Sync Testing
```powershell
# Update a user's display name on-prem to test sync propagation
Set-ADUser -Identity jsmith -DisplayName "John Smith - IT Lead"

# Force delta sync after change
Start-ADSyncSyncCycle -PolicyType Delta

# Verify the change appeared in Azure (wait 1-2 minutes then check portal)
```

---

## Troubleshooting Scenarios Practiced

| Issue | Root Cause | Resolution |
|---|---|---|
| Azure AD Connect install failed | .NET Framework version too low | Installed .NET 4.8 on DC, re-ran installer |
| Users not appearing in Entra ID after sync | UPN suffix mismatch between on-prem and Azure | Added barneslab.local as verified domain or changed UPN suffix to match Azure domain |
| MFA not prompting after Conditional Access policy | Policy set to Report-Only mode | Changed policy state from Report-Only to On |
| Sync errors on specific user accounts | Special characters in on-prem display name | Removed special characters from AD attributes |
| Cannot sign in to Azure portal as synced user | Account not licensed in Azure | Assigned Entra ID free license to user |
| SSPR not working | Authentication methods not configured | Enabled email and security question methods in SSPR settings |

---

## Skills Demonstrated

| Skill Category | Specific Skills |
|---|---|
| Hybrid Identity | Azure AD Connect installation, configuration, and sync management |
| Microsoft Entra ID | Tenant setup, user management, group management, directory roles |
| Conditional Access | Policy creation, MFA enforcement, legacy auth blocking, device compliance |
| Multi-Factor Authentication | MFA registration, authentication methods, SSPR configuration |
| Azure RBAC | Directory role assignment, least-privilege access, role validation |
| PowerShell | ADSync module, AzureAD module, sync cycle management, user queries |
| Troubleshooting | Sync error diagnosis, UPN mismatch resolution, policy mode correction |
| Cloud Administration | Azure portal navigation, Entra ID administration, license management |
| Documentation | Architecture diagrams, policy tables, command references, runbooks |

---

## Screenshots

> Screenshots are included in the `/screenshots` folder of this repository.

- `azure-ad-connect-sync.png` — Azure AD Connect dashboard showing successful sync
- `synced-users-entra.png` — Entra ID users list showing on-prem synced accounts
- `conditional-access-policy.png` — Conditional Access policy configuration screen
- `mfa-prompt.png` — MFA registration prompt for jsmith on first Azure login
- `rbac-assignments.png` — Directory role assignments in Entra ID
- `sspr-success.png` — Self-service password reset completion screen

---

## Project Phases

| Phase | Project | Status |
|---|---|---|
| Phase 1 | Active Directory Home Lab | ✅ Complete |
| Phase 2 | Splunk SIEM + Attack Detection | ✅ Complete |
| Phase 3 | Azure AD Connect + Hybrid Identity | ✅ Complete |
| Phase 4 | Cloud Security Monitoring (Microsoft Sentinel) | 📋 Planned |

---

## Resources Used

- [Microsoft Entra Connect Download](https://www.microsoft.com/en-us/download/details.aspx?id=47594)
- [Azure Free Account](https://azure.microsoft.com/en-us/free)
- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Conditional Access Documentation](https://learn.microsoft.com/en-us/entra/identity/conditional-access/)
- [Azure AD Connect Sync Documentation](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-sync-whatis)
- [Self-Service Password Reset](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-howitworks)
- [AzureAD PowerShell Module](https://learn.microsoft.com/en-us/powershell/module/azuread/)

---

## About This Project

Built by **Lloyd Barnes** — Systems Administrator | Cloud Security | Hybrid Identity | CompTIA CySA+

- 🔗 LinkedIn: [linkedin.com/in/lloyd-barnes-ii](https://www.linkedin.com/in/lloyd-barnes-ii)
- 🏅 Credly: [credly.com/users/lloyd-barnes.6c44ecd1](https://www.credly.com/users/lloyd-barnes.6c44ecd1)
- 💻 GitHub: [github.com/lbarnes86](https://www.github.com/lbarnes86)
