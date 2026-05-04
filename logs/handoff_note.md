# Handoff Note

**最終更新**: 2026-05-04T05:30:00Z @ training-goto
**ブランチ**: master (ATLAS / FTS とも master)
**直前のコミット**:
- ATLAS: `0397805` [change:spec] vectorbt L1 SHORT 評価バグ修正 (5.3.2 → 5.4.0)
- FTS: `725d1b1` [chore] hourly log snapshot (実装変更は `c70c09b` config.json コピー漏れ修正)

## 現在の作業（1 行サマリ）

11 件 Gate PASS + SCHEMA 5.4.0 bump + FTS Runner 再起動で **34 戦略 (Live 2 / Paper 32) Forward Test 稼働中**

## 詳細コンテキスト（5 行）

本セッションは多段階の成果: ① 「EMA なし純粋 Donchian + SL2x/TP4x」パターンで JPY ペア 8 件 Gate PASS（最高 L2 Sharpe=4.08）。② 専門家 5 名並列レビューで devil-advocate WFA バグ指摘が事実誤認、FTS imported 36 件中 21 件が現行 Tier2 基準未達と判明。③ Phase 0 健全化（5 戦略 archive + rescue 069 削除）。④ EUR/JPY H4 横展開で 3 件追加 Gate PASS（070-072）。⑤ vectorbt L1 SHORT 評価バグ根本修正（[change:spec] 5.4.0）。⑥ **FTS Live ポジション 2 件決済（+98 JPY 利益確定）→ Runner 全停止 → 再起動で 34 件 Forward Test 投入**。⑦ 058-066 の config.json コピー漏れを修正してコミット。

## 未コミット変更 / WIP コミット対象

なし（ATLAS / FTS / 親リポジトリ全てコミット・プッシュ済み）

## 現在の Runner 状態 (重要)

- **Runner PID 12080 で稼働中** (2026-05-04T13:58:48 起動、JST)
- discovery_complete: total=34 (Live 2 + Paper 32)
- 全 34 戦略 warmup 完了、main loop 起動中

### Live 戦略 (2 件、fixed_units=100, daily_loss_stop=-1000)

| sid | display | 累積 PnL |
|---|---|---|
| ATLAS-2026-0408-065 | USDJPY-MeanRev-Any-M15-016 | +103.10 JPY |
| ATLAS-2026-0417-003 | EURJPY-TrendFollow-Long-M15-001 | -28.50 JPY |

### Paper 戦略 (32 件、本日 13 件は新規 Forward Test 投入)

新規 Forward Test 投入 (本日 Gate PASS):
- 0504-058/060/063/065 (EUR/JPY H1, Donchian 10/20/15/5)
- 0504-061/062/064/066 (USD/JPY H4, Donchian 10/8/5/3)
- 0504-070/071/072 (EUR/JPY H4, Donchian 10/15/20)
- 0504-012/035 は既存 (待機状態から再 warmup)

既存 Forward Test 中 (19 件、bars=4〜340 で観察継続中)。OPEN ポジションあり: 0408-070, 0408-081, 0417-004, 0426-004, 0426-005

## OANDA Live Account 情報

- balance: **307,833.34 JPY** (本日 +98 JPY 利益確定)
- オープンポジション: 0 件
- account_id: 001-009-21110227-002 (live 環境)

## セッション成果サマリ (Gate PASS 11 件追加)

| ID | display | TF | L2 Sharpe | soft | 備考 |
|---|---|---|---|---|---|
| 058 | EURJPY-Breakout-Long-H1-004 | H1 | 2.295 | 0.749 | EMA なし Donchian10 |
| 060 | EURJPY-Breakout-Long-H1-005 | H1 | 2.087 | 0.802 | EMA なし Donchian20 |
| 061 | USDJPY-Breakout-Long-H4-002 | H4 | 3.334 | 0.864 | EMA なし Donchian10 |
| 062 | USDJPY-Breakout-Long-H4-003 | H4 | 3.611 | 0.863 | EMA なし Donchian8 |
| 063 | EURJPY-Breakout-Long-H1-006 | H1 | 2.045 | 0.745 | EMA なし Donchian15 |
| 064 | USDJPY-Breakout-Long-H4-004 | H4 | 3.960 | 0.878 | EMA なし Donchian5 |
| 065 | EURJPY-Breakout-Long-H1-007 | H1 | 2.180 | 0.705 | EMA なし Donchian5 |
| 066 | USDJPY-Breakout-Long-H4-005 | H4 | 4.080 | 0.879 | **本セッション最高 Sharpe** |
| 070 | EURJPY-Breakout-Long-H4-004 | H4 | 2.452 | 0.880 | EUR/JPY H4 横展開 D10 |
| 071 | EURJPY-Breakout-Long-H4-005 | H4 | 2.351 | 0.873 | EUR/JPY H4 横展開 D15 |
| 072 | EURJPY-Breakout-Long-H4-006 | H4 | 2.126 | 0.860 | EUR/JPY H4 横展開 D20 |

