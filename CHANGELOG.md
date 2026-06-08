# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Planned

- Initial public release.

### What it does

A Claude Code Agent Skill that extracts reusable patterns from a session (workarounds, decisions, hard-won solutions), self-evaluates their quality before saving, and determines the right save location: Global (`~/.claude/`) for patterns that apply across all projects, or Project (`.claude/` in the current repo) for project-scoped knowledge. The skill is designed to keep the knowledge surface from filling with low-signal observations that pollute future sessions.

### Components

- `skills/learn-eval/SKILL.md` — the skill body. Quality gate + placement decision workflow. Specifies what counts as a "non-obvious reusable pattern" (the trigger that promotes a session observation to a persisted knowledge artifact) and the Global vs Project split criteria.

### Scope

The skill assumes Claude Code as the harness and `~/.claude/skills/` / `.claude/skills/` as the knowledge storage convention. The quality gate (is the pattern reusable beyond this specific context, does it pass the "would a future me thank me?" test) is the load-bearing discipline and transfers to other agent harnesses without modification.

### Requirements

- The skill is documentation-only; the user confirms saves and chooses the target file. No runtime dependencies.

### Relationship to companion skills

| Skill | Role | When |
|---|---|---|
| [`search-first`](https://github.com/shimo4228/search-first) | Research before coding | Earlier phase of the AKC knowledge lifecycle (Search → Skill-stocktake → Learn-eval → Rules-distill) |
| [`skill-stocktake`](https://github.com/shimo4228/skill-stocktake) | Quality audit for existing skills / commands | Earlier phase; identifies what to retire before new extraction adds more |
| [`rules-distill`](https://github.com/shimo4228/rules-distill) | Promote cross-cutting principles to rules | Later phase; consumes learn-eval output and lifts repeating patterns from skills to rules |

This skill implements the **Extract** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) — a Zenodo-citable six-phase bidirectional growth loop for sustaining intent alignment between an AI agent and its operator over time.
