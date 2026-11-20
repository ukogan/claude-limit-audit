---
name: limit-audit
description: Audit a Claude setup for outdated claims that a Claude tool "can't" do something. Scans global and project instruction files (CLAUDE.md, AGENTS.md), saved memory notes, skills, commands and agent definitions; dates each limitation claim; retests stale ones with docs lookups and harmless read-only probes; reports still-true / no-longer-true / untestable; and stamps or fixes the notes only after approval. Use on "/limit-audit", "audit my Claude setup", "are my notes about Claude limits still true", "recheck what Claude can't do", or when a recalled limit is about to block a task.
argument-hint: "[--days N] [--path DIR ...] [--include-plugins] [--report-only]"
---

# Limit audit

Claude tooling changes every week. Notes like "routines can't be edited",
"headless runs can't publish", or "this connector is summaries-only" were true
when someone wrote them. Many are not true now. A stale "can't" is expensive:
the model refuses work it could do, or sends the person to do it by hand.

This skill finds every such claim in the person's setup, retests the stale
ones safely, and reports. It changes files only after the person approves.

## Arguments

- `--days N`: a claim is stale when its last check is older than N days.
  Default 14.
- `--path DIR`: extra project roots to scan. Repeatable. Default: the current
  working directory and its git root.
- `--include-plugins`: also scan installed plugin and marketplace files. These
  are third-party; report them, never edit them.
- `--report-only`: stop after the report. Propose nothing.

## Step 1. Find the files

Scan these, skipping any that do not exist:

1. Global: `~/.claude/CLAUDE.md`, `~/.claude/CLAUDE.local.md`,
   `~/.claude/rules/**/*.md`, `~/.claude/skills/**/SKILL.md` and other `.md`
   under skills, `~/.claude/commands/**/*.md`, `~/.claude/agents/**/*.md`,
   `~/.claude/output-styles/**/*.md`.
2. Memory: `~/.claude/projects/*/memory/**/*.md`. Read only the memory dirs
   for the projects in scope, unless the person asks for all of them.
3. Each project root: `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md` at any
   depth (skip `node_modules`, `.git`, `vendor`, build output),
   `.claude/skills`, `.claude/commands`, `.claude/agents`, `.claude/rules`,
   `.agents/skills`.
4. With `--include-plugins`: `~/.claude/plugins/**/*.md`.

Do not read settings JSON for claims. Do note any `permissions.deny` entries
whose comment says a tool is broken; treat that comment as a claim.

## Step 2. Pull out the claims

First narrow with a case-insensitive search for limit words, then read each
hit in context. Starter pattern:

```
can'?t|cannot|can not|unable|not (yet )?supported|unsupported|doesn'?t (work|support)|won'?t|unreliable|flaky|broken|blocked|refuses?|rejected|not available|unavailable|no way to|only (works|supports)|lacks?|missing|minimum|maximum|max\b|cap(ped)? at|limit(ed|ation)?|workaround|by hand|manually|hard[- ]stop
```

Keep a hit only if it claims something about **Claude's own tooling**:
Claude Code (CLI, desktop, web, IDE), its tools, hooks, settings, permission
modes, subagents, skills, plugins, MCP and connectors, scheduled or cloud
runs, headless `claude -p`, claude.ai features (artifacts, projects, docs,
design), the models, or the API. Also keep claims about what Claude can
reach (for example "the web fetch tool is blocked by site X").

Drop claims about the person's own code, their company's systems, or
third-party services, unless the claim is about Claude's access to them.
Drop style rules ("never use emojis"); those are preferences, not limits.

For each kept claim record: file, line, the claim in one sentence, the
context it was observed in if stated (interactive session, headless run,
subagent, cloud routine, a specific connector or plan), and its date.

## Step 3. Date each claim

Use the first of these that exists:

1. An explicit stamp near the claim: "checked YYYY-MM-DD", "as of", "verified".
2. A date in the note's own text or frontmatter (`modified`, `date`,
   `created`).
3. `git log -1 --format=%cs -- <file>` when the file is in a repo. This dates
   the file, not the line; use `git log -L` or `git blame` on the line when
   the file is long.
4. The file's modification time. Mark it "file date, weak".

A claim is stale when its date is older than `--days`. Undated claims are
stale.

## Step 4. Retest the stale claims

Use the cheapest test that settles the claim. Go down this ladder and stop at
the first rung that gives a clear answer:

1. **This session.** Is the tool or capability in your own tool list right
   now? Does `claude --help` or `claude <subcommand> --help` show it? Record
   the Claude Code version (`claude --version`).
2. **Official docs.** code.claude.com/docs, docs.claude.com,
   support.claude.com, the Claude Code changelog, and anthropic.com or
   claude.com release posts. If a `claude-code-guide` agent is available, ask
   it. Prefer pages dated after the claim.
3. **A harmless probe.** A read-only call: list, get, read, search, a
   `--dry-run`, or a write into a throwaway scratch folder that nothing else
   reads.

Never do any of these to test a claim: create, update, delete, publish,
send, post, fire or schedule anything; change settings or permissions;
spend money; touch shared or production state; contact another person. If
only one of those would prove the claim, mark it **needs a write** and
write the exact one-step test the person could approve.

**Context matters.** A limit can be true in one context and false in
another. "Routines refuse edits" was true for an agent running inside a
routine and false from the owner's own session. Record the context you
tested in. If it differs from the claim's context, say so, and do not call
the claim false on that evidence alone.

**Privacy.** Search the web with generic wording about the feature. Never
paste the person's file contents, names, project names, URLs or IDs into a
search or any outside service.

If there are more than about 15 stale claims, group them by product area and
hand each group to a read-only subagent with the rules in this step. Collect
their verdicts; do not let them edit files.

## Step 5. Report

Give one numbered line per claim, grouped by verdict, newest risk first:

1. **No longer true.** The claim, where it lives, the evidence (link or
   command and its result), and the fix you propose.
2. **Partly true.** What still holds, what changed.
3. **Needs a write to test.** The exact test to approve.
4. **Still true.** One line each, with the evidence.
5. **Fresh.** Count only. These were checked within `--days`.

Lead with one sentence: how many claims, how many were stale, how many are
now false. Keep the report on one screen when you can; put long evidence in
a file the person can open.

Stop here with `--report-only`.

## Step 6. Fix, after approval

Ask once which numbered items to apply. Then, for the approved items only:

1. **Still true:** add a stamp next to the claim:
   `(checked YYYY-MM-DD, Claude Code X.Y.Z: still true; <how>)`.
2. **No longer true:** rewrite the claim to say what is true now, then stamp
   it. If the whole note is now wrong, propose deleting it and wait for a yes.
3. **Partly true:** narrow the claim to the part that holds, then stamp it.
4. Keep any index or table of contents that lists the note in step with the
   new wording (for example a memory `MEMORY.md` line).

Never edit plugin or marketplace files. Never edit a file outside the scanned
roots. Commit nothing unless the person asks.

## Step 7. Leave a rule behind

If the person's setup has no rule about rechecking limits, offer to add one
line to their global instructions:

> A note that says a Claude tool can't do something is a dated observation.
> Before refusing or sending work back on its strength, try the action once.
> Write new limit notes with a "checked YYYY-MM-DD" date.
