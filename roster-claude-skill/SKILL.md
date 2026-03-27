---
name: roster-claude-skill
description: >
  Use this skill whenever creating, updating, or validating a monthly shift
  roster for an infrastructure operations team (Jio Platforms / Indian Bank DC).
  Triggers: any request mentioning "roster", "shift schedule", "monthly schedule",
  "weekoff", "WO assignment", "SLA headcount", or "shift rotation".
  Covers: L3/L2 role rules, WO logic, shift rotation (M→A→N), SLA validation,
  bank Saturday rules, night-only workers, and Excel output generation.
author: Syed Ali Masthan
version: 2.0.0
team: Indian Bank DC — Infrastructure Operations (Jio Platforms)
last_updated: March 2026
---

# Roster Claude Skill — v2.0

## Overview
This skill encodes ALL roster-generation rules, WO logic, SLA constraints,
and shift patterns for the Indian Bank DC Infrastructure Operations team.
Use it every time a monthly roster needs to be created or validated.
Source of truth: April 2026 final roster (Book1.xlsx).

---

## Team Composition

| Name | Skill | Level | Shift Type | WO Pattern |
|---|---|---|---|---|
| Syed Ali | Linux | L3 | Always Afternoon (A) | SAT+SUN (bank SAT comp on MON) |
| Purushothaman Kumar | VMware+Windows | L3 | Always Morning (M) | SAT+SUN (bank SAT comp on MON) |
| Vijayan | OpenShift | L2 | Night-only permanent | MON+TUE every week |
| Rajesh Chandran | Backup | L2 | Night-only permanent | TUE+WED every week |
| Srinivasan Palraj | VMware+Windows | L2 | Rotational M/A (N only for SLA gap) | THU+FRI every week |
| Reddy Srinivasulu | Backup | L2 | Rotational M/A (N only for SLA gap) | TUE+WED every week |
| Saravanan | VMware+Windows | L2 | Rotational M/A (N only for SLA gap) | THU+FRI every week |
| Dinesh Ananthaneni | Linux | L2 | Day-only M/A (never Night) | SUN+MON every week |
| VaraPrasad | VMware+Windows | L2 | Rotational M/A (N only for SLA gap) | WED+THU every week |
| Venkata Raju | Linux | L2 | Special — see below | 2-3 WOs in days 1-8 only |

---

## Shift Definitions

| Code | Shift | Timing | Duration |
|---|---|---|---|
| M | Morning | 07:00 AM – 04:00 PM | 9 hrs |
| A | Afternoon | 01:00 PM – 10:00 PM | 9 hrs |
| N | Night | 09:30 PM – 07:30 AM | 9 hrs |
| ES | Extended Shift | 07:00 AM – 07:00 PM | 12 hrs |
| WO | Week Off | — | — |
| L | Leave | — | — |
| CO | Comp Off | — | — |
| DH | Declared Holiday | — | — |
| NA | Not Available | — | — |

---

## Core Shift Rules

### Rule 1 — L3 Fixed Shifts
- **Syed Ali → Always Afternoon (A)**. Never M or N.
- **Purushothaman Kumar → Always Morning (M)**. Never A or N.
- L3 are never assigned Night under any circumstance.

### Rule 2 — Night-Only Workers (Vijayan & Rajesh only)
- **Vijayan** = Night (N) every working day. These two are the ONLY
- **Rajesh Chandran** = Night (N) every working day. dedicated permanent night workers.
- No other engineer is permanently assigned Night.
- Vijayan WO = **MON + TUE** every week (fixed, consecutive, never changes).
- Rajesh WO = **TUE + WED** every week (fixed, consecutive).
  - TUE+WED chosen because: MON+TUE clashes with Vijayan (both off = zero dedicated Night);
    WED+THU unsafe (THU already has 3 L2 on WO → SLA breach).
  - Every TUE: both Vijayan AND Rajesh are on WO → Night must be covered
    by rotational L2 on those days (SLA-driven Night assignment).

### Rule 3 — Rotational L2 Shift Assignment
Applies to: Srinivasan Palraj, Reddy Srinivasulu, Saravanan, VaraPrasad.

- **Primary rotation = Morning (M) and Afternoon (A) in weekly blocks.**
- **Night (N) is allowed** but ONLY when needed to meet the 2N/day SLA —
  specifically on days when Vijayan and/or Rajesh are on WO.
