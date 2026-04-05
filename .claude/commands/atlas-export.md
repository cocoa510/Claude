# ATLAS 戦略エクスポート

戦略を完全再現可能な形式でエクスポートしてください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — エクスポート対象の世代ID
- 引数なし — 最高スコアの優良戦略を自動選択

## 実行手順

### 1. エクスポート対象の確認
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history get <generation_id>
```
- 対象世代が「優良」判定（final_score >= 0.75）であることを確認
- 非優良の場合、警告を表示しユーザーに確認を求める

### 2. 既存ファイルの確認

`strategies/<generation_id>/` ディレクトリの内容を確認:
- `strategy.py` — 戦略コード
- `config.json` — パラメータ設定
- `spec.md` — 戦略仕様書
- `backtest/result.json` — バックテスト結果
- `evaluation/` — 評価レポート

### 3. エクスポートパッケージ生成

`export/<generation_id>/` に以下を生成:

```
export/<generation_id>/
├── README.md                    # 戦略概要・使用方法
├── strategy.py                  # 戦略コード
├── config.json                  # 全パラメータ
├── spec.md                      # 戦略仕様書（完全再現用）
├── reports/
│   ├── backtest_result.json     # バックテスト結果
│   ├── evaluation_report.json   # 26指標 + スコア
│   └── score_history.json       # スコア推移
├── human_readable/
│   └── trading_rules.md         # FX精通者向け完全ルール記述
└── reproduction/
    ├── backtest_config.json     # 再現用設定
    └── expected_metrics.json    # 期待メトリクス（検証用）
```

### 4. trading_rules.md の生成（Claude Codeが直接作成）

戦略コードと仕様書を読み、以下を自然言語で記述:
- エントリー条件: ロング/ショート別、全条件を具体的に
- エグジット条件: 利確/損切り/タイムアウトの条件
- フィルター条件: トレードしない状況
- リスク管理: ポジションサイジングルール
- 使用インジケータ: 計算式とパラメータ値
- 前提条件: 有効な市場環境、通貨ペア、時間足
- 注意事項: 既知の弱点、不利な市場環境

> **目的:** システム消失時でもFX経験者が手動で再現可能なレベル

### 5. 出力フォーマット
```
=== ATLAS エクスポート完了 ===
世代ID: <generation_id>
final_score: X.XX
出力先: export/<generation_id>/

生成ファイル:
  - strategy.py (戦略コード)
  - config.json (パラメータ設定)
  - spec.md (戦略仕様書)
  - reports/ (評価レポート)
  - human_readable/trading_rules.md (自然言語ルール)
  - reproduction/ (再現用ファイル)

戦略要約:
  ペア: USD_JPY | 時間足: H1
  PF: X.XX | Sharpe: X.XX | MaxDD: XX.X%
  エントリー: [要約]
  エグジット: [要約]
```

## エラーハンドリング
- バックテスト/評価が未実施 → 不足ステップを案内
- 仕様書が未作成 → 戦略コードから自動生成を提案

## 注意事項
- エクスポートは「システム消失時でも完全再現可能」が最優先
- human_readable はFX精通者が手動再現できるレベルの詳細さ
- 秘密情報（APIキー等）はエクスポートに含めない
