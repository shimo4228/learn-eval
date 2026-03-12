---
name: learn-eval
description: "Extract reusable patterns from the session, self-evaluate quality before saving, and determine the right save location (Global vs Project)."
origin: original
---

# /learn-eval - Extract, Evaluate, then Save

`/learn` の全フローに、保存前の品質評価と保存先判断を追加したコマンド。

## What to Extract

Look for:

1. **Error Resolution Patterns** — root cause + fix + reusability
2. **Debugging Techniques** — non-obvious steps, tool combinations
3. **Workarounds** — library quirks, API limitations, version-specific fixes
4. **Project-Specific Patterns** — conventions, architecture decisions, integration patterns

## Process

1. Review the session for extractable patterns
2. Identify the most valuable/reusable insight

3. **Determine save location:**
   - Ask: "Would this pattern be useful in a different project?"
   - **Global** (`~/.claude/skills/learned/`): 2+ プロジェクトで使える汎用パターン
   - **Project** (`.claude/skills/learned/` in current project): このプロジェクト固有の知見
   - 迷ったら Global

4. Draft the skill file using this format:

```markdown
---
name: pattern-name
description: "130文字以内の説明"
user-invocable: false
origin: auto-extracted
---

# [Descriptive Pattern Name]

**Extracted:** [Date]
**Context:** [Brief description of when this applies]

## Problem
[What problem this solves - be specific]

## Solution
[The pattern/technique/workaround - with code examples]

## When to Use
[Trigger conditions]
```

5. **Quality gate — チェックリスト + ホリスティック判定**

   #### 5a. 必須チェックリスト（実際にファイルを読んで確認）

   - [ ] `~/.claude/skills/` 配下をキーワード grep し、内容重複を確認した
   - [ ] MEMORY.md（プロジェクト + グローバル）との重複を確認した
   - [ ] 既存スキルへの追記で済むか検討した（knowledge-placement-decision 参照）
   - [ ] 一回限りの修正ではなく、再利用可能なパターンであることを確認した

   #### 5b. ホリスティック判定

   | Verdict | 意味 | 次のアクション |
   |---------|------|---------------|
   | **Save** | 独自・具体的・適切なスコープ | Step 6 へ |
   | **Improve then Save** | 価値はあるが要改善 | 改善点リスト → 修正 → 再判定（1回まで） |
   | **Absorb into [X]** | 既存スキルの一部として追記すべき | 追記先と追加内容を提示 → Step 6 へ |
   | **Drop** | 些末・冗長・抽象的 | 理由を説明して終了 |

6. **Verdict 別の確認フロー**

   - **Save**: 保存先パス + チェックリスト結果 + 判定理由1行 + ドラフト全文を提示
   - **Absorb into [X]**: 追記先パス + 追加内容（diff 形式）+ チェックリスト結果 + 判定理由を提示
   - **Drop**: チェックリスト結果 + 理由のみ表示

7. Save / Absorb to the determined location

## Notes

- Don't extract trivial fixes (typos, simple syntax errors)
- Don't extract one-time issues (specific API outages, etc.)
- Focus on patterns that will save time in future sessions
- Keep skills focused — one pattern per skill
- Absorb verdict が出た場合は新規ファイルを作らず、既存スキルへの追記を優先する
