# skills

Agent skills I publish. Every skill lives in this repository under `skills/<name>/`, and the whole set installs as one plugin.

## Install

Claude Code, as a plugin that updates itself:

```
/plugin marketplace add hcslomeu/skills
/plugin install hcslomeu-skills@hcslomeu
```

Any agent supported by [skills](https://github.com/vercel-labs/skills), as editable copies:

```
npx skills add hcslomeu/skills
```

Pick one per agent: installing both gives you every skill twice.

## Available

| Skill | What it does |
| ----- | ------------ |
| [handover](skills/session-handover) | Resume prompt for a cleared session that records which AI-reviewer comments you accepted, disputed or left pending, the decisions you made that no file holds yet, and what to ask you before acting |

## Adding a skill

1. Put it in `skills/<name>/SKILL.md`, with `name` and `description` in the frontmatter. Without `name`, the invocation name falls back to the install directory, which changes between installers.
2. Add `./skills/<name>` to `skills` in `.claude-plugin/plugin.json`, add a row to the table above, and bump `version` in that file.
3. Check the skill names nothing personal: no local paths, no private repositories, nothing that only exists on my machine.
4. Run `claude plugin validate .` before pushing.

## Licence

MIT. See [LICENSE](LICENSE).
