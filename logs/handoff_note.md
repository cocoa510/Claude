# Handoff Note

**最終更新**: 2026-05-04T01:30:00Z @ training-goto
**ブランチ**: master (ATLAS / FTS とも master)
**直前のコミット**: FTS: 5476cce [ops] 013 archive、ATLAS: 7a4fb58 rescue_candidates 069 削除

## 現在の作業（1 行サマリ）

H1/H4マトリクス探索 8件 Gate PASS + 専門家チーム評価 + Phase 0 健全化（rescue 069 削除 + Forward Test 5件 archive）完了

## 詳細コンテキスト（3〜5 行）

本セッション前半で「EMA フィルター除去 + 純粋 Donchian + SL2x/TP4x」パターンで JPY ペア H1/H4 が **8 件 Gate PASS**（最高 L2 Sharpe=4.08, USD/JPY H4 Donchian3）。後半で専門家 5 名（quant-analyst / fx-strategist / devil-advocate / risk-execution-engineer / platform-architect）の並列レビューを実施。

devil-advocate の WFA バグ指摘は事実誤認（Tier2 計算は正常動作中）と確認したが、**FTS imported 36 件のうち 21 件 (58%) が現行 Tier2 基準を満たさず、19 件は 2026-04-26 の Cluster top-2 selection で意図的 paper 投入されていた**ことが判明。

ユーザー判断（選択肢B）により、**L2 Sharpe<0 の 5 件 (012/013/0418-004/0421-001/0421-002) を archive**し、本日生成の高 Sharpe 戦略 8 件で枠を実質置換。013 は paper 降格済みかつ open position なしを確認の上 archive。

rescue_candidates から ATLAS-2026-0503-069 (L2 PF=0.714) を `removed_candidates` に移動し、filter_criteria に L2 PF>=1.0 必須を追記。

## 未コミット変更 / WIP コミット対象

なし（ATLAS / FTS / 親リポジトリ全てコミット・プッシュ済み）

## 現在の FTS ポートフォリオ状態

- imported: **31 件**（36 → 5 archive 後）、全 paper、Runner 停止中
- archive: 5 件（`trading_platform/strategies/archived_2026-05-04/`）
- 現行 Tier2 PASS の高品質戦略: 15 件（うち本日新規 8 件含む）
- 残 16 件は user_override 付き Forward Test (2026-04-26 投入、4ヶ月予定)

## 次にやること

### Phase 1（戦略多様性の構造的解決、優先順）

1. **EUR/JPY H4 データ取得**: `atlas data fetch EUR_JPY --timeframe H4 --years 7` → H4 no-EMA Donchian を 4 通貨ペア展開
2. **GBP/JPY / AUD/JPY データ取得**: 同パターン横展開で portfolio 通貨分散
3. **vectorbt L1 SHORT 評価バグ修正**: `[change:spec]` 手順で `_run_vectorbt_portfolio` を `entries`/`short_entries` 両渡しに改修。過去戦略再BTあり（影響範囲: parity test 全件）
4. **pips_per_unit 通貨ペア自動推論**: `atlas/common/instrument_meta.py::infer_pips_per_unit()` 新設（JPY=100、他=10000）

### Phase 2（Live 移行前の P0 課題）

5. **Per-Symbol/Per-Direction Net Exposure Cap**: PTRC 拡張で portfolio 集中検出
6. **Signal Coalescing**: 同方向クラスタ束ね注文（21戦略×2pips=42pips の自己悪化スリッページ防止）
7. **Capacity Test**: 1→5→13→31→50戦略のロードテスト
8. **Promotion Gate 形式化**: `promotion_gate.yaml`（Forward 60営業日 + 最低取引数 + Sharpe劣化≤30%）
9. **Kill Switch 3階層化** + ロールバック Runbook

### Phase 3（統計的堅牢性、並行進行）

10. 円高反転局面 isolated 検証（2022/10-12 USD/JPY 152→145 のみで PF 計算）
11. Monte Carlo trade-order shuffle + block bootstrap (block=20)
12. Portfolio correlation-adjusted DD シミュレーション（90%ile worst case が 30% 以内か）
13. コスト織込検証（swap、slippage、commission の織込確認）

## 関連文書・コマンド

- 発見パターン: `memory/project_no_ema_donchian_discovery.md`
- 専門家チーム評価結果: 本セッション会話履歴（要保存検討）
- 新規 Gate PASS: ATLAS-2026-0504-058〜066
- archive 詳細: `fx_trading_system/trading_platform/strategies/archived_2026-05-04/README.md`
- rescue 状態: `ATLAS/logs/rescue_candidates.json`
- 実行中ループ: なし

## 引継ぎ時の注意

- **pips_per_unit バグ**: 非JPYペアは `config.parameters` 内に `"pips_per_unit": 10000` 必須。top-level に書いても無視される（vectorbt_engine.py が `config.parameters.get(...)` で読むため）
- **EMA フィルターの効きどころ**:
  - USD/JPY H1: EMA50>EMA100 が有効（012）
  - EUR/JPY H1 / USD/JPY H4: EMA なしが有効（058〜066）
- **vectorbt L1 SHORT 制限**: SHORT エントリーを LONG として評価。SHORT 専用戦略の L1 結果は鵜呑みにせず、L2 で必ず確認すること
- **devil-advocate 評価の限界**: 数値だけ見て user_override や履歴を読まない傾向あり。指摘は鵜呑みにせず必ず文脈確認
- **archive の復活手順**: `git mv` で `archived_2026-05-04/` から `imported/` に戻すだけ
- **専門家チーム並行評価**: 5名並列起動が有効（quant-analyst, fx-strategist, devil-advocate, risk-execution-engineer, platform-architect）
