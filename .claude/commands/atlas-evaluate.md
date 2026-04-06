# ATLAS 評価・スコアリング

バックテスト結果に基づき、26指標の算出と3層スコアリングによる総合評価を実行します。
メトリクス算出・スコアリング・弱点検出はPython CLI、結果の深掘り解釈はquant-analystエージェントに委譲します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — 評価対象の世代ID（例: `ATLAS-2026-0405-001`）
- `<generation_id> --detailed` — 全26指標の詳細値を表示
- `<generation_id> --compare <other_id>` — 他世代との比較評価
- 引数なし — 最新のバックテスト完了世代を自動選択

## 実行手順

### 1. バックテスト完了確認
`strategies/<generation_id>/backtest/result.json` が存在するか確認。
未完了なら `/atlas-backtest` の実行を促す。

### 2. 26指標メトリクス算出 + スコアリング + 弱点検出（Python CLI）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main metrics <generation_id>
```

JSON出力に含まれるフィールド:
- `metrics` — 26指標（6カテゴリ: core, risk, trade_efficiency, market_adaptability, robustness, monte_carlo）
- `gate` — Backtest Gate判定結果（8条件 + passed/failed_conditions）
- `score` — 3層スコアリング（core_score, layer2_adjustment, quality_multiplier, final_score）
- `weaknesses` — ルールベース弱点検出（target_component, action_type, priority, description）
- `categories` — カテゴリ別メトリクス整理
- `summary` — 強み・懸念のサマリー
- `monte_carlo` — Monte Carlo結果

### 3. quant-analyst エージェントに深掘り解釈を委譲

Agent tool で `quant-analyst` エージェントを起動し、以下のタスクを渡す:

> 以下のFXトレード戦略のバックテスト評価結果を解釈してください。
>
> - 戦略ID: `<generation_id>`
> - メトリクスJSON: [Step 2の出力をそのまま渡す]
>
> 以下を実施すること:
> 1. 3層スコアリング結果の妥当性を検証（Python算出済み）
> 2. Backtest Gate基準との照合（Python判定済み）
> 3. 弱点の因果チェーン深掘り（Python一次検出 → エー��ェントが根本原因を特定）
> 4. 改善指示の策定（最大3件、優先度順、定量的期待効果付き）
> 5. 戦略の強みの明記
>
> 評価レポートを `ATLAS/strategies/<generation_id>/evaluation/report.json` に出力し、
> 人間可読なサマリーを `ATLAS/strategies/<generation_id>/evaluation/summary.md` に出力すること。

### 4. devil-advocate エージェントによる敵対的レビュー

**目的:** AI自己評価バイアス（Anthropic研究）への対策。quant-analystの評価を独立した視点で検証する。

Agent tool で `devil-advocate` エージェントを起動し、以下のタスクを渡す:

> 以下のquant-analystによる評価レポートに対して、敵対的レビューを実施してください。
>
> - 戦略ID: `<generation_id>`
> - メトリクスJSON: [Step 2の出力]
> - quant-analystの評価レポート: [Step 3の出力（report.json の内容）]
>
> 以下を検証すること:
> 1. 楽観バイアスの検出 - 強みの過大評価、弱点の過小評価がないか
> 2. 因果チェーンの論理検証 - 弱点の原因分析は正しいか、別の仮説はないか
> 3. 改善提案の実現可能性 - 他指標への副作用、期待効果の妥当性
> 4. Backtest Gate通過の現実性 - 現在地からの距離は埋められるか
> 5. オーバーフィッティングの見落とし - IS/OOS乖離、パラメータ感度
> 6. ATLASの既知失敗パターン（フィルター追加の罠、条件緩和のジレンマ等）との照合
>
> 敵対的レビューを `ATLAS/strategies/<generation_id>/evaluation/adversarial_review.json` に出力すること。

### 5. 評価統合 - 両エージェントの見解をマージ

quant-analystの評価レポートとdevil-advocateの敵対的レビューを統合する:

1. devil-advocateが `severity=high` のバイアスを検出した場合:
   - quant-analystの該当評価を下方修正する
   - 改善指示を devil-advocate の counter_proposal で補正する
2. devil-advocateが `abandon_recommendation=true` の場合:
   - 改善ではなく新規生成（`/atlas-generate`）を推奨する
3. 両者が合意した場合:
   - quant-analystの評価をそのまま採用する

統合結果を `ATLAS/strategies/<generation_id>/evaluation/integrated_report.json` に出力する。

### 6. 出力フォーマット
```
=== ATLAS 評価結果 ===
世代ID: <generation_id>

--- 総合スコア ---
final_score: X.XX / 1.00
判定: 優良(>=0.75) / 合格(>=0.55) / 中間帯(0.40-0.55) / 不合格(<0.40)

--- スコア内訳 ---
Layer 1 コアスコア: X.XX
  Sharpe(x0.30): X.XX  |  1-MaxDD(x0.25): X.XX
  PF(x0.15): X.XX      |  Consistency(x0.15): X.XX
  Recovery(x0.10): X.XX |  WinRate(x0.05): X.XX
Layer 2 補正: +X.XX / -X.XX
Layer 3 品質乗数: xX.XX

--- Backtest Gate ---
[PASS/FAIL] (N/8条件通過)
  各条件の判定結果

--- 弱点分析（quant-analyst） ---
[優先度1] <弱点パターン>
  根拠: <具体的な指標値>
  因果チェーン: <原因→結果の連鎖>
  改善提案: <具体的アクション（定量的期待効果付き）>

--- 強み ---
  [強みリスト]

--- 敵対的レビュー（devil-advocate） ---
  バイアス検出: X件 (high:X, medium:X, low:X)
  [severity=highの項目を表示]
  リスク警告: [最悪ケースシナリオ]
  判定: 評価に同意 / 評価を修正 / 廃棄推奨

--- 統合判定 ---
  最終評価: [quant-analyst + devil-advocate の統合結果]
  改善計画の修正: [あれば記載]

次のステップ:
  [優良] /atlas-export <generation_id>
  [合格/中間帯] /atlas-improve <generation_id>
  [不合格] /atlas-generate — 新規戦略を生成
```

## エラーハンドリング
- バックテスト未完了 → `/atlas-backtest` の実行を促す
- メトリクス算出でデータ不足 → 算出不能な指標を明記
- quant-analyst レポートが不完全 → 追加分析を依頼