- Night is NOT their default. It is a gap-fill role only.
- Within any week, shifts follow strict **M → A → N** order — no reversal:
  - Once on A this week → remaining days must be A or N (not back to M)
  - Once on N this week → remaining days must be N (not back to M or A)
  - Valid: M M M M M / A A A A A / M M A A A / M A A N N / A A N N N
  - Invalid: M A M / A M / N A / N M (any backward step) ✗
- After a Night week → WO days act as natural rest before next week resets.

### Rule 4 — Dinesh (Day-Only)
- Dinesh = **Morning (M) primarily**, Afternoon (A) occasionally.
- **Never Night under any circumstance** — not even for SLA gap fill.
- ES (Extended Shift 7AM–7PM) may be assigned occasionally.
- WO = **SUN + MON** every week. SUN is an exception for Dinesh only
  (all other L2 must not take SAT/SUN WO).

### Rule 5 — Venkata Raju (Special)
- Works **Night (N) for days 1–8** of the month only.
- Has **2–3 WOs** within that 8-day window.
- From day 9 onwards → **NA** (Not Available) for the rest of the month.

---

## WO Assignment Rules

### WO Cap Rule (CRITICAL)
- **WO cap per engineer = total number of SAT+SUN days in that month.**
- Example: May 2026 has 5 SAT + 5 SUN = 10 → WO cap = 10 for everyone.
- No engineer may exceed this count regardless of their pattern.
- If the pattern generates more WOs than the cap → trim the excess.
  When trimming, ask the engineer/manager which days to remove.

### Consecutive WO Rule (CRITICAL)
- **No more than 2 consecutive WO days for any engineer.**
- All standard patterns (MON+TUE, TUE+WED, THU+FRI, WED+THU, SUN+MON)
  produce exactly 2 consecutive days per week — this is by design.
- Never assign 3 or more consecutive WO days.

### No Weekend WO for L2 (except Dinesh's SUN)
- L2 engineers must NOT have SAT or SUN as WO days.
- **Exception**: Dinesh takes SUN as part of his SUN+MON pattern.
- All other L2 WOs must be weekdays only.

### Bank Saturday Rule (L3 only)
- 1st, 3rd, and 5th Saturday of the month = bank working days.
- At least one L3 must work on each bank Saturday.
- **Rotation**: Purushothaman works 1st and 5th SAT; Syed works 3rd SAT.
- **Compensation**: When an L3 works a bank Saturday → give WO on the
  following Monday as comp off.
  - If no Monday follows in that month (e.g., 5th SAT is last days) →
    use the nearest available weekday before that Saturday (e.g., Friday).
- On non-bank Saturdays (2nd and 4th) → both L3 take WO normally.

### Fixed WO Patterns (map to actual dates each month)

| Engineer | WO Pattern | Notes |
|---|---|---|
| Syed Ali L3 | SAT + SUN every week | Remove bank SAT, add comp MON |
| Purushothaman L3 | SAT + SUN every week | Remove bank SAT, add comp MON |
| Vijayan L2 | MON + TUE | Permanent Night-only |
| Rajesh Chandran L2 | TUE + WED | Permanent Night-only, consecutive |
| Srinivasan Palraj L2 | THU + FRI | Rotational M/A, N for SLA gap only |
| Reddy Srinivasulu L2 | TUE + WED | Rotational M/A, N for SLA gap only |
| Saravanan L2 | THU + FRI | Rotational M/A, N for SLA gap only |
| VaraPrasad L2 | WED + THU | Rotational M/A, N for SLA gap only |
| Dinesh L2 | SUN + MON | Day-only, SUN exception |

---

## SLA Rules

### Daily Headcount SLA
- **Minimum 6 people working every day** (includes L3 + L2).
- This is the Jio commitment to Indian Bank.
- Count working = total team − WO − Leave − NA − CO − DH.

### Night Shift SLA
- **Exactly 2 Night workers per day** (max and target).
- Primary Night: Vijayan + Rajesh cover 2N on their working days.
- On Vijayan/Rajesh WO days: rotational L2 fill the Night gap.
- Every TUE: both Vijayan+Rajesh on WO → rotational L2 must provide 2N.
- Every MON: Vijayan WO, Rajesh works → Rajesh = 1N, rotational L2 fill 2nd N.

