# Skills for Political Science Research

Agent skills I use for research work — mostly conventions I'd otherwise have to
re-explain at the start of every session. Each skill is a `SKILL.md` file that
tells agentic AI how I want a particular kind of work done.

**Work in progress.** More skills will be added over time.

## Skills

| Skill | What it covers |
| --- | --- |
| [`r-style`](skills/r-style/SKILL.md) | My R style for research data pipelines — pipes, script layout, naming, comments, I/O, and what not to do. |

## Using these

Copy a skill directory into `~/.claude/skills/` (available everywhere) or into
`.claude/skills/` inside a project (that project only):

```sh
git clone https://github.com/jcha1997/jcha_skills.git
cp -r jcha_skills/skills/r-style ~/.claude/skills/
```

Claude Code picks the skill up on the next session and invokes it when the
frontmatter `description` matches what you're doing.

These are my preferences, not general best practice. Fork and edit.

## License

[MIT](LICENSE).
