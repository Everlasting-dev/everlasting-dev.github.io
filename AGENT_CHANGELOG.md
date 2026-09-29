# Agent change log

Internal handoff log for Cursor agents and plugins. Newest entries first.
Read before editing; append after substantive changes.

## 2026-09-29 - Registry-first Quick Receive startup

- **Why:** Quick Receive should be present after reboot through the current-user Run registry key.
- **Changes:** Replaced scheduled-task-first startup with primary `HKCU\Software\Microsoft\Windows\CurrentVersion\Run\SlipQuickReceive` registration and remove any old scheduled task when Quick Receive is enabled or disabled.
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Source hash after publish staging: `B6FB6F65362FDECB97C86A82DF31A8A5A325573DEB663226FECF2D99837639F2`.

## 2026-09-29 - Clarify Repair command wording

- **Why:** A report showed Repair could finish successfully, but typing `slip` in the already-open parent shell still failed because that shell had not refreshed PATH.
- **Changes:** Updated `get` so Repair tells users that new windows can type `slip`, while already-open windows can run the full local `slip.cmd` path immediately.
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Source hash after publish staging: `F362EC6D1D1DEE99F9E449CAA4B580D7C3F0882E7178E4B6549067EA489562B1`.

## 2026-09-29 - Publish transfer debug and local Slip command

- **Why:** Field reports showed the old downloaded command wrappers could be blocked by Defender, and transfer debugging needed a single append-only log.
- **Changes:** Replaced `get` with the local Slip script that installs `AppData\Local\Slip\slip.ps1`, uses local wrappers/startup commands, adds `Documents\Slip\Debug\transfer-debug.log`, and exposes the Debug folder from More/Status.
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Source hash after publish staging: `2B447F53D1162D15F99C18829DDE1F3A8188CBAD08F621D540705A0FF897BEBA`.

## 2026-09-29 - Quick receive startup fallback and debug logs

- **Why:** The Teensy source installer reached Slip but failed when Windows denied `schtasks /Create` for the Quick receive logon task.
- **Changes:** Replaced `get` with the local Slip script that falls back to a per-user Run startup entry, logs under `Documents\Slip\Debug`, and keeps Quick receive install non-fatal when scheduled tasks are blocked.
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Source hash after publish staging: `210735A4659E6103FECB20643588940798DD03A0870E5CDCA630005AC2B76181`.

## 2026-09-28 — cmd menus and quick send to a chosen PC

- **Why:** the published `get` script should match the Command Prompt menu fix and the Quick receive port fix
- **Changes:** replaced `get` with the script that shows numbered rows in cmd, asks which PC to send to, and listens for Quick receive on ports 8787-8796
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Quick receive stays off until the user turns it on. An already-running listener must be turned off and on so it picks up the allowed port.

## 2026-09-28 — slip command, arrows, and quick send

- **Why:** the published `get` script should match the local Slip command, presence beacon, and optional quick receive
- **Changes:** replaced `get` with the script that installs `slip` on PATH, beacons while the menu is open, uses arrow keys, and can accept a send with no PIN when Quick receive is on
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Quick receive stays off until the user turns it on. Repair still must not delete Outbox or Inbox files.

## 2026-09-28 — shorter Slip menu and percent line

- **Why:** the published `get` script should match the local launcher
- **Changes:** replaced `get` with the script that opens Send first after Slip is installed and shows a one-line transfer percentage
- **Files:** `get`, `AGENT_CHANGELOG.md`
- **Notes:** Repair stays under More and still must not delete Outbox or Inbox files.

## 2026-09-28 — launcher instead of a direct Octane install

- **Why:** the short install command should open a menu on any online PC
- **Changes:** replaced `get` with the Slip launcher script. Added `.nojekyll` so Pages serves the file as-is.
- **Files:** `get`, `.nojekyll`, `README.md`, `AGENT_CHANGELOG.md`
- **Notes:** choosing Get Octane still downloads and silent-installs the latest Octane setup. Do not point `get` back at `download-octane.ps1` alone.
