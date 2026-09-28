# Agent change log

Internal handoff log for Cursor agents and plugins. Newest entries first.
Read before editing; append after substantive changes.

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
