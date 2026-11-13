---
name: close
description: Ask another Claude session in a zellij pane whether it is safe to close, and close it and its pane once it says yes. Use when the user types /zellij:close with a session name or @mention, or asks to close a finished session after checking with it.
argument-hint: "<session name or @mention>"
allowed-tools:
  - Bash(zellij action list-panes:*)
  - Bash(zellij action dump-screen:*)
---

# close

Close the Claude session named in `$ARGUMENTS`, but only after it agrees.
The session may be given as a bare name, `@name`, or `name [ref]`.

The `zellij` skill in this plugin has the commands for finding, reading
and closing a pane. Follow its "Close a Claude session and its pane"
section for the mechanics. This skill adds the order and the check.

## Steps

1. Find the pane. Match the session name against `list-panes --json`
   (the pane title or the `-n <name>` in `pane_command`). If none or
   more than one match, stop and tell the user.
2. Read the pane with `dump-screen --ansi`, not a plain `dump-screen`.
   Text after `❯` may be a grey suggestion rather than a draft, and a
   plain dump drops the colour that tells them apart. Text that starts
   with `ESC[2m` (faint) is a suggestion: `/exit` replaces it, so go
   ahead. Text without it is the user's draft: do not type over it;
   report it and stop. See "Tell a suggestion from a draft" in the
   `zellij` skill.
3. Ask the session. Send it a message (SendMessage, to the name exactly
   as `ListAgents` prints it):

   > Is it safe to close this session? Reply "safe to close" in one line
   > if no background job or cleanup is left; otherwise name what is
   > left in one line.

   Wait for the reply. Do not poll; the reply arrives as a message.
4. If it names something left, do not close. Tell the user what is left
   and stop.
5. If it says yes, exit and close as the `zellij` skill describes:
   `/exit`, Enter, wait for "Resume this session with:", then
   `close-pane`.
6. Report to the user in one or two lines: that it is closed, the reason
   it was safe, and the resume command from the screen.

If the session does not answer (it exited, or it holds the message for
its user's approval), tell the user and leave the pane open.
