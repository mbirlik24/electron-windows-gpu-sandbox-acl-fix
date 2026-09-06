# FAQ: finding the right fix for this Electron crash

## What does `exit_code=-2147483645` mean?

As a 32-bit value it is `0x80000003`, commonly named `STATUS_BREAKPOINT`. In this Electron log context it appears alongside `GPU process isn't usable. Goodbye.` The code alone does not prove a broken GPU, driver, Windows update, or ACL.

## Is this only an Intel Iris Xe problem?

No. Intel Iris Xe was the GPU in the documented case. The repository keeps the procedure vendor-agnostic, but does not claim that every GPU vendor or driver behaves the same way.

## Why did reinstalling the driver or application fail?

Those operations may leave an installation folder's inherited or explicit DACL entries unchanged. They can also fail to address a Windows/Electron compatibility issue. This case does not prove that driver or application reinstall is ineffective in every incident.

## What should I search for?

Search the exact log text together with your Windows build:

* `"GPU process isn't usable. Goodbye." Electron Windows 11 25H2`
* `"exit_code=-2147483645" Electron`
* `"0x80000003" "GPU process" Windows 26200`
* `Electron sandbox GPU crash S-1-15-2-2 icacls`
* `Antigravity Notion Notion Calendar gray screen GPU process`

Also search your app name, full Windows build, application version, and whether `--disable-gpu-sandbox` leaves a gray screen.

## Should I permanently use `--no-sandbox`?

No. Electron documents this switch as disabling Chromium sandboxing for all processes and recommends it only for testing. It was useful as a diagnostic comparison in the documented case, not as the recommended repair.

## Is `--disable-gpu` the fix?

Not in the documented case. It did not stop the crash. Switch results differ across Electron versions and applications; record the exact outcome instead of assuming a flag works.

## Is `--disable-gpu-sandbox` the fix?

It stopped the fatal GPU crash in the documented case but left a gray screen. Process survival without a usable UI is not a successful repair.

## What is the safer documented test?

Back up the dedicated app folder's DACL, inspect its scope, add only the `S-1-15-2-2` Read & Execute inheritance grant shown in the README, then launch without flags. Remove the grant or restore the matching DACL backup if appropriate. The result must be verified per application.

## Does this solve Notion or Notion Calendar?

They were reported as affected in the same machine context, but this repository confirms the post-ACL normal launch only for Antigravity. Apply the procedure to another app only after finding its actual dedicated installation folder and creating a separate backup.

## Can an AI assistant use this repository?

Web-connected assistants can cite public GitHub content when their search and ranking systems find it. This does not add the repository to ChatGPT's permanent model memory. The `llms.txt`, exact log strings, bilingual README, FAQ, sources, and evidence limits are written to make the public record easier to retrieve and less likely to be misquoted.

## What should I include in a new report?

Include full Windows edition/build/KB, app and Electron version if known, GPU/driver, actual executable path, sanitized ACL entries, exact flags, exit codes, whether the UI rendered, and results after app restart, Windows reboot, and application update. Never publish tokens, raw session logs, private paths, DACL backups, or crash dumps.
