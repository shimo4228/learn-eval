Language: English | [日本語](README.ja.md)

# learn-eval

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/learn-eval) [![GitMCP](https://img.shields.io/endpoint?url=https://gitmcp.io/badge/shimo4228/learn-eval)](https://gitmcp.io/shimo4228/learn-eval)

An Agent Skill that extracts reusable patterns from Claude Code sessions, self-evaluates their quality, and determines the right save location (Global vs Project) before saving.

## Install

### Claude Code (manual)

```bash
cp -r skills/learn-eval ~/.claude/skills/learn-eval
```

### SkillsMP

Coming soon.

## How It Works

The skill follows a 7-step process:

1. **Review** the session for extractable patterns
2. **Identify** the most valuable/reusable insight
3. **Determine save location** — Global (`~/.claude/skills/learned/`) for cross-project patterns, Project (`.claude/skills/learned/`) for project-specific knowledge
4. **Draft** the skill file using a standardized format
5. **Quality gate** — run checklist + holistic judgment
6. **Confirm** with the user (verdict-specific confirmation flow)
7. **Save** or absorb into the determined location

## Quality Gate

Every candidate pattern goes through a two-part evaluation:

### Checklist (must actually read files to verify)

- [ ] Grep `~/.claude/skills/` for keyword overlap — confirmed no content duplication
- [ ] Check MEMORY.md (project + global) for duplication
- [ ] Consider whether appending to an existing skill is sufficient
- [ ] Confirm the pattern is reusable, not a one-off fix

### Holistic Judgment

Based on the checklist results, one of four verdicts is issued:

| Verdict | Meaning | Next Action |
|---------|---------|-------------|
| **Save** | Unique, specific, properly scoped | Proceed to save |
| **Improve then Save** | Valuable but needs refinement | List improvements, revise, re-evaluate (once) |
| **Absorb into [X]** | Should be appended to an existing skill | Show target + diff, then save |
| **Drop** | Trivial, redundant, or too abstract | Explain reason and stop |

## What to Extract

1. **Error Resolution Patterns** — root cause + fix + reusability
2. **Debugging Techniques** — non-obvious steps, tool combinations
3. **Workarounds** — library quirks, API limitations, version-specific fixes
4. **Project-Specific Patterns** — conventions, architecture decisions, integration patterns

## What NOT to Extract

- Trivial fixes (typos, simple syntax errors)
- One-time issues (specific API outages, temporary version bugs)
- Patterns that can be found by searching the error message
- Standard documentation-level knowledge

## Supporting Files

- **[`skills/learn-eval/knowledge-placement-decision.md`](skills/learn-eval/knowledge-placement-decision.md)** — A decision tree for choosing between Memory, existing skill append, and new skill creation. Prevents skill sprawl and knowledge burial.

## References

The **grounding check** in the quality gate — verify each extracted pattern against the session's observed record (actual tool output, errors, user corrections) rather than the agent's own summary, because a purely self-evaluating loop drifts — is grounded in 2026 work on continual skill learning:

- [SkillLearnBench: Benchmarking Continual Learning Methods for Agent Skill Generation on Real-World Tasks](https://arxiv.org/abs/2604.20087) (Zhong et al., 2026) — finds that self-feedback alone induces *recursive drift*, while iterations grounded in external feedback yield genuine improvement.

This is the operational counterpart, one layer up the stack, of the model-collapse result: a generative process re-fed its own output degrades. The checklist forces the extraction to re-anchor on what was observed, not on the agent's own prior phrasing.

## About this skill

This skill implements the **Extract** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) — a Zenodo-citable six-phase bidirectional growth loop ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)) for sustaining intent alignment between an AI agent and its operator over time. AKC is one of three research lines by [@shimo4228](https://github.com/shimo4228), alongside [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) — autonomous agents grounded in four contemplative axioms — and [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)) — harness-neutral ADRs on accountability distribution.

## License

[MIT](LICENSE)
