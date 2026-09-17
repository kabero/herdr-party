# herdr-party 🎉

English | [日本語](README.ja.md)

A [Herdr](https://herdr.dev) plugin that draws the Claude Code sessions running in Herdr as a party in a side pane.
Every session is a stick figure and every space (workspace) is a stage.
You can tell at a glance who is working, who is waiting for your approval, and who has finished and is calling for you.

```
       2 dancing   1 waiting   1 done   2 chilling      00:09:29
                       Matsu 13m   Shizu
┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
   Sora [12]   Aoto [3]  Matsu [7]  ┆ Subaru [1]
     (54s)                (13m)      ┆    (2h)
      \o/         o        !o!       ┆    \o/
       |         /|\        |        ┆     |
      / \        / \       / \       ┆    / \
  ▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄
            master ✱3                 feat/x ✱1
                       herdr-party
```

## Install

Register it as a Herdr plugin. To link a local checkout:

```bash
git clone https://github.com/kabero/herdr-party ~/ghq/github.com/kabero/herdr-party
herdr plugin link ~/ghq/github.com/kabero/herdr-party
herdr plugin action list --plugin kabe.herdr-party
```

To install straight from GitHub: `herdr plugin install kabero/herdr-party`.

To bind it to a prefix key, add this to `~/.config/herdr/config.toml` and run `herdr server reload-config`:

```toml
[[keys.command]]
key = "prefix+space"
type = "plugin_action"
command = "kabe.herdr-party.toggle"
description = "party pane"
```

What the plugin provides:

| Kind | id | What it does |
| --- | --- | --- |
| action | `kabe.herdr-party.toggle` | Opens the party to the right of the current tab; closes it if it is already open |
| action | `kabe.herdr-party.open` / `close` | Open only / close only |
| pane | `party` | Opens as a Herdr-managed pane (`herdr plugin pane open --plugin kabe.herdr-party --entrypoint party --placement split`) |
| pane | `peek` | Peeks at the party in a popup (`--entrypoint peek`, close with `q`) |

## Usage

You can also run it directly inside a Herdr pane (`HERDR_ENV=1`) without the plugin:

```bash
./bin/herdr-party            # open a 30%-wide pane on the right; close it if already open (toggle)
./bin/herdr-party --open     # open only
./bin/herdr-party --close    # close only
./bin/herdr-party --width 0.4
./bin/herdr-party --focus    # move focus to the new pane
./bin/herdr-party -- --all   # invite agents other than claude too
./bin/herdr-party -- --list  # plain list instead of stick figures
```

Dependencies: `herdr`, `jq` (launcher), `python3` (viewer, standard library only)

## How to read it

### Stick figures (sessions)

| Figure | Mark | State |
| --- | --- | --- |
| `\o/` dancing (rainbow) | ◐ | working |
| ` o ` standing | ● | idle |
| `!o!` panicking | ◆ | blocked (waiting for an approval or an answer) |
| `\o ` waving | ✔ | done (finished in the background, not yet seen) |
| ` ? ` | ? | unknown |

- Every guest gets a romanized name derived from its session ID, so the same session keeps the same name across restarts.
- The number of instructions you have given the session appears next to the name, like `Sota [19]`. It is counted from the Claude Code conversation log (`~/.claude/projects/*/<session-id>.jsonl`, user messages excluding tool results), read incrementally, so it is exact even across viewer restarts. Sessions without a log show no number.
- Guests that are working, blocked, or done show how long they have been in that state under their name, like `(54s)`. The time of each state change is saved to `~/.local/state/herdr-party/state-since.json` (or `HERDR_PLUGIN_STATE_DIR` when opened as a plugin pane), so reopening the viewer keeps the elapsed time as long as the state has not changed. A guest whose state change has never been observed shows no time.
- The stage under the session you are currently in lights up yellow (the spotlight). The name of the focused space is shown in bold.

### Stages (spaces)

- A stage is 80% of the venue width by default, and performers stand centered on it. It is just one row of platform with the space name underneath.
- When a single row does not fit, the stage widens to the full venue, and anyone left over stands in a back row (above).
- Stages are per git repository. Worktrees of the same repository (separate spaces in Herdr) stand on one stage, divided into sections by `┆`. Each section shows its branch and change count underneath (`master ✱3 ↑1`: uncommitted changes, and ahead/behind of the remote), and the stage is named after the repository. Clicking a section jumps to that worktree's space.
- If the sections do not fit side by side, each becomes its own stage with the same repository name. Spaces outside git use the space label as the stage name.
- Each space gets its own stage color.

### What each dancer is doing

Under the stage name, every working guest gets a line like `Sota ▸ Relaunch the pane and commit`. The text is the description Claude Code writes for each tool call (Bash descriptions, or the tool name plus file for edits and searches), taken from the conversation log as it grows, so it changes in real time as the work proceeds and clears when you give a new instruction.

### Lobby

Guests that have been idle for a while (30 minutes by default; `--lobby-after MIN`, 0 disables) leave the stages and gather in a lobby at the bottom of the venue, drawn in a dim color with their idle time under the name, longest first. This keeps the stages to the sessions that matter right now. Idle time comes from the state change the viewer observed, or, for sessions that were already idle when the viewer started, from the last entry in the Claude Code conversation log. Lobby guests can still be hovered, selected, and clicked.

### Header

The first line shows the count per state and a clock. The word "dancing" flows in rainbow colors while somebody is working. When there are blocked or done guests, a second line lists their names and waiting times, blocked first, longest wait first.

## Controls

| Action | Effect |
| --- | --- |
| Hover over a figure | A bubble appears: line 1 is state, name, elapsed time, and time since you last looked; line 2 is the last thing you asked (`❯`); line 3 is the start of the last reply (`⏺`) or the pending approval (`?`) |
| Left-click a figure | Moves Herdr's focus to that session's pane (`herdr agent focus`) |
| Left-click a stage | Jumps to that space (`herdr workspace focus`) |
| `←` `→` (also `h` `l` `j` `k` `↑` `↓`) | Selects a guest. The selected guest's name turns pink and its bubble appears |
| `Tab` | Jumps through blocked and done guests |
| `Enter` / `o` | Opens the selected guest's session. With nothing selected, jumps to the guest who needs you most (blocked first, longest wait first). That guest gets a bobbing `▼` above the head and a bold name in its state color, and is marked `⏎` in the header |
| `Esc` | Clears the selection |
| `?` | Shows / hides the help line |
| `q` | Quits (and closes the pane) |

Toggling with the plugin action closes the party wherever it currently is, even if it has followed you to another space.

## Following you around

A pane belongs to one tab, so by itself the party would disappear when you switch tabs or spaces. By default the viewer follows you instead: when it sees a `tab.focused` event for another tab, it moves its own pane there with `pane move`, as a right split of the same width (`--width`, 30% by default). The pane only moves between tabs, so its content, elapsed times, and selection are all kept. Clicking on the party pane itself never moves it. Pass `--no-follow` to stay put.

```bash
./bin/herdr-party -- --no-follow   # stay in the tab it was opened in
```

## Keeping it in the sidebar

If you prefer the pane to stay put, Herdr's sidebar is another way to see the party everywhere.
Run the viewer with `--report`, or run the headless `--report-only`, and each session's pane gets its name and pose reported as metadata.

```bash
./bin/herdr-party -- --report          # open the party pane and also report to the sidebar
./bin/herdr-party-agents --report-only  # report only, no screen (for the background)
```

Add `$party_pose` and `$party_name` to the agent rows in `~/.config/herdr/config.toml`, and every agent in the sidebar gets a stick figure and a name.

```toml
[ui.sidebar.agents]
rows = [["state_icon", "machine", "workspace", "tab"], ["$party_pose", "$party_name", "agent"]]
```

Reports expire after 8 seconds, so stopping the viewer restores the sidebar.

## How it works

- The bubble's "asked", "reply", and "pending approval" lines are pulled from the pane text via `herdr agent read` whenever a session's state changes (the `❯` line, reply lines starting with `⏺`, and the first lines of an approval dialog box). "Since you last looked" is counted from `pane.focused` events.
- The viewer subscribes to `pane.focused`, `pane.agent_status_changed`, `pane.created`, and similar events through Herdr's socket API and reacts the moment they arrive. Focus changes move the spotlight straight from the event's pane_id; everything else triggers a fresh `herdr agent list` and `herdr workspace list`. A 2-second poll remains as a fallback, and the screen redraws every 0.5 seconds.
- It enables xterm mouse motion tracking (mode 1003), so hover and clicks reach the pane even with Herdr's `mouse_capture` enabled.
- Herdr clients before 0.9.1 could pull you back to the clicked pane's space right after an API workspace switch (fixed in 0.9.1, changelog #3760 #4153 #4171). On 0.9.0 and earlier the viewer waits 1.2 seconds after the button release before switching, then watches for 2 seconds and switches again if it was pulled back. On 0.9.1 and later the wait is 0.2 seconds. Run `herdr update` to get 0.9.1 or later if clicks should reliably jump.
- Set `HERDR_PARTY_LOG` to a file path to log click and focus handling to that file.
