# ATLAS 自動反復改善ループ

戦略の「生成 -> 検証 -> バックテスト -> 評価 -> 改��」サイクルを専門家エージェント群とPython CLIで自動駆動し、収束判定まで自動実行します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<戦略タイプ> <通貨ペア> [オプション]` — 新規生成から開始
- `--resume <generation_id>` — 既存世代から改良ループを再開
- `--max-generations <N>` — 世代上限（デフォルト: 10）
- `--target-score <S>` — 目標スコア（デフォルト: 0.75）
- `--types <type1,type2,...>` — 戦略タイプローテーション順序を明示指定
- `--no-rotate` — タイプ自動ローテーションを無効化（系統廃棄時にループ停止）
- `--dry-run` — 実行計画のみ表示（実行しない）
- 例: `trend_following USDJPY --max-generations 5`
- 例: `trend_following USDJPY --types trend_following,breakout,mean_reversion`
- 例: `--resume ATLAS-2026-0404-001 --target-score 0.80`

## 戦略タイプ自動切替

現在の戦略系統が**行き詰まった場合**（abandoned / converged_stagnation / converged_oscillation / 全バリアント品質下限割れ）、自動的に**現状とは異なる戦略タイプ**に切り替えて新規生成からループを再開する。

### 切替候補プール

以下の6タイプから現在のタイプを除外したものが候補:
```
trend_following / mean_reversion / breakout / momentum / volatility / hybrid
```

`--types <list>` 指定時はそのリストに限定する。

### 次タイプの選択ロジック

1. **過去結果に有望なタイプがある場合 → そのタイプを選択**
   - `ATLAS/strategies/` 配下の全戦略の `backtest/result.json` を走査
   - 現在のタイプを除いた中で **最良スコア** (PF×Sharpe など) を出したタイプを抽出
   - 候補が「未試行」または「平均PF >= 0.9」など有望な兆候があればそれを選択
2. **有望なタイプがない場合 → 候補プールからランダムに選択**
   - 直前のタイプは除外
   - `--no-rotate` 指定時はループ停止

> 厳密な順序ローテーションは行わない。ユーザー要求により「現状と違えば OK」という方針。

### 状態管理

`logs/loop_session.json` に以下を追記:
```json
{
  "session_start": "...",
  "limit_minutes": 300,
  "threshold_pct": 94,
  "available_types": ["trend_following", "mean_reversion", "breakout", "momentum", "volatility", "hybrid"],
  "current_type": "trend_following",
  "type_history": {
    "trend_following": {
      "lineages": ["ATLAS-2026-0408-001", ...],
      "best_pf": 2.2441,
      "best_sharpe": 0.283,
      "result": "abandoned"
    }
  }
}
```

### 切替発動条件と動作

| 発動条件 | 動作 |
|---------|------|
| `abandoned` 判定 | 別タイプで新規生成 |
| `converged_stagnation` | 別タイプへ切替 |
| `converged_oscillation` | 別タイプへ切替 |
| 全バリアントが品質下限割れ | 即座に別タイプへ |
| `--no-rotate` 指定時 | 上記の代わりにループ停止 |

### 切替時の引き継ぎ

次タイプへの遷移時は **過去全タイプの失敗履歴をコンテキストとして渡す**。`fx-strategist` エージェントは:
- 以前のタイプで失敗した原因（例: WR 38% 固定、PF 0.7 収束）を回避する設計を試みる
- 通貨ペア・時間足・データ範囲は維持する

## アーキテクチャ

```
/atlas-loop（このSkill = オーケストレーター）
    |
    |-- [新規] fx-strategist Agent --> 戦略コード生成
    |       |
    |-- atlas validate (Python CLI) --> 4段階安全性検証
    |-- code-safety-reviewer Agent --> 検証結果の解釈
    |       |
    |-- atlas backtest (Python CLI) --> 2層バックテスト
    |       |
    |-- atlas metrics (Python CLI) --> 26指標算出
    |-- quant-analyst Agent --> 結果解釈・弱点分析・改善方針
    |-- devil-advocate Agent --> 敵対的レビュー（バイアス検出）
    |-- [統合] 両エージェントの見解をマージ
    |       |
    |-- atlas converge (Python CLI) --> 収束判定
    |       |
    |-- [継続] fx-strategist Agent --> 改良版生成（統合評価に基づく）
    |-- [継続] devil-advocate Agent --> 改善提案のクロスバリデーション
    |-- [停止] 最終レポート出力
