# skillz

Brandon's collection of general-purpose Claude Code skills, packaged as one plugin.

## Install

```
/plugin marketplace add ~/code/promptctl_skillz
/plugin install skillz@promptctl-skillz
```

For a one-off session without installing: `claude --plugin-dir ~/code/promptctl_skillz`.

## Layout

Each skill is `skills/<name>/SKILL.md`; the plugin picks up new skill directories automatically.

| Skill | Purpose |
|---|---|
| `word-police` | Realign a project's bespoke vocabulary with the standard terms of its fields; renames only what you accept. |
| `clean-room` | Specify an existing application from the outside, precisely enough for an independent team to rebuild it without seeing it. |
| `spec` | Write and maintain the PRD, FSD and technical spec for a feature being built, tied by a traceability matrix. |
| `capability-backlog` | Break a request or founding document into a capability-first story map and tickets. |
| `delegate-some-shit` | Hand one work item to a worker subagent that drives it to an open PR, then validate the result. |
| `find-reviews` | List the code reviews of recent work, PR by PR. |
| `find-session` | Find past Claude Code sessions by topic and read context around a match. |
| `hire-a-minion` | Spawn a worktree-backed Claude minion in a new window of your own tmux session and run it as its controller. |
| `lets-take-a-step-back` | Stop a bogged-down ticket, return to where it started, and restart from the decision that caused the trouble. |
| `repo-session` | Start a Claude instance in another repo over tmux and delegate work to it. |
| `something-just-came-up` | Park the current ticket and file the real fix as top-ranked work. |
| `tmux-command` | Send slash commands and keystrokes into another agent's tmux pane. |
| `tmux-talk` | Hold a conversation with another Claude instance through tmux. |
| `validate-ticket` | Make a ticket ready for implementation: a clear goal, outcome-level acceptance criteria, and parts that agree, fixed in place. |

`tmux-shared/` is not a skill: it holds the tmux addressing scripts `tmux-talk` and `tmux-command` share.
