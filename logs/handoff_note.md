# Handoff Note

**最終更新**: 2026-05-05T11:00:00Z @ (current session)
**ブランチ**: master
**直前のコミット**: `2775dc8` [atlas] 157-158 Gate PASS

## 現在の作業（1 行サマリ）

自律ループ継続中: SL=1.5 TightSL + TP=5.0 拡張設計を3 JPY クロスペアで系統的に探索。本セッションで **24件 Gate PASS** を追加、FTS 64 戦略稼働中。

## 詳細コンテキスト

前セッション(handoff 引継ぎ): GBP/JPY H4 Donchian breakout 系列 D=4〜15 全 Gate PASS。
本セッション: SL=1.5 (TightSL) バリアントを USD/JPY、GBP/JPY、EUR/JPY の H4 全 D 値で網羅的に探索。
新たな発見: TP=5.0 延長 + max_hold=16 が短中期 D 値(D=3〜7)で有効、長期 D 値(D=10+)は L1 FAIL。
pips_per_unit=100 (JPY ペア) が正しい値。10000 は NG（SL が100倍タイトになる）。

## 本セッション Gate PASS 実績 (2026-05-04〜05)

### SL=1.5 TP=4.0 系列
| ID | ペア | D | soft | WFA | Sharpe |
|---|---|---|---|---|---|
| 0504-133 | USD/JPY H4 | D=3 | 0.880 | 0.989 | 3.93 |
| 0504-134 | GBP/JPY H4 | D=15 | 0.829 | 1.001 | 2.09 |
| 0504-137 | EUR/JPY H4 | D=5 | 0.873 | 0.992 | 2.72 |
| 0504-138 | EUR/JPY H4 | D=15 | 0.835 | 0.950 | 2.26 |
| 0504-139 | USD/JPY H4 | D=10 | 0.865 | 1.012 | 3.24 |
| 0504-140 | USD/JPY H4 | D=15 | 0.850 | 0.972 | 2.89 |
| 0504-142 | EUR/JPY H4 | D=3 | 0.828 | 0.982 | 2.63 |
| 0504-143 | EUR/JPY H4 | D=7 | 0.844 | 0.968 | 2.29 |
| 0504-145 | USD/JPY H4 | D=12 | 0.845 | 1.003 | 3.08 |
| 0504-150 | GBP/JPY H4 | D=3 | 0.786 | 0.978 | 2.32 |

### SL=1.5 TP=5.0 系列 (max_hold=16)
| ID | ペア | D | soft | WFA | Sharpe |
|---|---|---|---|---|---|
| 0504-149 | USD/JPY H4 | D=3 | 0.839 | 0.932 | 3.77 |
| 0504-152 | EUR/JPY H4 | D=5 | 0.848 | 0.982 | 2.57 |
| 0504-156 | GBP/JPY H4 | D=3 | 0.797 | 0.987 | 2.41 |
| 0504-157 | GBP/JPY H4 | D=5 | 0.803 | 0.952 | 2.38 |
| 0504-158 | EUR/JPY H4 | D=7 | 0.843 | 0.970 | 2.35 |

## 知見・発見

### D値フェールゾーン (SL=1.5)
| ペア | 通過D | 失敗D |
|---|---|---|
| USD/JPY H4 | D=3,5,10,12,15 | D=7 のみ |
| GBP/JPY H4 | D=3,5,15 | D=7〜12 (広いフェールゾーン) |
| EUR/JPY H4 | D=3,5,7,10,15 **全て通過** | なし |

### TP=5.0 有効範囲
- **有効**: D=3〜7 (短中期): USD/JPY D=3, EUR/JPY D=5,7, GBP/JPY D=3,5
- **無効**: D=10以上 (長期): GBP/JPY D=15, USD/JPY D=10 はL1 FAIL

### その他
- **pips_per_unit=100** が JPY ペアの正解（10000 は NG。SL/TP が100倍ずれる）
- **AUD/JPY、NZD/JPY** H4 breakout は L2 FAIL (carry効果弱)
- **0424-001 (GBP/USD M15 SHORT)** FTS Paper 稼働中

## FTS 状態

- 戦略数: **64件** (ATLAS 60件 + FWDTEST等)
- Runner: 稼働中 (最終再起動 2026-05-05)
- 全戦略 Paper モード、live_eligible=false

## 未コミット変更

なし（全コミット・プッシュ済み）

## 次にやること

### 高優先
1. **GBP/JPY D=5 TP=4.0 → SL=2.0 比較**: 131(SL=1.5 soft=0.827) vs 106(SL=2.0 soft=0.869) — SL=2.0 の優位性確認
2. **EUR/JPY D=10 TP=5.0** (TP=5.0 中期D値の EUR/JPY 境界確認)
3. **USD/JPY D=5 SL=1.5 TP=5.0** (130の TP 延長)

### 多様化検討
4. H1 タイムフレームの再評価 (H4 >>H1 の法則から H4 集中継続が効率的)
5. SHORT 戦略: 0424-001 が唯一の選択肢。現状維持

## 関連文書・コマンド

- ATLAS CLAUDE.md: `C:\data\works\FX\ATLAS\CLAUDE.md`
- FTS Runner: `fx_trading_system/scripts/start_unified_runner.ps1`
- 最新 ATLAS commit: `2775dc8`

## Runner 状態

- FTS Runner 稼働中（最終再起動 2026-05-05）
- 64戦略 Paper モード

## 引継ぎ時の注意

- **pips_per_unit**: JPY ペアは `100`。非 JPY ペア (EUR/USD 等) は `10000`
- **FTS Runner 再起動**: 新戦略追加後は `start_unified_runner.ps1` 実行必須
- **ATLAS git**: 親 FX repo とは別管理 (`cd /c/data/works/FX/ATLAS && git push`)
- **FTS 64 戦略**: ウォームアップ 30〜60 分かかる
- **相関リスク**: 全戦略 JPY クロス LONG。JPY 急騰時に同時ドローダウン
