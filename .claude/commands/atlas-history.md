# ATLAS 世代履歴の表示

戦略の親子系譜・改良履歴・improvement_rate 推移をターミナルに出力します。

## 引数
`$ARGUMENTS`:
- `<generation_id>` — 特定戦略の祖先 → 子孫系譜を表示
- `--top <N>` — Top N 戦略の系譜を一覧 (デフォルト 5)
- `--session <session_id>` — 特定 atlas-loop セッションの全世代を時系列表示
- 引数なし — Top 5 戦略の系譜サマリ

## 実行手順

### `<generation_id>` 指定時 (系譜トレース)
```bash
cd ATLAS && .venv/Scripts/python.exe -c "
import json
from pathlib import Path
import sys

target = '$ARGUMENTS'.split()[0]

# 1) ターゲット戦略の metadata を読む
def load_meta(sid):
    md = Path('strategies') / sid / 'metadata.json'
    if not md.exists():
        return None
    try:
        return json.loads(md.read_text(encoding='utf-8'))
    except Exception:
        return None

def load_result(sid):
    rj = Path('strategies') / sid / 'backtest' / 'result.json'
    if not rj.exists():
        return None
    try:
        return json.loads(rj.read_text(encoding='utf-8'))
    except Exception:
        return None

# 2) 祖先トレース (parent_id を遡る)
ancestors = []
cur = target
visited = set()
while cur and cur not in visited:
    visited.add(cur)
    m = load_meta(cur)
    if not m:
        break
    ancestors.append(cur)
    cur = m.get('parent_id')

ancestors.reverse()  # 古い順

# 3) 表示
print(f'=== {target} 系譜 ===')
print()
print(f'{\"世代ID\":<25} | {\"PF\":>6} | {\"Sharpe\":>7} | {\"trades\":>6} | {\"Tier1\":>6} | {\"soft\":>6} | {\"判定\":<15}')
print('-' * 95)

prev_score = None
for sid in ancestors:
    r = load_result(sid)
    if not r:
        print(f'{sid:<25} | (BT 未実行)')
        continue
    l1 = r.get('layer1', {}) or {}
    g = r.get('gate_check', {}) or {}
    pf = l1.get('profit_factor') or 0
    sharpe = l1.get('sharpe_ratio') or 0
    trades = l1.get('total_trades') or 0
    tier1_passed = g.get('passed_count', 0)
    tier1_total = g.get('total_conditions', 0)
    soft = g.get('soft_score') or 0
    judg = 'BREAKTHROUGH' if r.get('overall_passed') else (f'Tier1 {tier1_passed}/{tier1_total}' if tier1_total else 'L1 FAIL')

    # improvement rate
    irate = ''
    if prev_score is not None and soft > 0 and prev_score > 0:
        delta = (soft - prev_score) / max(prev_score, 0.01) * 100
        irate = f' ({delta:+.1f}%)'

    print(f'{sid:<25} | {pf:>6.3f} | {sharpe:>7.3f} | {trades:>6} | {tier1_passed:>6} | {soft:>6.3f} | {judg:<15}{irate}')
    if soft > 0:
        prev_score = soft

print()
print(f'計 {len(ancestors)} 世代')
"
```

### `--top <N>` 指定時 (Top N 系譜サマリ)
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history top --limit ${N:-5} 2>/dev/null | .venv/Scripts/python.exe -c "
import sys, json
data = json.load(sys.stdin)
print('=== ATLAS Top 戦略系譜サマリ ===')
print()
for s in data:
    sid = s.get('strategy_id', '')
    parent = s.get('parent_id') or '(初代)'
    gen = s.get('generation', 0)
    score = s.get('final_score') or 0
    print(f'{sid} (世代 {gen}, score {score:.3f})')
    print(f'  parent: {parent}')
    print(f'  status: {s.get(\"status\")}')
    print()
"
```

### `--session <session_id>` 指定時 (atlas-loop セッション時系列)
```bash
cd ATLAS && cat logs/loop_session.json 2>/dev/null | .venv/Scripts/python.exe -c "
import sys, json
d = json.load(sys.stdin)
target_session = '$ARGUMENTS'.split()[1] if '--session' in '$ARGUMENTS' else d.get('session_id')

if d.get('session_id') != target_session:
    print(f'(現在の loop_session.json は {d.get(\"session_id\")}、指定された {target_session} は履歴外)')
    sys.exit(0)

print(f'=== セッション {target_session} 全世代 ===')
print(f'session_start: {d.get(\"session_start\")}')
print(f'phase: {d.get(\"phase\")}')
print()

print(f'{\"世代ID\":<25} | {\"status\":<25} | {\"PF\":>6} | {\"Sharpe\":>7} | {\"trades\":>6} | {\"備考\":<30}')
print('-' * 110)
for g in d.get('completed_generations', []):
    sid = g.get('id', '')
    status = (g.get('status', '') or '')[:25]
    pf = g.get('l1_pf') or 0
    sharpe = g.get('l1_sharpe') or 0
    trades = g.get('l1_trades') or 0
    fail_reasons = g.get('tier1_fail_conditions') or g.get('improvement_note') or ''
    note = (str(fail_reasons)[:30] if fail_reasons else '')
    print(f'{sid:<25} | {status:<25} | {pf:>6.3f} | {sharpe:>7.3f} | {trades:>6} | {note:<30}')
"
```

## 出力例

```
=== ATLAS-2026-0426-014 系譜 ===

世代ID                    |     PF |  Sharpe | trades |  Tier1 |   soft | 判定
-----------------------------------------------------------------------------------------------
ATLAS-2026-0408-041       |  1.239 |   0.764 |    107 |      5 |  0.671 | Tier1 5/6
ATLAS-2026-0426-009       |  1.252 |   0.235 |     16 |      4 |  0.412 | Tier1 4/6 (-38.6%)
ATLAS-2026-0426-014       |  1.567 |   0.645 |     31 |      4 |  0.468 | Tier1 4/6 (+13.6%)

計 3 世代
```

## 注意事項
- improvement_rate は soft_score 基準
- 表示は祖先 → 子孫の時系列順
- `metadata.json::parent_id` がない戦略は系譜の起点 (初代)
