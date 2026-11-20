# limit-audit

Your Claude setup is full of notes about what Claude can't do. Many of them
stopped being true weeks ago.

`limit-audit` is a Claude Code skill. It finds every claim that a Claude tool
can't do something, in your global and project `CLAUDE.md` files, memory
notes, skills, commands and agents. It dates each claim, retests the stale
ones, and tells you which are now false.

- It tests with docs lookups and read-only probes. It never creates, deletes,
  publishes or sends anything to test a claim.
- It changes your files only after you approve each fix.
- It never sends your file contents to a web search.

## Install

```
/plugin marketplace add ukogan/claude-limit-audit
/plugin install limit-audit@limit-audit
```

## Use

Run the command, or ask Claude to "audit my Claude setup for outdated limits."

```
/limit-audit:limit-audit
/limit-audit:limit-audit --days 30
/limit-audit:limit-audit --path ~/code/other-repo --report-only
```

| Option | What it does |
|---|---|
| `--days N` | Treat a claim as stale after N days. Default 14. |
| `--path DIR` | Also scan this project. Repeatable. |
| `--include-plugins` | Also scan installed plugins. Reported, never edited. |
| `--report-only` | Report and stop. |

## What you get

1. **No longer true**: the claim, where it lives, the evidence, a proposed fix.
2. **Partly true**: what still holds.
3. **Needs a write to test**: the one-step test for you to approve.
4. **Still true**: with evidence.

Approved fixes add a stamp such as
`(checked 2026-11-19, Claude Code 2.1.328: still true; docs)` so the next
audit can skip fresh claims.

## Why

A real case: a memory note said cloud routines refuse prompt edits. That was
true for an agent running inside a routine. From the owner's own session the
edit worked on the first try. The note had made Claude send the owner to do
the edit by hand.

## License

MIT
