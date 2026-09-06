# Troubleshooting without widening permissions blindly

Use the backup and rollback procedure in [the README](../README.md) before editing ACLs. Record one change at a time.

| Result | Next check |
| --- | --- |
| Path not found | Resolve the shortcut and executable location; check whether the app has a versioned directory or a machine-wide installation |
| CMD reports invalid parameter | Confirm you are in CMD; use a plain `*`, not `\*`; use the quoted SID argument shown for PowerShell |
| Access denied | Confirm the account, ownership context, and management policy; use same-account elevation only where permitted |
| Backup partly fails | Stop; a partial backup is not a safe baseline |
| Restore reports privilege failure but zero failed files | Check the actual exit code and error text; reopen same-account elevated CMD and restore with explicit verified paths |
| Grant succeeds but app still crashes | Check executable/child ACLs, deny entries, and protected inheritance; save evidence and use rollback when suitable |
| App stays gray/white/black | Capture renderer/application errors; investigate UI/network/profile failures separately; process survival is not success |
| Works only with no-sandbox | The access hypothesis remains possible; do not widen the grant or retain the bypass as the permanent answer |
| Works once but fails after update | Re-resolve the executable; compare installation path and ACL changes; do not create an automatic re-grant scheduled task |
| Many unrelated applications fail | Inspect their actual paths and shared-parent inheritance read-only; involve Windows/IT support before changing parent permissions |

## Read-only evidence collection

Open `winver` and record the complete build. Use Settings > Windows Update > Update history for the installed KB and Settings > Windows Update > Windows Insider Program for enrollment. Record the app version from About or executable Properties. Record GPU model and driver version from Device Manager. For Electron/Chromium versions, use the app's About/system-information screen if available; do not guess them from its product version.

Optional PowerShell commands for local inspection:

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' |
  Select-Object DisplayVersion, CurrentBuildNumber, UBR
Get-CimInstance Win32_VideoController | Select-Object Name, DriverVersion
```

Inspect Event Viewer > Windows Logs > Application around the failed launch. Collect exception code, faulting module, and app version. A faulting module is a clue, not proof of the ultimate cause.

If the application supports Electron logging, a short local diagnostic launch in CMD can help:

```cmd
set "ELECTRON_ENABLE_LOGGING=1"
"%LOCALAPPDATA%\Programs\antigravity\Antigravity.exe"
set "ELECTRON_ENABLE_LOGGING="
```

Use a disposable CMD session and close it afterward. If that variable was already intentionally configured, preserve its value before the test. Logging support varies, and GUI output may appear in an app-specific log instead of the console. Logs can contain tokens even when the UI never opens. Sanitize before sharing.

For advanced investigation, use Microsoft's [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) to correlate relevant process startup and file-access results. Filter to the affected process tree and capture a short window. Isolated ACCESS DENIED events can be expected behavior; correlate the path and timing with the failure before concluding causation. Keep full trace files private unless a trusted support channel requests them.

## Stop conditions

Stop the generic ACL procedure if the path is a link to an unexpected location, contains private data, is controlled by an organization, already has unusual denies, or the backup cannot be completed. Do not remove unknown SIDs, grant Everyone Full Control, run recursive resets over a profile, disable antivirus permanently, or downgrade Windows solely because the error code matches.

App and Windows updates may be appropriate through supported vendor channels. Do not name a particular KB as the fix without release notes and matching test results. Report successful and unsuccessful outcomes with the issue template.
