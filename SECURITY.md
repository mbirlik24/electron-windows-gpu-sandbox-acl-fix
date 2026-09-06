# Security policy

This repository contains documentation and copyable command examples. It does not distribute an app repair executable, alter vendor sandbox defaults, or provide a security guarantee. Review the target folder and backup before changing its DACL.

## Report a security problem

If this repository has GitHub private vulnerability reporting enabled, use its Security tab and Report a vulnerability. Do not put exploit details, tokens, private paths, or permission backups in a public issue. If that private channel is unavailable, open a minimal public request asking maintainers to enable a private reporting channel, without sensitive details. No security email address is assumed and no response-time promise is made.

Report vulnerabilities in Electron, Chromium, Windows, or a vendor application through that project's official security channel. A normal startup failure or an unsuccessful workaround can use this repository's sanitized bug report form.

## Protect diagnostic material

Before sharing, remove account/device names, email addresses, tokens, CSRF values, authorization headers, host-bridge credentials, API keys, private project paths, document content, and signed URLs. Replace identifiers consistently so a reader can still compare before/after paths. Avoid publishing raw crash dumps or Process Monitor traces; they may contain private information.

The package-group grant broadens read/execute access under the chosen tree. Keep sensitive files outside it and do not generalize the grant to a profile or drive. Do not retain no-sandbox in everyday launches. A successful normal launch is not proof that an app's own sandbox design is secure.

Only the documentation on the default branch is maintained; there is no supported matrix of operating-system/app versions or security backport commitment.
