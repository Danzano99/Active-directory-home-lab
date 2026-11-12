# Incident Report 001 — FSRM File Screening Blocking Authorized File Operations

**Project:** Active Directory Home Lab  
**Environment:** Windows Server 2022 | VMware Workstation Pro  
**Domain:** Turtle.local  
**Incident Type:** File Access / Security Configuration  
**Severity:** Low — Isolated Lab Environment  
**Status:** Resolved  
**Root Cause:** Confirmed

---

## 1. Executive Summary

During an Active Directory home lab exercise involving shared-folder permissions and access control, an unexpected `Access Denied` error occurred when attempting to restore a JPG image to a shared directory.

The affected account possessed administrative privileges, and initial inspection confirmed that the local Administrators group had Full Control over the destination folder.

Troubleshooting included inspecting NTFS permissions, verifying the source file's attributes and readability, and testing file operations across multiple directories.

The root cause was ultimately identified as an **active File Server Resource Manager (FSRM) file-screening policy** configured to block image, audio, and video files.

Temporarily disabling the applicable file screen allowed the image to be restored. The file screen was subsequently re-enabled.

**Outcome:** The file was successfully restored, and the original file-screening restriction remained in place.

## 2. Environment and Affected Resources

| Component | Details |
|---|---|
| Operating system | Windows Server 2022 |
| Virtualization | VMware Workstation Pro |
| Domain | Turtle.local |
| Server role | Active Directory Domain Controller |
| File-sharing protocol | SMB |
| Resource management | File Server Resource Manager |
| Affected directory | `C:\Shared Folder\Wallpapers` |
| Affected file | `company.jpg` |
| Administrative tools | PowerShell, File Explorer, FSRM, icacls |

The Wallpapers directory was part of a larger shared-folder structure designed to support department-specific access through Active Directory security groups.

Regular employees were intended to have Read access to the Wallpapers folder, while administrators retained Full Control.

## 3. Problem Description

The file `company.jpg` was originally stored inside the Wallpapers directory and used by a Group Policy Object to configure desktop wallpaper for domain users.

During testing, I moved the image from its original location to the administrator's Desktop.

When attempting to return the image to the Wallpapers directory, Windows denied the operation.

An elevated PowerShell attempt produced an error similar to:

```text
Copy-Item : Access to the path
'C:\Shared Folder\Wallpapers\company.jpg'
is denied.
```

The error initially suggested an NTFS permission issue, particularly because the folder had recently undergone permission changes as part of an Active Directory group-based access-control exercise.

## 4. Troubleshooting and Investigation

### Investigation 1 — Verify NTFS permissions

**Objective:** Determine whether the administrator had sufficient permissions to modify the destination folder.

**Command:**

```powershell
icacls "C:\Shared Folder\Wallpapers"
```

**Relevant findings:**

```text
NT AUTHORITY\SYSTEM: (OI)(CI)(F)
BUILTIN\Administrators: (OI)(CI)(F)
TURTLE\DL_Wallpapers_Read: (OI)(CI)(RX,W)
```

<img width="593" height="133" alt="image" src="https://github.com/user-attachments/assets/8ac6aca5-0e4b-4b90-a933-0549a4fae60b" />

The Administrators group had Full Control over the directory.

This suggested that an elevated administrator process should be permitted to create and modify files.

**Assessment:** No obvious missing administrative NTFS permission was identified.

### Investigation 2 — Test file creation

**Objective:** Determine whether all writes to the destination folder were blocked.

**Command:**

```powershell
New-Item -Path "C:\Shared Folder\Wallpapers\permission-test.txt" -ItemType File
```

**Result:** Successful.

A new empty text file was created inside the directory.

**Assessment:** Basic file creation was permitted. The restriction appeared to affect certain operations or file types rather than all writes.

### Investigation 3 — Inspect source file attributes

**Objective:** Determine whether the JPG had restrictive attributes.

**Command:**

```powershell
Get-Item "$env:USERPROFILE\Desktop\company.jpg" |
    Format-List FullName,Attributes,Length,IsReadOnly
```

**Result:** The file was not marked Read-only.

**Assessment:** The JPG's Read-only attribute did not explain the failure.

<img width="522" height="114" alt="image" src="https://github.com/user-attachments/assets/ea340f7d-93ee-44b4-bd5b-b78e4cc2369a" />


### Investigation 4 — Verify source file readability

**Objective:** Determine whether the administrator could successfully read the original JPG.

**Command:**

```powershell
Get-FileHash "$env:USERPROFILE\Desktop\company.jpg"
```

