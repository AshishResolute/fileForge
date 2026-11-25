# FileForge

> Scan, organize, back up and restore your files from the terminal, safely.

**Status: early development.** Nothing below is released yet. This README describes what FileForge is being built to do, and the checkboxes show real progress.

---

## What is FileForge?

FileForge is a command-line file management tool built with Node.js and TypeScript. It is designed around one idea: **commands that touch your files must be safe by default.** Every destructive action can be previewed first, and every change can be undone or verified.

## Planned commands

| Command | What it will do |
|---|---|
| `fileforge stats <dir>` | Scan a directory and report file counts, total size, breakdown by type, largest and oldest files, and duplicates |
| `fileforge organize <dir>` | Sort a messy folder into category subfolders, with `--dry-run` preview and an undo log |
| `fileforge backup <src> --to <dest>` | Create an incremental backup with checksum verification and a manifest |
| `fileforge restore <manifest>` | Restore files from a backup, with integrity checks and conflict handling |

Running `fileforge` with no arguments in an interactive terminal will open a menu. Running it with arguments, or inside a script or pipe, will never prompt.

### Example (target behavior)

```bash
fileforge stats ./Downloads
fileforge stats ./Downloads --json > report.json
fileforge organize ./Downloads --dry-run
fileforge backup ./Downloads --to ~/backups
fileforge restore ~/backups/2026-01-01.json
```

## Design principles

- **Preview before acting.** A dry run builds the exact plan (`MovePlan`) and prints it. Applying runs the same plan.
- **No silent data loss.** Duplicates are reported, never auto-deleted. Restore never overwrites without an explicit conflict policy.
- **Everything is reversible or verifiable.** `organize` writes an undo log. `backup` writes a SHA-256 manifest. `restore` verifies it.
- **Script-friendly.** Results go to stdout, progress and errors go to stderr, and exit codes are meaningful (`0` success, `1` error, `2` bad usage).
- **Engine and UI are separate.** The core returns data, and the terminal UI only renders it.
- **Cross-platform.** Windows, macOS and Linux paths are handled through `node:path`.

## Architecture (planned)

```
src/
  cli.ts            entry point (shebang, argument handling)
  commands/         stats, organize, backup, restore
  core/
    scanner.ts      recursive scan -> FileEntry[]
    categorize.ts   extension -> category
    hash.ts         streaming SHA-256
    plan.ts         MovePlan builder (dry-run = print the plan)
    manifest.ts     read/write backup manifests
  ui/               Chalk, Boxen, Ora, Ink rendering
tests/
```

Core data structures:

```ts
type FileEntry = { path: string; size: number; mtimeMs: number; ext: string; category: string };
type MovePlan  = { from: string; to: string }[];
type Manifest  = { createdAt: string; sourceRoot: string; files: { relPath: string; size: number; sha256: string }[] };
```

## Roadmap

### Foundation
- [ ] Project setup (TypeScript, `package.json` `bin`, `npm link` for local use)
- [ ] Argument parsing (manual first, then Commander.js)
- [ ] Error handling and exit codes

### Scanner and stats
- [ ] Recursive directory scanner with ignore rules (`.fileforge/`, `node_modules`, etc.)
- [ ] File categorization by extension
- [ ] `fileforge stats` with plain text output
- [ ] Duplicate detection (group by size, then hash only candidates)
- [ ] `--json` output

### Organize
- [ ] `organize --dry-run` (plan only)
- [ ] Safe file moves with name-collision handling
- [ ] Undo log and `undo` support
- [ ] Interactive confirmation

### Backup and restore
- [ ] Pre-scan for totals and progress
- [ ] Streamed file copy with SHA-256 verification
- [ ] Incremental backups using the previous manifest
- [ ] Guard against destination inside source
- [ ] `restore` with integrity verification
- [ ] Conflict policies: skip, overwrite, rename

### Terminal UI
- [ ] Chalk colors and Boxen summary cards
- [ ] Ora spinners (on stderr)
- [ ] Block-bar breakdown charts
- [ ] Figlet banner
- [ ] Ink interactive menu

### Quality and release
- [ ] Unit and integration tests (Vitest, temporary directories)
- [ ] Cross-platform checks (Windows, macOS, Linux)
- [ ] Logging
- [ ] npm packaging and publish
- [ ] CI with GitHub Actions

## Tech stack

Node.js, TypeScript, `fs/promises`, `node:path`, `node:crypto`, `node:stream`, Commander.js, Chalk, Boxen, Ora, Figlet, Ink, Vitest.

## Development

```bash
git clone <your-repo-url>
cd fileforge
npm install
npm run build
npm link          # makes the `fileforge` command available locally
```

Commands above will work once the setup phase is complete.

## License

MIT (to be confirmed)
