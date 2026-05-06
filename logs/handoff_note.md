# Handoff Note

**最終更新**: 2026-05-06T00:10:00Z @ TRAINING-GOTO
**ブランチ**: master
**直前のコミット**: c1f9c2f [atlas] セッション0505後半: 349/352 Gate PASS (H1 volatility) + M15/CAD/AUD探索完了 Gate PASS累計28件

## 現在の作業（1 行サマリ）

戦略マトリックスのGate PASS=0セルを大規模開拓: 本日14件の新規Gate PASS（累計33件）。hybrid/momentum/volatilityタイプで新セルを多数開拓。FTS 70戦略稼働中。

## 詳細コンテキスト

### 本日の主要成果（0505セッション）

**新規Gate PASSセル一覧（0→PASS化）**:
| ID | 戦略 | ペア/TF | soft_score |
|----|------|---------|-----------|
| 324 | GBP/JPY H4 hybrid (Donchian+Pullback) | GBP/JPY H4 | **0.8605** |
| 335 | EUR/JPY H4 hybrid | EUR/JPY H4 | **0.8195** |
| 338 | AUD/JPY H4 hybrid | AUD/JPY H4 | **0.8335** |
| 329 | EUR/JPY H4 momentum (MACD) | EUR/JPY H4 | **0.8307** |
| 332 | USD/JPY H4 momentum (ADX) | USD/JPY H4 | **0.8413** |
| 334 | AUD/JPY H4 momentum (ADX) | AUD/JPY H4 | **0.7791** |
| 342 | GBP/JPY H4 momentum (ADX) | GBP/JPY H4 | 0.7599 ※OOS悪い |
| 345 | GBP/JPY H4 volatility (BB) | GBP/JPY H4 | 0.7789 |
| 346 | EUR/JPY H4 volatility (BB) | EUR/JPY H4 | **0.8835** |
| 348 | AUD/JPY H4 volatility (BB) | AUD/JPY H4 | **0.8468** |
| 349 | EUR/JPY H1 volatility (BB) | EUR/JPY H1 | **0.8335** |
| 352 | USD/JPY H1 volatility (BB) | USD/JPY H1 | 0.7957 ※OOS悪い |

**確定した知見**:
- H4 JPYクロス全タイプ（breakout/hybrid/momentum/volatility）で成功可能
- BB upper breakout = volatilityタイプとして有効（bollinger_upper feature名が正解）
- M15 BB breakoutはfalse breakout多発で全ペアFAIL
- H1 BB: EUR/JPY・USD/JPYは成功、GBP/JPY・AUD/JPYはFAIL
- CAD/JPY・NZD/JPY: Donchian以外の全アプローチ失敗確定

### FTS現状
- インポート済み: **70件**（本日新規：hybrid 3件 + momentum 5件 + volatility 6件 = 14件）
- 累計 realized PnL: **+400.88 JPY**（ペーパー）
- Runner稼働中（PID更新済み）

### 重要なバグ修正
`bollinger_upper`/`bollinger_lower`がATLASのbuilt-in feature名（`bb_upper`は誤り）。344から修正済み。

## 次にやること

1. **残り探索可能セルの確認**: GBP/JPY H4 trend_followingはすでに6 PASS。EUR/JPY H4 trend_followingは1 PASSのみ → 追加探索余地あり
2. **FTS同一セル3件上限の適用**: 現在5セルが3件超（EUR/JPY H4 breakout:6件など）。整理するか判断が必要
3. **OOS悪い戦略の精査**: 332（USD/JPY momentum OOS=-2.23）、342（GBP/JPY ADX OOS=-1.63）はFTSでのリスク要注意
4. **新規戦略タイプ探索**: M15 momentumの改善、H1 volatilityの他ペア展開
5. **Live投入候補**: 324（soft=0.8605）、346（soft=0.8835）、329（soft=0.8307）が最有力

## 関連文書・コマンド

- ループセッション: `ATLAS/logs/loop_session.json`（ATLAS-LOOP-2026-0505-001）
- 最新戦略ID: ATLAS-2026-0505-357（次は358）
- バックテスト: `cd /c/data/works/FX/ATLAS && .venv/Scripts/python.exe -m atlas.main backtest <ID>`
- FTS再起動: `cd /c/data/works/FX/fx_trading_system && PowerShell.exe -ExecutionPolicy Bypass -File scripts\start_unified_runner.ps1`

## 引継ぎ時の注意

- volatilityタイプ戦略でfeature登録は`"bollinger_upper"`（不変）
- pips_per_unit=100はJPYペアで必須
- 332/342はOOS Sharpe負 → FTSでは監視必要だがペーパーモードなので即時対応不要
- FTS同一系統3件上限: 今回適用済み（パラメータキーセット基準）。セル単位（inst×tf×type×dir）での整理は未実施
