# Electron GPU process crash on Windows 11: observed ACL fix

[Türkçe](README_TR.md) | [Case study](docs/case-study.md) | [Troubleshooting](docs/troubleshooting.md) | [Sources](docs/sources.md)

Windows 11 25H2 / build 26200.x, Electron or Chromium startup failure, `exit_code=-2147483645`, `0x80000003`, and `GPU process isn't usable. Goodbye.`: this repository documents an **observed workaround/fix on one affected system**. Antigravity opened normally after an application-folder permission change, without sandbox-disabling launch flags.

This is a community troubleshooting record, not an official Microsoft, Electron, Chromium, Google, or Notion fix. It does not establish that all crashes with this code share a cause, that Windows 25H2 introduced the problem, or that every Electron application is affected.

## What worked

After inspecting the installation folder's permissions, the user ran this command in **Command Prompt (CMD)**:

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /grant *S-1-15-2-2:(OI)(CI)(RX)
```

Antigravity then opened without flags. Read the backup, scope, and rollback instructions below before applying it. Do not paste this CMD syntax directly into PowerShell.

The change grants Read & Execute to ALL RESTRICTED APPLICATION PACKAGES on that installation folder, with inheritance. It leaves the application's default sandbox configuration in place. A successful launch alone does not independently verify every process's sandbox state.

## Symptoms and environment

The affected application failed during ordinary launch, repeatedly reporting:

```text
GPU process exited unexpectedly: exit_code=-2147483645
GPU process isn't usable. Goodbye.
```

The same 32-bit value is `0x80000003` in hexadecimal and `-2147483645` as a signed integer. Microsoft names the status `STATUS_BREAKPOINT`; it is not, by itself, proof of defective GPU hardware or an ACL denial. The [Microsoft status reference](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-0x3b--system-service-exception) is cited for the code mapping, not to characterize this application crash as a blue screen.

The troubleshooting record concerns Windows 11 25H2, with build `26200.9278` recorded in the conversation, and an Intel Iris Xe example. Antigravity's logs identify app version `2.12.2`. Exact bundled Electron/Chromium versions were not collected. Intel driver `32.0.101.7088` was discussed during troubleshooting; this is historical context, not a driver recommendation.

Notion and Notion Calendar were also reported as failing on the machine. The ACL fix was confirmed only for Antigravity. Treat other applications as candidates for separate investigation.

Microsoft lists 25H2 under OS build 26200, including General Availability releases. **26200 does not automatically mean Insider Preview.** Record the full build, KB, and enrollment separately. No specific Windows update is identified here as either the cause or the universal fix. [Windows release information](https://learn.microsoft.com/en-us/windows/release-health/windows11-release-information).

## Sandbox test matrix

These are the reported outcomes before the permission change. They are a historical record, not a checklist everyone must repeat.

| Launch condition | Result on the affected system | What it suggests |
| --- | --- | --- |
| Normal launch, no flags | GPU process crash; application exits | Startup fails in the default configuration |
| `--disable-gpu` | Crash remains | Disabling hardware acceleration alone did not resolve it |
| `--disable-gpu-sandbox` | Fatal crash stops, gray screen remains | Process survival is not a usable application |
| `--disable-gpu --disable-gpu-sandbox --in-process-gpu` | Gray/dark screen | This combination also failed to restore the UI |
| `--no-sandbox --disable-gpu` | Application works | Sandbox/access interactions warrant investigation; two settings changed |
| Folder ACL grant, then normal launch | Application opens without flags | Observed ACL workaround/fix on the affected system |

The record does not establish the result of `--no-sandbox` alone. A gray screen does not prove a renderer crash without supporting logs. Different applications or versions may ignore or interpret switches differently.

[Electron's sandbox documentation](https://www.electronjs.org/docs/latest/tutorial/sandbox) describes `--no-sandbox` as disabling Chromium sandboxing for all processes and recommends it only for testing. Do not make it a permanent shortcut, startup entry, or repair script. If a diagnostic run is necessary, keep it brief, avoid untrusted content and sensitive sessions, then close the application and remove the flag.

## Why permissions became a suspect

Changing sandbox-related flags changed the outcome, and a targeted folder grant was followed by a successful default launch. This supports an interaction between application-folder access and sandboxed startup. It does not identify the exact failing file, access check, or component.

A long `S-1-15-2-...` entry appeared in the folder ACL. Long AppContainer SIDs can be legitimate, even when Windows does not resolve a friendly name. Do not delete an unfamiliar SID solely because it appears as an unknown account. [Microsoft's AppContainer SID explanation](https://devblogs.microsoft.com/oldnewthing/20220502-00/?p=106550).

[Electron issue #51761](https://github.com/electron/electron/issues/51761) reports a similar DACL-related startup failure and the same restricted-package grant. That is corroborating issue evidence, not a vendor-confirmed diagnosis of this machine. We have no trace establishing who or what created its ACL entries. Claims that a particular app, agent sandbox, cleaner, installer, antivirus, or Windows update caused this incident remain unproven.

Reinstalling app files can leave permissions or inherited parent-folder entries in place. Updating a GPU driver does not ordinarily repair installation-folder DACLs. These are reasons such attempts might fail when access is involved; they do not rule out driver bugs, corrupt files, security software, or separate causes in another case. Repeating destructive profile cleanup is not justified by this error alone.

## Step-by-step repair with a backup

### 1. Close the app and identify its installation

Save work and fully exit Antigravity, including background instances. Use the shortcut's Properties or Task Manager's Open file location to confirm the actual executable and installation folder. In CMD:

```cmd
echo "%LOCALAPPDATA%\Programs\antigravity"
dir "%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
icacls "%LOCALAPPDATA%\Programs\antigravity"
```

Stop if the path does not match. The example targets installed program files, not `%APPDATA%\Antigravity`, your profile, or the whole `Programs` directory. Inspect for junctions/symbolic links before a recursive backup; do not proceed if they lead outside the intended application tree. Do not take ownership of WindowsApps or bypass a managed device's permission policy.

### 2. Save the DACL before editing

Use a fresh backup directory for each attempt; do not overwrite your only pre-change backup. These CMD commands save relative paths beginning with `antigravity`, so restore must use its parent directory:

```cmd
set "ACL_BACKUP=%TEMP%\antigravity-acl-%RANDOM%-%RANDOM%"
mkdir "%ACL_BACKUP%"
echo Backup location: "%ACL_BACKUP%"
pushd "%LOCALAPPDATA%\Programs"
icacls "antigravity" /save "%ACL_BACKUP%\before.dacl" /t
echo Exit code: %ERRORLEVEL%
popd
```

Run lines individually. Continue only after the backup finishes with exit code 0 and no failed files. Preserve the printed directory somewhere private and durable before temporary-file cleanup. `/save` backs up DACLs, not file contents, ownership, or all security-descriptor fields. Do not upload the backup; it includes local paths and identities.

If inspection, backup, or grant needs elevation, reopen CMD as administrator for the same Windows account and repeat the setup with verified paths. A different administrator account changes `%LOCALAPPDATA%`. Administrator rights may also be needed specifically for restore, even when save/grant worked without them. Do not disable UAC or use ownership resets to work around this.

### 3. Apply the limited grant

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /grant *S-1-15-2-2:(OI)(CI)(RX)
echo Exit code: %ERRORLEVEL%
```

