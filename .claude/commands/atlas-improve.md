# ATLAS 戦略改良

評価結果をフィードバックし、改良版の戦略コードを生成します。
コンテキスト構築はPython CLI、改良版コード生成はfx-strategist、安全性確認はcode-safety-reviewerに委譲します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<generation_id>` — 改良対象の世代ID（例: `ATLAS-2026-0404-001`）
- `<generation_id> --focus <領域>` — 改良フォーカス（entry, exit, risk, filter）
- `<generation_id> --max-changes <N>` — パラメータ変更上限（デフォルト: 3）
- `<generation_id> --aggressive` — 大幅改良モード（不合格戦略向け）
- 引数なし — 最新の評価済み世代を自動選択

## 実行手順

### 1. 改良コンテキストの取得（Python CLI）
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main context <generation_id> --mode improve
```
以下の構造化コンテキストJSONが出力される:
```
[PREVIOUS_STRATEGY]     <- 前回の戦略コード全文
[EVALUATION_RESULTS]    <- 26指標（閾値割れに★マーク）
[WEAKNESS_ANALYSIS]     <- 弱点サマリー（優先度順）
[IMPROVEMENT_DIRECTIVES]<- 改善指示（最大3件、優先度付き）
[CONSTRAINTS]           <- Immutable閾値 + 劣化禁止制約 + パラメータ変更上限
[HISTORY]               <- 直近3世代の改良履歴
```

### 1.5. 親 final_score チェック（QA-02）

`ATLAS/strategies/<generation_id>/evaluation/integrated_report.json`（なければ `report.json`）を読み、`final_score` を確認:

| 親 final_score | 改良方針 |
|-------------|---------|
| `null`（L1 FAIL） | **改良ではなく新規生成（`/atlas-generate`）を推奨**。改良を強行する場合は `--aggressive` 必須 |
| `< 0.30` | **系統廃棄推奨**。別タイプ/別通貨ペアでの新規生成を検討。本改良は `--aggressive` でロジック構造の変更も許可 |
| `0.30 - 0.75` | 通常改良フロー |
| `>= 0.75` | 既に `excellent`。改良ではなく `/atlas-export` を推奨 |

