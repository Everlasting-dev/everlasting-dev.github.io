# Agent change log

Internal handoff log for Cursor agents and plugins. Newest entries first.
Read before editing; append after substantive changes.

## 2026-10-03 - publish Slip 0.0.11 with the Octane menu

- **Why:** one open menu for both apps: install/update and repair Octane, lock its license on a PC, share the Octane template pack, and a short description for every option
- **Changes:** replaced `get` with version 0.0.11. Launcher shows Octane and Slip. The Octane menu has Install / update, Check & repair, Verify & lock license on this PC (admin prompt, read-only HKLM\SOFTWARE\EverlastingDev\Octane), Get template pack (backs up the PC's templates to the owner first), Publish my template pack and Collect template backups (owner), and Uninstall. Every Slip and Octane menu shows a one-line description for the highlighted option (inline when typing numbers). `slip octane` opens the Octane menu directly. Old Octane installs are now found by their versioned name ("Octane 0.9.5").
- **Files:** `get`, `README.md`, `AGENT_CHANGELOG.md`
- **Notes:** account features need `octane/supabase/licenses.sql` run in Supabase. Source of truth is `Documents\project Z\lanfile.ps1`.

## 2026-10-02 - reopen the menu after Diagnostics updates

- **Why:** Check and fix installed the downloaded version correctly, but then returned to the already-running menu process, whose in-memory version and code remained old until the user closed and reopened Slip.
- **Changes:** Version 0.0.10. A fetched upgrade now detects when its direct parent is the installed Slip menu. After installation it starts a hidden replacement helper from the newly installed script; the helper waits for the updater to finish, closes only that verified old menu process, and opens a fresh normal Slip menu. Direct command-line and unattended upgrades do not match the menu-parent check and keep their existing behavior.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** This works during the update from 0.0.9 because detection and replacement run inside the newly downloaded child script, not the old menu code.

## 2026-10-02 - faster normal startup

- **Why:** Opening `slip` was delayed because Auto chat ran the full command installation path before drawing every menu, including script hashing, launcher and shortcut rewrites, user PATH work, and a Windows environment broadcast that could wait up to five seconds.
- **Changes:** Version 0.0.9. Normal menu startup now performs only a fast Auto chat PID health check and starts the listener only when it is missing. Full command, profile, PATH, startup-entry, and Send To repairs remain in install, upgrade, explicit enable, and Repair. Command installation now skips the PATH write and environment broadcast when the user PATH is already correct.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Auto chat remains enabled by default and still self-recovers if its background listener has stopped; normal startup no longer rewrites installation state.

## 2026-10-02 - Explorer Send to Slip

- **Why:** Sending through Outbox and the full menu is too many steps for everyday one-PC-to-another transfers.
- **Changes:** Version 0.0.8. Install/upgrade/Repair now create `%APPDATA%\Microsoft\Windows\SendTo\Slip.lnk`, targeting the installed host script's new `sendto` entry. Explorer-selected files and folders bypass Outbox and PIN, reuse Quick receive, automatically choose the only available receiver, and show a short picker when several receivers are available. Multiple items and folders reuse `New-SendPayload` and arrive as one zip. Remove deletes the shortcut. Status and remote diagnosis report whether the shortcut exists.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** The receiving PC must have Quick receive on. A no-target or failed transfer stays visible with a plain explanation; success closes after two seconds. The shortcut launches PowerShell directly, so selected paths are passed by Explorer rather than expanded through a batch file.

## 2026-10-02 - scheduled update reminders

- **Why:** The user wants Slip to check after a chosen time and inform them when a new build exists, without silently installing it.
- **Changes:** Version 0.0.7. Added More -> Update reminders with Check now, a configurable daily `HH:mm` time, and Off. Enabling creates a limited current-user `Slip Update Reminder` scheduled task that runs the hidden `updatecheck` entry. Current/offline automatic checks stay silent; a newer version opens a normal `updatenotice` window once per version and points to Diagnostics -> Check and fix. State and the last-notified version live in `Documents\Slip\update-reminder.json`. Upgrade preserves and refreshes the task, Repair recreates it when missing, and Remove deletes it.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** The reminder is notification-only and never installs an update. It uses an Interactive/Limited principal and StartWhenAvailable, so a missed time runs when that user is next available. Scheduled-task creation failure leaves reminders off instead of claiming they are active.

## 2026-10-02 - paired remote diagnosis

- **Why:** The existing remote switch could only turn Quick receive and Auto chat on. It could not securely return enough state to diagnose another PC from here.
- **Changes:** Version 0.0.6. Added opt-in Remote help with its own background listener and `SUPPORT` beacon. The target shows a 96-bit pairing code; both PCs keep derived keys under current-user DPAPI protection. Status requests use HMAC-SHA256, a two-minute timestamp window, and replay nonces. Reports use AES-256-CBC plus encrypt-then-MAC and include fixed read-only Slip, Windows, network, firewall, disk, listener, finding, and recent Slip-log fields. Added `remote-diagnose.ps1` to pair, request, print, and save reports. Remote help survives upgrade and Repair when enabled, stays off by default, and can be disabled or re-keyed from More.
- **Files:** `lanfile.ps1`, `remote-diagnose.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** This deliberately is not remote desktop or remote PowerShell and cannot execute caller-supplied commands. First use on the other PC: update to 0.0.6, More -> Remote help -> turn on, then enter its pairing code once on this PC. Resetting the code revokes existing pairings.

## 2026-10-01 - remote enable for Quick receive and Auto chat

- **Why:** A PC with Slip open could be seen from here, but nothing was listening that could turn Quick receive or Auto chat on
- **Changes:** Version 0.0.5. While the Slip menu is open it listens on a Slip port and advertises that port in the OPEN beacon. `ENABLE quick` and `ENABLE chat` on that port turn the matching listener on. A PC that already has Quick receive accepts `ENABLE chat`. A PC that already has Auto chat accepts `ENABLE quick`. Added `enable-remote.ps1 -Name pink_juice` to send those from this PC.
- **Files:** `lanfile.ps1`, `enable-remote.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** The copy already running on pink_juice is older, so its OPEN beacon still has port 0. That PC needs Check and fix once. After the menu shows 0.0.5, `enable-remote.ps1 -Name pink_juice` can turn both listeners on without anyone typing there. These commands only enable. They do not run files or other actions.

## 2026-10-01 - open a remote chat test from this PC

- **Why:** The user needs the other PC's chat window tested without anyone using that keyboard
- **Changes:** Added `open-remote-chat.ps1`. It listens for Slip Auto chat beacons, and if none arrive it checks Slip ports on PCs already seen on the LAN. For each one it sends a chat invite and records whether the window connected back. Result is printed here and saved to `Documents\Slip\Debug\remote-chat-test.txt`. Stopped the earlier collector that waited for someone to run a script on the other PC.
- **Files:** `open-remote-chat.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** Auto chat must already be running on the other PC. Pass `-Name pink_juice` to test one PC and skip the others. A window that accepts the invite but does not connect back means that PC could not reach this PC (network or firewall).

## 2026-10-01 - chat open diagnose script

- **Why:** One of the user's PCs is not opening the chat window. They need a test window on that PC and the findings back on this PC.
- **Changes:** Added `diagnose-chat.ps1` (run on the problem PC) and `collect-chat-diagnose.ps1` (run on this PC). The diagnose script records version, Auto chat, listener, network, firewall, and recent chat log lines, opens a local Slip chat test window, and sends the report to TCP 8796. The collector writes `Documents\Slip\Debug\chat-diagnose-remote.txt`.
- **Files:** `diagnose-chat.ps1`, `collect-chat-diagnose.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** Default report target is 192.168.152.132. Port 8796 is inside the existing Slip TCP private-network rule. The collector holds that port until one report arrives or 20 minutes pass. Do not publish these as `get`.

## 2026-09-30 - Slip 0.0.4 installed here and published

- **Why:** Other PCs need one-to-one chat, the sectioned menus, and Check and fix that also sets the network to Private
- **Changes:** Installed the local 0.0.4 script with `SLIP_FETCHED_UPGRADE=1` and published it as `get` (was 0.0.3).
- **Files:** `AGENT_CHANGELOG.md`, published `get`
- **Notes:** Other PC: Diagnostics → Check and fix, then reopen Slip. The menu should show 0.0.4. Chat is one PC at a time. Check and fix can show one Windows approval prompt because it adds the firewall rules and sets the network to Private together.

## 2026-09-30 - Slip 0.0.4 single-person chat, sectioned menus, stronger one-click

- **Why:** The multi-select chat picker caused errors when sending to more than one PC; the user wants one-to-one click-to-connect chat and cleaner, sectioned menus, plus a network Private/Public toggle and a one-click diagnostic that also reinforces setup and makes the network Private.
- **Changes:** Version 0.0.4.
  - **Chat is one-to-one:** `Show-ChatPeople` rewritten to use `Read-ArrowMenu` (single select) - pick a PC, Enter/number opens the chat with just that PC. No checkboxes, no space-to-toggle, no multi-select. `Show-StartChat` still wraps the pick as a one-element list, so `chathost`/`Start-ChatWith` are unchanged.
  - **Menu sections:** `Read-ArrowMenu` gained `-SeparatorsAfter <int[]>` that draws a `----` line after the given item indices (purely visual; numbering and return values unchanged). Main menu: `Send files, Receive files | Chat | More` (Folders removed from here). More menu: `Folders, How to send and receive, Check connection | Quick receive, Auto chat, Network | Diagnostics`. Diagnostics: `Check and fix, Status, Repair, Remove | Rename this PC, Open Debug folder, Chat log` - Upgrade removed (Check and fix updates), Network moved to More.
  - **Network toggle:** new `Set-ActiveNetworksPublic`, `Show-NetworkToggle` (shows Private/Public and flips it, elevating when needed), and `netpublic` entry. More menu item shows the current type and toggles it.
  - **One-click reinforced:** `Repair-SlipStartupEntries` now runs `Ensure-SlipFiles` (recreate folders, refresh profile + command) like Repair. `Invoke-SlipSelfHeal` now actually makes the network Private (was report-only) and combines firewall + network admin work into a single `fixnet` elevated pass, so the user sees one UAC prompt instead of two. New `fixnet` entry = `Install-FirewallRules` + `Set-ActiveNetworksPrivate`.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` as 0.0.4. Parser clean; separator render verified. Group-Policy-locked machines may still refuse the network/firewall change; the report shows Firewall FAILED / Network `Still Public` in that case.

## 2026-09-30 - Slip 0.0.3 installed here and published

- **Why:** The other PCs need the chat window that stays open and explains a failed connect-back
- **Changes:** Installed the local 0.0.3 script with `SLIP_FETCHED_UPGRADE=1` and published it as `get` (was 0.0.2).
- **Files:** `AGENT_CHANGELOG.md`, published `get`
- **Notes:** Other PC: Diagnostics → Check and fix, then reopen Slip. The menu should show 0.0.3. A guest window that cannot connect back stays open and names the address and port. The PC that starts the chat still has to be on a Private network.

## 2026-09-30 - Slip 0.0.3 chatjoin shows why it could not connect back

- **Why:** A report that the "geforce" laptop (192.168.152.81) could not send or receive chats and "some kind of elevated protection" was suspected. Diagnosed live from the Vengeance PC (192.168.152.132, same /24): geforce's Auto chat listener was up and advertising CHAT on 8787 (QUICK on 8788), 8787 was reachable, a test INVITE returned OK, and a full end-to-end test (real host listener on 8790 + INVITE) had geforce's guest window spawn, connect back out, and send `HELLO` correctly. So geforce is not blocked - the chat mechanism works with it. The "window pops up and vanishes" is the guest failing to connect back to the PC that started the chat (that PC on a Public network or its chat port firewalled), and `chatjoin`'s catch just printed "chat closed" and exited, so the window flashed shut with no readable reason.
- **Changes:** Version 0.0.3. The `chatjoin` entry's failure path now clears the screen and shows a plain explanation - which PC/address:port it tried to connect back to, that the other PC likely is Public or firewalled, the fix (Check and fix + set Private on that PC), the exception detail - and waits on `Press Enter to close` instead of vanishing. The connect timeout now throws a specific `timed out connecting to <addr>:<port>` message. No protocol change.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Root cause of the original report is environmental: the initiating/other PC (likely Pink Juice, and earlier Vengeance) on a Public network, not geforce and not endpoint security. Diagnostic method worth reusing: from a peer, listen on UDP 47654 for beacons to confirm a PC's listener/port, TCP-probe its 8787-8796, then send a crafted `INVITE|room|hostPort|fromB64|bodyB64` and watch for `OK` and a connect-back to hostPort. Republish `get` as 0.0.3 (optional; this is a diagnostics/UX change, no wire change).

## 2026-09-30 - Slip 0.0.2 installed here and published

- **Why:** Both PCs have to be on 0.0.2 before the chat input fix can be tested
- **Changes:** Installed the local 0.0.2 script with `SLIP_FETCHED_UPGRADE=1` and published it as `get` (was 0.0.1).
- **Files:** `AGENT_CHANGELOG.md`, published `get`
- **Notes:** Other PC: Diagnostics → Check and fix, then reopen Slip. Test by sending several messages from one side while the other does not press a key. If messages still wait for Enter, read `chat.session.input` in that PC's `Documents\Slip\Debug\transfer-debug.log`. `mode=console-keys` is the new path. `mode=read-host` means the window had no live keyboard and used line input instead. Do not send chat key checks back through `$Host.UI.RawUI.KeyAvailable`.

## 2026-09-30 - Slip 0.0.2 fix chat receive stalling until a key is pressed

- **Why:** In a live chat, the receiver only saw the second and later messages after pressing Enter (or any key). The opening message showed fine because it is printed before the loop starts. Root cause: the chat input loop gated its key read with `$Host.UI.RawUI.KeyAvailable`, which under Windows Terminal / ConPTY (the Windows 11 default) can report a key available when there is none; `Read-SlipKey` (`$Host.UI.RawUI.ReadKey`) then blocked waiting for a real key, so the socket was not polled and incoming messages queued until the user pressed something.
- **Changes:** Version 0.0.2. Added `Test-ChatConsoleKeys` (probes `[Console]::KeyAvailable`) and `Read-ChatKey` (`[Console]::ReadKey($true)`, mapped to the same `@{ Name; Char }` shape). `Show-ChatSession` now decides raw vs fallback with `Test-ChatConsoleKeys`, gates input with `[Console]::KeyAvailable` (reliable and non-blocking under conhost and ConPTY), and reads keys with `Read-ChatKey` in both the main loop and the Esc leave-confirm. Idle poll is 10 ms. Logs the chosen input mode to transfer-debug as `chat.session.input`. The menu key helpers (`Read-SlipKey`, `Read-ArrowMenu`, `Read-PickerAction`) are unchanged - they only wait for input and have no async receive to starve.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` as 0.0.2; both PCs need it. No wire-protocol change. If a report still shows the stall, check the `chat.session.input` line in `Documents\Slip\Debug\transfer-debug.log`: `mode=read-host` means the window fell back to blocking line input (input redirected / not a real console), a different path. Do not route the chat's live input back through `$Host.UI.RawUI.KeyAvailable`.

## 2026-09-30 - Slip 0.0.1 is installed here and published

- **Why:** Go live so this PC and the other PCs run the same chat-window build
- **Changes:** Ran `lanfile.ps1 upgrade` with `SLIP_FETCHED_UPGRADE=1`, then cleared that variable. Installed host is `AppData\Local\Slip\slip-host.ps1` version 0.0.1. Published that file as `get` (was 1.3.5).
- **Files:** `AGENT_CHANGELOG.md`, published `get`
- **Notes:** Auto chat is on. Quick receive was already on and stayed on. The active network `Subzerolounge 3` is Public, so other PCs cannot connect until Diagnostics → Network makes it Private. Other PCs that are still on 1.3.x should run `slip upgrade` once; Check and fix exists only after they are on 0.0.1. Both PCs must be on 0.0.1 for chat to open in its own window.

## 2026-09-30 - Slip 0.0.1 one-click self-heal, chat in its own cmd window, version reset

- **Why:** Consolidate the scattered diagnostic actions into one "check and fix everything" flow with a report, and stop the chat from taking over / opening in the PowerShell window. Version scheme reset to 0.0.x per request.
- **Changes:** Version reset **1.3.7 -> 0.0.1** (`lanfile.ps1`, all `UI.md`/`NETWORKING_WSL.md` version strings).
  - **One-click self-heal:** New Diagnostics item **1 "Check and fix (one click)"** (`Show-SelfHealScreen` + `Invoke-SlipSelfHeal`). Runs in order, printing each step live so the window never looks frozen: (1) `Get-OnlineSlipVersion` downloads `get`, compares to `$script:Version`; if different, runs the fetched payload with `SLIP_FETCHED_UPGRADE=1 upgrade -Quiet`; (2) `Set-SlipScriptPolicy -Quiet` enables local scripts; (3) `Repair-SlipStartupEntries` re-adds the `slip` command + PATH (`Install-SlipCommand`) and the HKCU Run keys for Auto chat / Quick receive when missing or wrong; (4) firewall rules via `Install-FirewallRules`, elevating through `Invoke-ElevatedEntry -Entry 'firewall'` (UAC) when not admin; (5) network type reported (not forced). Ends with a per-check report (OK / FIXED / CHECK / FAILED) and an overall verdict. The standalone "Fix script policy" Diagnostics item from the previous build was removed (subsumed); `Set-SlipScriptPolicy` stays as the worker.
  - **Chat in its own cmd window:** New `Start-ChatConsole` opens a normal Command Prompt window (`%ComSpec% /c title ... & powershell -File ... <entry>`) that hosts the chat, with the title stripped to `[\w \-]` so a peer name cannot inject a command (entry args are base64 + IP/port/room only). `Start-ChatPopup` (incoming invite) now uses it. Hosting a chat no longer runs inline in the Slip window: `Show-StartChat` encodes the picked peers (`ConvertTo-Json` -> base64 `-PeopleB64`) and the message (`-OpeningB64`) and launches a new `chathost` entry, which decodes and calls `Start-ChatWith`, then pauses on `Press Enter to close`. Added the `chathost` token, the `-PeopleB64` option, and its dispatch. The hidden `chatlisten` background listener stays PowerShell (no window).
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` as 0.0.1. Verified: parser clean; people JSON base64 round-trips (single element preserved via `@()`); the cmd launch passes a spaced `-File` path + base64 args through to the entry parser correctly (tested with `Start-Process %ComSpec% -ArgumentList $full`). The self-heal update step reinstalls in a child process, so the running window keeps the old code until Slip is reopened - the report says so. No wire-protocol change; 0.0.1 chats with older builds. Firewall/update steps can raise UAC by design ("prompt the admin").

## 2026-09-30 - Slip 1.3.7 chat disconnect notices, clean exit, send back nav, script policy fix

- **Why:** Four reported issues. A PC that closed its window or pressed Ctrl+C left the other side hanging with no notice. There was no obvious, safe way to leave a chat. Backing out of the "Choose a PC" send screen dropped all the way to the main menu, losing the file selection. And one PC showed a red "running scripts is disabled" message at every PowerShell start.
- **Changes:** Version 1.3.7.
  - **Disconnect detection (issue 1):** Added `Test-SocketDead` (uses `Socket.Poll` + `Available` to spot a closed/reset TCP connection). `Show-ChatSession` now polls each member (host) and the host link (guest) every loop, so an abrupt close/Ctrl+C/network drop surfaces promptly as `left` (host broadcasts `SYS|left` to the rest) or `<host> left. Chat closed.` (guest). Graceful `BYE`/`SYS|closed` paths are unchanged. `Show-ChatSession` gained a `-Client` param; the `chatjoin` guest call now passes its `TcpClient` so the guest can detect a dead host link.
  - **Clean exit with confirm (issue 2):** Esc now leaves the chat. If the other PC is still connected it asks `[Y] Leave [N] Stay`; if the other PC has already gone, one Esc exits immediately. A guest window pauses on `Press Enter to close` at the end so the closing notice is readable before the popup disappears.
  - **Send back-navigation (issue 4):** `Show-SendFiles` wraps file selection and PC choice in one step loop. Backing out of the PC search (`0`, or `Q`/`else` on the no-PC-found fallback, or cancelling manual IP entry) now `continue`s back to the file list with the same items still checked, instead of returning to the main menu. `Q` on the file list itself still returns to the main menu, and a completed/failed send still ends the screen.
  - **Script policy fixer (issue 3), added on explicit user request:** New `Set-SlipScriptPolicy` and Diagnostics item **9 "Fix script policy"** (`Show-ScriptPolicyScreen`). It is user-initiated only (not run automatically on install), shows the account's current `Get-ExecutionPolicy -Scope CurrentUser`, asks `[Y] Fix now [N] Back`, then sets `CurrentUser` (and `LocalMachine` when already elevated) to `RemoteSigned` and unblocks Slip's installed script and profile files. Group-Policy-forced (`MachinePolicy`/`UserPolicy`) settings are reported as un-fixable. This addresses the red "running scripts is disabled on this system" message that appears when a PowerShell window opens on a PC left at `Restricted`.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` for 1.3.7. No wire-protocol change, so 1.3.7 chats with older builds. The script-policy fixer changes a Windows security setting, so it is deliberately behind a manual Diagnostics action with a confirmation prompt rather than run during install — keep it that way; do not auto-run execution-policy changes without a prompt.

## 2026-09-30 - Slip 1.3.6 instant chat typing, no double line

- **Why:** In a chat window each message showed twice — the live `> hi` draft line stayed on screen and then `vengeance: hi` printed under it — and typing felt laggy.
- **Changes:** Version 1.3.6. `Show-ChatSession` now evaluates the console input mode once (`$rawInput`) instead of calling `Test-SlipConsoleKeys` every loop. Added `Clear-ChatInput`/`Show-ChatInput`/`Write-ChatLine` helpers: the `> draft` line is drawn once at session start and fully cleared and redrawn on each keystroke, so backspacing over several characters no longer leaves artifacts. On Enter the draft line is cleared and replaced by a single `You: <text>` line (was a leftover `> text` line plus a separate `name: text` echo). Incoming messages and room notices (`joined`/`left`/`chat closed`) print through `Write-ChatLine`, which clears the input line, prints, then redraws the preserved draft, so an arriving message never collides with what you are typing. The `Read-Host` fallback path no longer re-echoes the sent text (the console already showed it), removing the duplicate there too. Idle poll sleep lowered from 15 ms to 8 ms for snappier incoming display.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` so other PCs pick up 1.3.6. The UI.md chat example already documented the single `You:` line; the code now matches it. No wire-protocol change, so a 1.3.6 window still talks to older builds. Only the raw-key console path (the normal popup) gets the live single-line prompt; the redirected-input fallback still uses `Read-Host`.

## 2026-09-30 - Teensy online bootstrap uses fetched Slip directly

- **Why:** The board should upgrade old laptops without depending on whatever stale local `slip` command is already installed there.
- **Changes:** Refreshed `teensy-slip-installer` and `teensy-slip-installer-source-debug` encoded payloads. The board opens PowerShell, downloads `https://everlasting-dev.github.io/get` to `%TEMP%\slip-upgrade.ps1`, validates that it is the Slip installer, sets `SLIP_FETCHED_UPGRADE=1`, and runs the fetched file with `upgrade -QuickReceive -Quiet`. The payload writes `Documents\Slip\Debug\teensy-upgrade-log.txt` with the downloaded version, byte count, installer exit code, and failures.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** The regular sketch auto-starts after 6 seconds with `REQUIRE_ARM_PIN=false`. Set it to `true` if you want physical arming on pin 2 before it types. Windows may still show an approval prompt for firewall/UAC; the Teensy payload handles the typing, not secure-desktop approvals.

## 2026-09-30 - Slip 1.3.5 visible online upgrade check

- **Why:** A laptop test made it look like `slip upgrade` was rerunning the local saved script, because the live script had been changed without a visible version bump.
- **Changes:** Version 1.3.5. `slip upgrade` still downloads `https://everlasting-dev.github.io/get` first on any PC that has the 1.3.2+ launcher/profile, and the upgrade path now writes explicit debug lines for `Upgrade downloaded online payload` and `Upgrade applying fetched payload` with the payload version and byte count. Docs now call out that pre-1.3.2 laptops need the one-time online bootstrap before plain `slip upgrade` can self-update.
- **Files:** `lanfile.ps1`, `UI.md`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** If a laptop still has an older local-only `slip` launcher, it cannot learn the new online behavior by running that same old local launcher. Run the online bootstrap command once; after the menu shows 1.3.5, future `slip upgrade` calls fetch online first.

## 2026-09-30 - Slip 1.3.4 upgrade and chat polish

- **Why:** The old upgrade wrappers could leave stale files behind, background listeners could hold the installed host script during upgrade, chat invites felt delayed, and chat failures were hard to diagnose from the debug folder.
- **Changes:** Version 1.3.4. Profile and in-session `slip` wrappers now generate from literal templates, so the upgrade regex cannot be broken by PowerShell string expansion. Command Prompt `slip upgrade` downloads `get` and forwards only the intended arguments to the fetched script. Upgrade stops old Quick receive and Auto chat listeners before refreshing `slip-host.ps1`, skips copying when source and target hashes match, restarts Auto chat from the latest host script, preserves Quick receive unless `-QuickReceive` is passed, and removes old `slip.ps1`, listener wrappers, and temp upgrade files. Chat invite polling is faster and chat discovery/invite/session breadcrumbs now append to `Documents\Slip\Debug\transfer-debug.log`. Added `NETWORKING_WSL.md` for the 172.x virtual adapter issue.
- **Files:** `lanfile.ps1`, `UI.md`, `upgrade-existing-slip.ps1`, `upgrade-existing-slip.cmd`, `NETWORKING_WSL.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get`. After one fetched upgrade, old PCs should be able to use plain `slip upgrade` for later releases. Use `slip upgrade -QuickReceive -Quiet` when the receiver should be enabled as part of the upgrade.

## 2026-09-30 - slip works when scripts are disabled

- **Why:** Typing `slip` in PowerShell on another PC failed with `running scripts is disabled` for `AppData\Local\Slip\slip.ps1`. PowerShell finds a `.ps1` on PATH before `slip.cmd` and then loads it under the normal script policy. The profile function written by 1.3.2 was also invalid: an expandable here-string ate `$'` in the upgrade regex, so the profile could not parse.
- **Changes:** Version 1.3.3. The installed script is `AppData\Local\Slip\slip-host.ps1`. `slip.cmd` stays beside it for windows that already have that folder on PATH, and a second copy is in `AppData\Local\Slip\bin`, which is the PATH entry for new windows. Both launchers use `-ExecutionPolicy Bypass`. The old `slip.ps1` name is deleted on install so it cannot shadow the command. The profile regex is `` `(?i:upgrade|update|migrate)`$ `` inside the expandable here-string so the written file contains a closed quote. This PC's broken profile quote was repaired in place.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`, and this PC's `Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`
- **Notes:** A PC that still has the old `slip` command does not get this until it runs the online updater once. Command Prompt:

```bat
powershell -NoProfile -ExecutionPolicy Bypass -Command "$p=Join-Path $env:TEMP 'slip-upgrade.ps1'; try { [Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12 } catch { }; Invoke-RestMethod https://everlasting-dev.github.io/get -OutFile $p -ErrorAction Stop; $env:SLIP_FETCHED_UPGRADE='1'; powershell -NoProfile -ExecutionPolicy Bypass -File $p upgrade; exit $LASTEXITCODE"
```

After that, type `slip` again. Expect version 1.3.3, Chat on the menu, and Auto chat on. Outbox, Inbox, and the display name stay. Quick receive stays as that PC left it. Do not put a file named `slip.ps1` on PATH. Do not write `$'` inside an expandable here-string; escape the dollar. Live file is `https://everlasting-dev.github.io/get`.

## 2026-09-30 - old slip upgrade never downloaded chat

- **Why:** `slip upgrade` on a PC installed before 1.3.1 runs that PC's local `slip.ps1`. The old command passes `menu upgrade` to the file already on disk, and that file copies itself onto itself, so Chat never arrives even though the live `get` file has it.
- **Changes:** Version 1.3.2. A fetched upgrade still installs the new script and turns Auto chat on when the firewall prompt is dismissed. The Command Prompt downloader exits 1 if the download throws. An older PC needs one online command (in UI.md) before `slip upgrade` starts downloading on its own.
- **Files:** `lanfile.ps1`, `UI.md`, `upgrade-existing-slip.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** Do not tell someone that typing `slip upgrade` on a pre-1.3.2 install will fetch Chat. They have to run the online command once.

## 2026-09-30 - slip upgrade downloads the latest script

- **Why:** `slip upgrade` in Command Prompt was reapplying the copy already on the PC, so chat and later fixes never arrived
- **Changes:** `slip`, `slip upgrade`, `slip update`, and `slip migrate` in Command Prompt and PowerShell download `get` and run that file. The upgrade turns Auto chat on, keeps Quick receive as it was, and leaves Outbox, Inbox, and the display name in place. Version is 1.3.1.
- **Files:** `lanfile.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** `slip` with no arguments still opens the menu. Republish `get`. Correction: a `slip.cmd` installed before 1.3.2 does not download. It reruns the local script. The 1.3.3 entry has the one command that replaces it.

## 2026-09-30 - shorter menus, version, auto chat on

- **Why:** The More list had grown into everyday actions plus repair and logs, and chat needed to open on the other PC the way Quick receive accepts a file
- **Changes:** Slip is version 1.3.0, shown on every menu. The main menu is Send, Receive, Chat, Folders, and More. Repair, Upgrade, Remove, Rename, Network, Debug, and the chat log sit under Diagnostics. Auto chat is on unless `Documents\Slip\chat.txt` says off. A chat invite opens a window, uses `joined`, `left`, and `chat closed`, and appends a DPAPI-encrypted log at `Documents\Slip\Chat\chat.log`.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Auto chat listens on 8787-8796 and beacons `CHAT`. Quick receive stays off until turned on. Republish `get` before other PCs can open a chat against this build.

## 2026-09-29 - reach the real LAN, not WSL or a Public profile

- **Why:** Send and receive failed on a PC with WSL 2 installed. Three separate
  causes: the firewall rules are scoped to Private but Windows had tagged the
  active WiFi Public, and a profile applies per interface, so neither rule was
  ever in force; `Get-LanIPv4` returned every non-loopback address including
  WSL's `vEthernet (WSL (Hyper-V firewall))` at 172.21.0.1, and returned it
  first, so the Sending screen printed an unreachable URL and Status showed the
  wrong LAN address; and every beacon went out through one unbound UdpClient to
  255.255.255.255, which Windows routes to the single lowest-metric interface,
  on this PC a Wi-Fi Direct stub at metric 25 rather than WiFi at 45.
- **Changes:** Added a `$script:NetHelpers` scriptblock holding
  `Test-VirtualAdapterAlias`, `Get-SubnetBroadcast`, `Get-LanAddressRows`,
  `Get-BroadcastTargets`, and `Send-BeaconPayload`, dot-sourced both into the
  script and into the presence-loop runspace so there is one copy. `Get-LanIPv4`
  now drops virtual adapters and orders what is left by interface metric, with
  `-IncludeVirtual` for the self-address checks in `Test-OwnAddress` and
  `Show-SendFiles`. All four beacon senders now bind a socket to each real local
  address and send a subnet-directed broadcast, so discovery no longer depends
  on the routing table picking the right adapter. Added
  `Get-ActiveNetworkProfiles`, `Get-PublicNetworkProfiles`,
  `Set-ActiveNetworksPrivate`, `Invoke-MakeNetworkPrivate`, and a `netprivate`
  elevated entry. `Show-NetworkWarning` now points at the fix, Install and
  Repair offer it, Status gained `Network:` and a plain `Receiving:` verdict,
  Check connection names the addresses it ignored, and More gained a Network row.
  Unattended install and upgrade never prompt; they warn and carry on, and take
  a new opt-in `-MakeNetworkPrivate` flag that does it in the same elevation
  prompt as the firewall rules.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Firewall rules stay `-Profile Private` on purpose. Quick receive
  takes a file with no PIN, so opening the ports on a Public network would make
  an Inbox that any device on that subnet can write to. Verified on this PC: a
  3 MB loopback `serve`/`recv` round trip matches by SHA256, the presence loop
  and the Quick receive listener both beacon on the real LAN, and discovery
  found six other PCs. Worth knowing when testing: this PC has two stray
  `Windows PowerShell` inbound Allow rules on the Public profile for every port,
  which is what made Slip appear to work here while the Slip rules were inert. A
  clean PC will not have them. Republish `get`.

## 2026-09-29 - old PC rescue and Quick Receive diagnostics

- **Why:** One upgraded PC still used an old PowerShell profile function that launched `slip.ps1` without `-ExecutionPolicy Bypass`, Repair hit the old self-copy bug, a pasted/repeated number crashed the send picker, and Quick Receive discovery needed a fallback when UDP beacons do not appear.
- **Changes:** Documented the `-NoProfile` online updater as the rescue path for old profile functions, changed the PowerShell profile function to prefer the local installed `slip.ps1` with `-ExecutionPolicy Bypass`, made the send file picker ignore oversized numeric input instead of throwing, added manual-IP Quick Receive sending with port scanning, and expanded Status to show Quick Receive state, registry startup, listener PID, and open port. The listener now writes `listen.port` beside `listen.pid`.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** If a receiving PC does not show up but Status says the listener is alive, senders can choose Type IP for Quick receive and enter the receiver's LAN address.

## 2026-09-29 - existing install upgrade path

- **Why:** PCs that already used an older Slip installer need a one-step migration to the local `slip.ps1`, registry Quick Receive startup, and newer debug logging without deleting user files.
- **Changes:** Added `upgrade` / `update` / `migrate` / `repair` command-line entries, made Repair refresh Quick Receive startup when it is already on, preserved existing display names during upgrades, skipped self-copy when Slip is already running from its installed local script, added `upgrade-existing-slip.ps1` plus `upgrade-existing-slip.cmd`, and switched the Teensy installer payloads to the upgrade entry.
- **Files:** `lanfile.ps1`, `UI.md`, `upgrade-existing-slip.ps1`, `upgrade-existing-slip.cmd`, `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `teensy-slip-installer-lan/teensy-slip-installer-lan.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Run `slip upgrade -QuickReceive -Quiet` on old PCs, or use the online updater command from `UI.md` if `slip` is not recognized.

## 2026-09-29 - registry-first Quick Receive startup

- **Why:** Quick Receive should be present after reboot through the registry, not only as a scheduled-task fallback.
- **Changes:** Made `HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SlipQuickReceive` the primary startup registration whenever Quick Receive is enabled, removed any old scheduled-task startup during enable/disable, and updated UI wording to call out the registry entry.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** The startup command points at the installed local script: `AppData\Local\Slip\slip.ps1 listen`.

## 2026-09-29 - transfer debug log and three-PC issue fixes

- **Why:** Received `bug*.txt` / `error.txt` reports showed three field issues: slow PCs saw the Teensy type only the tail of the Base64 payload into PowerShell, one PC blocked creation of the old `slip-listen.cmd` wrapper as potentially unwanted software, and one repair flow returned to a shell where `slip` was not yet on PATH.
- **Changes:** Added `Documents\Slip\Debug\transfer-debug.log` with append-only transfer events, added Debug folder access from More and Status, changed install to keep a local `AppData\Local\Slip\slip.ps1` copy instead of wrappers that re-download on every run, removed the stale listener wrapper, and made Quick Receive startup call the local script directly. Paced Teensy payload typing and raised the staged PowerShell wait to 2.2s for slower PCs.
- **Files:** `lanfile.ps1`, `UI.md`, `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Local install exits `0`; Quick Receive is running from the local script; loopback Quick Receive returned `HTTP/1.1 200 OK` and wrote `quick.receive.request` / `quick.receive.saved` to `transfer-debug.log`.

## 2026-09-29 - final Teensy timing and auto-exit polish

- **Why:** The presentation path needed less dead time between opening PowerShell and starting the install, and the debug installer should not wait for a final Enter press.
- **Changes:** Reduced the Run dialog wait to 1.0s and the staged PowerShell wait to 1.2s in both source installer sketches, added a delayed `exit` keystroke so the staging PowerShell closes after installation, and regenerated the debug encoded payload without `Read-Host` while writing clean ASCII logs to `Documents\Slip\Debug`.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Fresh debug and regular payload runs both returned exit `0`; reports show downloaded `90166` bytes, installer exit `0`, Quick receive on, startup fallback registered.

## 2026-09-29 - Quick receive startup fallback and faster Teensy payloads

- **Why:** The Teensy debug run reached Slip but failed because `schtasks /Create` returned `Access is denied`, and the debug log lived on Desktop. The regular Teensy sketch also still typed raw PowerShell, which could be corrupted by the active keyboard layout.
- **Changes:** Made Quick receive treat scheduled-task failure as non-fatal, fall back to a per-user Run startup entry, continue starting the listener for the current session, and write Slip logs to `Documents\Slip\Debug`. Updated both source Teensy sketches to use shorter waits and alphanumeric-only encoded payloads that log under `Documents\Slip\Debug`.
- **Files:** `lanfile.ps1`, `UI.md`, `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Local `lanfile.ps1 install -DisplayName $env:COMPUTERNAME -QuickReceive -Quiet` now exits `0` on this PC; `Documents\Slip\Debug\slip.log` shows the scheduled-task denial, startup-entry fallback, and listener start.

## 2026-09-29 - encoded Teensy debug payload

- **Why:** The raw PowerShell payload was corrupted by the Windows keyboard layout, turning quote-prefixed text into characters like `O`/`I` variants and leaving PowerShell at the `>>` continuation prompt.
- **Changes:** Replaced the source-debug sketch's raw PowerShell payload with a `-EncodedCommand` payload containing only alphanumeric base64, plus a simple `powershell -NoProfile -ExecutionPolicy Bypass -EncodedCommand` prefix.
- **Files:** `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Decoded the embedded base64 from the sketch and verified the decoded PowerShell parses, contains the live `get` URL, and writes `Desktop\slip-install-debug.txt`.

## 2026-09-29 - known-good Teensy debug payload

- **Why:** The board was detected as a keyboard, but the updated install command did not leave logs or enable Quick receive, so the payload needed a simpler visible diagnostic path.
- **Changes:** Made the source-debug sketch auto-run after 12 seconds, replaced alias-based download with `Invoke-WebRequest -UseBasicParsing`, and simplified the debug payload so each step writes to `Desktop\slip-install-debug.txt` and pauses visibly.
- **Files:** `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `teensy-slip-installer/teensy-slip-installer.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Extracted the actual typed payloads from the sketches and validated them with the PowerShell parser.

## 2026-09-29 - Teensy installer visible failure logging

- **Why:** The Teensy command opened PowerShell and then closed because `lanfile.ps1 install` calls `exit` when run as a script file.
- **Changes:** Updated the Teensy source installer commands to launch the downloaded installer in a child `powershell.exe` process, added Desktop logs for silent installs, and made the debug sketch keep the parent PowerShell open with a transcript at `Desktop\slip-install-debug.txt`.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `teensy-slip-installer-lan/teensy-slip-installer-lan.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Extracted `INSTALL_SCRIPT` / `DEBUG_SCRIPT` from the sketches and validated them with the PowerShell parser.

## 2026-09-29 - receive open reliability and script hardening

- **Why:** Files could transfer successfully without Inbox opening, and Windows Defender blocked the script because the old help/elevation path still contained an inline download-and-execute pattern.
- **Changes:** Removed `irm | iex` style execution, hardened the installed `.cmd` wrappers against stale temp files and download failures, forced received-file Explorer reveal after completed transfers, added background receive logging at `Documents\Slip\slip.log`, made Quick receive startup check scheduled-task failures, sanitized received file names, and added TLS 1.2 setup before web downloads.
- **Files:** `lanfile.ps1`, `UI.md`, `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `teensy-slip-installer-lan/teensy-slip-installer-lan.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Parser check passes, and `powershell -File .\lanfile.ps1 recv` now reaches the expected argument error instead of Defender blocking the script. Published to `Everlasting-dev/everlasting-dev.github.io` through final commit `2b0c45d`; live Pages verified with the new markers.

## 2026-09-29 - Teensy arming and Run timing guard

- **Why:** The Teensy can type into the wrong focused app if Windows has not opened the Run dialog yet.
- **Changes:** Added a pin-2-to-GND arming step, LED waiting feedback, a `REQUIRE_ARM_PIN` switch for timed auto-start testing, a desktop focus step before Run, longer Run and PowerShell waits, and the same guardrails to the deprecated LAN sketch.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `teensy-slip-installer-lan/teensy-slip-installer-lan.ino`, `AGENT_CHANGELOG.md`
- **Notes:** The Teensy is still blind HID, so the arm step is the intentional safety gate before it sends keystrokes.

## 2026-09-29 - staged Teensy PowerShell entry

- **Why:** Long `-EncodedCommand` payloads typed directly into the Windows Run dialog can appear to do nothing because Run/ShellExecute handling is flaky with very long commands.
- **Changes:** Updated the main and source-debug Teensy sketches to open PowerShell with a short Run command, then type the Slip installer command into the PowerShell window. The debug sketch stays visible and pauses with status.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Use the debug sketch first when testing a new PC. Upload with USB Type `Keyboard` to avoid ProECU reacting to serial/COM interfaces.

## 2026-09-29 - published GitHub Pages get

- **Why:** The Teensy source installer depends on `https://everlasting-dev.github.io/get`, which was still serving an older Slip build without unattended Quick receive support.
- **Changes:** Published current `lanfile.ps1` to `Everlasting-dev/everlasting-dev.github.io` as root `get` in commit `4abf7a63beea61f9ebf595fd19a6886bb061f0eb`.
- **Files:** remote `get`, `AGENT_CHANGELOG.md`
- **Notes:** Verified GitHub raw and GitHub Pages both contain `QuickReceive`, `Enable-QuickReceive`, and `Send-EnvironmentChanged`.

## 2026-09-29 - Teensy source debugging and PATH notification

- **Why:** The source Teensy install can fail invisibly when the public `get` file is stale, and `slip` may not be visible to newly opened CMD windows until Windows notices the PATH change.
- **Changes:** Added `Send-EnvironmentChanged` after PATH edits and added `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, a visible source installer that reports when the online `get` file has not been republished yet.
- **Files:** `lanfile.ps1`, `teensy-slip-installer-source-debug/teensy-slip-installer-source-debug.ino`, `AGENT_CHANGELOG.md`
- **Notes:** Current public `get` was checked and appears older than local `lanfile.ps1`; republish before using the silent source sketch.

## 2026-09-29 - source Teensy installer with Quick receive

- **Why:** The LAN and local-path sketches were useful diagnostics, but the desired demo path is to fetch the installer from the published source, name the PC automatically, and enable Quick receive.
- **Changes:** Added unattended install support for `-QuickReceive` / `-EnableQuickReceive` / `-AutoReceive`, made unattended installs write a display name defaulting to `$env:COMPUTERNAME`, refactored Quick receive enabling for no-prompt setup, and updated `teensy-slip-installer.ino` to download from `https://everlasting-dev.github.io/get`.
- **Files:** `lanfile.ps1`, `teensy-slip-installer/teensy-slip-installer.ino`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Republish `get` before testing the source sketch on another PC. Windows UAC/firewall approval can still appear when firewall rules are missing.

## 2026-09-29 - Teensy diagnostics and LAN installer

- **Why:** The local-path Teensy sketch only works on this PC, and another PC needs either a visible keyboard test or a private way to fetch the installer.
- **Changes:** Added `teensy-keyboard-test/teensy-keyboard-test.ino` for Notepad verification and `teensy-slip-installer-lan/teensy-slip-installer-lan.ino` for fetching `lanfile.ps1` from this PC at `192.168.152.132:8000`.
- **Files:** `teensy-keyboard-test/teensy-keyboard-test.ino`, `teensy-slip-installer-lan/teensy-slip-installer-lan.ino`, `AGENT_CHANGELOG.md`
- **Notes:** LAN sketch requires a local web server from the project folder before plugging into the target PC.

## 2026-09-29 - Teensy local demo sketch

- **Why:** A Teensy 4.1 demo should launch the supported unattended installer without driving Slip's interactive menus.
- **Changes:** Added `teensy-slip-installer/teensy-slip-installer.ino`, which opens Windows Run and executes the local `lanfile.ps1 install -DisplayName 'Demo PC' -Quiet` command through PowerShell encoded-command mode.
- **Files:** `teensy-slip-installer/teensy-slip-installer.ino`, `AGENT_CHANGELOG.md`
- **Notes:** This targets the local script path on this PC. The online `get` URL needs to be republished before using an internet-hosted unattended install command.

## 2026-09-29 - unattended install entry

- **Why:** Presentations and scripted demos need a supported no-menu install path instead of timed keyboard input through the interactive launcher.
- **Changes:** Added `install`/`setup` entry parsing, optional `-DisplayName`, `-Quiet`, and `-Silent` flags, a shared display-name cleaner, and an unattended installer function that returns a process exit code. Existing interactive install and repair behavior stays unchanged.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** This is unattended, not stealth. Windows firewall/UAC approval still appears when needed.

## 2026-09-28 - Explorer reveal and transfer polish

- **Why:** Finished receives opened Inbox inconsistently, Quick receive had its own rough save path, and a few generated names still assumed the app was always called Slip.
- **Changes:** Added shared Explorer focusing/reveal helpers, selected the saved file after normal or Quick receive when possible, added Open Inbox to the main menu, cleaned up empty-Outbox refresh, made generated command/listener/task names follow `$script:CommandName`, preserved Quick receive filenames with UTF-8-safe headers, gave multi-item zips clearer names, and removed partial downloads if a transfer stops early.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Run the parser and loopback serve/recv checks after edits. Republish `get` so the online command matches.

## 2026-09-28 - cmd menus and quick send to a chosen PC

- **Why:** In Command Prompt the menu keys never settled on a numbered list, and Quick receive listened on a random port the firewall blocks, so a small send sat there and Inbox never opened
- **Changes:** Menus read keys through the console host, and every row shows `[1]`. Quick send lists PCs first and shows the size, then connects with a short timeout to a port in 8787-8796. A finished quick receive opens Inbox.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Turn Quick receive off and on once so the old listener, which used a random port, is replaced. Republish `get`.

## 2026-09-28 - slip command, arrows, and quick send

- **Why:** `slip` was only a profile function, so the window that just finished Repair could not run it, and Check connection only saw a PC that was already sending
- **Changes:** Install and Repair put `slip.cmd` on the user PATH and define `slip` in the current window. The open Slip menu beacons once a second. Menus take arrow keys and numbers. More has Quick receive (off by default); when on, a logon task listens and a sender can choose Send now with no PIN. A finished receive opens Inbox and brings that Explorer window forward.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** Quick receive state is `Documents\Slip\quick.txt`. The listener entry is `listen`. Remove stops the listener, the logon task, and the PATH command. Do not revert the checkbox picker. Republish `get`.

## 2026-09-28 - shorter Slip menu and percent line

- **Why:** Send was buried, and the transfer used the bulky PowerShell progress bar
- **Changes:** after install, Open Slip and `slip` open Send, Receive, Outbox, and More. Repair no longer blocks that menu. Send and receive show one updating percentage line. Added `UI.md` with each screen.
- **Files:** `lanfile.ps1`, `UI.md`, `AGENT_CHANGELOG.md`
- **Notes:** the Outbox checkbox picker stays. Quiet `serve` and `recv` do not print the percentage. Republish `get` so the online command matches.

## 2026-09-28 - Slip launcher and LAN transfer

- **Why:** one online PowerShell command should open a menu to install Octane or transfer files on the local network
- **Changes:** added `lanfile.ps1`. Launcher offers Get Octane and Get Slip. Slip installs or repairs without deleting Outbox/Inbox, stores a display name, and streams files over the LAN with a PIN. The same bytes are published as `get` on the GitHub Pages repo.
- **Files:** `lanfile.ps1`, `AGENT_CHANGELOG.md`
- **Notes:** app name is `$script:AppName` (`Slip`). Get Octane follows `ecutek_logs/octane/scripts/download-octane.ps1` plus Runscope/Octane removal from `install-octane.bat`. Repair must not delete Outbox or Inbox. Do not put the PIN in the UDP beacon. Direct test entry points are `serve` and `recv` (recv needs `-Token` as well as address, port, and PIN).
