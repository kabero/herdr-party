# herdr-party 🎉

English | [日本語](README.ja.md)

A [Herdr](https://herdr.dev) plugin that draws the Claude Code sessions running in Herdr as a party in a side pane.
Every session is a stick figure and every space (workspace) is a stage.
You can tell at a glance who is working, who is waiting for your approval, and who has finished and is calling for you.

```
       2 dancing   1 waiting   1 done   1 chilling      00:09:29
                    ⏎ Matsu 13m   Shizu 1m
┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
  herdr-party  master ✱3
  ──────────────────────────────────────────────────────────────
  \o/  Sora [12]  (54s)
   |   > make the stage vertical
  / \
  ▄▄▄
  ┈┈ feat/x ✱1 ┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈
  !o!  ⏎ Matsu [7]  (13m)
   |   > fix the failing test in the worktree
  / \  ? Bash command: rm -rf build
  ▄▄▄
  chezmoi  master
  ──────────────────────────────────────────────────────────────
  \o   Shizu [4]  (1m)
   |\  > tidy up the herdr config
  / \  < Done: moved the tab bar to the bottom and reloaded.
  ▄▄▄
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

### Groups (repositories and spaces)

- Guests are grouped by git repository, with the repository name and its branch and change count (`master ✱3 ↑1`: uncommitted changes, ahead/behind of the remote) as a heading. Worktrees of the same repository (separate spaces in Herdr) sit under the same heading, each in its own section marked by a `┈┈ feat/x ✱1 ┈┈` rule. Spaces outside git use the space label as the heading. Clicking a heading or a section rule jumps to that space.
- Each guest is one card: the stick figure on the left standing on a small platform, and on the right the name, prompt count, elapsed time, and the text lines described below. The platform under the session you are currently in lights up yellow (the spotlight). Headings of the focused space are bold.

### What each card says

- `> …` is the last prompt you gave that session, wrapped to two lines, read from the conversation log as it grows, so it updates the moment you send a new instruction. Slash commands show as their name and arguments.
- `< …` appears on done guests: the start of the reply that finished, from the same log.
- `? …` appears on blocked guests: what the approval dialog or question is asking, read from the pane text.
- `見てから 12m` after the name is how long since you last looked at that session.

### Lobby

Guests that have been idle for a while (30 minutes by default; `--lobby-after MIN`, 0 disables) leave their groups and gather in a lobby at the bottom of the venue, drawn in a dim color with their idle time next to the name, longest first. This keeps the groups to the sessions that matter right now. Idle time comes from the state change the viewer observed, or, for sessions that were already idle when the viewer started, from the last entry in the Claude Code conversation log. Lobby guests can still be hovered, selected, and clicked.

### Header

The first line shows the count per state and a clock. The word "dancing" flows in rainbow colors while somebody is working. When there are blocked or done guests, a second line lists their names and waiting times, blocked first, longest wait first.

## Controls

| Action | Effect |
| --- | --- |
| Hover over a card | Highlights it (the name turns pink) |
| Left-click a card | Moves Herdr's focus to that session's pane (`herdr agent focus`) |
| Left-click a heading or section rule | Jumps to that space (`herdr workspace focus`) |
| `↑` `↓` (also `j` `k` `←` `→` `h` `l`) | Selects a guest. The selected guest's name turns pink |
| `Tab` | Jumps through blocked and done guests |
| `Enter` / `o` | Opens the selected guest's session. With nothing selected, jumps to the guest who needs you most (blocked first, longest wait first). That guest gets a `⏎` before the name, shown bold in its state color, and is marked `⏎` in the header too |
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

- Prompts, replies, and prompt counts come from the Claude Code conversation log (`~/.claude/projects/*/<session-id>.jsonl`), read incrementally. The pending-approval line is pulled from the pane text via `herdr agent read` when a session becomes blocked. "Since you last looked" is counted from `pane.focused` events.
- The viewer subscribes to `pane.focused`, `pane.agent_status_changed`, `pane.created`, and similar events through Herdr's socket API and reacts the moment they arrive. Focus changes move the spotlight straight from the event's pane_id; everything else triggers a fresh `herdr agent list` and `herdr workspace list`. A 2-second poll remains as a fallback, and the screen redraws every 0.5 seconds.
- It enables xterm mouse motion tracking (mode 1003), so hover and clicks reach the pane even with Herdr's `mouse_capture` enabled.
- Herdr clients before 0.9.1 could pull you back to the clicked pane's space right after an API workspace switch (fixed in 0.9.1, changelog #3760 #4153 #4171). On 0.9.0 and earlier the viewer waits 1.2 seconds after the button release before switching, then watches for 2 seconds and switches again if it was pulled back. On 0.9.1 and later the wait is 0.2 seconds. Run `herdr update` to get 0.9.1 or later if clicks should reliably jump.
- Set `HERDR_PARTY_LOG` to a file path to log click and focus handling to that file.