The original report showed one processed item and zero failures. The command names the root folder; inheritance can still affect descendants. It deliberately omits `/t`, `/reset`, `/grant:r`, ownership changes, and Full Control.

For readers using PowerShell, the equivalent grant is below. Use it only after completing a backup; it is an alternative to the CMD grant, not an additional step:

```powershell
icacls "$env:LOCALAPPDATA\Programs\antigravity" /grant '*S-1-15-2-2:(OI)(CI)(RX)'
$LASTEXITCODE
```

### 4. Verify permissions and launch without flags

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity"
icacls "%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
"%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
```

Look for SID `S-1-15-2-2` or its localized group name with RX and inheritance on the folder. Check the executable and relevant child resources for inherited access. If a child has protected inheritance or a deny entry, stop and investigate rather than granting recursively.

Remove diagnostic switches from shortcuts, wrappers, and startup entries. Check any sandbox-disabling environment overrides configured during earlier troubleshooting. Confirm the UI loads and a normal workflow works. Fully close and reopen the app, then verify after reboot and after an app update. These longer-term checks are recommendations; the supplied record confirms only the immediate normal launch.

### 5. Roll back when appropriate

If **no explicit grant for this SID existed on the target folder before the change**, and no later changes need to be preserved, the narrow rollback is:

```cmd
icacls "%LOCALAPPDATA%\Programs\antigravity" /remove:g *S-1-15-2-2
echo Exit code: %ERRORLEVEL%
```

This removes grants for that SID on the named object; it is not a subtraction of just RX. If the SID already had an explicit grant, use the DACL backup instead. Do not add `/t` to this removal.

In the same CMD session where `ACL_BACKUP` is set, restore from the matching parent directory:

```cmd
icacls "%LOCALAPPDATA%\Programs" /restore "%ACL_BACKUP%\before.dacl"
echo Exit code: %ERRORLEVEL%
```

In a reopened/elevated CMD, first set `ACL_BACKUP` to the actual saved backup location; the variable does not survive closing the terminal. Verify the target user's path as well. Require exit code 0, read the complete output, and compare folder/child ACLs with the backup. A restore can fail for lack of privilege even when its summary says zero failed files.

Restore reapplies saved DACLs and can overwrite intervening permission changes. Coordinate with the owner/admin if an installer or policy has changed permissions since backup. Files created after the snapshot are not individually represented; do not claim an exact whole-tree rollback after an update. See [Microsoft icacls documentation](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) for command semantics.

## What the grant means and what it risks

| Component | Meaning |
| --- | --- |
| `S-1-15-2-2` | ALL RESTRICTED APPLICATION PACKAGES; different from `S-1-15-2-1` |
| `*` before the SID | Numeric SID syntax understood by icacls |
| `(OI)(CI)` | File and directory inheritance |
| `(RX)` | Read & Execute, not Write or Full Control |
| `/grant` | Adds permission without replacing the SID's prior explicit grants |

This grants access to a package group, not just to Antigravity's own identity. It broadens read/execute access within the selected tree and is not risk-free. Avoid installation folders that contain private keys, tokens, personal documents, or unrelated projects. A deny entry, protected child DACL, or parent traversal issue may still prevent access. A successful icacls command is not an effective-access audit.

Prefer this limited, reversible application-folder change over retaining `--no-sandbox` when it resolves the affected case. Do not apply it to a drive root, `C:\Windows`, an entire user profile, `%LOCALAPPDATA%`, `%APPDATA%`, or every Electron app at once.

## Other apps, troubleshooting, and reporting

For Notion, Notion Calendar, and other Electron/Chromium applications, follow [the adaptation guide](docs/other-apps.md). It explains executable discovery, versioned installation folders, managed installations, and the difference between a reported failure and a confirmed fix.

If the change fails, use [troubleshooting](docs/troubleshooting.md). File an issue using the bug report form, including sanitized logs, full versions, and the exact before/after outcome. An unsuccessful result is useful evidence. Read [CONTRIBUTING](CONTRIBUTING.md) and [SECURITY](SECURITY.md) first.

## Repository and search terms

Suggested repository name: `electron-windows-gpu-sandbox-acl-fix`. Ready-to-copy description, topics, and publishing steps are in [PUBLISHING.md](PUBLISHING.md). This documentation and its examples use the [MIT License](LICENSE); vendor software is not included.

Search phrases: Windows 11 25H2 Electron crash; build 26200 GPU process; Antigravity not opening; Notion startup crash; Notion Calendar gray screen; Chromium sandbox permissions; GPU process isn't usable Goodbye; exit_code=-2147483645; 0x80000003 STATUS_BREAKPOINT; ALL RESTRICTED APPLICATION PACKAGES; S-1-15-2-2 icacls RX; Intel Iris Xe Electron crash. These aid discovery and do not imply confirmed compatibility or vendor-specific causation.
