# ATLAS バックテスト実行

検証済み戦略のバックテストを2層構成で実行してください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — バックテスト対象の世代ID
- `<generation_id> --layer <1|2>` — 特定レイヤーのみ実行
- 引数なし — 最新の検証済み（validated）世代を自動選択

## 実行手順

### 1. 事前チェック
- 対象世代の `strategies/<generation_id>/strategy.py` が存在するか確認
- 必要な市場データ（Parquet）が `data/market/` に存在するか確認
- データ不足の場合 `/atlas-data fetch <instrument>` の実行を提案

### 2. Python CLI でバックテスト実行

```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main backtest <generation_id>
```

オプション:
- `--layer 1` — Layer 1（vectorbt高速スクリーニング）のみ
- `--layer 2` — Layer 2（Event-Driven精密シミュレーション）のみ
- 省略時は両方実行（L1→L2→Monte Carlo→Gate判定）

### 3. 実行内容（自動）

**Layer 1: vectorbt高速スクリーニング**
- 戦略のgenerate_signal()を全バーに適用
- IS/OOSデータでの基本メトリクス算出
- パラメータ感度分析（±10%/±20%変動によるSharpe CV）
- 最低基準（取引>=20, PF>=0.8, Sharpe>=0.3）不合格でLayer 2スキップ

**Layer 2: Event-Driven精密シミュレーション**
- バー逐次処理（1バー約定遅延、スプレッド、スリッページモデル）
- SL/TP逆指値判定、ポジションサイジング
- フルメトリクス算出（PF, Sharpe, Sortino, Calmar, DD等）
- Walk Forward Analysis（IS/OOS分割、WFA効率比、Strategy Drift）

**Monte Carloシミュレーション**
- 10,000回のブートストラップリサンプリング
- 95%信頼区間ドローダウン、破産確率

**Backtest Gate判定（Immutable 8条件）**

| 条件 | 閾値 |
|------|------|
| Profit Factor | >= 1.2 |
| Max Drawdown | <= 20% |
| Sharpe Ratio | >= 0.8 |
| OOS PF | >= IS PF x 0.7 |
| Strategy Drift | <= 50% |
| 最低取引数 | >= 50件 |
| パラメータ感度スコア | <= 0.5 |
| WFA Efficiency Ratio | >= 0.7 |

### 4. 結果確認

CLIの出力JSONを解釈し、以下のフォーマットで報告:

```
=== ATLAS バックテスト結果 ===
世代ID: <generation_id>
データ: <instrument> <timeframe> (<bars>バー)

--- Layer 1 (vectorbt) ---
PF: X.XX | Sharpe: X.XX | MaxDD: XX.X% | 取引数: XXX
感度スコア: X.XX
判定: PASS / FAIL

--- Layer 2 (Event-Driven) ---
PF: X.XX | Sharpe: X.XX | Sortino: X.XX | MaxDD: XX.X%
取引数: XXX | 勝率: XX.X% | 平均PnL: ¥XXX
WFA効率比: X.XX | Strategy Drift: X.XX
OOS PF: X.XX

--- Monte Carlo (10,000回) ---
95% MaxDD: XX.X% | 破産確率: X.XX%
リターン中央値: XX.X%

--- Backtest Gate ---
[PASS] / [FAIL]  (N/8条件通過)
  各条件の判定結果を表示

次のステップ:
  [PASS] → /atlas-evaluate <generation_id>
  [FAIL] → /atlas-improve <generation_id>
```

### 5. 結果保存
- `strategies/<generation_id>/backtest/result.json` に自動保存済み
- History Store への記録は `/atlas-evaluate` で実施

## エラーハンドリング
- 戦略ファイル未存在 → `/atlas-validate` の実行を促す
- データ不足 → `/atlas-data fetch <instrument>` を案内
- Layer 1不合格 → Layer 2スキップの旨を明記
- 実行エラー → エラー内容を報告し対処法を提案

## 注意事項
- Backtest Gate判定は **Layer 2結果のみ** で実施（Layer 1は参考値）
- 品質スコア < 0.80 のデータは拒否される
- 結果は `strategies/<generation_id>/backtest/` に自動保存
