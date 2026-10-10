Language: English | [日本語](README.ja.md)

# learn-eval

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/learn-eval)

An Agent Skill that extracts reusable patterns from a Claude Code session, checks each one against what actually happened in the session, and saves it only where a future session will actually find it: appended to an existing skill, rule, or `hooks/README.md` section (the file where the author's harness documents its hooks; a target only if you keep one), or promoted to a skill of its own. A pattern that is trivial, redundant, or has no such destination is dropped.

It edits those skill, rule, and README files only after you confirm that candidate. The author's other work is listed under [More from the author](#more-from-the-author).

## Install

There are two routes; pick one. Both need [uv](https://docs.astral.sh/uv/) on your PATH, because the quality gate's overlap check runs a bundled Python script (Python 3.11 or later, no third-party dependencies) through it. No paid key is needed.

**Clone this repository** to install learn-eval alone, and run it as `/learn-eval`:

```bash
git clone https://github.com/shimo4228/learn-eval.git
mkdir -p ~/.claude/skills
cp -r learn-eval/skills/learn-eval ~/.claude/skills/learn-eval
```

The skill calls its script at `~/.claude/skills/learn-eval`, so keep the folder at that path. Absorbing a pattern into an existing file needs nothing else; promoting one to a new skill calls `skill-creator`, which this route does not install, so there you write the `SKILL.md` yourself against the [Agent Skills specification](https://agentskills.io/specification).

**Install the [akc-cycle](https://github.com/shimo4228/akc-cycle) Claude Code plugin** to get the same skill as `/akc-cycle:learn-eval`, together with `skill-creator` and the other skills of the Agent Knowledge Cycle (AKC: the author's six-phase, human-gated cycle that turns a coding agent's repeated experience into skills and rules). learn-eval is the cycle's Extract phase. The plugin copy calls the script from inside the plugin.

```
/plugin marketplace add shimo4228/akc-cycle
/plugin install akc-cycle@akc-cycle
```

This repository is synced one way from the same source as the plugin, so between syncs it can trail the plugin.

Run it at the end of a work session: type the command for your route, or ask Claude Code to save what the session taught (the skill's description also matches 「今回の学びを残して」).

## How It Works

The skill follows an 8-step process:

1. **Review** the session for extractable patterns
2. **Identify** the most valuable, reusable insights; each one becomes its own candidate
3. **Pick the destination.** A kept pattern goes to exactly one of two: **absorb** the pattern into an existing skill, rule, or `hooks/README.md` section that already owns the topic (the default), or **promote** it to a skill of its own when it has an independent trigger that no installed skill answers. If neither fits, the verdict is Drop and nothing is saved
4. **Draft** the candidate as a scratch note (name, description, Problem, Solution, When to Use), written to the session's scratchpad so the overlap script can read it
5. **Quality gate**: overlap candidates (installed skills and memory lines the draft may duplicate), checklist, draft-specific questions, then one holistic verdict
6. **Confirm** with the user, one candidate at a time (`[y/n/skip]`), evidence first, never batch approval
7. **Write**: append in place and show the diff, or hand the draft to [`skill-creator`](https://github.com/shimo4228/claude-harness/tree/main/skills/skill-creator), which owns shape, boundaries, and its own draft gate
8. **Reachability check**: state in one line what will route to the saved content in a future session. If the honest answer is "someone would have to grep for it", the save was wrong; go back to step 3. There is no notes directory to fall back on

Before it asks you to confirm (step 6), the skill shows its evidence in this shape, adapted from the output format in SKILL.md. The values are SKILL.md's illustration, not a recorded run. SKILL.md's format has no Grounding row; it is added here for the last checklist item, the grounding check, which the gate also runs:

```
### Overlap candidates (from scripts/overlap_candidates.py)
- skills: 0.60 git-workflow [bash, c, cd, git, permission, status] → same knowledge, Absorb
- memory: MEMORY.md:45 feedback_git_dash_c_over_cd, 4 shared terms → already recorded

### Checklist
- [x] Candidates judged one by one: (verdict per candidate, quoting shared terms)
- [x] Append-to-existing considered: should append to [X]
- [x] Reusability: confirmed
- [x] Grounding: (the tool output, error or user correction it rests on)

### Draft-specific questions
- [Yes] Q1: ... — one-line evidence
- [No]  Q2: ... — one-line evidence → (on Improve: one-line fix plan)

### Verdict: Absorb into [X]
**Rationale:** (1–2 sentences; always mentions any No questions)
```

## Quality Gate

Every candidate that has a destination goes through three layers (one with no destination was already dropped at step 3). The first two produce evidence, and only the third produces a verdict.

### 1. Overlap candidates (script)

`scripts/overlap_candidates.py` takes the draft as a file and enumerates what it may duplicate: installed skills, ranked by how much of each skill's *description* the draft covers (the description is what routes a future session), and the index lines of MEMORY.md (Claude Code's memory index) from both the project and the global memory. It prints JSON with the shared terms for every candidate and never says "this is a duplicate"; that judgment stays with the model. It also lists every file it could not read, so an unread file is not silently counted as "no overlap".

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
| **Absorb into [X]** | Belongs inside an existing skill, rule, or `hooks/README.md` section | Show the target and the diff, confirm, then append |
| **Drop** | Trivial, redundant, abstract, or unreachable | Explain why and stop |

A No on the grounding check (the last checklist item) leans the verdict to Drop even when everything else is Yes.

## What to Extract

1. **Error Resolution Patterns** — root cause + fix + reusability
2. **Debugging Techniques** — non-obvious steps, tool combinations
3. **Workarounds** — library quirks, API limitations, version-specific fixes
4. **Project-Specific Patterns** — conventions, architecture decisions, integration patterns

## What NOT to Extract

- Trivial fixes (typos, simple syntax errors): the verdict is Drop
- One-time issues (specific API outages, temporary version bugs): the checklist requires a reusable pattern, not a one-off fix

## More from the author

- **[Not Reasoning, Not Tools — What If the Essence of AI Agents Is Memory?](https://dev.to/shimo4228/not-reasoning-not-tools-what-if-the-essence-of-ai-agents-is-memory-4k4n)** ([日本語](https://zenn.dev/shimo4228/articles/agent-essence-is-memory)): where learn-eval came from, and how it joined search-first and rules-distill in one loop over an agent's memory.
- **[LLM-as-Judge Shouldn't Aggregate Scores: Binary Checks as Evidence, One Holistic Verdict](https://dev.to/shimo4228/llm-as-judge-shouldnt-aggregate-scores-binary-checks-as-evidence-one-holistic-verdict-822)** ([日本語](https://zenn.dev/shimo4228/articles/llm-judge-checks-not-scores)): why this skill's quality gate answers yes/no questions as evidence for one named verdict and never sums them, drawn from running this gate and a skill audit.
- **[akc-cycle](https://github.com/shimo4228/akc-cycle)**: installs this skill together with the rest of the cycle as one Claude Code plugin.
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: the reasoning behind each phase of the cycle, Extract among them, recorded as dated design decisions.
- **[rules-distill](https://github.com/shimo4228/rules-distill)**: a later phase, which finds principles that belong in the always-loaded rules and drafts them as rules, with your confirmation for each.
- **[skill-stocktake](https://github.com/shimo4228/skill-stocktake)**: audits the skills you have saved for staleness, conflicts and redundancy, with a verdict per skill.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with AKC next to the author's other long-running projects and their DOIs.

## License

[MIT](LICENSE)

<details>
<summary>For tools and AI assistants</summary>

learn-eval is an Agent Skill for Claude Code that extracts a reusable pattern from a work session, gates it against what actually happened in the session, and saves it only where a future session will be routed to it, for people who want an agent's lessons to be found again rather than piled up. Every pattern it keeps is either absorbed into an existing skill, rule or `hooks/README.md` section (the Absorb into [X] verdict) or promoted to a skill of its own (the Save verdict); anything else is dropped.

It exists because saved notes that nothing points at are not read. The author's earlier version kept each pattern under `skills/learned/`; over 74 days (2026-06-10 to 2026-08-23) those notes were opened 184 times (file reads by the agent recorded in the harness's usage log), 161 of them on six days when audit skills were deciding whether to keep them, and only 12, across 8 notes, inside a repository where real work was going on. So every save now has to name what will route to it, and the extraction is checked against the session's observed record (tool output, errors, user corrections) rather than the agent's own summary, because a self-evaluating loop drifts. That grounding check follows [SkillLearnBench](https://arxiv.org/abs/2604.20087) (Zhong et al., 2026), which finds that self-feedback alone induces recursive drift while iterations grounded in external feedback yield genuine improvement; in a skill library this is the failure that model collapse ([Shumailov et al., 2024](https://www.nature.com/articles/s41586-024-07566-y)) describes for model training, applied one level up to what an agent writes into its own skills. The draft-specific yes/no questions and the rule that a failed question becomes an improvement item are ported from BinEval, [Ask, Don't Judge: Binary Questions for Interpretable LLM Evaluation and Self-Improvement](https://arxiv.org/abs/2606.27226), and the answers are never summed because that paper's own stated limits say that on holistic quality dimensions the share of affirmed questions does not map linearly to quality.

Canonical facts: MIT license; a `SKILL.md` plus one Python script, `scripts/overlap_candidates.py` (Python 3.11 or later, standard library only, run through `uv` with the bundled `uv.lock`, tests under `tests/`); maintained by one author (@shimo4228). Status: active, synced one way from the author's Claude Code harness by `scripts/sync-from-local.sh` (it never commits), and also shipped in the akc-cycle plugin, so this repository can trail the plugin between syncs. Requirements: Claude Code, where the skill is developed and tested (it is written to be portable to other Agent Skills-compatible agents, but on another agent the overlap script still compares against Claude Code's locations, skills under `~/.claude/skills` and memory under `~/.claude/projects`, unless given `--skills-root` for the skills and `--no-auto-memory` to stop the `~/.claude/projects` lookup; `--memory` only adds MEMORY.md files to compare), and `uv` on PATH; no paid key. The skill edits skill, rule and `hooks/README.md` files, each only after a one-at-a-time `[y/n/skip]` confirmation (the scratch draft it passes to the overlap script is written before that), and hands new skills to `skill-creator`.

Example: the overlap script takes the draft as a file; with the clone install the skill runs `uv run --frozen --project ~/.claude/skills/learn-eval --directory ~/.claude/skills/learn-eval python scripts/overlap_candidates.py --draft <draft.md> --project "$PWD"` (the plugin copy runs it from inside the plugin), and it prints JSON listing installed skills ranked by how much of each description the draft covers, plus matching MEMORY.md index lines, each with its shared terms and every file it could not read; it never says "duplicate". The model then answers a fixed checklist and 3 to 5 draft-specific yes/no questions and issues one verdict: Save, Improve then Save, Absorb into [X], or Drop.

Links: [skills/learn-eval/SKILL.md](skills/learn-eval/SKILL.md) is the skill itself; [llms.txt](llms.txt) and [llms-full.txt](llms-full.txt) are the machine-readable summary and reference. The skill implements the Extract phase of the [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle), concept DOI [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726); cite AKC by that DOI. The installable form of the whole cycle is [akc-cycle](https://github.com/shimo4228/akc-cycle).

</details>
