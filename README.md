# work-mgt — Claude Code plugin for executive-assistant work management

Reusable skills and slash commands for a Claude-based executive assistant. The
plugin doesn't bundle any MCP servers itself — it operates against MCP servers
that the user configures separately and exposes high-level workflows on top of
them.

## Status

`v0.1` — first feature: **daily Trello summary** of the `Doing` and `Done`
lists on a configured board.

## Required MCP servers

| Server      | Repo                                                | Required for           | Slug expected in `.mcp.json` |
|-------------|-----------------------------------------------------|------------------------|------------------------------|
| Trello-MCP  | https://github.com/RPAzevedo/Trello-MCP             | daily Trello summary   | `trello`                     |

Verify with `claude mcp list` — the tool names should appear as
`mcp__trello__list_boards`, `mcp__trello__list_lists`, and
`mcp__trello__get_cards`.

## Install

The repo ships a `.claude-plugin/marketplace.json`, so the GitHub repo *is*
the marketplace. Install from inside Claude Code:

```
/plugin marketplace add RPAzevedo/WorkMgt-Plugin
/plugin install work-mgt@work-mgt
```

The first command registers this repo as a marketplace named `work-mgt`;
the second installs the `work-mgt` plugin from it (the
`<plugin>@<marketplace>` syntax — both happen to be `work-mgt` here).

Verify with:

```
/plugin list
```

You should see `work-mgt@work-mgt` and `/daily-summary` should appear in
the slash-command list.

### Updating

```
/plugin marketplace update work-mgt
/plugin update work-mgt@work-mgt
```

### Uninstalling

```
/plugin uninstall work-mgt@work-mgt
/plugin marketplace remove work-mgt
```

### Local development install

If you're hacking on the plugin itself, point Claude Code at your checkout
as a directory-source marketplace instead of the GitHub one:

```
/plugin marketplace add /path/to/WorkMgt-Plugin
/plugin install work-mgt@work-mgt
```

Edits to skill/command files are picked up on the next Claude Code session
(or after `/plugin marketplace update work-mgt`).

## Configure

Create `~/.claude/work-mgt/config.json`. Use `config.example.json` in this repo
as a template.

```json
{
  "trello": {
    "default_board": "Personal Ops",
    "lists": {
      "doing": ["Doing", "In Progress", "WIP"],
      "done":  ["Done", "Shipped", "Completed"]
    },
    "stale_days": 5,
    "done_head": 10
  },
  "state_dir": "~/.claude/work-mgt/state"
}
```

- `default_board` — the Trello board to summarize when no argument is given.
- `lists.doing` / `lists.done` — aliases the resolver will match against actual
  list names on the board (case-insensitive, first match wins).
- `stale_days` — Doing cards untouched for at least this many days are flagged
  for attention.
- `done_head` — number of recently-completed cards to show for context.
- `state_dir` — where the plugin caches resolved board/list IDs and the
  last-run timestamp. Created on first run.

## Usage

```
/daily-summary
/daily-summary "Other Board Name"
```

Output is a markdown report covering:

1. **Completed since last summary** — Done cards whose `date_last_activity` is
   after the last run.
2. **Started or active since last summary** — Doing cards with activity since
   the last run.
3. **In flight (full Doing list)** — every card in Doing, oldest activity
   first; cards untouched longer than `stale_days` are flagged.
4. **Recently completed (full Done list, head N)** — context for run-rate.

On first run there's no last-run timestamp, so the plugin uses "24h ago" and
prints a one-line note.

## Reusing skills programmatically

The plugin ships three skills:

- `trello-board-resolver` — name → board/list IDs (with cache). Internal.
- `trello-list-snapshot` — `get_cards(list_id, since?)` plus structured +
  markdown output. Internal.
- `daily-trello-summary` — the v1 user-facing feature.

Other plugins or workflows can compose against the two internal skills to
build their own Trello reports without re-implementing the discovery or
formatting layers.

## Roadmap

- Calendar review (Google Calendar MCP).
- Email triage and follow-ups (Gmail MCP).
- Cross-source prioritization.
- Subagent parallelism for multi-board / multi-source briefings.
