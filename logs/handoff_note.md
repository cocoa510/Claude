# Handoff Note (親リポ — リダイレクトのみ)

**最終更新**: 2026-04-25T00:30:00Z (JST 2026-04-25 09:30:00+09:00) @ training-goto
**ブランチ**: master

## このノートは ATLAS 引継ぎを保持しません

ATLAS は `bc20d49` (2026-02 頃) で親リポ (`claude/`) から独立 git 管理に分離済み
（リモート: `https://github.com/cocoa510/ATLAS.git`）。

**現在進行中の作業（ATLAS / fx_trading_system）の引継ぎ状態は以下の正本を参照すること**:

| 対象 | 引継ぎノート正本 |
|-----|----------------|
| ATLAS | `ATLAS/logs/handoff_note.md` + `ATLAS/logs/handoff_state.json` |
| fx_trading_system (FTS) | `fx_trading_system/logs/handoff_note.md`（存在する場合） |

`/handoff pull` 実行時は、CWD に応じてサブプロジェクト側の note を読みに行く運用とする。
親リポ側の note は「親リポ独自のメタ作業（CLAUDE.md 改訂・スキル定義・複数システム横断作業等）」が
発生した時のみ更新する。

## 親リポ単独の作業状況（あれば）

現時点で親リポ単独の未完了作業はなし。直近の親リポコミットは:

- `69df9e9 [wip:handoff] アンサンブル方針 4 専門家検証完了、Option A/B/C 選択待ち @ training-goto`
  → このコンテキスト自体は **2026-04-25 時点で陳腐化**。実体は ATLAS Phase 3 系列に統合済み
  （Phase 2 / Phase 3 Option A / Phase 3.6 完了、次は Phase 3b refactor）。
  詳細は `ATLAS/logs/handoff_note.md` 参照。

## 引継ぎ時の注意

- 親リポ note を最新状態と誤読しないこと。**ATLAS 側 note の `generated_at` と必ず比較**し、新しい方を信用する
- 親リポ側を更新するのは、複数システム横断の判断ログ（例: ATLAS から FTS へのチャンピオン投入決定等）が
  発生した場合に限る
