# ATLAS 世代履歴の閲覧・比較

世代別の履歴を閲覧し、任意の世代間で比較分析を行ってください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — 特定戦略の全世代一覧
- `<id1> <id2>` — 2戦略の比較（差分表示）
- `--top <N>` — スコア上位N件を表示（デフォルト: 5）
- 引数なし — スコア上位5件の一覧表示

## 実行手順

### 1. 履歴データの取得

```bash
# スコア上位一覧（デフォルト）
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history top --limit 5

# 特定戦略の全世代
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history list <generation_id>

# 特定世代の詳細
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <generation_id> --generation <N>

# 次の世代ID採番
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history next-id
```

### 2. 収束状態の確認（特定戦略の場合）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main converge <generation_id>
```

### 3. 2戦略の比較（id1 id2 指定時）

両戦略のバックテスト結果を取得し、`comparator.py` の結果を解釈:
```bash
# 各戦略の最新レコードを取得
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <id1>
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <id2>
```

評価結果ファイルがあれば読み取り:
- `strategies/<id>/backtest/result.json`
- `strategies/<id>/evaluation/`

### 4. 表示内容

#### スコア上位一覧（デフォルト）
```
=== ATLAS 世代履歴 ===

順位 | 戦略ID              | 世代 | スコア | 状態
  1  | ATLAS-2026-0405-001 |  3   | 0.76  | excellent
  2  | ATLAS-2026-0405-002 |  2   | 0.58  | passed
  3  | ATLAS-2026-0404-001 |  5   | 0.49  | passed
```

#### 特定戦略の全世代
```
=== ATLAS-2026-0405-001 系譜 ===

世代 | スコア | 状態          | 改善率
  0  | 0.42  | passed        | -
  1  | 0.55  | passed        | +31.0%
  2  | 0.68  | passed        | +23.6%
  3  | 0.76  | excellent     | +11.8% ★ 優良到達
```

#### 2戦略の比較
```
=== 戦略比較: <id1> vs <id2> ===

指標              | <id1>    | <id2>    | 差分     | 変化
final_score       |   0.68   |   0.76   | +0.08    | ↑
Profit Factor     |   1.38   |   1.52   | +0.14    | ↑
Max Drawdown      |  16.2%   |  13.5%   | -2.7pp   | ↑ (改善)
Sharpe Ratio      |   1.15   |   1.28   | +0.13    | ↑
```

## エラーハンドリング
- History Storeが空 → `/atlas-generate` の実行を促す
- 指定IDが存在しない → 利用可能な戦略IDを `history top` で表示
- DBファイルが存在しない → `/atlas-init` を案内

## 注意事項
- 表示はHistory Store（SQLite: `data/atlas_history.db`）からリアルタイム取得
- 比較は同一通貨ペア・時間足の戦略同士が最も有意義
- 詳細な比較分析が必要な場合は `/atlas-compare` を使用
