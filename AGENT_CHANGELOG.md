# Agent change log

Internal handoff log for Cursor agents and plugins. Newest entries first.
Read before editing; append after substantive changes.

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
