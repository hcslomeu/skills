---
name: handover
description: Generate a structured session-resume prompt before /clear or starting a fresh session. Use when the user wants to carry forward project state, pending work, active task info, running background processes, open AI-reviewer threads (Claude, Codex, CodeRabbit, Gemini), decisions the owner made that are not written down yet, questions to ask the owner before acting, or session-specific knowledge into a new conversation. Triggers on phrases like "handover", "session handover", "resume prompt", "before I clear", "/handover", or any request to summarise current session state for continuation.
---

# Session Handover

Produce a self-contained prompt that the user can paste verbatim into a new
session to resume work without losing context.

## Configuration

Vocabulary, not requirements. Edit the values to match the project; the rest of
the skill refers to the names on the left. Every setting is a word, not a file,
so nothing here has to exist on disk before the skill works.

| Setting         | Default                     | Meaning                                                 |
| --------------- | --------------------------- | ------------------------------------------------------- |
| `TASK_UNIT`     | work package (WP)           | The project's unit of work, and how its IDs are written  |
| `TASK_GROUPING` | track                       | The grouping a task belongs to (workstream, epic, area)  |
| `SCOPE_TERM`    | allowed-surface directories | The directories a task is permitted to modify            |
| `REVIEWERS`     | Claude Code, Codex, CodeRabbit, Gemini Code Assist | AI reviewers that comment on this project's PRs |

No filenames are configured here on purpose. Which documents the next session
must read is a property of the repository, discovered when the handover is
written, not declared in advance. See Bootstrap below.

## When to invoke

- User explicitly asks for a "handover", "resume prompt", or similar before
  running `/clear`.
- User signals they're about to start a fresh session and need to carry state
  forward.
- After a major milestone (PR merge, task completion) when the user is wrapping
  up a working block.

## Output rules

1. **Wrap the entire handover in a single fenced ```code block```** so the user
   can copy/paste it verbatim. Use Markdown headings inside the block.
2. **Follow the output template below exactly**: same headings, same order,
   same numbering.
3. **Every section appears every time.** A section with nothing to report is
   written out with `none` as its body. Never omit a heading. Stable structure
   is the point: it lets the reader scan to a known position, and it
   distinguishes "checked, nothing there" from "not checked". Dropping empty
   sections reintroduces exactly the drift the template exists to remove.
4. **Language: English.** The handover represents persisted-state knowledge and
   may live in chat, but it mirrors what would otherwise sit in project
   Markdown files. Even when the surrounding chat is in another language, the
   handover body is English.
5. **Self-contained.** Assume the next session has zero context. Repeat the
   facts the next agent needs, even if they are "obvious" right now.
6. **Cap length at ~90 lines** unless the current session has unusually deep
   state (multi-PR, multi-task, mid-incident, many open review threads).
7. **Use exact identifiers**: commit hashes, PR numbers, issue numbers, branch
   names, file paths, background shell IDs, reviewer comment IDs. No vague
   references.
8. **Skip platform and environment boilerplate** (model name, OS, shell). The
   next session resolves its own environment.
9. **After the fenced block**, surface pending uncommitted local changes with
   the exact `git` commands the user should run before clearing. These are easy
   to miss otherwise.

## Output template

Fill in each placeholder. Keep the headings and their order verbatim.

