# Handoff Note

**最終更新**: 2026-05-04T08:00:00Z @ training-goto
**ブランチ**: master (ATLAS / FTS とも master)

## 現在の作業（1 行サマリ）

SHORT 戦略開発ループ: 12 戦略生成・全 Gate PASS 失敗、ただし既存 ATLAS-2026-0424-001 が vectorbt 5.4.0 で **Tier 1 ALL PASS** 判明（FTS Paper 投入候補）

## 詳細コンテキスト（5 行）

ユーザー指示「許可確認なしで戦略開発ループを回し続ける」を受け、SHORT/balanced 戦略を 12 件生成（ATLAS-2026-0504-073〜084）。USD/JPY/EUR/USD/GBP/JPY/GBP/USD/EUR/JPY を MR/Breakout/balanced で網羅的に試行。すべて overall_passed=false。同セッションで既存 SHORT/balanced 戦略を vectorbt 5.4.0 で 8 件再評価し、**ATLAS-2026-0424-001 (GBPUSD-MeanRev-Short-M15-001) が Tier 1 全 8 条件 PASS** することを発見（B&H 超過 +0.65%、Secondary PF=0.94、L2 PF=1.01、SHORT 696 trades）。Tier 2 soft_score=0.35 < 0.70 で overall=false だが、Tier 1 真の Immutable は満たす。本セッションで Gate PASS 達成戦略は 0 件、ただし重要な構造的洞察を多数獲得。

## 未コミット変更 / WIP コミット対象

なし（12 戦略生成 + 5 件 5.4.0 再評価結果すべてコミット済み）

## SHORT 戦略開発の構造的結論

| 失敗パターン | 件数 | 原因 |
|---|---|---|
| L1 Sharpe FAIL | 7 | 073, 074, 076, 077, 080, 081, 082 |
| L1 trade 数不足 (<20) | 3 | 078 (制約過剰), 079 (同), 083 (H4 noise) |
| L1 PASS / L2 過学習 | 1 | 075 (IS PF=1.19 → OOS PF=0.88) |
| L2 PASS だが Tier 1 #1/#4 | 1 | 084 (PF<1 + B&H excess<0) |

主要構造的要因:
1. **2019-2026 JPY weakness era**: USD/JPY 140→156, GBP/JPY 130→200 → SHORT 構造的逆風
2. **EUR/USD sideways 2023-26**: Donchian SHORT/Breakout SHORT は fade される
3. **vectorbt 5.4.0 SHORT bug fix**: 既存 SHORT 戦略の評価値を下方修正（過去 false positive 解消）
4. **Tier 2 soft_score 上限**: PF<1.5 / Sharpe<1.5 だと自動的に soft<0.4 で SHORT には厳しい

## 唯一の Tier 1 ALL PASS SHORT 戦略

**ATLAS-2026-0424-001 (GBPUSD-MeanRev-Short-M15-001)** — 重要発見

| Tier 1 条件 | 値 | 閾値 | 判定 |
|---|---|---|---|
| Profit Factor | 1.0088 | >=1.0 | ✓ |
| Max Drawdown % | 12.10 | <=30 | ✓ |
| Total Trades | 696 | >=50 | ✓ |
| **B&H Excess Return %** | **+0.65** | >=0 | ✓ |
| Secondary Period PF | 0.9436 | >=0.8 | ✓ |
| Direction Bias | LONG=0 SHORT=696 | short_only | ✓ |
| RiskGuard Halt/yr | 0.0 | <=3.48 | ✓ |
| KillSwitch Trigger | 0 | ==0 | ✓ |

**Tier 2: soft_score=0.3521 < 0.70 で overall=false** だが、Tier 1（真の Immutable）は完全充足。

戦略仕様:
- パラメータ: bb_std=1.5, rsi_overbought=70, atr_sl_multiplier=1.5, tp_rr_ratio=2.0
- session: UTC 7-20 時 (London + NY)
- エントリ: RSI(14)>70 AND close>BB_upper(20, 1.5)
- エグジット: RSI<50 OR SL=1.5×ATR OR TP=SL×2.0

## 改良試行の結果

0424-001 から 081/082 を fork して Tier 2 強化試行:
- **081** (bb_std=2.0, rsi=73, ADX<25 厳格化): WR 62.7→34.6% に激減、L1 FAIL
- **082** (RR=1.5, early RSI exit): WR 62.7→48.1%、Sharpe 0.39→0.19 に低下

**結論**: 0424-001 のパラメータは既に sweet spot で、改良は逆効果。Tier 2 突破は構造的に困難。

## 次にやること

### 即実施候補（次セッション）

1. **ATLAS-2026-0424-001 の FTS Paper 投入判断**:
   - Tier 1 ALL PASS = 真の Immutable 条件は満たす
   - Tier 2 0.35 = 旧 deploy (0410-008 soft=0.49, 0426-013 soft=0.13 等) と同水準
   - 方向性集中リスク (FTS 34件全 long_only) を緩和する数少ない選択肢
   - **判断はユーザーに委ねる**: overall=false なので autonomous 投入は避けた

2. **追加の SHORT 探索 (代替アプローチ)**:
   - GBP/JPY M15 MR SHORT（M15 + JPY pair で MR、未試行）
   - EUR/USD H1 Donchian SHORT（077 は MR、Donchian は未試行）
   - 多時間足 (MTF) trend filter 追加（H4 trend 下抜けで SHORT 限定）
   - session-bound SHORT（NY セッション only など）

3. **rescue_candidates 救済 (handoff_state next_actions)**:
   - 0430-005 (PF=3.08, trades=28) アンサンブル化

### 中期

4. WFA Efficiency / Strategy Drift が null になる原因調査（多くの戦略で null）
5. Per-Symbol/Direction Net Exposure Cap (P3)
6. Promotion Gate 形式化（Paper → Live 昇格基準）

## 関連文書・コマンド

- **重要発見**: ATLAS-2026-0424-001 5.4.0 再評価結果 → `ATLAS/strategies/ATLAS-2026-0424-001/backtest/result.json`
- セッション内全戦略コミット: 073-084 各戦略の `[atlas] ATLAS-2026-0504-XXX ...` コミット
- spec_change_log: 2026-05-04 vectorbt L1 SHORT 評価バグ修正 (5.3.2→5.4.0)
- 実行中ループ: なし（手動 1 戦略ずつ生成）

## Runner 状態

前セッションから継続: PID 12080 で稼働中、Live 2 + Paper 32 = 34 戦略 Forward Test 中。本セッションでは Runner に変更なし。

## OANDA Live Account（変更なし）
- balance: 307,833.34 JPY
- Open positions: 0 件

## 引継ぎ時の注意

- **0424-001 投入は要ユーザー判断**: Tier 1 ALL PASS だが overall=false。autonomous 投入は CLAUDE.md「確認無し連続運用」の `Gate PASS した戦略は FTS ペーパートレードに即投入する` に厳密には該当しない（`Gate PASS = overall_passed=true`）
- **既知バグ集**: 
  - インジケータ名 `bb_upper` (誤) → `bollinger_upper` (正、builtin map 必須)
  - `hasattr()` 禁止 → `bar.get("timestamp", None)` パターン
  - `pips_per_unit=10000` (非 JPY) / `=100` (JPY) を `config.parameters` に必須
- **fx-strategist の傾向**: AND 厳格化と OR 緩和の両方に振れる。明示的に「sweet spot を維持」と指示しないと逆方向に動くことあり
- **vectorbt 5.4.0 SHORT 真値**: 過去 SHORT 戦略の Tier 1 値は信頼できる（pre-5.4.0 は false positive 含む）
