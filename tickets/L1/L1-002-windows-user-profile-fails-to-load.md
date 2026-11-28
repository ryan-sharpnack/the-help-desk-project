# [L1-002: Windows User Profile Fails to Load]

| Field | Details |
|---|---|
| **Ticket ID** | L1-002 |
| **Status** | Resolved |
| **Difficulty** | Intermediate |
| **Category** | Windows; User Profiles |
| **Environment Type** | Homelab |
| **Author** | ryan-sharpnack |
| **Date Resolved** | 2026-11-27 |

## Environment

- **Virtualization Platform:** Proxmox VE
- **VM ID:** 100
- **VM Name:** WIN11-CLIENT
- **Operating System:** Windows 11 Enterprise Evaluation, Version 26H2
- **Affected Account:** profiletest (local standard user)
- **Administrative Account:** labadmin
- **Profile Directory:** C:\Users\profiletest

A disposable standard user account named profiletest was used to reproduce the issue. The existing labuser and labadmin accounts were not modified.

A Proxmox snapshot was created before introducing the controlled fault:

L1-002-Clean-Profile-Baseline

## Problem Description

A Windows user is unable to load their normal user profile during sign-in. Windows displays a notification stating that it cannot sign in to the account and may load a temporary profile instead.

This scenario was reproduced in the homelab by deliberately changing the affected account's ProfileImagePath registry value to an incorrect directory.

## Business Impact

The user may be able to access the Windows desktop through a temporary profile, but the normal user environment is unavailable.

Changes made while using a temporary profile may not persist after signing out. The user may also lose access to their normal desktop configuration, application preferences, and profile-specific settings.

## Symptoms

- Windows displays: **"We can't sign in to your account."**
- Windows presents **Sign out** and **Close** options.
- A desktop loads, but it does not reflect the original user's profile.
- A TEMP directory is visible under C:\Users.
- The original profile directory remains present.
- The original profile's registry hive file, NTUSER.DAT, remains present.

## Troubleshooting

1. **Step:** Identify the affected account and SID
   **Why:**  Identify the affected local account and obtain its Security Identifier (SID) for registry inspection.
   **Action:** Executed the following PowerShell command from an elevated PowerShell session: Get-LocalUser -Name "profiletest" | Select-Object Name, SID, Enabled
   **Result:** The account was present and enabled. SID: S-1-5-21-3786117702-1378666247-2130385978-1003

2. **Step:** Inspect the existing user profile
   **Why:** Establish the profile's baseline status before introducing the controlled failure.
   **Action** Executed:

   Get-CimInstance Win32_UserProfile |
Where-Object { $_.LocalPath -eq "C:\Users\profiletest" } |
Select-Object SID, LocalPath, Loaded, Special

   **Result:**
   - The expected profile directory was identified.
   - Loaded was False because the affected account was signed out.
   - Special was False.
  
3. **Step:** Inspect the profile registry configuration
   **Why:** Verify that Windows has the expected registry mapping for the affected user profile.

   **Registry Location:**

   HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList

   **Action:** Inspected the SID key ending in 1003 using Registry Editor.
   **Result:**
   - ProfileImagePath pointed to C:\Users\profiletest.
   - State was 516.
   - RefCount was not present.
   - No .bak registry key existed before introducing the fault.
  
   The original profile directory and NTUSER.DAT file were present.

4. **Step:** Create an additional profile hive backup
   **Why:** Preserve an additional copy of the original registry hive before introducing the controlled failure.
   **Action:**

   Executed:

   Copy-Item "C:\Users\profiletest\NTUSER.DAT" "C:\Users\profiletest\NTUSER.DAT.L1-002-backup"

   Verified the copy:

   Test-Path "C:\Users\profiletest\NTUSER.DAT.L1-002-backup"

   **Result:** Returned True. The additional backup file was successfully created.

5. **Step:** Introduce the controlled profile configuration fault
   **Why:** Reproduce a profile-loading failure in a disposable account without affecting the other local accounts.
   **Action:** In Registry Editor, modified the ProfileImagePath value under the affected user's SID key.

   Original value:

   C:\Users\profiletest

   Test value:

   C:\Users\profiletest-broken

   **Result:** The incorrect registry path was confirmed. No profile files were deleted or modified during this step.

6. **Step:** Reproduce the sign-in failure
   **Why:** Observe Windows behavior when the affected account's profile path points to a nonexistent directory.
   **Action:** Signed out of labadmin and attempted to sign in as profiletest.
   **Result:**

   Windows displayed:

   We can't sign in to your account.

   A desktop loaded, and a TEMP directory was visible under C:\Users.

   The expected profiletest-broken directory was not present.

   The behavior was consistent with Windows loading a temporary user profile rather than the original profile.

7. **Step:** Inspect the registry after the failed sign-in
   **Why:** Determine how Windows represented the affected profile registration following the failed sign-in attempt.
   **Action:** Signed out of the temporary session, returned to labadmin, and inspected the ProfileList registry keys.
   **Result:** The previously observed SID key ending in 1003 was no longer listed under that exact name. A key ending in 1003.bak was present.

   The ProfileImagePath value under the .bak key pointed to:

   C:\Users\profiletest-broken

   This registry-key change was observed during the controlled failure and was used as part of the recovery process.


