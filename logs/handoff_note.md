# Handoff Note

**最終更新**: 2026-05-06T02:00:00Z @ TRAINING-GOTO
**ブランチ**: master
**直前のコミット**: 5b1c284 [atlas] 231-234: D=8/D=9 探索完了

## 現在の作業（1 行サマリ）

GBP/JPY H4 パラメータ空間を網羅的に探索。全時代最高値 soft=0.9483 (ATLAS-2026-0504-216: D=10 SL=2.0 TP=10 min_atr=0.40) 完全確定。FTS 127戦略稼働中。

## 詳細コンテキスト

### 確定した最優秀設定（不動の全時代最高）
**ATLAS-2026-0504-216: GBP/JPY H4 D=10 SL=2.0 TP=10 min_atr=0.40**
- soft_score=0.9483
- Tier1 ALL PASS (8/8)
- 検証済みパラメータ空間:
  - D値: 3〜15 (全探索済み、D=10が頂点)
  - TP値: 6,8,9,10,11,12 (TP=10が頂点)
  - min_atr: 0.30〜0.50 (0.40が頂点)
  - SL値: 1.5, 2.0, 2.5 (2.0が頂点)
  - ATR期間: 14, 21 (14が頂点)
  - max_hold: 24, 28, 40 (同等)
  - cooldown: 3, 5 (3が若干有利)

### 対ペア最優秀一覧
| ペア | 最優秀ID | soft | D | SL | TP | min_atr |
|---|---|---|---|---|---|---|
| **GBP/JPY** | **216** | **0.9483** | **10** | **2.0** | **10** | **0.40** |
| EUR/JPY | 219 | 0.907 | 5 | 1.5 | 12 | 0.40 |
| USD/JPY | 187 | 0.893 | 3 | 2.0 | 15 | 0.10 |

### 重要な知見
- **SL=2.0 はGBP/JPY専用**: EUR/JPY D=3,5,7 + USD/JPY D=10 はL1 FAIL
- **min_atr=0.40 はD≥8向け**: D=3/5では効果なし、D≥8で有効
- **非JPYペア**: GBP/USD, EUR/USD はLONG-only構造が不適合 (carry bias なし)
- **H1 タイムフレーム**: GBP/JPY H1はノイズ多くL2 FAIL
- **EUR/JPY D=7 TP=12 min_atr=0.40 (228): soft=0.890** (177:0.881から改善)

### FTS 状況
- インポート済み戦略: **127件**
- 最新ID: ATLAS-2026-0504-234
- Runner 再起動済み (2026-05-06)

## 次にやること

1. **全時代最高の超越を狙う — 新方向**:
   - GBP/JPY D=10 SL=2.0 TP=10 min_atr=0.40 + base_risk=0.01 (ポジション倍増検証)
   - GBP/JPY D=10 SL=2.0 TP=10 min_atr=0.40 + cooldown=2 (冷却短縮)
   - EUR/JPY D=5 SL=1.5 TP=12 min_atr=0.40 + TP=10 (EUR最高値への攻め)
2. **GBP/JPY D1 タイムフレーム** (日足データ未取得、要 `atlas data` コマンド)
3. **SHORT戦略探索** (ダウントレンド対応、全戦略がLONG-onlyで方向集中リスク)

## 関連文書・コマンド

- 最新戦略ID: ATLAS-2026-0504-234
- 次の採番: ATLAS-2026-0504-235
- バックテスト: `cd /c/data/works/FX/ATLAS && .venv/Scripts/python.exe -m atlas.main backtest <ID>`
- FTS 再起動: `cd /c/data/works/FX/fx_trading_system && PowerShell.exe -Command "& .\scripts\start_unified_runner.ps1"`
- データ取得: `cd /c/data/works/FX/ATLAS && .venv/Scripts/python.exe -m atlas.main data <PAIR> <TF>`

## 引継ぎ時の注意

- **pips_per_unit=100 は必須** (JPYペア。非JPYは10000)
- **SL=2.0 は GBP/JPY 専用**: EUR/JPY は全D値でL1 FAIL
- バックテスト前の validate は不要 (Stage 4 は long-only戦略で誤判定)
- 現状 127戦略全て LONG-only → SHORT戦略の追加を検討すべき時期
- GBP/JPY D=10 SL=2.0 TP=10 min_atr=0.40 soft=0.9483 が真の全時代最高値
