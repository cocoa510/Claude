# ATLAS 市場データ管理

バックテスト用の市場データの取得・管理・品質チェックを行ってください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `fetch <通貨ペア>` — データ取得（例: `fetch USD_JPY`）
- `fetch <通貨ペア> --years <N>` — 取得年数を指定（デフォルト: 3年）
- `fetch <通貨ペア> --timeframe <TF>` — 時間足を指定（デフォルト: H1）
- `status` — 保有データの一覧と品質サマリー
- `check <通貨ペア>` — データ品質チェック（欠損、異常値検出）
- `check <通貨ペア> --timeframe <TF>` — 特定時間足の品質チェック
- 引数なし — 保有データのサマリーを表示

## 実行手順

### fetch（データ取得）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main data fetch <instrument> --years <N> --timeframe <TF>
```
- OANDA v20 APIから指定期間のOHLCVデータを取得
- Parquet形式で `data/market/<instrument>_<timeframe>.parquet` に保存
- メタデータJSON `data/market/<instrument>_<timeframe>_meta.json` も生成
- 品質チェックも自動実行

出力JSONの主要フィールド:
- `status`: "success" / エラー時はトレースバック
- `total_bars`: 取得バー数
- `date_range`: 開始〜終了日時
- `quality`: 品質チェック結果

### status（保有データ一覧）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main data status
```
出力例:
```
=== 市場データステータス ===
通貨ペア   | 時間足 | バー数   | 期間                      | サイズ   | 品質
USD_JPY    | H1     | 18,720  | 2023-01-02 〜 2025-12-31  | 1.2 MB  | 0.95
EUR_USD    | H1     | 15,600  | 2023-06-01 〜 2025-12-31  | 980 KB  | 0.92
```

### check（品質チェック）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main data check <instrument> --timeframe <TF>
```
チェック内容:
- 欠損バーの検出と割合（週末ギャップは除外）
- 異常値（価格0、High<Low、NaN）
- 連続欠損区間（ギャップ）の特定
- 品質スコア算出（0.80以上で合格）

## 出力フォーマット

結果表示:
```
=== データ品質チェック ===
通貨ペア: USD_JPY (H1)
全バー数: 18,720
期間: 2023-01-02 00:00 〜 2025-12-31 23:00
欠損バー: 42 (0.22%)
ギャップ: 3箇所
品質スコア: 0.9778 [PASS]
警告: なし
```

## エラーハンドリング
- OANDA_API_KEY 未設定 → `.env` ファイルへの設定方法を案内
- OANDA_ACCOUNT_ID 未設定 → 同上
- 認証エラー → APIキーの有効性確認を促す
- ネットワークエラー → リトライを提案
- データなし → 通貨ペア名が正しいか確認（OANDA形式: `USD_JPY`）

## 注意事項
- 通貨ペア名はOANDA形式（`USD_JPY`, `EUR_USD` 等）
- 対応時間足: M1, M5, M15, M30, H1, H4, D
- `data/market/` は `.gitignore` 対象（サイズが大きいため）
- 品質スコア < 0.80 のデータはバックテスト時に拒否される
- 大量データ取得は時間がかかるため、最初は `--years 1` で動作確認推奨
