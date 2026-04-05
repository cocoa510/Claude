# ATLAS 環境診断・トラブルシューティング

ATLASの実行環境、依存関係、データ整合性を診断し、問題があれば修正方法を提案してください。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- 引数なし — 全項目の診断を実行
- `--env` — 環境変数・APIキーの確認のみ
- `--deps` — 依存ライブラリの確認のみ
- `--db` — History Store（SQLite）の整合性チェック
- `--fix` — 自動修正可能な問題を修正

## 診断項目

### 1. Python環境
```bash
cd ATLAS && python --version && pip list --format=columns | grep -E "anthropic|pydantic|vectorbt|pandas|numpy|aiosqlite|structlog"
```
- Python 3.11+ であること
- 必須ライブラリのバージョン確認
- 仮想環境の有効性確認

### 2. 環境変数
```bash
cd ATLAS && python -c "
from atlas.config.settings import get_settings
s = get_settings()
print('ANTHROPIC_API_KEY:', 'SET' if s.anthropic_api_key else 'MISSING')
print('OANDA_API_TOKEN:', 'SET' if s.oanda_api_token else 'MISSING')
print('OANDA_ACCOUNT_ID:', 'SET' if s.oanda_account_id else 'MISSING')
"
```

### 3. ディレクトリ構成
必要なディレクトリとファイルの存在確認。欠損があれば報告。

### 4. History Store整合性
```bash
cd ATLAS && python -m atlas.history.queries health_check
```
- SQLiteデータベースの接続確認
- テーブル構造の検証
- 孤立レコードの検出
- データ一貫性チェック

### 5. 市場データ
- `data/market/` のデータ存在確認
- 品質スコアの確認

## 出力フォーマット
```
=== ATLAS 環境診断 ===

[OK] Python環境: 3.11.5
[OK] 依存ライブラリ: 全11パッケージ確認済み
[OK] ANTHROPIC_API_KEY: 設定済み
[NG] OANDA_API_TOKEN: 未設定
  → .env に OANDA_API_TOKEN を設定してください
[OK] ディレクトリ構成: 正常
[OK] History Store: 正常（24世代記録済み）
[WARN] 市場データ: GBPUSD の2023-06以前が不足
  → /atlas-data fetch GBPUSD 2022-01:2023-05

診断結果: 1件のエラー、1件の警告
```

## エラーハンドリング
- 診断自体が失敗した場合、失敗した項目と原因を表示
- `--fix` モードでの修正前に必ず確認を求める

## 注意事項
- APIキーの値自体は表示しない（設定済み/未設定のみ）
- `--fix` は安全な修正のみ実行（ディレクトリ作成、DB初期化等）
