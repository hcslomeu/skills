# skills

Plugin catalogue for the agent skills I publish. Each skill lives in its own
repository; this repo holds only the manifest that lists them.

## Use

From inside Claude Code:

```
/plugin marketplace add hcslomeu/skills
/plugin install session-handover@hcslomeu
```

Update later with `/plugin marketplace update`.

## Available

| Plugin | What it does |
| ------ | ------------ |
| [session-handover](https://github.com/hcslomeu/session-handover) | Resume prompt for a cleared session that records which AI-reviewer comments you accepted, disputed or left pending |

## Why the catalogue is its own repo

A marketplace name is registered once per user: adding a second marketplace
called `hcslomeu` would replace the first rather than extend it, so every
plugin published under `@hcslomeu` has to be listed in this one file. Keeping
that file here, instead of inside one of the skill repositories, means the
catalogue outlives any individual skill and the install address stays the same
as skills are added, renamed or retired.

## Adding a skill

1. Create a repository for the skill with `.claude-plugin/plugin.json` and a
   `SKILL.md` at the root. A root `SKILL.md` with no `skills/` directory is
   loaded as a single-skill plugin; set `name` in its frontmatter so the
   invocation name stays stable across updates.
2. Add an entry to `plugins` in `.claude-plugin/marketplace.json` here, with a
   `github` source pointing at that repository.
3. Run `claude plugin validate .` before pushing.

## Licence

MIT. See [LICENSE](LICENSE).
