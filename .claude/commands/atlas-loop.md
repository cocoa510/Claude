# ATLAS 自動反復改善ループ

戦略の「生成 -> 検証 -> バックテスト -> 評価 -> 改��」サイクルを専門家エージェント群とPython CLIで自動駆動し、収束判定まで自動実行します。

## 引数
`$ARGUMENTS` — 以下の形式で指定:
- `<戦略タイプ> <通貨ペア> [オプション]` — 新規生成から開始
- `--resume <generation_id>` — 既存世代から改良ループを再開
- `--max-generations <N>` — 世代上限（デフォルト: 10）
- `--target-score <S>` — 目標スコア（デフォルト: 0.75）
- `--dry-run` — 実行計画のみ表示（実行しない）
- 例: `trend_following USDJPY --max-generations 5`
- 例: `--resume ATLAS-2026-0404-001 --target-score 0.80`

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
    |-- quant-analyst Agent --> 結果解釈・��点分析・改善方針
    |       |
    |-- atlas converge (Python CLI) --> 収束判定
    |       |
    |-- [継続] fx-strategist Agent --> 改良版生成（コンテ��スト付き）
    |-- [停止] 最終レポート出力
```

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

#### Step 3.4: 収束判定
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main converge <generation_id>
```

収束判定結果に基づいて分岐:

| 判定 | アクション |
|------|----------|
| `excellent` | ループ停止。最終レポート出力 |
| `converged_stagnation` | ル���プ停止。改善停滞を報告 |
| `converged_oscillation` | ループ停止。振動検出を報告 |
| `abandoned` | この系統を廃棄。新規生成に切り替え |
| `max_generations` | ループ停止。世代上限到達を報告 |
| `continue` | 次のステップ（Step 3.5）へ |

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

**Step 3.1 に戻る（新世代で反復）**

### フェーズ4: 最終レポート

ループ完了時に以下を出力する。

#### ループ中の各世代ログ（蓄積表示）
```
--- 世代 ATLAS-2026-0404-001 (1/10) ---
[検証]     PASS (4/4 stage通過)
[BT]       PF:1.45 Sharpe:1.12 MaxDD:15.2% Gate:PASS
[評価]     final_score: 0.62 (合格)
[収束判定] 継続（目標0.75未達、改善率+8.2%）
[改良]     -> ATLAS-2026-0404-002 生成完了
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
- 全バリアントが品質下限割れした場合、戦略タイプの変更を提案
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
