---
name: zellij
description: Drive other Claude Code sessions in zellij panes from a coordinator session. Use when opening a pane that runs `claude --remote-control`, typing a slash command such as /rename or /remote-control into another Claude pane, or reading what another pane shows. Needs zellij 0.44 or later and uses only the built-in `zellij action` CLI.
allowed-tools:
  - Bash(zellij action list-panes:*)
  - Bash(zellij action dump-screen:*)
---

# zellij

You are a coordinator running inside a zellij pane. Other Claude Code
sessions run in sibling panes. This skill covers three things you do to
them: open one, type a slash command into one, and read one.

Everything here is the built-in `zellij action` CLI (tested on 0.45.1). No
plugin is loaded. Every action that touches a pane takes `--pane-id`, so
nothing depends on which pane has focus.

## Rules

- **Never type into your own pane.** Your pane id is `$ZELLIJ_PANE_ID`.
  Writing there injects text into your own prompt box. Check the target id
  against it before every `write-chars` or `write`.
- **Never answer a prompt for the user.** That covers the folder trust
  prompt, permission prompts, and login. If a pane stops on one, report it
  and leave it.
- **Find panes by listing, not by memory.** Ids are stable for the life of
  a pane, but a pane you remember may have closed. List again before you
  write.
- **A zellij pane name is not a Claude session name.**
  `zellij action rename-pane` changes only the title in the zellij frame.
  The Claude session keeps its name; change that with `/rename` inside it.

## Find panes

```bash
zellij action list-panes --json --command
```

Each terminal pane has `id`, `title`, `pane_command`, `pane_cwd` and
`exited`. A Claude pane started by this skill has a `pane_command` that
begins with `claude --remote-control <name>`. A Claude pane sets its own
title from the conversation, so match on `pane_command`, not `title`.

```bash
zellij action list-panes --json --command |
  jq -r '.[] | select(.is_plugin | not) | [.id, .title, .pane_cwd] | @tsv'
```

## Open a Claude session in a new pane

Start the session with a name and no prompt, then send the brief with
your messaging tool (SendMessage) addressed to that name:

```bash
zellij action new-pane --no-focus --cwd "$dir" --name "$name" -- \
  claude --remote-control "$name" -n "$name"
```

- Do not put the brief on the command line. The shell expands backticks
  and `$` in it, so code spans such as `` `$wgFoo = true` `` run as
  commands and vanish from the prompt, and the whole line stays visible
  to anyone who runs `list-panes --command`.
- Once the prompt box shows, send the brief with SendMessage to `$name`.
  The text arrives verbatim, and the same channel carries the follow-ups.
- It prints the new pane id as `terminal_<id>`. Keep it.
- `--remote-control <name>` makes the session reachable from claude.ai under
  that name. `-n <name>` sets the name shown in the prompt box and in
  `/resume`. Give both the same value.
- `--no-focus` leaves the user where they are.
- `$dir` must be a folder Claude Code already trusts. In a new folder the
  session stops on the trust prompt, and the rules above say you do not
  answer it. Ask the user to open the folder in Claude once, or pick a
  trusted parent.

Confirm it started, then brief it:

```bash
zellij action dump-screen --pane-id "$id"
```

You should see the Claude Code prompt box, idle. If you see "Do you trust
the files in this folder?", stop and tell the user. Otherwise send the
brief by SendMessage to `$name`; if the name is not listed yet, wait a
moment and list agents again.

## Type a slash command into another pane

Type, check, pause, submit:

```bash
zellij action write-chars --pane-id "$id" '/rename femiwiki-cpu'
zellij action dump-screen --pane-id "$id"
sleep 1
zellij action write --pane-id "$id" 13
```

- `write-chars` puts the text in the prompt box without submitting.
- `dump-screen` shows the box before you commit. If the text is not
  there, or something else is in the box (a half-typed message from the
  user, an open menu), do not press Enter. Report what you saw.
- The pause keeps the Enter out of the same paste as the text. Without it
  the two can arrive as one paste, and the Enter becomes a newline in the
  box instead of a submit.
- `write ... 13` sends a carriage return, which submits.

Then check the result:

```bash
zellij action dump-screen --pane-id "$id"
```

`/rename` and `/remote-control` take effect even while that session is in
the middle of a turn. Ordinary prompt text typed into a busy session is
queued until the turn ends, so to message another Claude session prefer
its messaging tool (SendMessage) over typing.

Keep each `write-chars` to one line. A newline inside the text may submit
early.

## Read a pane

```bash
zellij action dump-screen --pane-id "$id"          # what is on screen
zellij action dump-screen --pane-id "$id" --full   # with scrollback
```

Without `--path` it prints to stdout. Read the bottom of the dump: the
prompt box and the status line under it tell you whether the session is
idle, busy, or waiting on a prompt.

## Not covered

- Layout files and tabs. `zellij action new-tab --layout` and
  `override-layout` exist; this skill does not use them.
- Waiting until a pane is idle. Poll `dump-screen`, or ask the session to
  message you when it is done.