```
# Session handover: <repo name>, <date>

## 1. Project context
- Repo: <absolute path>
- Stack: <one-line lock summary>
- Hard rules: <language, commit conventions, push policy, parallelism>

## 2. Bootstrap
Read in this order before doing anything:
1. <file that exists in this repo>
2. <file that exists in this repo>
3. Skim <agent instructions file>

Re-hydrate reviewer state on demand:
- gh pr view <num> --comments
- gh api repos/<owner>/<repo>/pulls/<num>/comments

## 3. State
- Last completed <TASK_UNIT>: <ID> | PR #<num> | <commit hash>
- Open branches: <branch> (<what it holds>)
- Uncommitted local changes: <paths, or none>
- Session-only state not yet on disk: <what, or none>

## 4. Running state
- Background shell <shell ID>: <command>, <purpose>. Kill: <command>
- Dev server <name> on port <port>, started by <shell ID>. Stop: <command>
- Worktree <path> on branch <branch>. Remove: <command>
- Other live processes: <what, or none>

## 5. Active <TASK_UNIT>
- ID: <id>, <TASK_GROUPING>: <name>, issue #<num>
- Depends on: <ids, or none>
- <SCOPE_TERM>: <paths>
- Acceptance criteria: <one-line summary>
(If nothing is active, write: none, no <TASK_UNIT> in flight.)

## 6. Pre-push verification gate
<exact commands, one shell line if possible>

## 7. Merge flow
- Branch protection: <rules>
- Expected re-review from: <REVIEWERS>
- Thread-resolution gotchas: <notes>
- Merge command: <command>

## 8. Open review threads
### PR #<num>: <title>
- <reviewer> | <file>:<line> | <ask in one phrase>
  - Disposition: accepted, fixed in <commit hash>
- <reviewer> | <file>:<line> | <ask in one phrase>
  - Disposition: disputed, <one-line counter-argument>
- <reviewer> | <file>:<line> | <ask in one phrase>
  - Disposition: pending, <planned action>
(If there are none, write: none, no open review threads.)

## 9. External system mapping
- <issue tracker> to GitHub: <mapping>
- MCP availability: <notes>
- Other integrations: <what this session learned, or none>

## 10. Parallelism
- Default: <serial or parallel teammates>
- Caveats: <worktree isolation gotchas, etc., or none>

## 11. Owner decisions not yet on disk
- <decision or preference> | Why: <owner's reason> | Applies to: <scope>
(If there are none, write: none, every decision is already in a file.)

## 12. Ask the owner before acting
- <question only the owner can answer> | Assumed meanwhile: <default, or none>
(If there are none, write: none, nothing to confirm.)

## 13. Immediate next action
<single concrete first step, actionable within 30 seconds>
```

## Section notes

What qualifies for each section. The template fixes the shape; these notes fix
the content.

1. **Project context.** The rules that would surprise a fresh session if
   violated, not a general project description. Rules already written in the
   repository go here as a pointer; decisions the owner made in this session
   that no file records yet go in section 11.
2. **Bootstrap.** The files the next session must read first, in order.
   Discover them, do not assume them: look at what this repository actually
   keeps. Typically agent instructions (`CLAUDE.md`, `AGENTS.md`), a progress
   or status document, the spec for the task in flight, a relevant ADR. List
   only files that exist. A handover pointing at a file that is not there is
   worse than one pointing at nothing, because the next session will go looking
   for it and conclude the repository is broken.
   Then the exact `gh` commands to fetch reviewer state. Store the commands,
   not their output: comment bodies go stale the moment someone replies,
   whereas a command returns current truth whenever it is run.
3. **State.** What is on disk and in git: last completed `TASK_UNIT` and its PR
   with hash and number, branches still open, uncommitted changes, and any
   facts the session learned that have not been written down anywhere. What
   the owner *decided* belongs in section 11, not here.
4. **Running state.** What is alive but not on disk, and therefore lost
   entirely when the session is cleared. Background shells started with
   `run_in_background` must be listed with their **shell ID**: without the ID
   the next session cannot query or terminate them, and will only discover them
   later as a port already in use. Include dev servers and their ports, open
   `git worktree` paths with their branches, and anything else still running
   (tunnels, file watchers, containers). Every entry carries its own teardown
   command, for example:
   - `Background shell abc123: npm run dev, dev server on :3000. Kill: KillShell(abc123)`
   - `Worktree ../repo-task-12 on branch task-12-parser. Remove: git worktree remove ../repo-task-12`
5. **Active `TASK_UNIT`.** ID, `TASK_GROUPING`, issue number, dependencies,
   `SCOPE_TERM`, and an acceptance-criteria summary. If nothing is in flight,
   say so explicitly rather than leaving the reader to infer it.
6. **Pre-push verification gate.** The exact commands this project requires,
   combined into one shell line where possible.
