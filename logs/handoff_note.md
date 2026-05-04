# Handoff Note

**最終更新**: 2026-05-05T22:40:00Z @ TRAINING-GOTO
**ブランチ**: master
**直前のコミット**: 7e61557 [atlas] 210-213 GBP/JPY D=10 TP精密探索

## 現在の作業（1 行サマリ）

GBP/JPY H4 SL=2.0 D値・TP値探索で全履歴最高値 soft=0.9468 (ATLAS-2026-0504-211: D=10 SL=2.0 TP=10) 確定。FTS 106戦略稼働中。

## 詳細コンテキスト

### 今セッション主要発見
1. **SL=2.0 がGBP/JPYに革命的**: SL=1.5では完全failだったD=7〜12が、SL=2.0で全て Gate PASS
2. **全時代最高値**: GBP/JPY D=10 SL=2.0 TP=10 → soft=0.9468 (ATLAS-2026-0504-211)
3. **TP最適化完了**: D=10 SL=2.0 TP曲線: 6(0.866)→8(0.943)→9(0.931)→10(0.9468)→11(0.941)→12(0.919)
4. **EUR/JPY SL=2.0は全D値でL1 FAIL**: GBP/JPY固有の特性
5. **USD/JPY SL=2.0 最適**: D=3 TP=15 → soft=0.893

### 確定した最適値
- **GBP/JPY H4 絶対最優秀**: D=10 SL=2.0 TP=10 (211) soft=0.9468 ← 全履歴最高
- **EUR/JPY H4 最優秀**: D=5 SL=1.5 TP=12 (175) soft=0.899
- **USD/JPY H4 最優秀**: D=3 SL=2.0 TP=15 (187) soft=0.893

### 全時代TOP 10 (soft_score)
| Rank | ID | ペア | D | SL | TP | soft |
|---|---|---|---|---|---|---|
| 1 | 211 | GBP/JPY | 10 | 2.0 | 10 | **0.9468** |
| 2 | 212 | GBP/JPY | 10 | 2.0 | 11 | 0.9414 |
| 3 | 197 | GBP/JPY | 10 | 2.0 | 8 | 0.9433 |
| 4 | 213 | GBP/JPY | 10 | 2.0 | 9 | 0.9305 |
| 5 | 192 | GBP/JPY | 3 | 2.0 | 12 | 0.9269 |
| 6 | 182 | GBP/JPY | 5 | 2.0 | 12 | 0.9252 |
| 7 | 180 | GBP/JPY | 5 | 2.0 | 8 | 0.925 |
| 8 | 199 | GBP/JPY | 12 | 2.0 | 8 | 0.9339 |
| 9 | 196 | GBP/JPY | 7 | 2.0 | 12 | 0.9025 |
| 10 | 175 | EUR/JPY | 5 | 1.5 | 12 | 0.899 |

### FTS 状況
- インポート済み戦略: 106件
- 最新追加: ATLAS-2026-0504-213まで全Gate PASS戦略インポート済み

## 次にやること

1. **GBP/JPY H1 D=10 SL=2.0 TP=10 試行** — H4最高値。H1でも有効か確認
2. **GBP/USD H4 D=10 SL=2.0 TP=10** — 非JPYペアへの転用
3. **EUR/JPY D=5 SL=1.5 TP=12 に SL=1.0 変形** — さらなる win rate 向上
4. **GBP/JPY D=10 SL=2.0 TP=10 に min_atr 調整** — 現在 min_atr=0.30
5. **FTS runner 再起動確認** — 106戦略で正常動作確認

## 関連文書・コマンド

- 最新戦略ID: ATLAS-2026-0504-213
- 次の採番: ATLAS-2026-0504-214
- バックテスト: `cd /c/data/works/FX/ATLAS && .venv/Scripts/python.exe -m atlas.main backtest <ID>`
- FTS インポート先: `C:\data\works\FX\fx_trading_system\trading_platform\strategies\imported\`
- FTS 再起動: `cd /c/data/works/FX/fx_trading_system && PowerShell.exe -Command "& .\scripts\start_unified_runner.ps1"`

## 引継ぎ時の注意

- **pips_per_unit=100 は必須** (JPYペア。非JPYは10000)
- **SL=2.0 は GBP/JPY 専用**: EUR/JPY は全D値でL1 FAIL
- バックテスト前の validate は不要 (Stage 4 は long-only戦略で誤判定)
- GBP/JPY D=10 SL=2.0 TP=10 が全時代最高。min_atr / cooldown 調整でさらなる改善余地あり
- USD/JPY でも SL=2.0 TP=15 は有効 (soft=0.893)
