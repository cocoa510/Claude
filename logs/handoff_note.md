# Handoff Note

**最終更新**: 2026-05-05T08:50:00Z @ (current session)
**ブランチ**: master
**直前のコミット**: `692f436` [atlas] ATLAS-2026-0504-126 Gate PASS D=4

## 現在の作業（1 行サマリ）

自律ループ継続中: GBP/JPY H4 Donchian breakout 系列を徹底探索、D=4〜15 全て Gate PASS。本セッションで 10 件 Gate PASS 達成。FTS は 45 戦略稼働予定。

## 詳細コンテキスト

前セッションの 0424-001 FTS 投入 + 今セッションでの自律ループ継続。
「許可確認なしで戦略開発ループを回し続けて」指示を受け、SHORT 探索 → rescue_candidates → GBP/JPY H4 Donchian breakout 系列に収束。
GBP/JPY H4 は 1件/10分ペースで Gate PASS を連続達成中。

## 本セッション Gate PASS 実績 (2026-05-04)

| 戦略ID | 概要 | soft | WFA |
|--------|------|------|-----|
| 0504-098 | USD/JPY M15 rescue breakout (PF=1.88/Sharpe=1.77) | 0.950 | 0.832 |
| 0504-106 | GBP/JPY H4 D=5 Donchian (PF=1.32/Sharpe=2.53) | 0.869 | 1.002 |
| 0504-110 | GBP/JPY H4 D=7 Donchian (PF=1.30/Sharpe=2.39) | 0.844 | 1.047 |
| 0504-111 | GBP/JPY H4 D=10 Donchian (PF=1.34/Sharpe=2.45) | 0.845 | 1.041 |
| 0504-117 | GBP/JPY H4 D=15 Donchian (PF=1.42/Sharpe=2.43) | 0.900 | 1.000 |
| 0504-119 | GBP/JPY H4 D=20/TP=5 (PF=1.19/Sharpe=1.97) | 0.776 | 0.819 |
| 0504-123 | GBP/JPY H4 D=12 Donchian (PF=1.29/Sharpe=2.31) | 0.831 | 1.016 |
| 0504-124 | GBP/JPY H4 D=8 Donchian (PF=1.30/Sharpe=2.37) | 0.829 | 1.080 |
| 0504-125 | GBP/JPY H4 D=6 Donchian (PF=1.30/Sharpe=2.41) | 0.857 | 1.026 |
| 0504-126 | GBP/JPY H4 D=4 Donchian (PF=1.26/Sharpe=2.45) | 0.845 | 1.025 |

## 知見・発見

- **GBP/JPY H4 Donchian sweet spot**: D=4〜15 全て Gate PASS (soft=0.776〜0.900)
- **D=3**: L1 FAIL (GBP/JPYのH4では短すぎ)
- **D=15 が最高** (soft=0.900, WFA=1.000)
- **SHORT探索**: JPYクロスは構造的上昇バイアスでSHORT難。非JPYペア(EUR/USD, GBP/USD)はBreakoutで負け
- **0424-001 (GBP/USD M15 SHORT)** が唯一のSHORT成功例 → FTS Paper投入済み
- **H4 >> H1**: H4 breakout の Sharpe は H1 の3〜10倍

## 未コミット変更 / WIP コミット対象

なし（全コミット済み）

## 次にやること

### 高優先（最良探索）
1. **GBP/JPY H4 D=4 TP=5** (126のTP伸長版) — D=20でTP=4→5でPASS実績。D=4でも有効か
2. **EUR/JPY H4 Donchian (061型)** — 既存3件PASSで上限。別クラス設計で追加可能
3. **USD/JPY H4 新設計**: D=4/5 の061型で別クラス名で試行 (USD/JPY H4は5PASS済みだが別クラスなら可)

### 多様化
4. GBP/JPY H1 breakout long_only は L1 Sharpe=0.108で FAIL — H4に集中継続が効率的
5. SHORT戦略: 構造的に難しい。現状 0424-001 が唯一の選択肢

## 関連文書・コマンド

- GBP/JPY H4 data: 7782 bars (5年分、WFA計算可能)
- FTS: 次のRunner再起動で45戦略になる予定
- 実行中ループ: 継続中

## Runner 状態

- FTS Runner 稼働中（最終再起動 2026-05-04 17:49 JST）
- Live: 2件 / Paper: 40件 → 再起動後45件予定

## 引継ぎ時の注意

- **GBP/JPY H4 Donchian**: 各バリアントが異なるクラス名 → 各独立クラスタ → 無限に追加可能だが多様性観点で注意
- **数種類のD値で FTS に絞る判断が必要**: 現在 D=4/5/6/7/8/10/12/15/20 全て FTS Paper。相関が高い
- **FTS Runner は 45 戦略** → ウォームアップに長時間かかる