## 次にやること

### 短期（次セッション着手候補）

1. **24-48 時間後に `/fts-status` で Forward Test 状況確認** — 新規 13 戦略の bars / signals / fills 蓄積を観察
2. **SHORT 戦略の探索** — vectorbt 5.4.0 で正しく評価できるが edge 発見が必要
   - 候補1: USD/JPY mean_reversion SHORT（過買い RSI 戻り）
   - 候補2: GBP/JPY データ取得 + 同パターン展開
   - 候補3: EUR/USD H4 SHORT (H4 ノイズ少ない、2022 強下落)
3. **rescue_candidates 救済** — 0430-005 (PF=3.08, trades=28) → アンサンブル設計

### 中期

4. **Per-Symbol/Direction Net Exposure Cap (P3)** — SHORT 戦略生成後に必要性確認
5. **pips_per_unit 通貨ペア自動推論** — 非 JPY ペアの設定ミス防止
6. **Promotion Gate 形式化** — Paper → Live 昇格基準を `promotion_gate.yaml` 化
7. **Capacity Test** — 1→5→13→34→50 戦略のロードテスト

### 長期

8. **Kill Switch 3 階層化** + ロールバック Runbook
9. **Heartbeat + Prometheus** — 34 件 Paper 戦略のヘルス監視

## テストフロー (再確認)

```
① L1 (ATLAS BT)        — 過去IS期間で簡易スクリーニング
↓ PASS
② L2 (ATLAS BT)        — 過去IS+OOS全期間 + Secondary でGate判定
↓ Gate PASS
③ Paper Trading (FTS) = フォワードテスト  ← 本セッション現在地
   - リアルタイム市場で仮想口座取引
   - 推奨観察期間: 4 ヶ月以上
↓ Promotion Gate 通過
④ Live Trading (FTS)
   - fixed_units=100 (最小ロット) から開始
```

「フォワードテスト」と「Paperトレード」は同一概念（別名）。

## 関連文書・コマンド

- 発見パターン: `memory/project_no_ema_donchian_discovery.md`
- 専門家チーム評価結果: 本セッション会話履歴
- spec_change_log エントリ: 2026-05-04 vectorbt L1 SHORT 評価バグ修正
- baseline parity: `ATLAS/logs/parity_baseline_5_3_2.json`
- archive 詳細: `fx_trading_system/trading_platform/strategies/archived_2026-05-04/README.md`
- rescue 状態: `ATLAS/logs/rescue_candidates.json`
- Runner ログ: `fx_trading_system/logs/unified_runner.stdout.log`
- 実行中ループ: なし

## 引継ぎ時の注意

- **Runner 起動中 (PID=12080)**: 次セッションでも継続稼働するはず。停止確認は `tail logs/unified_runner.jsonl` で最新タイムスタンプ確認
- **Runner 停止が必要な場合**: `touch logs/unified_runner.stop` でグレースフル停止、または PowerShell `scripts/start_unified_runner.ps1` で再起動
- **vectorbt L1 SHORT 修正済み (5.4.0)**: SHORT 戦略の L1 結果は信頼できる。それ以前の SHORT 戦略 (045-047) の結果は誤情報
- **bit-exact parity 維持**: 既存 long_only の L1/L2 数値は完全一致を確認済み (058/066/070)
- **EMA フィルターの効きどころ**: USD/JPY H1 のみ EMA 必要、それ以外の JPY pair H1/H4 は EMA なしが最適
- **pips_per_unit バグ**: 非 JPY ペアは `config.parameters` 内に `"pips_per_unit": 10000` 必須
- **archive の復活**: `git mv archived_2026-05-04/{sid} imported/{sid}` + Runner 再起動で復活
- **devil-advocate 評価の限界**: 数値だけ見て履歴・user_override を読まない傾向。指摘は鵜呑みにせず必ず文脈確認
- **FTS imported 配置**: 戦略追加時は ATLAS から `config.json + metadata.json + strategy.py` を必ずコピー（gate_results.json/runner_config.json だけだと discovery で skip）
- **Runner discovery タイミング**: 起動時のみ実行 → imported 変更には Runner 再起動が必要 (動的 reload なし)
- **Live と Paper は同一プロセス**: UnifiedRunner で 1 プロセス管理、Paper のみ再起動は不可能
