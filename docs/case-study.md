# Case study and evidence boundaries

Prepared on 2026-09-06 from a user-supplied troubleshooting conversation and the user's explicit account. The private conversation, screenshots, original account name, full personal paths, and unredacted logs are not distributed.

## Sequence

1. The user reported startup failures involving Antigravity, Notion, and Notion Calendar. In the available Antigravity console excerpt, GPU process exits repeat before the fatal message.
2. Earlier troubleshooting included app reinstall/profile-cache cleanup, disk-space work, and Intel driver updates according to the conversation recap. These did not resolve the reported startup failure. Exact installer steps and full before/after driver inventories were not preserved here; do not treat this as a controlled reinstall experiment.
3. Disabling GPU acceleration alone did not help. Disabling the GPU sandbox avoided immediate termination but left a gray screen. A combined in-process-GPU experiment also produced an unusable UI.
4. The user explicitly reported that `--no-sandbox --disable-gpu` opened the app normally.
5. The installation-folder ACL was inspected. It contained inherited SYSTEM, Administrators, user entries, and a long AppContainer-like SID. The displayed root ACL did not show `S-1-15-2-2`. The complete child DACLs and effective access were not captured.
6. The targeted grant returned one successfully processed item and zero failures. The user then confirmed the application opened after being asked to launch without any flags.

The README presents the cleaned, valid CMD syntax. Escaping artifacts from chat export are not part of the command.

## Evidence ledger

| Claim | Evidence level | Limit |
| --- | --- | --- |
| Antigravity failed with the stated log signature | Console excerpt | No crash dump or stack analysis |
| Antigravity app version 2.12.2 | Console startup line | Bundled runtime versions unknown |
| Windows 25H2 / 26200.9278 | Recorded conversation context | Original screenshot not redistributed |
| Intel Iris Xe / driver 32.0.101.7088 context | Troubleshooting account | No attached hardware inventory |
| Combined no-sandbox flags restored UI | Explicit user report | Does not isolate either flag |
| Grant completed successfully | icacls output | Does not audit every child's access |
| Default launch worked after the grant | User confirmation immediately after instruction | No independently recorded process-token inspection |
| Notion and Notion Calendar also failed | User-provided case context | No confirmed ACL repair for either |
| The fix survives reboot/update | Not established | Further verification requested |
| A Windows regression or a particular app changed the ACL | Hypothesis | No origin trace or controlled reproduction |

## Cleaned log signature

```text
GPU process exited unexpectedly: exit_code=-2147483645
GPU process isn't usable. Goodbye.
```

PID, timestamps, source line numbers, and unrelated service startup messages were omitted. Original logs contained session-related values; those are intentionally excluded.

## What would strengthen the diagnosis?

Useful follow-up evidence includes a full build and update channel, executable/runtime versions, sanitized before/after ACLs, access-denied traces tied to a specific child process and file, and results after reboot and application update. A controlled reproduction belongs in an isolated test installation, not a user's working profile. Do not manufacture malformed SIDs on a production machine to reproduce an issue.

The strongest justified conclusion is: **the limited ACL grant was followed by successful flags-free launch on the affected system**. The repository's hypothesis is a sandbox/access interaction. Neither the original cause of the permissions nor a universal repair is established.