```

**エージェント構成（5名体制）:**
| エージェント | モデル | 役割 |
|-------------|--------|------|
| fx-strategist | opus | 戦略コード生成・改良 |
| quant-analyst | opus | 定量評価・弱点分析 |
| devil-advocate | sonnet | 敵対的レビュー・バイアス検出 |
| code-safety-reviewer | sonnet | コード安全性検証 |
| オーケストレーター（このSkill） | - | 全体制御・統合判定 |

## 実行フロー

### フェーズ1: 初期化
1. 引数を解析し、設定を読み込む
2. `--resume` の場合は History Store から前回の状態を復元
3. 新規の場合は世代IDを採番

### フェーズ2: 初期生成（--resume でない場合）

#### Step 2.1: fx-strategist に生成を委譲
Agent tool で `fx-strategist` エージェントを起動:

> 以下の条件でFXトレード戦略を生成してください。
> - 戦略ID: `<generation_id>`
> - 戦略タイプ: `<戦略タイプ>`
> - 通貨ペア: `<通貨ペア>`
> - 時間足: `<時間足>`
> - 出力先: `ATLAS/strategies/<generation_id>/`
>
> strategy.py, spec.md, config.json, metadata.json を生成すること。
> ATLAS/CLAUDE.md と ATLAS/atlas/common/models.py を読んでインターフェースに準拠すること。

### フェーズ3: 反復ループ（各世代で実行）

以下を**各世代ごとに順次実行**する。

#### Step 3.1: 安全性検証
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main validate <generation_id>
```

Agent tool で `code-safety-reviewer` エージェントを起動し、検証結果JSONを渡して解釈を依頼。

- **FAIL の場合:** fx-strategist に修正指示を渡して再生成（最大3回）。3回失敗で廃棄。
- **PASS の場合:** 次のステップへ。

#### Step 3.2: バックテスト実行
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main backtest <generation_id>
```
結果JSONを保存。

#### Step 3.3: メトリクス算出・評価
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main metrics <generation_id>
```

Agent tool で `quant-analyst` エージェントを起動し、メトリクスJSONを渡す:

> 以下のFXトレード戦略のバックテスト結果を評価してください。
> - 戦略ID: `<generation_id>`
> - メトリクスJSON: [metricsの出力]
>
> 1. 3層スコアリング結果の妥当性検証
> 2. Backtest Gate判定
> 3. 弱点の因果チェーン分析
> 4. 改善指示の策定（最大3件）
>
> ATLAS/strategies/<generation_id>/evaluation/ に report.json と summary.md を出力すること。

#### Step 3.3b: 敵対的レビュー（devil-advocate）

Agent tool で `devil-advocate` エージェントを起動し、quant-analystの評価を検証:

> 以下のquant-analystによる評価レポートに対して、敵対的レビューを実施してください。
> - 戦略ID: `<generation_id>`
> - メトリクスJSON: [metricsの出力]
> - quant-analystの評価レポート: [Step 3.3の report.json]
>
> 楽観バイアス検出、因果チェーン検証、改善提案の実現可能性、既知失敗パターンとの照合を行うこと。
> ATLAS/strategies/<generation_id>/evaluation/adversarial_review.json に出力すること。

#### Step 3.3c: 評価統合

quant-analyst と devil-advocate の見解を統合する:
- devil-advocate が `severity=high` を検出 → 改善指示を修正
- devil-advocate が `abandon_recommendation=true` → 収束判定で `abandoned` 扱い
- 統合結果を `integrated_report.json` に出力

#### Step 3.4: 収束判定
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main converge <generation_id>
```

収束判定結果に基づいて分岐:

| 判定 | アクション |
|------|----------|
| `excellent` | ループ停止。最終レポート出力 |
| `converged_stagnation` | **タイプ切替発動** — 別の戦略タイプで新規生成（`--no-rotate` 時はループ停止） |
| `converged_oscillation` | **タイプ切替発動** — 別の戦略タイプで新規生成（`--no-rotate` 時はループ停止） |
| `abandoned` | **タイプ切替発動** — 別の戦略タイプで新規生成（`--no-rotate` 時はループ停止） |
| `max_generations` | ループ停止。世代上限到達を報告 |
| `continue` | 次のステップ（Step 3.5）へ |

**タイプ切替発動時の手順:**
1. `logs/loop_session.json` の `type_history[current_type].result` に終了理由を記録
2. `ATLAS/strategies/` 配下の過去結果から「現在のタイプを除く有望タイプ」を探索
   - 各タイプの best_pf / best_sharpe を集計
   - PF >= 0.9 などの基準で「有望」を判定
3. 有望タイプが見つかればそれを選択、見つからなければ `available_types` から `current_type` を除いてランダム選択
4. 次タイプを `current_type` に設定し、フェーズ2（初期生成）に戻る
5. fx-strategist 起動時に過去全タイプの失敗履歴をコンテキストに渡す

#### Step 3.5: 改良版生成（継続時のみ）

改良コンテキストを取得:
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main context <generation_id> --mode improve
```

