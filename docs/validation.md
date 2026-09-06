# Package validation

Preparation checks performed on 2026-09-06:

* English and Turkish CMD/PowerShell command blocks were compared and match exactly.
* Relative Markdown file links resolve inside the source package.
* Markdown fences, UTF-8 text, trailing whitespace, JSON, and YAML were checked.
* The bug report form was parsed and its field IDs checked for duplicates. GitHub-hosted rendering has not been tested.
* Repository metadata was checked for description length, topic count, and topic syntax.
* Publication switches were compared with the installed GitHub CLI help. No remote repository was created.
* The source package was inspected for private paths, account names, session values, and raw diagnostic files. Only short cleaned error lines are included.
* The ZIP was checked for archive integrity and byte-for-byte agreement with every packaged source file. A SHA-256 checksum accompanies the ZIP.

## Permission-command verification

Permission changes were tested only in a disposable workspace folder containing a nested sample file. The actual Antigravity, Notion, and Notion Calendar installations were not modified during repository preparation.

Both the CMD grant/removal syntax and the quoted PowerShell SID syntax completed successfully. The sample child inherited the restricted-package RX grant. Removing the root grant returned the root DACL to its baseline and removed the inherited child grant.

DACL save completed successfully on the disposable tree. DACL restore could not be completed in the available non-elevated process: Windows reported a privilege-related error while the summary showed zero failed files. This limitation is documented in both READMEs; full restore must not be treated as verified here. The backup/restore path pairing was checked against the Microsoft command reference. The narrow removal was used to return the disposable tree to baseline.

These checks validate documentation syntax and the limited grant/removal behavior. They do not reproduce the original application crash, independently prove its cause, or establish success after reboot or an application update.
