# ATLAS プロジェクト初期化

ATLASプロジェクトの初期セットアップを実行してください。

## 引数
`$ARGUMENTS` — オプション。以下の形式で指定可能:
- `--force` — 既存ディレクトリがある場合も上書き再初期化
- 引数なし — 通常の初期化

## 実行手順

### 1. ディレクトリ構成の作成
`ATLAS/` 配下に以下のディレクトリ構成を作成してください:

```
ATLAS/
├── atlas/
│   ├── __init__.py
│   ├── main.py
│   ├── orchestrator/
│   │   ├── __init__.py
│   │   ├── loop_controller.py
│   │   ├── convergence.py
│   │   └── state.py
│   ├── generator/
│   │   ├── __init__.py
│   │   ├── strategy_generator.py
│   │   ├── prompt_templates.py
│   │   └── code_formatter.py
│   ├── validator/
│   │   ├── __init__.py
│   │   ├── syntax_checker.py
│   │   ├── interface_checker.py
│   │   └── sandbox_runner.py
│   ├── backtest/
│   │   ├── __init__.py
│   │   ├── runner.py
│   │   ├── data_loader.py
│   │   ├── vectorbt_engine.py
│   │   ├── event_simulator.py
│   │   └── result_parser.py
│   ├── analyzer/
│   │   ├── __init__.py
│   │   ├── metrics.py
│   │   ├── scorer.py
│   │   ├── weakness_detector.py
│   │   ├── report_builder.py
│   │   └── comparator.py
│   ├── evaluator/
│   │   ├── __init__.py
│   │   ├── strategy_evaluator.py
│   │   └── prompt_templates.py
│   ├── feedback/
│   │   ├── __init__.py
│   │   ├── composer.py
│   │   └── constraints.py
│   ├── history/
│   │   ├── __init__.py
│   │   ├── store.py
│   │   ├── models.py
│   │   └── queries.py
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py
│   │   └── defaults.py
│   └── common/
│       ├── __init__.py
│       ├── logger.py
│       ├── claude_client.py
│       └── exceptions.py
├── data/
│   ├── market/
│   └── generations/
├── tests/
│   ├── __init__.py
│   ├── unit/
│   │   └── __init__.py
│   ├── integration/
│   │   └── __init__.py
│   └── fixtures/
└── docs/
```

### 2. 設定ファイルの生成
以下のファイルを生成してください:

- **`ATLAS/pyproject.toml`** — プロジェクトメタデータ、依存関係（anthropic, pydantic, pydantic-settings, vectorbt, pandas, numpy, aiosqlite, structlog, pytest, pytest-asyncio, python-dotenv）
- **`ATLAS/.env.example`** — 環境変数テンプレート（ANTHROPIC_API_KEY, OANDA_API_TOKEN, OANDA_ACCOUNT_ID, LOG_LEVEL等）
- **`ATLAS/.gitignore`** — .env, __pycache__, .venv, data/market/, *.pyc 等
- **`ATLAS/CLAUDE.md`** — ATLAS固有の開発ルール（ImplementationPlan.mdの内容を参照して、戦略インターフェース規約、Backtest Gate基準等を記載）

### 3. 基底モデルの初期コード
以下のスケルトンコードを生成してください:
- `atlas/config/settings.py` — Pydantic Settingsによる設定クラス
- `atlas/config/defaults.py` — Backtest Gate基準値等の定数定義
- `atlas/common/exceptions.py` — カスタム例外クラス群
- `atlas/common/logger.py` — structlogベースのロガー設定
- `atlas/history/models.py` — Pydanticデータモデル（StrategyConfig, TradeSignal, GenerationRecord等）

### 4. 確認出力
初期化完了後、以下を表示してください:
```
=== ATLAS 初期化完了 ===
作成ディレクトリ数: XX
作成ファイル数: XX
次のステップ:
  1. ATLAS/.env を作成し、APIキーを設定
  2. cd ATLAS && pip install -e .
  3. /atlas-generate で戦略を生成
```

## エラーハンドリング
- `ATLAS/` ディレクトリが既に存在し `--force` がない場合、ユーザーに確認を求める
- ファイル書き込みに失敗した場合、エラー内容を表示して中断
- 既存ファイルを上書きする前に必ず確認する

## 注意事項
- `ATLAS/ImplementationPlan.md` の仕様に厳密に従うこと
- `.env` ファイル自体は生成しない（`.env.example` のみ）
- 全てのPythonファイルには `from __future__ import annotations` を含める
- 型ヒントを必ず付ける
