Language: English | [日本語](README.ja.md)

# claude-skill-learn-eval

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

## License

[MIT](LICENSE)
