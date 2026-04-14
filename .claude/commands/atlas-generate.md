# ATLAS 戦略コード生成

新しいFXトレード戦略のコードを専門家エージェント（fx-strategist）に委譲して自動生成します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<戦略タイプ> <通貨ペア> [オプション]`
- 例: `trend_following USDJPY --timeframe 1H --variants 3`
- 例: `mean_reversion EURUSD GBPUSD --timeframe 4H`
- 例: `breakout USDJPY --temperature 0.8`
- 引数なし — 対話的に戦略タイプと通貨ペアを質問

### 戦略タイプ（strategy_type）
- `trend_following` — トレンドフォロー系
- `mean_reversion` — 平均回帰系
- `breakout` — ブレイクアウト系
- `momentum` — モメンタム系
- `volatility` — ボラティリティ系
- `hybrid` — 複合型（上記の組み合わせ）

### オプション
- `--timeframe <TF>` — 時間足（M1, M5, M15, H1, H4, D）。デフォルト: H1
- `--variants <N>` — 同時生成バリアント数（1-5）。デフォルト: 1
- `--parent <generation_id>` — 親戦略ID（改良ベース指定時）
- `--direction <bias>` — 方向制約（`any` / `long_only` / `short_only` / `balanced`）。デフォルト: `any`。Period Bias対策のSHORT強制生成に使用

## 実行手順

### 1. 設定確認と世代ID採番
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history next-id
```
設定読み込みに失敗した場合、`/atlas-init` の実行を促す。

### 2. 戦略ディレクトリの準備
```bash
mkdir -p ATLAS/strategies/<generation_id>
```

### 3. コンテキスト取得（改良時のみ）
親戦略が指定されている場合、改良用コンテキストを取得:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main context <parent_id> --mode improve
```

### 4. 生成コンテキスト取得（方向制約付き）

```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main context generate \
  --instrument <通貨ペア> --timeframe <時間足> \
  --strategy-type <戦略タイプ> --direction <direction_bias>
```

出力JSONには `direction_bias` と `direction_constraint.instruction` が含まれる。
`short_only` 指定時は、fx-strategist が LONG シグナルを返すコードを生成した場合、
そのまま不合格（Gate 以前にインターフェース違反として再生成対象）。

### 5. fx-strategist エージェントに生成を委譲

Agent tool で `fx-strategist` エージェントを起動し、以下のタスクを渡す:

> 以下の条件でFXトレード戦略を生成してください。
>
> - 戦略ID: `<generation_id>`
> - 戦略タイプ: `<戦略タイプ>`
> - 通貨ペア: `<通貨ペア>`
> - 時間足: `<時間足>`
> - 方向制約: `<direction_bias>` — [direction_constraint.instruction をそのまま貼付]
> - 出力先: `ATLAS/strategies/<generation_id>/`
>
> 以下のファイルを生成すること:
> 1. `strategy.py` — Strategy基底クラスを継承した戦略コード
> 2. `spec.md` — 戦略仕様書（手動再現可能なレベル）
> 3. `config.json` — StrategyConfigパラメータ
> 4. `metadata.json` — 生成メタデータ（`direction_bias` フィールドを含めること）
>
> ATLAS/CLAUDE.md と ATLAS/atlas/common/models.py を読んでインターフェースに準拠すること。
> 方向制約は不可侵の最優先制約であり、違反したコードは即座に破棄される。

### 5. 生成結果の記録
History Store に生成レコードを保存:
```bash
cd ATLAS && .venv/Scripts/python.exe -c "
import asyncio, json
from atlas.history.store import HistoryStore
from atlas.history.models import GenerationRecord, GenerationStatus

async def save():
    async with HistoryStore('data/atlas_history.db') as store:
        rec = GenerationRecord(
            strategy_id='<generation_id>',
            generation=0,
            instrument='<通貨ペア>',
            timeframe='<時間足>',
            status=GenerationStatus.VALIDATING,
            strategy_code=open('strategies/<generation_id>/strategy.py').read(),
        )
        await store.save(rec)

asyncio.run(save())
"
```

### 6. 出力フォーマット
```
=== ATLAS 戦略生成完了 ===
世代ID: <generation_id>
戦略タイプ: <戦略タイプ>
通貨ペア: <通貨ペア>
時間足: <時間足>

生成ファイル:
  - strategies/<generation_id>/strategy.py
  - strategies/<generation_id>/spec.md
  - strategies/<generation_id>/config.json
  - strategies/<generation_id>/metadata.json

戦略概要:
  [spec.mdから要約を表示]

次のステップ:
  /atlas-validate <generation_id>  — 安全性検証
```

## エラーハンドリング
- fx-strategist エージェントが生成したコードがインターフェース不適合の場合、再生成を指示（最大3回）
- 全リトライ失敗時は、生成されたコードとエラー内容をユーザーに提示
- ディレクトリ作成に失敗した場合は `/atlas-doctor` の実行を促す
