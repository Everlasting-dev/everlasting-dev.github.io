# Agent change log

Internal handoff log for Cursor agents and plugins. Newest entries first.
Read before editing; append after substantive changes.

## 2026-09-28 — launcher instead of a direct Octane install

- **Why:** the short install command should open a menu on any online PC
- **Changes:** replaced `get` with the Slip launcher script. Added `.nojekyll` so Pages serves the file as-is.
- **Files:** `get`, `.nojekyll`, `README.md`, `AGENT_CHANGELOG.md`
- **Notes:** choosing Get Octane still downloads and silent-installs the latest Octane setup. Do not point `get` back at `download-octane.ps1` alone.
