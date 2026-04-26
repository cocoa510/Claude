# ATLAS 戦略比較分析

複数戦略を横断比較してターミナルに出力します。

## 引数
`$ARGUMENTS`:
- `<id1> <id2> [id3...]` — 指定戦略を横断比較 (差分テーブル)
- `--top <N>` — Top N 戦略を比較 (デフォルト 5)
- 引数なし — Top 5 を自動比較

## 実行手順

### 指定 ID 比較
```bash
cd ATLAS && .venv/Scripts/python.exe -c "
import json, sys
from pathlib import Path

ids = [a for a in '$ARGUMENTS'.split() if not a.startswith('--')]
if not ids:
    print('使用法: /atlas-compare <id1> <id2> [...]'); sys.exit(1)

def load(sid):
    rj = Path('strategies') / sid / 'backtest' / 'result.json'
    if not rj.exists(): return None
    return json.loads(rj.read_text(encoding='utf-8'))

results = {sid: load(sid) for sid in ids}
missing = [sid for sid, r in results.items() if r is None]
if missing:
    print(f'(BT 結果なし: {missing})')
    results = {sid: r for sid, r in results.items() if r is not None}
    if not results:
        sys.exit(1)

def get_metric(r, path):
    for p in path.split('.'):
        if r is None: return None
        r = r.get(p) if isinstance(r, dict) else None
    return r

metrics = [
    ('Instrument', 'instrument', None),
    ('Direction', 'metadata.direction_bias', None),
    ('--- L1 ---', None, None),
    ('PF (L1)', 'layer1.profit_factor', 'max'),
    ('Sharpe (L1)', 'layer1.sharpe_ratio', 'max'),
    ('Trades (L1)', 'layer1.total_trades', 'max'),
    ('WR (L1)', 'layer1.win_rate', 'max'),
    ('MaxDD% (L1)', 'layer1.max_drawdown_pct', 'min_abs'),
    ('--- L2 ---', None, None),
    ('PF (L2)', 'layer2.profit_factor', 'max'),
    ('Sharpe (L2)', 'layer2.sharpe_ratio', 'max'),
    ('Trades (L2)', 'layer2.total_trades', 'max'),
    ('--- Gate ---', None, None),
    ('Tier1 PASS', 'gate_check.passed_count', 'max'),
    ('Soft Score', 'gate_check.soft_score', 'max'),
    ('Overall PASS', 'overall_passed', None),
]

# header
header = f'{\"指標\":<20} | ' + ' | '.join(f'{sid[-12:]:>12}' for sid in results.keys())
print('=== 戦略横断比較 ===')
print(header)
print('-' * len(header))

for label, path, best_op in metrics:
    if path is None:
        print(label)
        continue
    values = {}
    for sid, r in results.items():
        v = get_metric(r, path)
        values[sid] = v

    # best 判定
    best_sid = None
    if best_op and any(v is not None for v in values.values()):
        valid = {sid: v for sid, v in values.items() if v is not None and isinstance(v, (int, float))}
        if valid:
            if best_op == 'max':
                best_sid = max(valid, key=valid.get)
            elif best_op == 'min_abs':
                best_sid = min(valid, key=lambda k: abs(valid[k]))

    cells = []
    for sid, v in values.items():
        if v is None:
            disp = 'N/A'
        elif isinstance(v, bool):
            disp = '✓' if v else '✗'
        elif isinstance(v, float):
            disp = f'{v:.3f}'
        else:
            disp = str(v)[:12]
        marker = ' ★' if sid == best_sid else ''
        cells.append(f'{disp:>10}{marker:<2}')

    print(f'{label:<20} | ' + ' | '.join(cells))

print()
print('★ = 各指標の最良値')
"
```

### `--top <N>` 比較 (Top N 自動)
```bash
cd ATLAS && .venv/Scripts/python.exe -m atlas.main history top --limit ${N:-5} 2>/dev/null | .venv/Scripts/python.exe -c "
import sys, json
data = json.load(sys.stdin)
ids = [s.get('strategy_id') for s in data]
print(f'対象戦略: {ids}')
print(f'指定 ID 比較を実行: /atlas-compare ' + ' '.join(ids))
"
# 上記出力の「実行コマンド」を続けて Bash で実行
```

## 出力例

```
=== 戦略横断比較 ===
指標                 |  -2026-0408-041 |  -2026-0426-014
-----------------------------------------------------------------
Instrument           |        USD_JPY  |        USD_JPY
Direction            |     long_only ★ |     long_only ★
--- L1 ---
PF (L1)              |          1.239  |        1.567 ★
Sharpe (L1)          |          0.764 ★|          0.645
Trades (L1)          |          107 ★  |             31
WR (L1)              |          0.486  |        0.581 ★
MaxDD% (L1)          |          -1.99 ★|          -0.68 (※-0.68 が小)
--- L2 ---
PF (L2)              |          1.218  |        1.364 ★
Sharpe (L2)          |          0.625 ★|          0.570
Trades (L2)          |          112 ★  |             43
--- Gate ---
Tier1 PASS           |             5 ★ |              4
Soft Score           |          0.671 ★|          0.468
Overall PASS         |              ✗  |              ✗

★ = 各指標の最良値
```

## 注意事項
- 異なる instrument/timeframe の比較は数値の絶対比較に意味がない場合あり
- 詳細な指標 (26 metrics) は `evaluation/report.json` を直接参照
