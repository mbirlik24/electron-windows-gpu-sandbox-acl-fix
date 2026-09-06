---
name: Report an observed result
about: Share success, partial success, or failure for a specific app and system.
title: "[Result]: "
labels: ''
assignees: ''
---

## System and application

* Windows edition, version, full build, KB, and channel:
* App and version; Electron/Chromium if known:
* GPU and driver:
* Sanitized installation layout:

## Observation

* Failure signature before the change:
* Tests already performed (do not run unsafe tests to fill this out):
* Backup completed:
* Exact ACL change and command exit code:
* Did the SID have a grant beforehand?
* Normal launch after change; does the UI work?
* After full app restart / Windows reboot / app update (separate each, or not tested):
* Rollback result, if attempted:
* Other changes that could explain the result:

## Sanitized evidence

Short log excerpt or relevant ACL entries. Do not attach raw ACL backups, secrets, or private trace files.

## Scope

Describe this as an observed outcome on your system. Avoid claiming a universal cause or fix.
