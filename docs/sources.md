# Sources and claim boundaries

Reviewed 2026-09-06. Links may change after publication. Issue authors' hypotheses are not treated as vendor conclusions. No private conversation link is required to use this repository.

| Source | What it supports | What it does not establish |
| --- | --- | --- |
| [Microsoft icacls reference](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) | Grant/remove/save/restore syntax and inheritance | That changing a given app's ACL fixes this crash |
| [Microsoft AppContainer SID explanation](https://devblogs.microsoft.com/oldnewthing/20220502-00/?p=106550) | Names of package SIDs; long app-specific SIDs can be legitimate | That an unresolved SID is corrupt or malicious |
| [Electron process sandboxing](https://www.electronjs.org/docs/latest/tutorial/sandbox) | Sandbox purpose and testing-only no-sandbox guidance | The internal sandbox state of this Antigravity build |
| [Microsoft exception-code reference](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/bug-check-0x3b--system-service-exception) | 0x80000003 maps to STATUS_BREAKPOINT | A blue screen occurred, or an ACL denial is the cause |
| [Windows 11 release information](https://learn.microsoft.com/en-us/windows/release-health/windows11-release-information) | 25H2/26200 release context, including General Availability | Every 26200 installation is Insider or affected |
| [Electron issue #51761](https://github.com/electron/electron/issues/51761) | A reporter describes a similar ACL-sensitive crash and restricted-package grant | A proven source of this user's permission entries |
| [GitHub Desktop issue #22306](https://github.com/desktop/desktop/issues/22306) | A separate report of sandbox-sensitive startup failures on 26200/26300 | Universal diagnosis or an Antigravity ACL test |
| [Electron issue #52324](https://github.com/electron/electron/issues/52324) | A separate NVIDIA/build-26200 report | Confirmation that its suggested switches work in this case |
| [Microsoft Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) | An official tool for advanced access investigation | That any individual denied event caused the crash |

The supplied user record establishes the test sequence and immediate Antigravity success as reported observations. It is summarized in [case-study.md](case-study.md). No source here identifies a universal fixed Windows KB, and no issue title is accepted as proof of causation.
