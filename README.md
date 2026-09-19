Language: English | [日本語](README.ja.md)

# learn-eval

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/learn-eval)

An Agent Skill that extracts reusable patterns from a Claude Code session, checks each one against what actually happened in the session (the grounding checklist), and saves it only where a future session will actually find it: appended to an existing skill, rule, or doc section, or promoted to a skill of its own. A pattern that is trivial, redundant, or has no such destination is dropped.

## Why there is no notes directory

Earlier versions saved each pattern as a note under `skills/learned/`. Between 2026-06-10 and 2026-08-23 (74 days) those notes were opened 184 times, counting each file read by the agent that the harness's usage log recorded. Of the 184, 161 fell on six days when audit skills were deciding whether to keep the notes, and only 12, spread over 8 notes, happened inside a repository where real work was going on. Nothing routed a working session to a note, so a note was found only when someone already suspected it existed and searched for it, and the 12 reads show how rarely that happened. The directory was retired, and every save now has to name what will route to it.

## Install

```bash
git clone https://github.com/shimo4228/learn-eval.git
cp -r learn-eval/skills/learn-eval ~/.claude/skills/learn-eval
```

The overlap check in the quality gate runs a bundled Python script through [uv](https://docs.astral.sh/uv/), so `uv` needs to be on your PATH.

Once installed, run it at the end of a work session: type `/learn-eval`, or ask Claude Code to save what the session taught (the skill's description also matches 「今回の学びを残して」).

## How It Works

The skill follows an 8-step process:

1. **Review** the session for extractable patterns
2. **Identify** the most valuable, reusable insight
3. **Pick the destination.** There are exactly two: **absorb** the pattern into an existing skill, rule, or doc section that already owns the topic (the default), or **promote** it to a skill of its own when it has an independent trigger that no installed skill answers. If neither fits, the verdict is Drop
4. **Draft** the candidate as a scratch note (name, description, Problem, Solution, When to Use)
5. **Quality gate**: overlap candidates, checklist, draft-specific questions, then one holistic verdict
6. **Confirm** with the user, one candidate at a time (`[y/n/skip]`), evidence first, never batch approval
7. **Write**: append in place and show the diff, or hand the draft to a skill-authoring skill that owns shape, boundaries, and its own draft gate. The author's harness uses [`skill-creator`](https://github.com/shimo4228/claude-harness/tree/main/skills/skill-creator); without one, write the `SKILL.md` by hand against the [Agent Skills specification](https://agentskills.io/specification)
8. **Reachability check**: state in one line what will route to the saved content in a future session. If the honest answer is "someone would have to grep for it", the save was wrong; go back to step 3

## Quality Gate

Every candidate goes through three layers. The first two produce evidence, and only the third produces a verdict.

### 1. Overlap candidates (script)

`scripts/overlap_candidates.py` takes the draft as a file and enumerates what it may duplicate: installed skills, ranked by how much of each skill's *description* the draft covers (the description is what routes a future session), and MEMORY.md index lines from both the project and the global memory. It prints JSON with the shared terms for every candidate and never says "this is a duplicate"; that judgment stays with the model. It also lists every file it could not read, so an unread file is not silently counted as "no overlap".

### 2. Checklist and draft-specific questions

- [ ] Each surviving overlap candidate judged one by one, quoting its shared terms
- [ ] Appending to an existing skill considered first
- [ ] The pattern is reusable, not a one-off fix
- [ ] The pattern is grounded in the session's observed record (tool output, errors, user corrections), not in the agent's own summary

The skill then writes 3 to 5 yes/no questions specific to the draft, each testing one verifiable claim and phrased to seek disconfirmation ("Does the code example run as-is in the stated environment?"). Answers are Yes or No plus one line of evidence. They are never added up into a score.

### 3. Holistic verdict

Weighing the checklist, the answers, and the draft together, the skill issues exactly one verdict and lists every No as grounds for it:

| Verdict | Meaning | Next Action |
|---------|---------|-------------|
| **Save** | Stands on its own: unique, concrete, well scoped, with a trigger no installed skill answers | Confirm, then promote it to a skill of its own |
| **Improve then Save** | Valuable but needs fixes | Each No becomes an improvement item; fix, then re-judge once with the same questions |
| **Absorb into [X]** | Belongs inside an existing skill, rule, or doc section | Show the target and the diff, confirm, then append |
| **Drop** | Trivial, redundant, abstract, or unreachable | Explain why and stop |

A No on a grounding question leans the verdict to Drop even when everything else is Yes.

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

## References

The **grounding check** in the quality gate verifies each extracted pattern against the session's observed record (actual tool output, errors, user corrections) rather than the agent's own summary, because a purely self-evaluating loop drifts. It follows 2026 work on continual skill learning:

- [SkillLearnBench: Benchmarking Continual Learning Methods for Agent Skill Generation on Real-World Tasks](https://arxiv.org/abs/2604.20087) (Zhong et al., 2026) finds that self-feedback alone induces *recursive drift*, while iterations grounded in external feedback yield genuine improvement.

Recursive drift in a skill library is the same failure that model collapse ([Shumailov et al., 2024](https://www.nature.com/articles/s41586-024-07566-y)) describes for model training: a generative process re-fed its own output degrades. The grounding check applies that lesson one level up, to what an agent writes into its own skills. The checklist forces the extraction to re-anchor on what was observed, not on the agent's own prior phrasing.

The **draft-specific yes/no questions** and the rule that a failed question becomes an improvement item are ported from BinEval:

- [Ask, Don't Judge: Binary Questions for Interpretable LLM Evaluation and Self-Improvement](https://arxiv.org/abs/2606.27226). The decision not to aggregate the answers into a score follows the paper's own stated limits: on holistic quality dimensions the share of affirmed questions does not map linearly to quality.

## About this skill

This skill implements the **Extract** phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle), a six-phase bidirectional growth loop for sustaining intent alignment between an AI agent and its operator over time ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)).

Related work by [@shimo4228](https://github.com/shimo4228): [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) and [Agent Attribution Practice](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)).

## License

[MIT](LICENSE)
