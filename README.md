# claude-zellij

A Claude Code plugin for a coordinator session that runs other Claude Code
sessions in zellij panes. It teaches Claude to:

- open a pane running `claude --remote-control <name>`,
- type a slash command such as `/rename` into another Claude pane and
  submit it,
- read what another pane shows.

It uses only the built-in `zellij action` CLI, so there is no zellij plugin
to load. Needs zellij 0.44 or later; tested on 0.45.1.

## Install

```text
/plugin marketplace add chaotic-ground/claude-zellij
/plugin install zellij@claude-zellij
```

## What it will not do

- Type into its own pane.
- Answer a trust, permission or login prompt in another pane. It reports
  the prompt and stops.

Both are in [`skills/zellij/SKILL.md`](skills/zellij/SKILL.md) with the
rest of the procedure.

## Repository settings

`.tf/` holds this repository's GitHub settings as OpenTofu, planned by
[repo-settings-as-code](https://github.com/chaotic-ground/repo-settings-as-code).

## License

GPL-3.0-or-later. See [`LICENSE`](LICENSE).
