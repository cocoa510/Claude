# ATLAS ステータス表示

現在の世代状況、スコア推移、戦略ランキングを表示してください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- 引数なし — 全体サマリーを表示
- `<generation_id>` — 特定戦略の系譜とスコア推移を表示
- `--top <N>` — スコア上位N件を表示

## 実行手順

### 1. History Storeからデータ取得
```bash
# 上位戦略一覧
cd ATLAS && .venv/Scripts/python.exe -m atlas.main status

# 特定戦略の系譜
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history list <strategy_id>

# スコア上位
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history top --limit 10
```

### 2. 収束状態の確認（特定戦略の場合）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main converge <strategy_id>
```

### 3. 保有データの確認
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main data status
```

### 4. 表示内容

#### 全体サマリー（デフォルト）
```
=== ATLAS ステータス ===

--- 戦略ランキング ---
順位 | 世代ID              | 世代 | スコア | 状態
  1  | ATLAS-2026-0405-001 |  3   | 0.76  | excellent
  2  | ATLAS-2026-0405-002 |  2   | 0.58  | passed
  3  | ATLAS-2026-0404-001 |  5   | 0.49  | passed

--- 市場データ ---
USD_JPY (H1): 18,720バー (2023-01 〜 2025-12) 品質: 0.95

--- 次のアクション ---
- ATLAS-2026-0405-001 は優良到達 → /atlas-export
- ATLAS-2026-0405-002 は改善余地あり → /atlas-improve
```

#### 特定戦略の詳細
```
=== ATLAS-2026-0405-001 系譜 ===

世代 | スコア | 状態          | 収束判定
  0  | 0.42  | passed        |
  1  | 0.55  | passed        |
  2  | 0.68  | passed        |
  3  | 0.76  | excellent     | ★ 優良到達

収束状態: stop_excellent
推奨: /atlas-export ATLAS-2026-0405-001
```

## エラーハンドリング
- History Storeが空 → `/atlas-generate` の実行を促す
- 指定IDが存在しない → 利用可能な戦略IDを表示
- DBファイルが存在しない → `/atlas-init` を案内
