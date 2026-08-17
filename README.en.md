# dsh-session-management · Session Management for DSH

English | [中文](README.md)

![Settings page](docs/screenshots/settings.png)

dsh-session-management is a session management plugin for DeepSeek Harness (DSH) Web that **fills the gaps left by the official session management**. From the Settings panel you can manage chat sessions in one place: archive, unarchive, **truly delete local chat records**, and export data in bulk. It is fully bilingual, switching instantly with the DSH locale setting.

![Archived chats manage dialog](docs/screenshots/manage.png)

## Why this plugin

DSH Web does not ship complete session management today, and long-term users hit three pain points:

- **Chats cannot be truly deleted**: the official UI only offers "archive". Archiving merely hides a session from the sidebar; the chat records stay fully on disk, with no entry point to actually remove them.
- **Archived chats cannot be restored**: there is no "unarchive". Once archived, a session is unreachable from the list for good — invisible yet still on disk.
- **Export is one-by-one manual**: the official export handles a single session at a time, which means a lot of clicking once sessions pile up.

This plugin closes all three gaps: **real deletion** (removes the session log files on disk), **restoration** (unarchive one session or a whole group), and **bulk export** (ZIPs identical to the official format).

| Capability | Official DSH | With this plugin |
| --- | --- | --- |
| Archive sessions | Supported | Supported, plus one-click archive-all |
| Unarchive | No entry point | Single / per-workspace / per-month, in bulk |
| Truly delete chat records | Hidden only, never deleted | Deletes the on-disk log files; running sessions are skipped |
| Export session data | One at a time, manual | One-click bulk export, byte-compatible with the official format |

## Features

### Archived chats management

Open the manage dialog and switch between two grouping views:

- **By workspace**: one group per workspace, rows sorted by time;
- **By month**: grouped by year-month (`August 2026`, `July 2026`...), so archives across years are immediately clear.

Sort by created / updated date (ascending / descending), collapse or expand groups at will, and run batch actions per group — unarchive the group, or **delete the archived chats of the group** (only the group's archived sessions are affected; unarchived records are never touched).

| Grouping & sorting | Group-level batch actions |
| --- | --- |
| ![Manage dialog](docs/screenshots/manage.png) | ![Manage dialog](docs/screenshots/manage.png) |

### Archive all chats

Archive every session at once: records stay fully intact, merely hidden from the sidebar list — unarchive them any time from the manage dialog.

### Delete all chats

**Truly delete** local chat records: removes the session log files on disk (`session.jsonl` / `session.jsonl.zstd` and its twin), and cleans up workspace accounting and archive marks. Running sessions are skipped automatically to prevent logs from being rewritten.

### Export data

One-click bulk export of every session: it walks all root sessions (subagent sessions included) and produces an official-format ZIP archive for each (`dsh-session-<id>.zip`) — no more exporting them one by one.

- **Complete contents**: session log, subagent sessions and media attachments in one package;
- **Format compatible**: byte-compatible with the official export format, ready for the official tooling to inspect or migrate;
- **Backup before deletion**: run "Export data" before "Delete all chats" to keep what matters while cleaning up the rest.

### Bilingual UI

Copy switches instantly with Settings > General > Language; documentation and UI are maintained in both languages.

## Installation

DSH plugins mount through a **profile** (`dsh web` uses the `web` profile). **Restart `dsh web`** after installing.

**Prerequisites**: Node.js (with npm); `dsh plugin` relies on `pnpm` (see FAQ if missing).

### Option 1: from npm (recommended)

The plugin is published to npm as `dsh-session-management`. The command depends on how you run dsh:

- **Global dsh install** (`dsh` is on your PATH):

  ```sh
  dsh plugin --profile web add dsh-session-management
  ```

- **Running dsh via npx** (not installed globally; you start it with `npx @deepseek-ai/dsh web`):

  ```sh
  npx -y @deepseek-ai/dsh plugin --profile web add dsh-session-management
  ```

After installing, restart `dsh web` and open Settings to find "Session Manager". To upgrade, replace `add` with `update` (keep the `npx -y @deepseek-ai/dsh` prefix when applicable).

> **Version note**: if npm/pnpm metadata caching or a mirror registry causes an older version to be installed (warning `declares no dsh.bundle`), pin the latest version and re-install:
>
> ```sh
> npx -y @deepseek-ai/dsh plugin --profile web add dsh-session-management@<latest-version>
> ```
>
> Check the latest version: `npm view dsh-session-management version`.

### Option 2: from GitHub

```sh
git clone https://github.com/cokiscarazo-rgb/dsh-session-management.git
cd dsh-session-management

# Windows (PowerShell)
powershell -ExecutionPolicy Bypass -File scripts/install.ps1

# macOS / Linux
bash scripts/install.sh
```

The installer is idempotent and safe to re-run. It does two things:

1. Copies the package into `$DSH_HOME/profiles/node_modules/dsh-session-management/`;
2. Registers the loader entry in `$DSH_HOME/profiles/web/cordis.patch.yml`:

```yaml
- insert:
    - id: dsh-session-management
      name: dsh-session-management
```

### Verification & uninstall

After installing, restart `dsh web` — the "Session Manager" entry appears in Settings. You can also confirm the plugin layer with `dsh --profile web --dump-config` (prepend `npx -y @deepseek-ai/dsh` when dsh is not global). If no entry shows up, the usual culprit is a missing restart.

To uninstall, run `dsh plugin --profile web remove dsh-session-management` (prepend `npx -y @deepseek-ai/dsh` when applicable) and restart `dsh web`; for manual installs, remove `$DSH_HOME/profiles/node_modules/dsh-session-management/` and delete the insert entry from `cordis.patch.yml`.

### FAQ

- **`'pnpm' is not recognized` / `pnpm: command not found`**: `dsh plugin` relies on pnpm internally. Install it first:

  ```sh
  npm install -g pnpm
  ```

  or enable it via Node's built-in corepack: `corepack enable pnpm`. Verify with `pnpm --version`.

- **An older version is installed / warning `declares no dsh.bundle`**: npm/pnpm metadata caching or a mirror registry delay. Check your registry with `pnpm config get registry` (if it is a mirror like npmmirror, switch back with `pnpm config set registry https://registry.npmjs.org/`), then re-install with the exact version (see the version note above).

- **`ERR_PNPM_IGNORED_BUILDS` on first install**: pnpm refuses build scripts; add the reported packages to `allowBuilds` in the profile's `pnpm-workspace.yaml` and re-run.

- **No "Session Manager" entry in Settings**: make sure you restarted `dsh web`, and that `$DSH_HOME/profiles/web/cordis.patch.yml` contains an insert entry with `id: dsh-session-management`.

## How it works & limitations

- **Archive**: built on the official `workspaceRegistry.archiveSession`; the archive set is persisted in the workspace domain and synced to clients via the official frame mechanism. **Unarchive** is not provided by the official API, so the plugin updates the workspace domain archive set directly; the change is broadcast through the official `domain/changed` event.
- **Delete means delete**: the session log files (including the zstd twin) are located and removed via the system command, then workspace accounting and archive marks are cleaned up; the search index is reconciled automatically by the official SQLite backend.
- **Limitations**:
  - Running sessions are refused for deletion (to avoid logs being rewritten);
  - Image attachments use content-addressed storage and may be shared across sessions, so they are not removed with a session;
  - Subagent sessions are independent records; deleting a parent session does not cascade (they can be deleted individually).

## License

[MIT](LICENSE) © cokiscarazo-rgb
