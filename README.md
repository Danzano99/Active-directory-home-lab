# Active Directory Home Lab

## Project Overview

This project documents the development of a virtualized Windows Active Directory environment designed to simulate a small enterprise network.

The purpose of this lab is to develop hands-on experience with Windows Server administration, identity and access management, Group Policy, file sharing, and security troubleshooting.

The lab is an ongoing learning project. New configurations, administrative exercises, and troubleshooting case studies will be documented as they are completed.

## Lab Environment

| Component | Technology |
|---|---|
| Hypervisor | VMware Workstation Pro |
| Domain Controller | Windows Server 2022 |
| Client Workstation | Windows 11 Enterprise |
| Active Directory Domain | Turtle.local |
| Directory Services | Active Directory Domain Services |
| Policy Management | Group Policy Management |
| File Sharing | SMB and NTFS |
| Storage Management | File Server Resource Manager |
| Administration | PowerShell, ADUC, Event Viewer |

## Active Directory Structure

The domain is organized into three geographical regions:

- USA
- Europe
- Asia

The USA region currently contains departmental organizational units for IT, Accounting, HR, Sales, and Management.

Security groups are used to manage employee membership and access to departmental resources.

### Group-Based Access Control

The lab uses the AGDLP model:

**Accounts → Global Groups → Domain Local Groups → Permissions**

For example, IT employees are members of `GG_USA_IT`, which is nested inside `DL_IT_Modify`. The Domain Local group is assigned Modify permissions on the IT shared folder.

This allows permissions to be managed through group membership rather than individual user accounts.

## Implemented Features

### Active Directory
- Created regional and departmental organizational units
- Created domain user and computer accounts
- Configured Global and Domain Local security groups
- Implemented nested group membership

### Group Policy
- Configured a desktop wallpaper policy
- Deployed a mapped network drive
- Tested Control Panel restrictions
- Configured removable-storage restrictions for testing

### File Sharing and Permissions
- Created an SMB network share
- Configured Share and NTFS permissions
- Implemented department-specific folder access
- Tested authorized and unauthorized access using different employee accounts

### File Server Resource Manager
- Created file storage quotas
- Configured notification thresholds
- Tested file-screening restrictions

## Troubleshooting and Lessons Learned

During the development of this lab, several issues required investigation:

**Group Policy Administrative Lockout** — Investigated a restrictive Group Policy that unintentionally affected the Domain Administrator account.

**Network Drive Mapping** — Troubleshot a network drive that appeared connected through Command Prompt but was not accessible in File Explorer.

**FSRM Management Console Failure** — Investigated errors involving the FSRM MMC snap-in and PowerShell quota enumeration.

**FSRM File Screening** — Identified a file-screening rule that prevented an administrator from restoring a JPG to a shared folder despite sufficient NTFS permissions.

Detailed troubleshooting reports will be added as separate Markdown documents.

## Future Development

Planned exercises include Windows security auditing, additional Group Policy configurations, user lifecycle management, permission-change investigations, and Windows Event Log analysis.

## Disclaimer

This project is an isolated educational home lab. It is not a production enterprise deployment. All accounts and resources used for testing are part of the simulated environment.
