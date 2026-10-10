Language: [English](README.md) | 日本語

# learn-eval

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/learn-eval)

Claude Code セッションから再利用可能なパターンを抽出し、その 1 件ずつを、セッションで実際に起きたことに照らして確かめ、将来のセッションが実際にたどり着く場所にだけ保存する Agent Skill です。保存先は、既存の skill・rule・`hooks/README.md`（著者の harness が hook を説明しているファイル。持っている場合だけ保存先になります）の節への追記か、独立した skill への昇格のどちらかです。些末なもの、既にあるもの、そのどちらの保存先にも当てはまらないものは捨てます。

これらの skill・rule・README のファイルを変えるのは、その候補をあなたが確認したあとだけです。著者のほかの仕事は[著者のほかの仕事](#著者のほかの仕事)にまとめています。

## インストール

インストールの方法は 2 つあり、どちらか 1 つを選びます。どちらでも [uv](https://docs.astral.sh/uv/)（英語）に PATH が通っている必要があります。品質ゲートの重複チェックが、同梱の Python スクリプト（Python 3.11 以上、外部ライブラリなし）を uv 経由で実行するためです。有料キーは要りません。

**このリポジトリを clone する**と learn-eval だけが入り、`/learn-eval` で呼びます。

```bash
git clone https://github.com/shimo4228/learn-eval.git
mkdir -p ~/.claude/skills
cp -r learn-eval/skills/learn-eval ~/.claude/skills/learn-eval
```

スキルはスクリプトを `~/.claude/skills/learn-eval` から呼ぶので、フォルダはこの場所に置いてください。既存のファイルへの追記にはほかに何も要りません。新しい skill への昇格では `skill-creator` を呼びますが、この方法では入らないので、その場合は [Agent Skills 仕様](https://agentskills.io/specification)（英語）に沿って `SKILL.md` を自分で書きます。

**Claude Code プラグイン [akc-cycle](https://github.com/shimo4228/akc-cycle)（英語）を入れる**と、同じスキルが `/akc-cycle:learn-eval` という名前で、`skill-creator` と Agent Knowledge Cycle のほかのスキルと一緒に入ります。Agent Knowledge Cycle（AKC）は著者の 6 フェーズのサイクルで、コーディングエージェントが繰り返した経験を、人の承認を経て skill と rule に変えます。learn-eval はこのサイクルの Extract（抽出）フェーズです。プラグイン版はスクリプトをプラグインの中から呼びます。

```
/plugin marketplace add shimo4228/akc-cycle
/plugin install akc-cycle@akc-cycle
```

このリポジトリはプラグインと同じ元から一方向に同期しているので、同期と同期の間はプラグインより古いことがあります。

作業セッションの終わりに実行します。入れた方法に応じたコマンドを入力するか、Claude Code に「今回の学びを残して」と頼んでください。

## 仕組み

このスキルは次の 8 つのステップで動きます。

1. **レビュー** — セッションから抽出できるパターンを探します
2. **特定** — 最も価値があり再利用できる知見を選びます。1 件ずつが別の候補になります
3. **保存先の決定** — 残すパターンの行き先は 2 つのうちどちらか 1 つです。そのトピックを既に持っている既存の skill・rule・`hooks/README.md` の節へ**追記する**（既定）か、インストール済みのどの skill も受けない独立した発火条件を持つ場合に、独自の skill へ**昇格させる**かです。どちらにも当てはまらなければ判定は Drop で、何も保存しません
4. **ドラフト** — 候補をスクラッチノートとして書きます（name / description / Problem / Solution / When to Use）。重複チェックのスクリプトが読めるよう、セッションのスクラッチパッドに書き出します
5. **品質ゲート** — 重複候補（ドラフトと重なりうるインストール済みの skill とメモリの行）の列挙、チェックリスト、ドラフト固有の質問を経て、最後に総合判断で判定を 1 つ出します
6. **確認** — 候補を 1 件ずつ、証拠を提示してから `[y/n/skip]` で確認します（一括承認はしません）
7. **書き込み** — その場で追記して diff を見せるか、形・境界・独自のドラフトゲートを受け持つ [`skill-creator`](https://github.com/shimo4228/claude-harness/tree/main/skills/skill-creator)（英語）にドラフトを渡します
8. **到達可能性の確認** — 保存した内容へ将来のセッションを導くものは何かを 1 行で述べます。正直な答えが「誰かが grep するしかない」なら、その保存は誤りです。ステップ 3 に戻ります。逃げ込めるノート置き場はありません

確認（ステップ 6）を求める前に、スキルは次の形で証拠を示します。SKILL.md が定める出力形式をもとにした形で、値は SKILL.md の例示であり、実際の実行記録ではありません。SKILL.md の形式には Grounding の行がありません。この行は、ゲートが同じく行うチェックリスト最後の項目（接地チェック）に合わせてここで足しています。

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

## 品質ゲート

保存先が決まった候補は、すべて 3 つの層を通ります（保存先のない候補は、ステップ 3 で既に Drop になっています）。最初の 2 層は証拠を出すだけで、判定を出すのは 3 層目だけです。

### 1. 重複候補の列挙（スクリプト）

`scripts/overlap_candidates.py` はドラフトをファイルとして受け取り、重複しうるものを列挙します。インストール済みの skill は、各 skill の *description* をドラフトの語がどれだけ覆うかで順位づけします（将来のセッションを導くのは description だからです）。MEMORY.md（Claude Code のメモリの索引）の index 行は、プロジェクトとグローバルの両方を対象にします。出力は JSON で、候補ごとに共有語が付きます。スクリプトは「これは重複だ」とは言いません。その判断はモデル側に残します。読めなかったファイルはすべて一覧に出すので、読めていないファイルが黙って「重複なし」に数えられることはありません。

### 2. チェックリストとドラフト固有の質問

- [ ] 残った重複候補を 1 件ずつ、共有語を引用して判定した
- [ ] 既存 skill への追記を先に検討した
- [ ] 一回限りの修正ではなく、再利用可能なパターンである
- [ ] エージェント自身の要約ではなく、セッションの観測記録（ツール出力・エラー・ユーザーの訂正）に接地している

そのうえでスキルは、ドラフト固有の yes/no 質問を 3〜5 個作ります。1 問につき検証可能な主張を 1 つだけ問い、反証を探す形で書きます（例:「コード例は、書かれている環境でそのまま動くか」）。答えは Yes / No と 1 行の証拠です。これを足し合わせてスコアにすることはありません。

### 3. 総合判断

チェックリスト・質問への答え・ドラフトをまとめて見て、判定を 1 つだけ出します。No だった質問は、判定の根拠として必ず列挙します:

| Verdict | 意味 | 次のアクション |
|---------|------|---------------|
| **Save** | 単独で立つ。独自・具体的・適切なスコープで、インストール済みのどの skill も受けない発火条件を持つ | 確認のうえ、独自の skill へ昇格 |
| **Improve then Save** | 価値はあるが要修正 | No の質問がそのまま改善項目。修正後、同じ質問で 1 回だけ再判定 |
| **Absorb into [X]** | 既存の skill・rule・`hooks/README.md` の節の中に置くべき | 追記先と diff を提示し、確認のうえ追記 |
| **Drop** | 些末・冗長・抽象的、または到達不能 | 理由を説明して終了 |

接地チェック（チェックリストの最後の項目）が No なら、他がすべて Yes でも判定は Drop 側に倒します。

## 抽出対象

1. **エラー解決パターン** — 根本原因 + 修正方法 + 再利用性
2. **デバッグ技法** — すぐには思いつかない手順、ツールの組み合わせ
3. **ワークアラウンド** — ライブラリの癖、API制限、バージョン固有の修正
4. **プロジェクト固有パターン** — 規約、アーキテクチャ判断、統合パターン

## 抽出しないもの

- 些末な修正（タイポ、単純な構文エラー）: 判定は Drop です
- 一回限りの問題（特定のAPI障害、一時的なバージョン問題）: チェックリストが、一回限りの修正ではなく再利用可能なパターンであることを求めます

## 著者のほかの仕事

- **[推論でもツールでもない — AIエージェントの本質は「記憶」ではないか](https://zenn.dev/shimo4228/articles/agent-essence-is-memory)**（[English](https://dev.to/shimo4228/not-reasoning-not-tools-what-if-the-essence-of-ai-agents-is-memory-4k4n)）: learn-eval がどこから来たのか、そして search-first・rules-distill と一緒にエージェントの記憶を回す 1 つのループになった経緯です。
- **[LLM-as-judge はスコアを集計しない — チェックは証拠、判定は総合判断](https://zenn.dev/shimo4228/articles/llm-judge-checks-not-scores)**（[English](https://dev.to/shimo4228/llm-as-judge-shouldnt-aggregate-scores-binary-checks-as-evidence-one-holistic-verdict-822)）: このスキルの品質ゲートが yes/no の答えを 1 つの判定の証拠として扱い、足し合わせない理由を、このゲートとスキル監査の運用から書いています。
- **[akc-cycle](https://github.com/shimo4228/akc-cycle)**: このスキルをサイクルのほかのスキルと一緒に、1 つの Claude Code プラグインとして入れます（英語）。
- **[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)**: Extract を含むサイクルの各フェーズがなぜあるのかを、日付付きの設計判断として記録しています。
- **[rules-distill](https://github.com/shimo4228/rules-distill)**: 後のフェーズです。常時読み込まれる rule に置くべき原則を見つけ、1 件ずつあなたの確認を取って rule にします（英語）。
- **[skill-stocktake](https://github.com/shimo4228/skill-stocktake)**: 保存した skill が古くなっていないか、矛盾や重複がないかを監査し、skill ごとに判定を出します。
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: 著者の hub です。AKC をほかの長期プロジェクトとその DOI と並べています。

## ライセンス

[MIT](LICENSE)

<details>
<summary>ツールと AI アシスタント向けの資料</summary>

learn-eval は、作業セッションから再利用できるパターンを抽出し、セッションで実際に起きたことに照らして確かめ、将来のセッションが導かれる場所にだけ保存する Claude Code 向けの Agent Skill です。エージェントの学びを積み上げるのではなく、もう一度見つかるようにしたい人向けです。残すパターンはすべて、既存の skill・rule・`hooks/README.md` の節への追記（判定 Absorb into [X]）か、独立した skill への昇格（判定 Save）のどちらかで、それ以外は Drop になります。

このスキルがあるのは、何からも指されていないノートは読まれないからです。著者の以前の版はパターンを `skills/learned/` に保存していました。74 日間（2026-06-10 から 2026-08-23）でこのノート群が開かれたのは 184 回（harness の利用ログに残った、エージェントによるファイル読み込み）で、そのうち 161 回は残すかどうかを監査 skill が判定していた 6 日間に集中し、実際の作業が進んでいる repo の中で読まれたのは 8 件のノートで 12 回だけでした。そのため、いまは保存のたびに何がそこへ導くのかを名指しさせ、抽出をエージェント自身の要約ではなくセッションの観測記録（ツール出力・エラー・ユーザーの訂正）に照合します。自己評価だけで回るループは少しずつずれていくからです。この接地チェックは [SkillLearnBench](https://arxiv.org/abs/2604.20087)（Zhong et al., 2026）に沿っています。この研究は、自己フィードバックだけでは recursive drift（再帰的ドリフト）が起き、外部フィードバックに接地した反復なら実際に改善すると報告しています。skill ライブラリで起きるこのずれは、model collapse（[Shumailov et al., 2024](https://www.nature.com/articles/s41586-024-07566-y)）がモデルの学習について述べた失敗と同じで、自分の出力を食べさせ続けると生成の質が落ちていきます。接地チェックは、その教訓を一段上の、エージェントが自分の skill に書き込む内容に当てはめたものです。ドラフト固有の yes/no 質問と、No だった質問がそのまま改善項目になる規則は、BinEval（[Ask, Don't Judge: Binary Questions for Interpretable LLM Evaluation and Self-Improvement](https://arxiv.org/abs/2606.27226)、英語）から移植しました。答えを足し合わせないのも、同じ論文が自ら認める限界に従ったものです。品質を全体として見る観点では、Yes と答えた質問が多いほど品質が高いとは言えない、と論文は述べています。

基本情報: ライセンスは MIT です。`SKILL.md` と Python スクリプト 1 本、`scripts/overlap_candidates.py`（Python 3.11 以上、標準ライブラリのみ、同梱の `uv.lock` で `uv` から実行、テストは `tests/`）からなり、著者 1 人（@shimo4228）が保守しています。状態は現役で、著者の Claude Code harness から `scripts/sync-from-local.sh`（commit はしません）で一方向に同期しています。akc-cycle プラグインにも入っているので、同期と同期の間はこのリポジトリがプラグインより古いことがあります。必要なものは Claude Code と、PATH の通った `uv` です。スキルは Claude Code で開発・検証しており、ほかの Agent Skills 互換のエージェントへ移せるように書いていますが、そこでも重複チェックのスクリプトは Claude Code の場所（skill は `~/.claude/skills`、メモリは `~/.claude/projects`）と比べます。skill の場所は `--skills-root` で変えられ、`~/.claude/projects` を探すのを止めるのは `--no-auto-memory` です（`--memory` は比べる MEMORY.md を足すだけです）。有料キーは要りません。スキルは skill・rule・`hooks/README.md` のファイルを編集しますが、どれも 1 件ずつ `[y/n/skip]` で確認を取ってからです（重複チェックのスクリプトに渡すスクラッチのドラフトは、その前に書かれます）。新しい skill は `skill-creator` へ渡します。

例: 重複チェックのスクリプトはドラフトをファイルとして受け取ります。clone で入れた場合、スキルは `uv run --frozen --project ~/.claude/skills/learn-eval --directory ~/.claude/skills/learn-eval python scripts/overlap_candidates.py --draft <draft.md> --project "$PWD"` で実行し（プラグイン版はプラグインの中から実行します）、スクリプトは JSON を出します。インストール済みの skill を、各 description をドラフトがどれだけ覆うかで順位づけし、一致する MEMORY.md の index 行も挙げ、それぞれに共有語と読めなかったファイルを付けます。「重複だ」とは言いません。そのうえでモデルが固定のチェックリストと 3〜5 個のドラフト固有の yes/no 質問に答え、Save・Improve then Save・Absorb into [X]・Drop のどれか 1 つの判定を出します。

リンク: [skills/learn-eval/SKILL.md](skills/learn-eval/SKILL.md) がスキル本体（英語）、[llms.txt](llms.txt) と [llms-full.txt](llms-full.txt) が機械可読の要約と参照資料（英語）です。このスキルは [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) の Extract フェーズを実装しています。AKC の concept DOI（常に最新版へつながる代表 DOI）は [10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726) で、引用はこの DOI で行います。サイクル全体を入れる形は [akc-cycle](https://github.com/shimo4228/akc-cycle) です。

</details>