**Result:** A SHA-256 hash was successfully generated.

**Assessment:** The original file was readable.

<img width="977" height="108" alt="image" src="https://github.com/user-attachments/assets/3728b184-dcf3-4616-a3ae-71c3b7aac1fe" />


### Investigation 5 — Compare destination directories

**Objective:** Determine whether the file-copy failure occurred across the server or only in the affected directory.

I attempted to copy the original JPG to a separate FSRM test directory.

**Command:**

```powershell
Copy-Item "$env:USERPROFILE\Desktop\company.jpg" `
    "C:\FSRM-Test\wallpaper-test.jpg"
```

**Result:** Successful.

<img width="669" height="63" alt="image" src="https://github.com/user-attachments/assets/f2c49a35-569c-4419-94b4-0911ecffe46a" />


I then attempted to copy the image into the Wallpapers directory using a different filename.

**Result:** Access Denied.

<img width="959" height="187" alt="image" src="https://github.com/user-attachments/assets/24ca19a1-cc34-401b-85dd-650dcf214a53" />


**Assessment:** The problem was specific to the destination directory or a control affecting that directory. The source file itself was not the issue.

### Investigation 6 — Review FSRM file-screening policies

**Objective:** Identify whether another Windows Server feature was blocking the file operation despite sufficient NTFS permissions.

I reviewed the configuration under:

`File Server Resource Manager → File Screening Management → File Screens`

An existing active file-screening rule was identified that blocked the following file categories:

- Image files
- Audio files
- Video files

The rule applied to the shared directory containing the Wallpapers folder.

**Finding:** The JPG matched a prohibited file category and was being blocked by FSRM.

## 5. Root Cause Analysis

The root cause was an **FSRM active file screen configured to prevent image files from being saved within the shared-folder structure**.

The file-screening policy had been enabled after `company.jpg` was originally placed in the Wallpapers directory.

FSRM did not automatically remove the existing image when the rule was activated.

However, once the JPG was moved out of the directory, restoring it required a new file-write operation. FSRM evaluated the operation against the active file screen and blocked it.

This explains why:

- The original JPG could exist in the directory before it was moved.
- The administrator could create an empty text file.
- The administrator could copy the JPG to another directory.
- The administrator could not restore the JPG to the protected Wallpapers directory.

The failure was caused by FSRM file screening rather than missing NTFS permissions.

## 6. Resolution

The following steps were taken:

1. Opened File Server Resource Manager.
2. Located the active file screen affecting the Wallpapers directory.
3. Temporarily disabled the applicable file screen.
4. Restored `company.jpg` to its original location.
5. Re-enabled the file screen.

**Result:** The image was successfully restored, and the original file-screening restriction was reinstated.

The final configuration maintained the lab's existing file-type restrictions while preserving the wallpaper required by the Desktop Wallpaper GPO.

## 7. Lessons Learned

**Layered security controls**

NTFS permissions are not the only controls that determine whether a file operation succeeds. Additional mechanisms, including FSRM file screening, can deny operations even when an account possesses administrative permissions.

**Systematic troubleshooting**

Testing file creation, source-file readability, and multiple destinations helped narrow the problem without unnecessarily modifying permissions.

**Configuration dependencies**

A security policy intended to restrict unwanted files can interfere with legitimate administrative operations if exceptions are not considered.

**Administrative privileges**

Running a process with elevated permissions does not automatically bypass file-screening policies.

**Importance of documenting changes**

The file-screening rule was created during an earlier exercise. Remembering that configuration ultimately helped identify the problem. Maintaining a configuration-change log would make similar investigations faster.

## 8. Future Improvements

To improve the configuration, I plan to:

- Evaluate an FSRM file-screening exception for the Wallpapers directory.
- Keep employee wallpaper access restricted to Read & Execute.
- Continue using Domain Local security groups to control resource permissions.
- Document security-policy changes and their intended scope.
- Validate newly configured restrictions against both authorized and unauthorized operations.

## 9. Conclusion

This incident demonstrated the importance of investigating multiple security layers when troubleshooting Windows Server file-access failures.

Although the initial symptoms suggested an NTFS permission issue, controlled testing established that the destination folder could accept other file operations and that the JPG was readable.

Reviewing FSRM configuration revealed the actual cause: an active file-screening policy blocking image files.

The issue was resolved without removing the existing departmental access-control structure or permanently weakening the file-screening policy.

---

**Skills Demonstrated:** Windows Server Administration · FSRM · NTFS Permissions · PowerShell · Access Control · Root Cause Analysis · Technical Troubleshooting
