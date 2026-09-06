# Contributing

English and Turkish reports and corrections are welcome. This repository documents a bounded observation; preserve that scope in titles and claims.

Use the bug report form for failures and the result report template for follow-up outcomes. Include full Windows build/channel, application version, Electron/Chromium version if known, GPU/driver, installation layout, exact flags, and the ACL change. Mark missing information as unknown. Report immediate launch, reboot, and update results separately.

For pull requests:

* Explain the incorrect or missing instruction and the change you propose.
* Keep README.md and README_TR.md consistent for commands, risks, and evidence status.
* Cite an official document for command semantics and link the original issue for community observations.
* Keep changes limited to dedicated app folders with a backup and rollback path.
* Validate CMD and PowerShell syntax separately. Test permission-changing examples only in disposable folders or isolated app installations.
* Check relative links, Markdown code fences, issue-template formatting, and privacy before submitting.

Do not add automatic elevation, bulk repairs, recursive profile resets, unsupported certainty, or permanent sandbox-bypass launchers. Do not add application binaries, full private chats, screenshots with account details, ACL backups, crash dumps, or logs containing secrets.

Read SECURITY.md before reporting a vulnerability. Contributions are provided under the repository's MIT License. Maintainers may ask for narrower claims or reproducible evidence; a report that the workaround failed is also valuable.
