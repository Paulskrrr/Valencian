# STRATEGY.md — Exact basic strategy tables (encode these cell-for-cell)

Source: the human's reference chart. There are two variants in the chart. The
**right-hand variant** is for **H17 (dealer hits soft 17)** with Late Surrender
and the small numbers are Hi-Lo index/deviation plays. The **left-hand variant**
is the simpler S17-style baseline. Encode BOTH if practical, and pick the table
based on the H17/S17 setting. The PRIMARY target is the H17 version.

Dealer upcard columns are always: 2 3 4 5 6 7 8 9 10 A

Legend:
- Pairs: Y = split. Y/N = split only if DAS offered, else don't. N = don't split.
- Soft/Hard: H = hit. S = stand. D = double if allowed else hit. Ds = double if
  allowed else stand. SUR = surrender.
- The small superscript numbers in the chart (e.g. 6*, 5*, -1*, 3+) are Hi-Lo TRUE
  COUNT deviation indices: take the indicated action when the true count meets the
  threshold (positive number = at-or-above that TC; negative = at-or-below).
  Implement deviations as an OPTIONAL layer in Phase 4 — get the base action right
  first.

================================================================================
## PAIR SPLITTING
================================================================================

### Baseline (S17-ish, left chart)
Pair | 2   3   4   5   6   7   8   9   10  A
A,A  | Y   Y   Y   Y   Y   Y   Y   Y   Y   Y
T,T  | N   N   N   N   N   N   N   N   N   N
9,9  | Y   Y   Y   Y   Y   N   Y   Y   N   N
8,8  | Y   Y   Y   Y   Y   Y   Y   Y   Y   Y
7,7  | Y   Y   Y   Y   Y   Y   N   N   N   N
6,6  | Y/N Y   Y   Y   Y   N   N   N   N   N
5,5  | N   N   N   N   N   N   N   N   N   N
4,4  | N   N   N   Y/N Y/N N   N   N   N   N
3,3  | Y/N Y/N Y   Y   Y   Y   N   N   N   N
2,2  | Y/N Y/N Y   Y   Y   Y   N   N   N   N

### H17 variant (right chart) — primary
Pair | 2   3   4    5    6    7   8   9   10  A
A,A  | Y   Y   Y    Y    Y    Y   Y   Y   Y   Y
T,T  | N   N   6*   5*   4*   N   N   N   N   N
9,9  | Y   Y   Y    Y    Y    N   Y   Y   N   N
8,8  | Y   Y   Y    Y    Y    Y   Y   Y   Y   Y
7,7  | Y   Y   Y    Y    Y    Y   N   N   N   N
6,6  | Y/N Y   Y    Y    Y    N   N   N   N   N
5,5  | N   N   N    N    N    N   N   N   N   N
4,4  | N   N   N    Y/N  Y/N  N   N   N   N   N
3,3  | Y/N Y/N Y    Y    Y    Y   N   N   N   N
2,2  | Y/N Y/N Y    Y    Y    Y   N   N   N   N

Notes on T,T deviations (H17): the 6*/5*/4* mean "split tens only at very high
true counts vs dealer 4/5/6" — advanced; gate behind the deviation toggle. Default
play for T,T is always Stand (never split).

================================================================================
## SOFT TOTALS
================================================================================

### Baseline (left chart)
Hand | 2   3   4   5   6   7   8   9   10  A
A,9  | S   S   S   S   S   S   S   S   S   S
A,8  | S   S   S   S   Ds  S   S   S   S   S
A,7  | Ds  Ds  Ds  Ds  Ds  S   S   H   H   H
A,6  | H   D   D   D   D   H   H   H   H   H
A,5  | H   H   D   D   D   H   H   H   H   H
A,4  | H   H   D   D   D   H   H   H   H   H
A,3  | H   H   H   D   D   H   H   H   H   H
A,2  | H   H   H   D   D   H   H   H   H   H

