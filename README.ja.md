Language: [English](README.md) | 日本語

# learn-eval

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/learn-eval) [![GitMCP](https://img.shields.io/endpoint?url=https://gitmcp.io/badge/shimo4228/learn-eval)](https://gitmcp.io/shimo4228/learn-eval)

Claude Code セッションから再利用可能なパターンを抽出し、品質を自己評価した上で、適切な保存先（Global / Project）を判断して保存する Agent Skill。

## インストール

### Claude Code（手動）

```bash
cp -r skills/learn-eval ~/.claude/skills/learn-eval
```

### SkillsMP

近日対応予定。

## 仕組み

7ステップのプロセスで動作します:

1. **レビュー** — セッションから抽出可能なパターンを探す
2. **特定** — 最も価値のある再利用可能な知見を選ぶ
3. **保存先の判断** — Global（`~/.claude/skills/learned/`）はプロジェクト横断パターン、Project（`.claude/skills/learned/`）はプロジェクト固有の知見
4. **ドラフト** — 標準フォーマットでスキルファイルを作成
5. **品質ゲート** — チェックリスト + ホリスティック判定を実施
6. **確認** — Verdict に応じた確認フローをユーザーに提示
7. **保存** — 決定した場所に保存、または既存スキルに追記

## 品質ゲート

すべての候補パターンは2段階の評価を通過します:

### チェックリスト（実際にファイルを読んで確認）

- [ ] `~/.claude/skills/` 配下をキーワード grep し、内容重複を確認した
- [ ] MEMORY.md（プロジェクト + グローバル）との重複を確認した
- [ ] 既存スキルへの追記で済むか検討した
- [ ] 一回限りの修正ではなく、再利用可能なパターンであることを確認した

### ホリスティック判定

チェックリストの結果に基づき、4つの Verdict のいずれかを発行します:

| Verdict | 意味 | 次のアクション |
|---------|------|---------------|
| **Save** | 独自・具体的・適切なスコープ | 保存へ進む |
| **Improve then Save** | 価値はあるが要改善 | 改善点リスト → 修正 → 再判定（1回まで） |
| **Absorb into [X]** | 既存スキルの一部として追記すべき | 追記先 + diff を提示して保存 |
| **Drop** | 些末・冗長・抽象的 | 理由を説明して終了 |

## 抽出対象

1. **エラー解決パターン** — 根本原因 + 修正方法 + 再利用性
2. **デバッグ技法** — 非自明な手順、ツールの組み合わせ
3. **ワークアラウンド** — ライブラリの癖、API制限、バージョン固有の修正
4. **プロジェクト固有パターン** — 規約、アーキテクチャ判断、統合パターン

## 抽出しないもの

- 些末な修正（タイポ、単純な構文エラー）
- 一回限りの問題（特定のAPI障害、一時的なバージョン問題）
- エラーメッセージで検索すれば解決するパターン
- 標準的なドキュメント参照レベルの知識

## 補助ファイル

- **[`skills/learn-eval/knowledge-placement-decision.md`](skills/learn-eval/knowledge-placement-decision.md)** — Memory・既存スキル追記・新規スキル作成の3択を判断するデシジョンツリー。スキルの乱立と知識の埋没を防ぎます。

## このスキルについて

このスキルは [Agent Knowledge Cycle (AKC)](https://github.com/shimo4228/agent-knowledge-cycle) の **Extract** フェーズを実装する — エージェント行動とオペレーターの判断が共発展する 6 フェーズ双方向成長ループ ([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726))。AKC は [@shimo4228](https://github.com/shimo4228) の 3 つの研究ラインの 1 つで、他に [Contemplative Agent](https://github.com/shimo4228/contemplative-agent) ([DOI 10.5281/zenodo.19212118](https://doi.org/10.5281/zenodo.19212118)) — 4 つの contemplative 公理に基づく自律エージェント — と [Agent Attribution Practice (AAP)](https://github.com/shimo4228/agent-attribution-practice) ([DOI 10.5281/zenodo.19652013](https://doi.org/10.5281/zenodo.19652013)) — 自律 AI エージェントの責任分配に関するハーネス中立 ADR — がある。

## ライセンス

[MIT](LICENSE)
