# claude-renamed

A tiny Bash CLI to **list, resume, rename, unrename, and archive your Claude Code sessions** — from one screen.

Claude Code lets you give a session a sticky custom title (via `/name` or the rename UI), but there's no first-class way to browse those named sessions across all your projects, jump back into one, or change/clear a name after the fact. `claude-renamed` does exactly that.

```
  Claude Session Manager (3 active, 1 archived)
  ──────────────────────────────────────────────────

  ── Archived ──────────────────────────────────────
   4  old-spike                          /home/me/code/api                 2026-04-02 11:10

  ── Active ────────────────────────────────────────
   3  vacation-trip-planner              /home/me/personal                 2026-05-19 20:33
   2  refactor-auth                      /home/me/code/api                 2026-05-20 15:21
   1  lovereport                         /home/me/code/lovereport          2026-05-20 15:22

  Commands:  <number>        resume session
             d <number>      unrename (remove name, keep session)
             r <number>      rename to new name
             a <number>      archive (sort to bottom)
             ua <number>     unarchive
             q               quit
```

## Why

If you use Claude Code seriously across many projects, you accumulate sessions fast. The built-in `claude --resume` picker is per-directory and unsorted. Giving important sessions a name and then having one global, sorted, navigable list is a small thing — but it changes the way you work.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/TuringTzu/claude-renamed/main/claude-renamed \
  -o ~/.local/bin/claude-renamed
chmod +x ~/.local/bin/claude-renamed
```

Make sure `~/.local/bin` is on your `PATH`. Then just run:

```bash
claude-renamed
```

Requires: `bash`, `python3` (system Python is fine; no third-party libraries), and the official `claude` CLI on your `PATH`.

## How it works

Claude Code stores each session's transcript as JSONL under `~/.claude/projects/<encoded-cwd>/<session-id>.jsonl`. When you rename a session, Claude appends a single line like:

```json
{"type":"custom-title","customTitle":"refactor-auth","timestamp":...}
```

`claude-renamed` scans every JSONL, keeps the sessions that have a `custom-title` entry, and presents them oldest-first so the most-recently-active session sits at the bottom, right next to the prompt — numbered from the bottom up, so the newest session is always **#1**. Resume re-execs `claude --resume <session-id>` in that session's original working directory. Rename and unrename atomically rewrite the JSONL.

Nothing here is "official" — it just reads the on-disk format that Claude Code already writes. Should the format ever change, this script will need a small patch.

## Archiving

`a <number>` archives a session and `ua <number>` un-archives it. Archived sessions aren't deleted — they're moved into a separate **Archived** section at the top of the list (dimmed), out of the way of your active sessions, which stay at the bottom near the prompt. Everything still works on them by number (resume, rename, unrename).

Unlike rename, archive state is **not** written into the session JSONL. It's kept in a small separate file, `~/.config/claude-renamed/archived.json`:

```json
{ "archived": ["<session-id>", "<session-id>"] }
```

This keeps Claude Code's own session files untouched. Override the location with `CLAUDE_RENAMED_ARCHIVE=...`.

## Optional: path remapping

If you've moved your code between machines (e.g. an old Linux box where projects lived at `/srv/code/` and a new Mac where they live at `/Volumes/ssd/srv/code/`), Claude Code remembers the original `cwd`. Resume would `cd` into a path that no longer exists.

Create `~/.config/claude-renamed/paths.json`:

```json
{
  "prefix_maps": [
    {"from": "/srv/", "to": "/Volumes/ssd/srv/"},
    {"from": "/home/oldname/", "to": "/Users/me/"}
  ],
  "exact_maps": {
    "/old/exact/project/path": "/new/exact/project/path"
  }
}
```

`prefix_maps` rewrites any session `cwd` that starts with `from` to start with `to` instead. `exact_maps` handles one-off moves (a renamed project directory). Without this file, no remapping is done.

You can also point at a different config file via `CLAUDE_RENAMED_CONFIG=...`, or a different projects dir via `CLAUDE_PROJECTS_DIR=...`.

## Environment variables

| Var | Default | What |
|-----|---------|------|
| `CLAUDE_PROJECTS_DIR` | `~/.claude/projects` | Where to scan for session JSONL files. |
| `CLAUDE_RENAMED_CONFIG` | `~/.config/claude-renamed/paths.json` | Path-remapping config (optional). |
| `CLAUDE_RENAMED_ARCHIVE` | `~/.config/claude-renamed/archived.json` | Where archived session ids are stored. |

## Caveats

- Reads the unofficial on-disk format of Claude Code session files. If Anthropic changes the format, this script will need updating.
- "Rename" works by appending/updating a `custom-title` line in the JSONL transcript. The change is local-only; it doesn't sync anywhere.
- The script `exec`s into `claude` on resume, replacing itself. That's intentional — you end up in a normal Claude Code session, not nested inside the manager.

## License

MIT. See [LICENSE](./LICENSE).

## Author

Built by [@turingtzu](https://x.com/turingtzu). PRs welcome.
