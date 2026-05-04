# Handoff Note

**最終更新**: 2026-05-04T04:30:00Z @ training-goto
**ブランチ**: master (ATLAS / FTS とも master)
**直前のコミット**:
- ATLAS: `0397805` [change:spec] vectorbt L1 SHORT 評価バグ修正 (5.3.2 → 5.4.0)
- FTS: `ba034b6` [feat] ATLAS-070/071/072 FTS paper追加

## 現在の作業（1 行サマリ）

H1/H4 マトリクス探索 + 専門家チーム評価 + Phase 0 健全化 + Phase 1-(1) EUR/JPY H4 横展開 + Phase 1-(2) vectorbt SHORT 修正 = **本セッション 11 件 Gate PASS、SCHEMA 5.4.0 へ bump**

## 詳細コンテキスト（5 行）

本セッションでは以下を順次達成: ① 「EMA フィルター除去 + 純粋 Donchian + SL2x/TP4x」パターンで JPY ペア H1/H4 を 8 件 Gate PASS（最高 L2 Sharpe=4.08, USD/JPY H4 D3）。② 専門家 5 名並列レビューで devil-advocate の WFA バグ指摘が事実誤認と判明、FTS imported 36 件中 21 件が現行 Tier2 基準未達と発見。③ ユーザー判断（選択肢B）で L2 Sharpe<0 の 5 戦略を archive。④ EUR/JPY H4 データ取得 + Donchian D10/D15/D20 で 3 件追加 Gate PASS（070/071/072）。⑤ vectorbt L1 SHORT 評価バグの根本修正（METRICS_SCHEMA_VERSION 5.4.0）で SHORT 戦略の正しい L1 評価を実現、既存 long_only 戦略は bit-exact parity 維持。

## 未コミット変更 / WIP コミット対象

なし（ATLAS / FTS / 親リポジトリ全てコミット・プッシュ済み）

## セッション成果サマリー

### Gate PASS 戦略（本セッション 11 件追加）

| ID | display_name | TF | L2 Sharpe | soft | 備考 |
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

### 重要な発見・学習

1. **EMA フィルターの効きどころ**:
   - USD/JPY H1: EMA50>EMA100 が有効（012）
   - USD/JPY H4 / EUR/JPY H1 / EUR/JPY H4: **EMA なしが有効**
   - USD/JPY H4 はルックバック短いほど高 Sharpe (D3=4.08, D5=3.96, D8=3.61, D10=3.33)
   - EUR/JPY H1 はルックバック長いほど安定 (D20=soft 0.802 が最良、D5=0.705 ギリギリ)
   - EUR/JPY H4 は D10 が最良 (USD/JPY H4 と逆傾向)

2. **vectorbt L1 SHORT バグ修正効果**:
   - 修正前: SHORT 戦略の L1 評価が誤っていた (LONG として処理)
   - 修正後: 047 (旧 SHORT) は L1 PF=1.19 → 0.80 に変化、正しく FAIL 判定
   - 副次発見: EUR/USD H1 純粋 Donchian SHORT は edge なし (073 Sharpe=-0.27)

3. **専門家チーム評価のメタ知見**:
   - devil-advocate は数値だけ見て user_override や履歴を読まない傾向
   - WFA バグ指摘 (5名中 1 名) は事実誤認だったが、副産物として 21 件の旧基準 PASS 戦略混在が判明
   - 5 名並列レビューは効果的（コスト < 単発深掘り）

### Phase 0 健全化結果

- ATLAS-2026-0408-012/013, 0418-004, 0421-001, 0421-002 → `archived_2026-05-04/` へ git mv
- rescue_candidates から 069 (L2 PF=0.714) → `removed_candidates` へ移動
- FTS imported: 36 → 31 (archive 後) → **34** (070/071/072 追加後)

### METRICS_SCHEMA_VERSION

5.3.2 → **5.4.0**（vectorbt L1 SHORT 評価ロジック変更、既存 long_only は bit-exact 維持）

## 次にやること

### 短期（次セッション着手候補）

1. **SHORT 戦略の探索**: 修正後の vectorbt で正しく評価できる SHORT 戦略を見つける
   - 候補1: USD/JPY mean_reversion SHORT（過買い RSI 戻り）
   - 候補2: GBP/JPY データ取得 + 同パターン展開
   - 候補3: EUR/USD H4 SHORT (H4 ノイズ少ない、2022 強下落あり)
2. **rescue_candidates 救済**: 0430-005 (PF=3.08, trades=28) → アンサンブル設計

### 中期

3. **Per-Symbol/Direction Net Exposure Cap (P3)**: SHORT 戦略生成後に必要性確認
4. **pips_per_unit 通貨ペア自動推論** (`atlas/common/instrument_meta.py`): 非 JPY ペアの設定ミス防止
5. **Promotion Gate 形式化**: Paper → Live 昇格基準を `promotion_gate.yaml` 化
6. **Capacity Test**: 1→5→13→34→50 戦略のロードテスト

### 長期

7. **Kill Switch 3 階層化** + ロールバック Runbook
8. **Heartbeat + Prometheus**: 34件 Paper 戦略のヘルス監視

## 関連文書・コマンド

- 発見パターン: `memory/project_no_ema_donchian_discovery.md`
- 専門家チーム評価結果: 本セッション会話履歴
- spec_change_log.md エントリ: 2026-05-04 vectorbt L1 SHORT 評価バグ修正
- baseline parity: `ATLAS/logs/parity_baseline_5_3_2.json`
- archive 詳細: `fx_trading_system/trading_platform/strategies/archived_2026-05-04/README.md`
- 実行中ループ: なし

## 引継ぎ時の注意

- **vectorbt L1 SHORT 修正済み**: 5.4.0 以降、SHORT 戦略の L1 結果は信頼できる。それ以前のセッションで生成した SHORT 戦略 (例: 045-047) の L1 結果は誤情報なので再 BT 必須
- **bit-exact parity 維持**: 既存 long_only 戦略の L1/L2 数値は完全一致を確認済み (058/066/070 で実測)。再 BT で数値が変わったら実装バグ
- **EMA フィルターの効きどころ** (再掲): USD/JPY H1 のみ EMA 必要、それ以外の JPY pair H1/H4 は EMA なしが最適
- **pips_per_unit バグ**: 非 JPY ペアは `config.parameters` 内に `"pips_per_unit": 10000` 必須 (top-level に書いても無視される)
- **archive 復活**: `git mv archived_2026-05-04/{sid} imported/{sid}` で簡単に復元可能
- **devil-advocate 評価の限界**: 数値だけ見て履歴・user_override を読まない傾向。指摘は鵜呑みにせず必ず文脈確認
- **FTS Runner 停止中**: `unified_runner.jsonl` 最終 2026-04-10。再起動時は 34 件 paper を一斉稼働させる前にロードテスト推奨
- **次セッションで SHORT 探索する場合**: 修正後の L1 評価値を信頼できるが、L2 secondary period (2019-2022) で direction_bias チェックも忘れずに
