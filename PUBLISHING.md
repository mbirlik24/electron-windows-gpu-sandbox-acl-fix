# Publish this repository

Suggested name:

```text
electron-windows-gpu-sandbox-acl-fix
```

Suggested GitHub About description:

```text
Observed ACL fix for Electron/Chromium GPU process crashes on Windows 11 25H2 (26200.x): 0x80000003 / -2147483645. Antigravity case, safe steps, rollback, EN/TR docs.
```

Suggested topics (20):

```text
electron chromium windows11 windows-11-25h2 windows-26200 gpu-process-crash 0x80000003 status-breakpoint sandbox acl dacl icacls antigravity notion notion-calendar troubleshooting windows-permissions intel-iris-xe read-execute all-restricted-application-packages
```

The same metadata is in [repository-metadata.json](repository-metadata.json). The keywords describe the topic, not confirmed repairs for every named app. Search ranking is not guaranteed.

## Upload in the browser

Create an empty public GitHub repository with the suggested name. Upload the contents of this folder, including `.github`, `.gitignore`, and `.gitattributes`, at the repository root. Do not upload only the ZIP or add an extra containing directory. Check hidden files are included; Git is more reliable if the browser hides them. Add the description and topics in About, enable Issues, and consider enabling private vulnerability reporting under Security settings.

## Publish with Git and GitHub CLI

Open a terminal in this repository folder. These commands publish publicly; run them only when ready. Authentication and your Git author identity must already be configured. This source package has no fabricated Git identity, remote, or commit history.

```sh
git init -b main
git add .
git diff --cached --check
git commit -m "Document observed Windows Electron sandbox ACL fix"
gh auth status
gh repo create electron-windows-gpu-sandbox-acl-fix --public --source=. --remote=origin --push
gh repo edit --description "Observed ACL fix for Electron/Chromium GPU process crashes on Windows 11 25H2 (26200.x): 0x80000003 / -2147483645. Antigravity case, safe steps, rollback, EN/TR docs."
gh repo edit --add-topic "electron,chromium,windows11,windows-11-25h2,windows-26200,gpu-process-crash,0x80000003,status-breakpoint,sandbox,acl,dacl,icacls,antigravity,notion,notion-calendar,troubleshooting,windows-permissions,intel-iris-xe,read-execute,all-restricted-application-packages"
```

If the name is already in use, choose the intended destination explicitly. For an organization, specify its owner/name when creating the repo and verify membership/permissions. If a repository was already created in the browser, attach that verified remote and push instead of creating a duplicate. Do not blindly repeat publication commands after a partial failure.

## License choice

MIT is included to permit reuse of the original documentation and command examples with attribution and a warranty disclaimer. The copyright notice uses the repository's contributor name collectively; a maintainer may substitute their chosen personal or organization attribution before publication. Third-party documentation is linked, and vendor binaries and private logs are excluded. The repository license does not relicense those third-party materials.

## Release checks

Review both READMEs, inspect all files for personal data, confirm the issue templates appear, and check that links work on GitHub. If making a release, attach the source ZIP and its SHA-256 checksum as assets. A release number represents the documentation revision, not a universally supported repair-tool version. Do not claim a reboot/update outcome until someone reports it with evidence.
