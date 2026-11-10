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