7. **Merge flow.** Branch-protection and ruleset behaviour, the re-review
   pattern expected from each of the `REVIEWERS`, and the exact merge command.
   Include thread-resolution gotchas where they matter: some reviewers block
   the merge until every thread is resolved, while others post inline comments
   that do not block but still need triage.
8. **Open review threads.** For each open PR, every unresolved comment from the
   `REVIEWERS` that touched this session's work, grouped by PR:
   - **Reviewer**: which of the `REVIEWERS` posted it
   - **PR number and `file:line`**
   - **Ask** in one phrase
   - **Disposition**, one of:
     - *accepted*, with the commit hash that fixed it
     - *disputed*, with a one-line counter-argument, so the next session does
       not re-litigate a decision that has already been made
     - *pending*, with the planned action

   This is the section that survives `/clear` least well by default, and it
   matters most. Without it, a fresh session reads a reviewer's comment with no
   record of the reasoning that rejected it, and reopens a settled argument.
9. **External system mapping.** Issue-tracker to GitHub mapping, MCP
   availability notes, and anything else this session learned about external
   integrations.
10. **Parallelism.** The current default and any caveats discovered during the
    session.
11. **Owner decisions not yet on disk.** Choices and preferences the owner
    stated in this session, each with the reason they gave, that no file in
    the repository or the agent instructions records yet: a process rule
    ("the napkin goes in the task PR, not its own"), a scope cut, a rejected
    edge case, a repository that is retired, a constraint about the business,
    budget or deadline. These are what a fresh session gets wrong most often,
    because the state sections carry the task but not the owner's rules for
    doing it. One line each; the reason matters, because without it the next
    session cannot tell a firm rule from a passing remark. If a decision is
    already in a file, leave it out and cite the file in Bootstrap instead.
12. **Ask the owner before acting.** What the next session must confirm before
    it starts, because only the owner knows: the goal behind the next step,
    its audience, a constraint this session did not settle, or anything that
    may have been decided elsewhere since this handover was written. Give the
    default this session would assume, so the owner can answer with a yes.
    Leave out anything the repository or a command can answer. When this
    section is not `none`, the immediate next action is to ask these questions
    and wait for the answers.
13. **Immediate next action.** One concrete step. If a review thread is the
    next action, name the reviewer, the PR and the `file:line`. If section 12
    has questions, the step is asking them.

## Anti-patterns

- Don't include exploratory output, file dumps, or large code excerpts.
  Reference paths instead.
- Don't paste full review-comment bodies. One-phrase ask plus disposition only.
  The next session runs the `gh` commands from Bootstrap to fetch the full text
  on demand.
- Don't restate generic `CLAUDE.md` content the next session will re-load on
  its own. Focus on session-specific deltas and active state. Section 11 does
  not break this rule: it holds only what is not written anywhere yet.
- Don't paraphrase ADRs or specs already on disk; cite them by path.
- Don't include conversation chronology or "what we tried" narratives. A
  handover is forward-looking state, not a session log. A decision and its
  reason go in section 11; the discussion that led to it does not.
- Don't fill section 12 with questions the next session can answer itself by
  reading files or running commands. It is for what only the owner knows.
- Don't add a session-summary preamble in the chat language before the fenced
  block; the user wants to copy and paste, not read narrative.
- Don't drop a section because it is empty. Write `none` and move on.

## Curation hint

If the current session uncovered a recurring lesson worth keeping beyond the
next session, raise it *separately* from the handover, even when it also
appears in section 11 so that it survives the next hop, and let the user decide
where it lives: agent instructions, an ADR, a runbook, whatever the project
already keeps.

The two have different lifetimes and must not be mixed. A handover is transient
state that expires the moment the next session absorbs it. A lesson is durable
and belongs wherever the project is read from at the start of every session.
Putting a durable lesson in a handover loses it after one hop; putting
transient state in a long-lived document slowly turns a runbook into a log.
Section 11 is the bridge between them: it carries an unwritten decision for
one hop, and the curation suggestion is how it gets written down for good.

Suggest, don't write. This skill produces a handover and nothing else. It never
creates or edits project files.

If one of the `REVIEWERS` repeatedly flags the same class of issue across PRs,
that is a durable lesson, not a handover entry.