Agent tool で `fx-strategist` エージェントを起動し、改良版を生成:

> 以下のFXトレード戦略の改良版を生成してください。
> - 親戦略ID: `<generation_id>`
> - 新戦略ID: `<new_generation_id>`
> - パラメータ変更上限: 3箇所
>
> **改良コンテキスト:**
> [contextコマンドの出力をそのまま渡す]
>
> ATLAS/strategies/<new_generation_id>/ に出力すること。
>
> **重要:** 改良コンテキストに devil-advocate の敵対的レビュー結果が含まれている場合、
> その指摘事項にも対処すること。

#### Step 3.5b: 改善提案のクロスバリデーション（devil-advocate）

Agent tool で `devil-advocate` エージェントを起動し、改良版を検証:

> fx-strategistの改良版戦略コードに対して敵対的レビューを実施してください。
> - 親戦略ID: `<generation_id>` / 新戦略ID: `<new_generation_id>`
> - 改良版コード: [生成された strategy.py]
> - 改良コンテキスト: [contextの出力]
>
> 副作用予測、制約違反チェック、既知失敗パターンとの照合を行うこと。

- `severity=high` → fx-strategist に修正を依頼（1回まで）
- `abandon_recommendation=true` → この改良を中止、新規生成へ

**Step 3.1 に戻る（新世代で反復）**

### フェーズ4: 最終レポート

ループ完了時に以下を出力する。

#### ループ中の各世代ログ（蓄積表示）
```
--- 世代 ATLAS-2026-0404-001 (1/10) ---
[検証]     PASS (4/4 stage通過)
[BT]       PF:1.45 Sharpe:1.12 MaxDD:15.2% Gate:PASS
[評価]     final_score: 0.62 (合格)
[敵対的RV] バイアス:0件 リスク警告:1件(medium) -> 評価維持
[収束判定] 継続（目標0.75未達、改善率+8.2%）
[改良]     -> ATLAS-2026-0404-002 生成完了
[改良RV]   devil-advocate: PASS（副作用リスクなし）
```

#### 最終サマリー
```
=== ATLAS 自動ループ完了 ===
停止理由: <判定結果>
総世代数: XX
実行世代: XX

--- スコア推移 ---
ATLAS-2026-0404-001: 0.38 ########
ATLAS-2026-0404-002: 0.45 #########
ATLAS-2026-0404-003: 0.52 ##########
ATLAS-2026-0404-004: 0.61 ############
ATLAS-2026-0404-005: 0.68 #############
ATLAS-2026-0404-006: 0.76 ############### * 優良

--- 最優秀戦略 ---
世代ID: ATLAS-2026-0404-006
final_score: 0.76
戦略タイプ: <タ���プ>
系譜: 001 -> 002 -> 003 -> 004 -> 005 -> 006

次のステップ:
  /atlas-export ATLAS-2026-0404-006 — 本番候補としてエクスポート
  /atlas-status — 全世代の詳細を確認
  /atlas-history ATLAS-2026-0404-006 — 改良履歴を表示
```

## エラーハンドリング
- エージェント起動に失敗した場合、エラー内容をログに記録しリトライ（最大3回）
- Python CLIがエラーを返した場合、エラーJSONの内容に基づいて判断
- 全バリアントが品質下限割れした場合、**自動的に次の戦略タイプにローテーション**（`--no-rotate` 時は提案のみ）
- ユーザーによる中断（Ctrl+C）時、現在の状態はHistory Storeに保存済み。`--resume` で再開可能

## 自動実行（ユーザー操作なし）

完全自動実行するには:
1. `dontAsk` パーミッションモードで Claude Code を起動
2. permissions.allow にATLAS用のBashパターンを登録
3. `/atlas-loop <引数>` を実行

定期実��するには:
```
/loop 10m /atlas-loop --resume <latest_generation_id>
```

## 注意事項
- 長時間実行となるため、各世代の開始時に進捗を表示
- 全世代の中間結果は History Store にリアルタイム保存
- `--dry-run` で実行計画と推定ステップ数を事前確認可能
- ループ中断後は `/atlas-status` で現在状態を確認可能
- 各エージェントは順次実行される（並列実行はAgent Teams実験機能で対応予定）