### 2. 新世代IDの採番
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history next-id
```

### 3. 改良版ディレクトリの準備
```bash
mkdir -p ATLAS/strategies/<new_generation_id>
```

### 4. fx-strategist エージェントに改良版生成を委譲

Agent tool で `fx-strategist` エージェントを起動し、以下のタスクを渡す:

> 以下のFXトレード戦略の改良版を生成してください。
>
> - 親戦略ID: `<generation_id>`
> - 新戦略ID: `<new_generation_id>`
> - 改良モード: improve
> - パラメータ変更上限: <N>箇所
> - 改良フォーカス: <entry/exit/risk/filter/なし>
>
> **改良コンテキスト:**
> [Step 1で取得したコンテキストJSONをそのまま渡す]
>
> **厳守ルール:**
> 1. [IMPROVEMENT_DIRECTIVES] の改善指示を最優先で対処すること
> 2. パラメータ変更は上限<N>箇所を厳守（`agents/fx-strategist.md` の **F3 厳密カウント** a-e 分類に従い、spec.md 改良履歴に内訳を明記）
> 3. 親世代で閾値を満たしていた指標を劣化させないこと（**F2 親統計プロファイル継承** ±30% 以内）
> 4. Backtest Gate基準（Immutable）は絶対に緩和しない
> 5. 親と異なる instrument / timeframe の場合は **F1 クロスインスツルメント移植プロトコル** 5 項目を spec.md に必ず記載
> 6. 親世代の `spec.md` の「前提条件」「フィルター条件」を改変する場合は spec.md 改良履歴に理由を必須記述
>
> 出力先: `ATLAS/strategies/<new_generation_id>/`
> 以下を生成: strategy.py, spec.md（改良履歴を含む）, config.json, metadata.json

### 5. devil-advocate エージェントによる改善提案のクロスバリデーション

**目的:** fx-strategistの改善提案が本当に有効か、副作用がないかを事前検証する。

Agent tool で `devil-advocate` エージェントを起動し、以下のタスクを渡す:

> 以下のfx-strategistによる改良版戦略コードに対して、敵対的レビューを実施してください。
>
> - 親戦略ID: `<generation_id>`
> - 新戦略ID: `<new_generation_id>`
> - 改良コンテキスト: [Step 1の改良コンテキストJSON]
> - 改良版コード: [生成された strategy.py の内容]
> - 改良内容の説明: [fx-strategistが出力した改良サマリー]
>
> 以下を検証すること:
> 1. 改善の副作用予測 - エントリー条件変更がトレード頻度・品質にどう影響するか
> 2. 改善の優先順位の妥当性 - 最もインパクトのある改善が最優先になっているか
> 3. 制約違反の可能性 - パラメータ変更上限の実質的超過（**F3 a-e 分類で厳密カウント**）、Immutable制約への抵触
> 4. **agents/devil-advocate.md の既知失敗パターン照合を必須**（Gen42-55 atr_expansion、Gen85 rsi_exit、フィルター追加の罠、条件緩和のジレンマ、L1 sensitivity 逆方向）
> 5. 親世代で閾値を満たしていた指標が劣化するリスク
> 6. **各 bias_detected エントリに D2 adoption_level（must_fix / should_consider / informational）を必ず付与**
> 7. **D4 定性表現禁止** — 「なんとなく不安」等は informational 以下に格下げ、数値根拠を必須
>
> 敵対的レビューを `ATLAS/strategies/<new_generation_id>/adversarial_review.json` に出力すること。

**レビュー結果の処理（QA-05 adoption_level 3 段階分岐）:**
- `adoption_level=must_fix` の問題:
  - fx-strategist にレビュー結果を渡して**修正必須**（1 回まで）
  - 修正後、再度 devil-advocate に確認を依頼
  - 修正できなかった場合はこの改良を中止
- `adoption_level=should_consider` の問題:
  - `improvement_directives` に追記、fx-strategist は判断で採否を決める
  - 却下時は `spec.md` に理由を明記
- `adoption_level=informational` の問題:
  - 記録のみ、次のステップへ
- `abandon_recommendation=true` の場合:
  - この改良を中止し、別アプローチの検討を促す

### 6. code-safety-reviewer エージェントによる安全性確認

Agent tool で `code-safety-reviewer` エージェントを起動し、以下のタスクを渡す:

> 以下の改良版戦略コードの安全性を確認してください。
>
> - 戦略ID: `<new_generation_id>`
> - コードパス: `ATLAS/strategies/<new_generation_id>/strategy.py`
>
> 以下を検証すること:
> 1. 禁止パターン（os, subprocess, exec等）が含まれていないか
> 2. Strategy基底クラスのインターフェースに準拠しているか
> 3. 親戦略から追加されたロジックに問題がないか
>
> FAIL の場合は具体的な修正指示を出力すること。

### 7. 安全性確認の結果処理
- **PASS の場合:** History Store に記録し、次のステップへ
- **FAIL の場合:** fx-strategist に修正指示を渡して再生成（最大3回）

### 8. 改良結果の記録
```bash
cd ATLAS && .venv/Scripts/python.exe -c "
import asyncio, json
from atlas.history.store import HistoryStore
from atlas.history.models import GenerationRecord, GenerationStatus

async def save():
    async with HistoryStore('data/atlas_history.db') as store:
        rec = GenerationRecord(
            strategy_id='<new_generation_id>',
            generation=<new_gen_num>,
            parent_id='<generation_id>',
            instrument='<通貨ペア>',
            timeframe='<時間足>',
            status=GenerationStatus.VALIDATING,
            strategy_code=open('strategies/<new_generation_id>/strategy.py').read(),
        )
        await store.save(rec)

asyncio.run(save())
"
```

### 9. 出力フォーマット
```
=== ATLAS 戦略改良結果 ===
親世代: <generation_id> (final_score: X.XX)
新世代: <new_generation_id>

--- 改良内容 ---
[1] <改善指示1への対応内容>
[2] <改善指示2への対応内容>
[3] <改善指示3への対応内容>

--- 変更差分 ---
  変更パラメータ数: X / 上限<N>
  新規追加ロジック: <あれば記載>
  削除ロジック: <あれば記載>

--- 敵対的レビュー ---
  devil-advocate: PASS / 修正要求 / 廃棄推奨
  [指摘事項があれば記載]

--- 安全性確認 ---
  code-safety-reviewer: PASS

次のステップ:
  /atlas-validate <new_generation_id> — 4段階安全性検証
  /atlas-backtest <new_generation_id> — バックテスト実行
```

## エラーハンドリング
- 評価未完了の世代が指定された場合、`/atlas-evaluate` の実行を促す
- 改良版生成が3回連続FAILした場合、新規生成（`/atlas-generate`）への切り替えを推奨
- コンテキスト取得に失敗した場合、History Store の状態を確認
- --aggressive モードではパラメータ変更上限を緩和し、ロジック構造の変更も許可

## 注意事項
- パラメータ変更は上限数を厳守（デフォルト3箇所）
- 改良履歴（parent_id -> child_id）は History Store に系譜として記録
- Immutable制約（Backtest Gate基準）は絶対に緩和しない
