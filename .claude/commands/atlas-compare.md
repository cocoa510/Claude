# ATLAS 戦略比較分析

複数の戦略を横断的に比較し、最適な戦略の選定を支援してください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<id1> <id2> [id3...]` — 指定した世代同士を比較
- `--top <N>` — スコア上位N件を比較（デフォルト: 5）
- 引数なし — スコア上位5件を自動比較

## 実行手順

### 1. 比較データ取得（Python CLI）

指定ID同士の場合:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <id1>
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <id2>
```

上位N件の場合:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history top --limit <N>
```

各世代のメトリクスも取得:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main metrics <generation_id>
```

### 2. 比較分析

取得したデータを基に以下を分析:

#### 基本比較テーブル
主要15指標を横断比較し、各指標の最良値に ★ マークを付与:
- コア指標: PF, Sharpe, Sortino, MaxDD, Win Rate, Total Trades
- 効率指標: Recovery, Calmar, Edge Ratio, Trade Efficiency
- 堅牢性: WFA Efficiency, Strategy Drift, Param Sensitivity, Consistency, SQN

#### トレンド分析（同一系統の場合）
- 世代間のスコア推移（改善/安定/劣化）
- 振動検出（スコアが上下交互に振れていないか）

#### 2世代差分（改良前後の場合）
- 改善された指標リスト（差分 + 変化率）
- 劣化した指標リスト
- 純改善数（改善数 - 劣化数）

### 3. 出力フォーマット
```
=== 戦略比較 ===

指標              | <id1>         | <id2>         | <id3>
final_score       | 0.76 ★        | 0.58          | 0.49
PF                | 1.52 ★        | 1.41          | 1.35
MaxDD             | 13.5% ★       | 17.2%         | 19.8%
Sharpe            | 1.28 ★        | 1.05          | 0.92
Sortino           | 2.34 ★        | 1.87          | 1.42
Win Rate          | 58.3%         | 62.1% ★       | 51.2%
Trades            | 342           | 456 ★         | 289
WFA Eff           | 0.85 ★        | 0.72          | 0.68
Drift             | 0.20 ★        | 0.35          | 0.42

★ = 各指標の最良値

--- トレンド ---
推移: 改善傾向
最良世代: <id1> (score: 0.76)

--- 推奨 ---
次のステップ:
  最良が優良(>=0.75) → /atlas-export <id1>
  改善の余地あり → /atlas-improve <id1>
```

## エラーハンドリング
- 比較対象が1件のみ → 単体サマリーを表示し、他候補の検索を提案
- バックテスト未完了の世代が含まれる → 除外して比較し警告表示
- History Store にデータがない → `/atlas-backtest` の実行を促す

## 注意事項
- 異なる通貨ペア・時間足の戦略比較ではその旨を明記
- 世代間比較は同一 strategy_id の系統で最も意味がある
