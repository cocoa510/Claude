# Handoff Note

**最終更新**: 2026-04-22T09:59:10Z @ training-goto
**ブランチ**: master
**直前のコミット**: 3235589 [docs] fx-strategist: on_fill を optional として記述（Phase 1b Step 7 反映）

## 現在の作業（1 行サマリ）

アンサンブル運用方針（FTS live + Gate 調整 + 3 通貨ペア限定）を 4 専門家並列検証完了、ユーザーに Next Step Option A/B/C を提示中。

## 詳細コンテキスト（3〜5 行）

既存 83 戦略に FTS live 実績がゼロであり、Gen41/Gen65 をローカル特化アンサンブル（uptrend×SHORT、range×LONG 等）として運用する方針がユーザーから提示された。7 段階方法論（Phase 2 Gate 階層化 → Phase 3 新アーキ移動 → FTS live 投入 → Gate 調整 → 3 ペア運用 → strategy_tags 分類基盤 → 高頻度低リスク初期設定）について、fx-strategist / quant-analyst / platform-architect / devil-advocate の 4 名を CLAUDE.md「新規 Phase 提案プロトコル」5 項目遵守で並列相談し、条件付き YES の合意を得た。合意済 Phase 順序は 2a → 3 → 3.6 → 2b → 3.5-P/D → 3.7 → Phase 4 → Phase 5（9-10 週、推定 34 人日）。strategy_tags（MF-2）前倒しが 3 名合意、Gate #14 Secondary PF≥1.0 緩和禁忌が quant 指摘。

## 未コミット変更 / WIP コミット対象

- `logs/handoff_note.md` (new): このハンドオフノート本体
- `logs/handoff_state.json` (new): 機械可読スナップショット
- `.claude/commands/handoff.md` (untracked): handoff スキル本体（必要に応じ別途コミット検討）
- `.claude/scheduled_tasks.lock` (untracked): ロックファイル、コミット対象外

## 次にやること

1. ユーザーが Option A/B/C のいずれを選ぶか決定待ち:
   - **(A) 最小 BLOCK 解消先行** — ①システム完成の operational 定義合意 → ②Gen41 OANDA 実コスト検証（net PF>1.05）→ ③高頻度 Gate 通過率シム（合計 2-3 日）、その後 Phase 2a 着手
   - **(B) Phase 順序のみ確定して即着手** — 2a から着手、BLOCK は並行解消
   - **(C) strategy_tags (MF-2) 最優先** — 83 戦略 backfill を先にして多様性実態を可視化
2. 選択された Option に従い着手:
   - A 選択時: BLOCK-1「システム完成定義」合意シート作成 → OANDA 実コスト計測スクリプト → 高頻度 Gate シム
   - B 選択時: Phase 2a（Gate Tier 1/2/3 階層化）に即着手
   - C 選択時: `scripts/backfill_strategy_tags.py` 設計 + StrategyTags Pydantic モデル platform-architect 起動
3. Phase 順序確定後は 2a → 3 → 3.6 → 2b → 3.5-P/D → 3.7 → Phase 4 → Phase 5 を 9-10 週で進行

## 関連文書・コマンド

- 参照: `ATLAS/CLAUDE.md`（Gate 14 条件正規定義・新規 Phase 提案プロトコル）
- 参照: `ATLAS/docs/Redesign_v2_Plan.md`（Phase 分割計画）
- 参照: `ATLAS/docs/spec_change_log.md`（直近 5 エントリの判断記録）
- 参照: Gen41 params（rsi_oversold=28, bb_std=2.0, adx_max=22, atr_sl_multiplier=2.0, position_size_ratio=0.01）
- FTS live 投入時コマンド例: `python fx_trading_system/scripts/paper_add.py --strategy-id ATLAS-2026-0408-041`
- 実行中ループ: なし（`.claude/scheduled_tasks.lock` 存在するが loop_session.json の current_step=未更新）

## 引継ぎ時の注意

- **4 専門家合意済の must_fix 4 件**（着手前に優先対処推奨）:
  1. 「システム完成」の operational 定義を文書化合意（devil B-1）
  2. Gen41 の OANDA 実コスト計測で net PF>1.05 の事前検証（fx + quant）
  3. FTS live 投入目的を「spread/slippage キャリブレーション限定」に明記、収益検証ではない（devil B-1）
  4. 高頻度 Gate（trades/year≥80, MaxDD≤8%）の 83 戦略通過率シミュレーション（fx + quant）
- **Gate #14 Secondary PF≥1.0 は緩和禁忌**（quant 厳格指摘、Period Bias 対策の中核）
- **strategy_tags 前倒し必須**（fx + platform + quant 合意、MF-2 として Phase 3.6 から最前に繰上げ候補）
- **高頻度低リスク初期設定の Exp D 再発懸念**（devil 指摘、forbidden_patterns 注入下 abandon_rate=1.0 の再現を警戒）
- **G-3 ガードレール（2 週上限）遵守**: 各 Phase で G-3 違反時は即停止
- **CLUSTER_CAP_PER_GROUP=3**: 83 戦略中 42% が MeanReversionRSIBollingerLongOnlyStrategy 同族のため Phase 3.6 backfill で overflow 発生予想
- **ATLAS_CHAMPIONS**: Gen41（USD_JPY M15 range LONG, L1 PF=1.2388, Secondary PF=0.5866）と Gen65（Gen41 派生, atr_sl_multiplier 1.8）のみ FTS live 候補。Gen66-87 は過剰最適化で Fwd 劣後済（MEMORY 参照）
