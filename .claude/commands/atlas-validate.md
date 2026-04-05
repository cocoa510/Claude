# ATLAS 戦略検証（4段階安全性チェック）

生成された戦略コードの安全性・適合性を4段階で検証します。
Python CLIで検証を実行し、結果の解釈・修正指示はcode-safety-reviewerエージェントに委譲します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — 検証対象の世代ID（例: `ATLAS-2026-0405-001`）
- `<generation_id> --stage <N>` — 特定段階のみ実行（1-4）
- 引数なし — 最新の未検証世代を自動選択

## 実行手順

### 1. 検証対象の確認
戦略コードファイルの存在を確認:
```bash
ls ATLAS/strategies/<generation_id>/strategy.py
```

### 2. 4段階検証の実行（Python CLI）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main validate <generation_id>
```

特定段階のみ実行する場合:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main validate <generation_id> --stage <N>
```

出力はJSON形式:
```json
{
  "generation_id": "<id>",
  "overall": "PASS" or "FAIL",
  "stages": {
    "stage1_static": { "passed": true/false, "errors": [], "warnings": [] },
    "stage2_interface": { "passed": true/false, "errors": [], "warnings": [] },
    "stage3_sandbox": { "passed": true/false, "cpu_sec": N, "errors": [] },
    "stage4_runtime": { "passed": true/false, "avg_signal_ms": N, "signal_distribution": {} }
  }
}
```

### 3. code-safety-reviewer エージェントに結果解釈を委譲（FAILの場合）

FAILの場合のみ、Agent tool で `code-safety-reviewer` エージェントを起動:

> 以下の戦略コード検証結果を解釈し、修正指示を出力してください。
>
> - 戦略ID: `<generation_id>`
> - 検証結果JSON: [Step 2の出力をそのまま渡す]
> - 戦略コードパス: `ATLAS/strategies/<generation_id>/strategy.py`
>
> 1. FAIL原因を特定し、具体的な行番号と修正方法を明記
> 2. 修正の優先順位を付ける
> 3. 戦略ロジックを崩さない範囲での修正方法を提案
>
> 結果を `ATLAS/strategies/<generation_id>/validation_report.json` に出力すること。

### 4. 検証結果の記録
History Store に検証結果を保存:
```bash
cd ATLAS && .venv/Scripts/python.exe -c "
import asyncio, json
from atlas.history.store import HistoryStore
from atlas.history.models import GenerationStatus

async def save():
    async with HistoryStore('data/atlas_history.db') as store:
        rec = await store.get('<generation_id>', <gen_num>)
        if rec:
            rec.validation_log = [json.dumps(<検証結果JSON>)]
            rec.status = GenerationStatus.BACKTESTING if '<overall>' == 'PASS' else GenerationStatus.FAILED_VALIDATION
            await store.save(rec)

asyncio.run(save())
"
```

### 5. 出力フォーマット
```
=== ATLAS 戦略検証結果 ===
世代ID: <generation_id>

Stage 1 [静的解析]     : PASS / FAIL
  - 構文チェック       : OK
  - 禁止パターン       : OK（検出なし）
  - import安全性       : OK

Stage 2 [インターフェース]: PASS / FAIL
  - Strategy継承       : OK
  - 必須メソッド       : OK（4/4）
  - 型整合性           : OK

Stage 3 [サンドボックス]  : PASS / FAIL
  - 実行完了           : OK（CPU: X.Xs, シグナル: N件）
  - 戻り値検証         : OK
  - 例外               : なし

Stage 4 [ランタイム品質]  : PASS / FAIL
  - 平均実行時間       : Xms（上限: 100ms）
  - シグナル分布       : 正常
  - confidence分布     : 正常

総合判定: PASS / FAIL
  FAIL項目: [あれば列挙]

次のステップ:
  [PASS] /atlas-backtest <generation_id>
  [FAIL] code-safety-reviewer の修正指示に基づいて修正 -> /atlas-validate <generation_id>
```

## エラーハンドリング
- 指定した `generation_id` のstrategy.pyが存在しない場合、利用可能な世代一覧を表示
- Stage 1で FAIL の場合、Stage 2以降はスキップ（Python CLI側で自動制御）
- 各Stageの詳細なエラー内容を必ず表示すること

## 注意事項
- 検証結果は History Store に自動記録される
- FAIL時はcode-safety-reviewerエージェントが具体的な修正指針を提示
- Stage 3のサンドボックスはダミーOHLCVデータで実行（実市場データ不要）