### H17 variant (right chart) — primary
Hand | 2   3   4    5    6    7   8   9   10  A
A,9  | S   S   S    S    S    S   S   S   S   S
A,8  | S   S   S    3*   0*   S   S   S   S   S     (Ds appears at the deviation TCs vs 4/5/6)
A,7  | Ds  Ds  Ds   Ds   Ds   S   S   H   H   H
A,6  | 1*  D   D    D    D    H   H   H   H   H
A,5  | H   H   D    D    D    H   H   H   H   H
A,4  | H   H   D    D    D    H   H   H   H   H
A,3  | H   H   H    D    D    H   H   H   H   H
A,2  | H   H   H    D    D    H   H   H   H   H

Notes: A,8 vs 4/5/6 and A,6 vs 2 carry Hi-Lo deviations (the small numbers). Base
action for A,8 is Stand; base for A,6 vs 2 is Hit. Gate the starred plays behind
the deviation toggle.

================================================================================
## HARD TOTALS
================================================================================

### Baseline (left chart)
Tot | 2   3   4   5   6   7   8   9   10  A
17  | S   S   S   S   S   S   S   S   S   S
16  | S   S   S   S   S   H   H   H   H   H
15  | S   S   S   S   S   H   H   H   H   H
14  | S   S   S   S   S   H   H   H   H   H
13  | S   S   S   S   S   H   H   H   H   H
12  | H   H   S   S   S   H   H   H   H   H
11  | D   D   D   D   D   D   D   D   D   D
10  | D   D   D   D   D   D   D   D   H   H
9   | H   D   D   D   D   H   H   H   H   H
8   | H   H   H   H   H   H   H   H   H   H

### H17 variant (right chart) — primary
Tot | 2    3    4    5    6    7   8   9    10   A
17  | S    S    S    S    S    S   S   S    S    S
16  | S    S    S    S    S    H   H   4*   0*   3*
15  | S    S    S    S    S    H   H   H    4*   5*
14  | S    S    S    S    S    H   H   H    H    H
13  | -1*  S    S    S    S    H   H   H    H    H
12  | 3*   2*   0*   S    S    H   H   H    H    -1*
11  | D    D    D    D    D    D   D   D    D    D
10  | D    D    D    D    D    D   D   D    4*   3*
9   | 1*   D    D    D    D    3*  H   H    H    H
8   | H    H    H    H    H    H   H   H    H    H

Notes: starred cells are Hi-Lo deviation indices (e.g. 16 vs 10 = stand at TC 0 or
higher, which is the famous "16 vs 10" deviation; 12 vs 3 = stand at TC +2; etc.).
BASE action ignoring count: 16/15/13/12 follow the baseline table values. Gate the
deviations behind the toggle. The base for 10 vs 10/A and 9 vs 2/7 is Hit/Double
per baseline; the stars only flip them at the indicated counts.

================================================================================
## LATE SURRENDER
================================================================================

### Baseline (left chart)
- 16 vs 9, 10, A → SUR
- 15 vs 10 → SUR

### H17 variant (right chart) — primary
- 17 vs A → SUR
- 16 vs 9 (at deviation), 16 vs 10 → SUR, 16 vs A → SUR
- 15 vs 10 → SUR, 15 vs A → SUR (with deviation markers on some cells)

Surrender is checked BEFORE other actions and only when Late Surrender is enabled
in settings. If surrender is disabled, fall through to the hard-total action.

================================================================================
## INSURANCE / EVEN MONEY
================================================================================
- Baseline: never take insurance / even money.
- H17 + counting: take insurance only when true count is +3 or higher
  (this is the "Insurance at 3+" note in the chart). Gate behind deviation toggle.

================================================================================
## DECISION ORDER (engine must follow)
================================================================================
1. If dealer shows Ace → insurance decision (per setting/deviation).
2. If Late Surrender enabled AND hand qualifies → surrender (unless deviation/
   count says otherwise).
3. If hand is a pair AND splitting allowed → consult pair table (respect DAS for
   Y/N cells, respect re-split limits).
4. Else if hand is soft → consult soft table.
5. Else → consult hard table.
6. D resolves to Hit if doubling not allowed (e.g. after hit, or 9-10-11-only
   rule); Ds resolves to Stand if doubling not allowed.

When the deviation toggle is OFF, ignore all starred values and use the baseline
action under each starred cell. When ON, apply the starred action once the true
count crosses the listed threshold.