### Morning and Afternoon SLA
- **Minimum 2 Morning workers per day.**
- **Minimum 2 Afternoon workers per day.**
- L3 contribute: Syed=A always, Puru=M always (when not on WO/Leave).

### SLA Unsafe Days (by pattern — always check these)
- **THU**: Srinivasan + Saravanan + VaraPrasad all on WO → 3 off. Adding
  Rajesh WO on THU would be 4 off → unsafe. Rajesh must NOT use THU.
- **SUN**: Syed + Puru + Dinesh all on WO → 3 off. No additional L2 WO on SUN.
- **TUE**: Vijayan + Rajesh + Srinivasulu all on WO → rotational L2 must cover Night.

---

## WO Safety Validation Checklist

Before finalising WOs each month:
1. Map all pattern-based WO days to actual dates of the new month.
2. Count total WOs per person → must not exceed SAT+SUN count for that month.
3. Check no 3+ consecutive WO days for anyone.
4. Check L2 WOs are weekdays only (except Dinesh's SUN).
5. For each day: count people on WO + Leave + NA → working must be ≥ 6.
6. For each day: count Night workers → must be exactly 2 (or flag if <2).
7. On bank Saturdays → verify at least 1 L3 is working.
8. On Vijayan+Rajesh joint WO days (every TUE) → verify rotational L2 covers 2N.

---

## Night Coverage on Critical Days

| Day Pattern | Vijayan | Rajesh | Fixed Night | Action Needed |
|---|---|---|---|---|
| MON | WO | Working (N) | 1N | 1 rotational L2 → N |
| TUE | WO | WO | 0N | 2 rotational L2 → N |
| WED | Working (N) | WO | 1N | 1 rotational L2 → N |
| THU–FRI–SAT–SUN | Working (N) | Working (N) | 2N | No action needed |

Rotational L2 available for Night gap-fill (not on WO those days):
- MON: Srinivasan ✓, Saravanan ✓, VaraPrasad ✓ (Srinivasulu WO on TUE/WED — check)
- TUE: Srinivasan ✓, Saravanan ✓ (Srinivasulu WO, VaraPrasad WO on WED/THU)
- WED: Srinivasan ✓, Saravanan ✓ (check their THU/FRI WO pattern)

---

## Month Preparation Checklist

1. Confirm month and year → map exact day names to all dates.
2. Count SAT+SUN in the month → set WO cap for all engineers.
3. Identify 1st, 3rd, 5th Saturdays → bank working Saturdays.
4. Collect approved leaves per person.
5. Note team changes (joining, leaving, role changes).
6. Compute WO days from fixed patterns mapped to new month's calendar.
7. Apply bank Saturday rule for L3 → remove bank SAT from WO, add comp MON.
8. Validate WO safety: cap check, consecutive check, SLA check (min 6/day).
9. Assign shifts:
   - L3: fixed (Syed=A, Puru=M) with WO/Leave.
   - Vijayan: N always, WO on pattern days.
   - Rajesh: N always, WO on pattern days.
   - Dinesh: M/A rotation, WO SUN+MON, never Night.
   - Rotational L2: M/A weekly blocks, N only for SLA gap-fill on Vijayan/Rajesh WO days.
10. Validate shift rotation: no backward step (M→A→N only within each week).
11. Validate Night cap: exactly 2N per day.
12. Generate Excel in existing template format.
13. Add per-person shift count summary (M/A/N/WO/L/NA/Working Days).

---

## Excel Output Format

### Sheet structure
- Row 1: "Attendence Report for THE MONTH OF [MONTH YEAR]"
- Row 2: Skill | Mob No | Location | Name | D1..D(n) | WO | G | M | A | N | PL Applied | DH Availed | CO Available | No of working Days
- Row 3: Day numbers (1 to n)
- Rows 4+: One row per engineer with shift codes per day
- Bottom: Shift type count rows (M/A/N/ES/G/WO/PL/CO/DH/L counts per day)

### Color coding
| Code | Color | Hex |
|---|---|---|
| M | Light blue | #D6E8FF |
| A | Light green | #D5F5E3 |
| N | Light orange | #FCE4D6 |
| WO | Light grey-green | #EAF4EA |
| L | Light yellow | #FFF9C4 |
| NA | Light grey | #EEEEEE |
| Header | Dark navy | #1F4E79 |
| Weekend day names | Red | #C00000 |
| Weekday day names | Blue | #2E75B6 |

### Summary columns (right side of day columns)
WO count | G count | M count | A count | N count | PL Applied | DH Availed | CO Available | No of Working Days

### Working days formula
Total days − WO − Leave − NA − CO − DH

---

## Reference: April 2026 Actual Roster (Source of Truth)

From uploaded Book1.xlsx (sheet: DC_DR_April_Final):

```
Days:           1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
Day names:     WED THU FRI SAT SUN MON TUE WED THU FRI SAT SUN MON TUE WED THU FRI SAT SUN MON TUE WED THU FRI SAT SUN MON TUE WED THU

Syed Ali:       A  A  L WO WO  A  A  A  A  A WO WO  A  L  A  A  A  A WO WO  A  A  A  A WO WO  M  A  A  A
Purushothaman:  M  M  M  M WO WO  M  M  M  M WO WO CO  L  M  M  M WO WO  M  M  M  L  M WO WO  M  M  M  M
Srinivasan:     A  N WO WO  M  M  A  A WO WO  A  A  A  A  A WO WO  M  M  A  A  A WO WO  A  A  A  L  L WO
Venkata Raju:   N WO WO WO  A  N  N  N NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA
Rajesh:        WO  A  N  N  N  N  N WO  N  N  N  N  N  N WO  N  N  N  N WO  N  N WO WO WO WO  L  A  N  N
Srinivasulu:   WO WO  A  A  A  A WO WO  A  A  A  A  M WO WO  A  A  A  A  N WO WO  N  N  N  N  N WO  A  A
Saravanan:      M WO WO  M  M  M  M  M WO WO  M  M  N  N  N WO WO  M  M  A WO WO  A  A  A  A  A  N WO  A
Dinesh:         M  M  M  A WO WO  M  M  M  M WO WO  M ES  M  M  M  M WO  M  M  M  M  M  M WO WO  M  M WO
Vijayan:        N  N  N  N  N WO WO  N  N  N  N  N WO WO  N  N  N  N  N WO WO  N  N  N  N  N WO WO  N  N
VaraPrasad:    WO WO  A  A  A WO WO  L  L  M  M  M  M  M WO WO  M  A  A  N  N WO  M  M  M ES  N  N WO  M
```

April 2026 actuals:
- Vijayan:      N=22 WO=8
- Rajesh:       N=19 A=2 WO=8 L=1
- Syed Ali:     A=19 M=1 WO=8 L=2
- Purushothaman:M=19 WO=8 L=2 CO=1
- Dinesh:       M=20 A=1 WO=8 ES=1
- Srinivasan:   A=14 M=4 N=1 WO=9 L=2
- Srinivasulu:  A=14 M=1 N=6 WO=9
- Saravanan:    M=10 A=7 N=4 WO=9
- VaraPrasad:   M=10 A=5 N=4 WO=8 L=2 ES=1

Note: Srinivasan/Srinivasulu/Saravanan/VaraPrasad show some N in April
because they filled Night SLA gaps on Vijayan/Rajesh WO days.
This is correct and expected — Night is allowed for SLA gap-fill only.

---

## Key Decisions Log

| Decision | Rule | Reason |
|---|---|---|
| Rajesh WO = TUE+WED | Consecutive, avoids MON/TUE | MON+TUE = Vijayan WO → 0 dedicated Night. THU unsafe (3 others off). |
| WO cap = SAT+SUN count | Per month | Fairness — WO mirrors natural weekend count of that month. |
| No 3+ consecutive WO | Quality of life | Engineer cannot have 3+ days off consecutively. |
| Rotational L2 Night = SLA gap only | Not primary | Primary Night = Vijayan+Rajesh only. Others fill when needed. |
| L3 bank SAT comp = next MON | Policy | Working bank SAT earns a comp Monday off. |
| No SAT/SUN WO for L2 | Policy | Except Dinesh's SUN which is his fixed pattern exception. |
| M→A→N weekly order | Shift health | No juggling shifts within a week. Once progressed, no reversal. |