8. **Step:** Restore the original profile registry entry
   **Why:** Restore the original SID registration and correct the profile directory mapping.
   **Action:**

   Using Registry Editor:

   1. Renamed the SID key ending in 1003.bak to remove the .bak suffix.
   2. Selected the restored SID key ending in 1003.
   3. Changed ProfileImagePath back to:
  
   C:\Users\profiletest

   4. Left the other registry values unchanged.
  
   **Result:** The original SID key name and profile path were restored.

9. **Step:** Verify successful profile recovery
   **Why:** Confirm that Windows could once again load the original user profile.
   **Action:** Signed out of labadmin and signed in as profiletest.

   Executed:

   $env:USERPROFILE

   **Result:**

   Returned:

   C:\Users\profiletest

   The normal desktop loaded, and the previous profile sign-in warning no longer appeared.

10. **Step:** Verify the active profile status
    **Why:** Confirm that Windows had loaded the expected profile rather than another temporary profile.
    **Action:**

    Executed:

    Get-CimInstance Win32_UserProfile |
    Where-Object { $_.LocalPath -eq "C:\Users\profiletest" } |
    Select-Object LocalPath, Loaded, Special

    **Result:**

    - LocalPath: C:\Users\profiletest
    - Loaded: True
    - Special: False
   
    The original profile was successfully loaded.

11. **Step:** Verify profile backup and cleanup status
    **Why:** Confirm that the original profile hive remained intact and document the remaining lab artifact.
    **Action:**

    Signed out of profiletest and returned to labadmin.

    Verified both files using Test-Path:

    Test-Path "C:\Users\profiletest\NTUSER.DAT"

    Test-Path "C:\Users\profiletest\NTUSER.DAT.L1-002-backup"

    **Result:**

    Both commands returned True.

    An attempt to remove the additional backup file using elevated PowerShell was unsuccessful because of insufficient access permissions.

    The backup file was left in place. The original NTUSER.DAT was not modified or deleted.

## Root Cause

The controlled lab failure was caused by an incorrect ProfileImagePath registry value associated with the profiletest account.

The registry value was deliberately changed from:

C:\Users\profiletest

to:

C:\Users\profiletest-broken

The incorrect path prevented Windows from loading the original profile as expected. Windows displayed a profile sign-in warning and loaded a temporary profile.

This was an intentionally introduced configuration fault for the lab exercise, not an organically occurring production incident.

## Resolution

Restored the original SID registry key name by removing the .bak suffix observed after the failed sign-in attempt.

Corrected the ProfileImagePath value to:

  C:\Users\profiletest

No changes were made to the original profile's NTUSER.DAT file.

The user was subsequently able to sign in and load the original profile successfully.

## Verification

The following checks confirmed successful recovery:

- Windows no longer displayed the temporary-profile sign-in warning.

- The normal profiletest desktop loaded.

- $env:USERPROFILE returned C:\Users\profiletest.

- Win32_UserProfile reported Loaded = True.

- Win32_UserProfile reported Special = False.

- The original NTUSER.DAT file remained present.

- The original SID registry key and profile path were restored.

**Final Status: Resolved**

## User Communication and Closure

This was a controlled homelab troubleshooting exercise using a disposable local account. No production user or workstation was affected.

In a production support scenario, the user would be informed that the original profile had been restored and asked to verify access to their normal desktop, files, and applications before ticket closure.

## Lessons Learned and Escalation Considerations

- Verify the affected user's SID and ProfileImagePath before making registry changes.

- Confirm whether the user is signed in with a temporary profile rather than assuming that a desktop loading successfully means the original profile loaded.

- Inspect the original profile directory and NTUSER.DAT before attempting recovery.

- Understand the significance of the .bak registry-key pattern when investigating Windows temporary-profile incidents.

- Preserve original profile data and use a recovery point before making changes.

- Verify recovery using both the active profile path and Windows profile status.

- Avoid unnecessary changes to unrelated user accounts or registry values.

- If the original profile cannot be recovered, or profile corruption or storage problems are suspected, escalate for further investigation before considering profile recreation.

The additional NTUSER.DAT.L1-002-backup file remains in the lab profile directory because its removal was blocked by file permissions. It does not prevent the recovered profile from functioning.

## Supporting Material

**Proxmox Recovery Snapshot:** L1-002-Clean-Profile-Baseline
**Affected Account:** profiletest
**Registry Location:** HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList
**Original Profile Directory:** C:\Users\profiletest

---

**Submission Checklist**

- [x] I removed passwords, keys, tokens, and other secrets.
- [x] I redacted or replaced real IP addresses, hostnames, usernames, and personal or employer information where necessary.
- [x] I verified the commands and configurations where possible.
- [x] I accurately represented whether this incident was reproduced, researched, or performed in an authorized production environment.
- [x] I wrote this myself or have the right to share it.
- [x] Ticket metadata completed.
- [x] Problem description and symptoms documented.
- [x] Troubleshooting steps recorded in chronological order.
- [x] Commands, configurations, and observed results documented.
- [x] Root cause identified and accurately explained.
- [x] Resolution documented.
- [x] Recovery verified with appropriate evidence.
- [x] Remaining cleanup tasks or artifacts disclosed.
- [x] Unrelated accounts, systems, and configurations were not modified.
- [x] GitHub Markdown formatting reviewed.
- [x] YouTube walkthrough recorded and linked.
