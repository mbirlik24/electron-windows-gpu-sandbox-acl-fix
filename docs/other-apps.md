# Adapting the investigation to other applications

The confirmed outcome in this repository is Antigravity only. Notion and Notion Calendar were reported as affected in the same case; their response to the ACL grant is unverified. A shared Electron/Chromium foundation makes comparisons useful, but app packaging and sandbox policy differ.

## Locate the real target

1. Open the app shortcut's Properties. If it starts an updater or launcher, find the actual application executable through Task Manager's Open file location during a brief launch, or through the vendor's installation information.
2. Inspect the executable's containing tree. Some installers use a dedicated product folder with versioned `app-*` children; others use `%LOCALAPPDATA%\Programs`, Program Files, or a custom location. Do not assume Notion and Notion Calendar share a directory.
3. Identify the smallest dedicated installation tree containing the executable and its required program resources. Keep the user-data/profile directory out of scope. If unsure which tree is dedicated, obtain vendor guidance rather than granting on a common parent.
4. Inspect that folder's DACL and relevant children. Check for links, private files, denied permissions, and inheritance boundaries. The presence of an unfamiliar SID is not permission to delete it.
5. Save the tree's DACLs with paths relative to its parent, following the README. Preserve a separate backup for each app. Replace both the grant target and the restore parent consistently.
6. Apply the grant only if the evidence and scope support the test. Launch without diagnostic flags, verify the UI, and document the exact outcome. A failure remains a failure even if a blank window survives.

No executable Notion/Notion Calendar repair command is supplied because a guessed installation path can target the wrong files. The substitution pattern is the actual dedicated installation folder, with the same SID and RX rights used in the README. Do not edit WindowsApps or take ownership of a Store-managed package. For a managed device or protected installation, use the vendor or IT support route.

## Updates and scope

Granting on a version-specific folder may stop helping when an updater replaces that version. Granting on a product root may inherit into later versions, but has a wider scope. Choose deliberately and verify after updating. Neither option justifies granting access across all of `%LOCALAPPDATA%`.

| Application context | Status in this repository |
| --- | --- |
| Antigravity 2.12.2 in the supplied case | ACL grant followed by successful normal launch |
| Notion | Startup failure reported; ACL outcome unverified |
| Notion Calendar | Startup failure reported; ACL outcome unverified |
| GitHub Desktop | Separate public report linked in sources; not locally tested |
| Other Electron apps / Chromium / CEF | Similar symptoms can be investigated; no blanket compatibility claim |

When contributing another application result, include the full Windows/app/runtime versions, sanitized path layout, the precise ACL change, and whether the app worked after reboot/update. Do not call the method vendor-agnostic proof merely because it changes a Windows permission: the instructions avoid assuming a GPU vendor, while effectiveness must still be tested per case.
