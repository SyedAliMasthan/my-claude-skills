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
version: 1.0.0
team: Indian Bank DC — Infrastructure Operations (Jio Platforms)
---

# Roster Claude Skill

## Overview
This skill encodes all roster-generation rules, WO logic, SLA constraints,
and shift patterns for the Indian Bank DC Infrastructure Operations team.
Use it every time a monthly roster needs to be created or validated.

---

## Team Composition

| Name | Skill | Level | Type |
|---|---|---|---|
| Syed Ali | Linux | L3 | Always Afternoon (A) |
| Purushothaman Kumar | VMware+Windows | L3 | Always Morning (M) |
| Vijayan | OpenShift | L2 | Night-only (permanent) |
| Rajesh Chandran | Backup | L2 | Night-only (permanent) |
| Srinivasan Palraj | VMware+Windows | L2 | Rotational M/A/N |
| Reddy Srinivasulu | Backup | L2 | Rotational M/A/N |
| Saravanan | VMware+Windows | L2 | Rotational M/A/N |
| Dinesh Ananthaneni | Linux | L2 | Day-only (M primary) |
| VaraPrasad | VMware+Windows | L2 | Rotational M/A/N |
| Venkata Raju | Linux | L2 | Special (see below) |

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

### L3 Rules (Syed Ali & Purushothaman)
- Syed Ali → **Always Afternoon (A)**. Never M or N.
- Purushothaman → **Always Morning (M)**. Never A or N.
- L3 WO pattern: **SAT + SUN** every week.
- **Bank Saturday rule**: 1st, 3rd, and 5th Saturday of the month are bank
  working days. At least one L3 must work on each bank Saturday.
  Rotate between Syed (works 3rd SAT) and Purushothaman (works 1st and 5th SAT).
  On non-bank Saturdays (2nd, 4th), both L3 can be WO.

### Night-Only Workers (Vijayan & Rajesh)
- **Vijayan**: Night (N) every working day. WO = **MON + TUE** every week. Fixed, never changes.
- **Rajesh**: Night (N) every working day. WO = **TUE + WED** every week (consecutive).
  - WO must be consecutive weekdays. Cannot split across non-adjacent days.
  - TUE+WED chosen because: MON+TUE clashes with Vijayan (both Night workers off = 0 Night coverage);
    WED+THU is unsafe (THU already has 3 others on WO → SLA breach).
  - On Rajesh WO days (TUE), Vijayan is also WO → Night must be covered by rotational L2.

### Day-Only Worker (Dinesh)
- Dinesh → **Morning (M) primarily**, Afternoon (A) occasionally. **Never Night (N)**.
- WO = **SUN + MON** every week. Sunday is an exception allowed for Dinesh only
  (all other L2 cannot take SAT/SUN WO).
- ES (Extended Shift) may be assigned occasionally.

### Rotational L2 (Srinivasan, Srinivasulu, Saravanan, VaraPrasad)
- Rotate through M, A, N in **weekly blocks**.
- **Shift order within a week is strictly M → A → N**. No reversal allowed:
  - Once on A this week → remaining days must be A or N only (not back to M).
  - Once on N this week → remaining days must be N only (not back to M or A).
- After a Night week → WO days act as rest before next week starts fresh.
- Max **2 Night shifts per day** across entire team.

### Special: Venkata Raju
- Works **Night (N) for days 1–8** of the month only.
- Has **2–3 WOs** within that 8-day window.
- From day 9 onwards → **NA** (Not Available) for the rest of the month.
- On days when Venkata covers Night (days 1–8), Vijayan is the 2nd Night worker.

---

## WO Assignment Rules

### General Rules
1. **No SAT/SUN WO for L2** — except Dinesh's SUN (his fixed pattern).
2. **WO must be consecutive days** — never split across non-adjacent weekdays.
3. **Target 8–9 WOs per L2 per month** depending on month length.
4. **2 WOs per week** per L2 as the standard rhythm.
5. **No SLA breach** — minimum 6 people working every single day.

### Fixed WO Patterns (derive actual day numbers from the month's calendar)
| Engineer | WO Pattern | Notes |
|---|---|---|
| Vijayan | MON + TUE | Fixed, permanent Night-only |
| Rajesh | TUE + WED | Fixed, consecutive, avoids Vijayan overlap |
| Srinivasan | THU + FRI | Rotational L2 |
| Srinivasulu | TUE + WED | Rotational L2 |
| Saravanan | THU + FRI | Rotational L2 |
| VaraPrasad | WED + THU | Rotational L2 |
| Dinesh | SUN + MON | Day-only, SUN exception allowed |

### SLA Validation Logic
- Count working people per day = Total team − WO − Leave − NA
- Minimum = **6 people working every day** (including L3)
- Max **2 Night workers per day** (caps Night shift at exactly 2)
- On Vijayan+Rajesh joint WO days (every TUE): Night must come entirely
  from rotational L2 — ensure at least 1 rotational L2 is on Night that day.

### WO Safety Check Before Finalising
For each proposed WO day, verify:
- Adding this person's WO does not drop working count below 6.
- THU is typically unsafe (Srinivasan + Saravanan + VaraPrasad all WO on THU).
- SUN is typically unsafe (Syed + Purushothaman + Dinesh all WO on SUN).
- Rajesh cannot take MON/TUE WO (Vijayan already WO → 0 dedicated Night workers).
- Rajesh cannot take THU WO (3 already off → SLA breach).

---

## Shift Rotation Rule (M → A → N)

Within any given week for rotational L2:
```
Valid:   M M M M M  (all morning)
Valid:   A A A A A  (all afternoon)
Valid:   N N N N N  (all night)
Valid:   M M A A A  (morning then afternoon)
Valid:   M A A N N  (morning → afternoon → night)
Valid:   A A N N N  (afternoon then night)
Invalid: M A M ...  (back to M after A) ✗
Invalid: A M ...    (A then M) ✗
Invalid: N A ...    (N then A) ✗
Invalid: N M ...    (N then M) ✗
```
Once the shift level goes up within a week, it cannot come back down.

---

## Excel Output Format (match existing template)

### Sheet structure
- **Row 1**: Title — "Attendance Report for THE MONTH OF [MONTH YEAR]"
- **Row 2**: Headers — Skill | Mob No | Location | Name | D1..D31 | WO | G | M | A | N | PL Applied | DH Availed | CO Available | No of working Days
- **Row 3**: Day numbers (1 to 31)
- **Rows 4+**: One row per engineer with shift codes per day
- **Bottom section**: Shift count summary rows (M, A, N, ES, G, WO, PL, CO, DH, L counts per day)

### Color coding
- M (Morning): Light blue `#D6E8FF`
- A (Afternoon): Light green `#D5F5E3`
- N (Night): Light orange `#FCE4D6`
- WO (Week Off): Light green `#EAF4EA`
- L (Leave): Light yellow `#FFF9C4`
- NA (Not Available): Light grey `#EEEEEE`
- Header rows: Dark navy `#1F4E79`
- Weekend day name cells: Red `#C00000`
- Weekday day name cells: Blue `#2E75B6`

### Summary columns (after day columns)
- WO count
- G (General) count
- M count
- A count
- N count
- PL Applied
- DH Availed
- CO Available
- No of Working Days

### Per-person working days formula
`= Total days − WO − Leave − NA − CO − DH`

---

## Month Preparation Checklist

Before generating any roster:
1. **Confirm month and year** → map exact day names (MON–SUN) to dates
2. **Identify bank Saturdays** → 1st, 3rd, 5th Saturday
3. **Collect approved leaves** → per person, exact days
4. **Note team changes** → any additions, removals, or role changes
5. **Derive WO days** from fixed patterns mapped to the new month's calendar
6. **Validate WO safety** → run SLA check for all 31/30 days
7. **Assign shifts** → L3 first (fixed), Night-only next, then rotational L2
8. **Validate shift rotation** → no M→A reversal, no A→M, no N→M/A within week
9. **Validate Night cap** → max 2N per day
10. **Generate Excel** → match existing template format exactly
11. **Add shift count summary** → per person M/A/N/WO/L/NA totals at end of sheet

---

## Night Coverage Priority on Critical Days

On days where both Vijayan and Rajesh are WO (every TUE in the month):
- Rotational L2 must cover Night
- Prefer: assign Srinivasulu or VaraPrasad to Night that week
- Srinivasan and Saravanan share THU+FRI WO so they are available on TUE

On days where Vijayan is WO and Rajesh is working (MON):
- Rajesh alone covers Night (1 dedicated + rotational if needed for 2N)

---

## Reference: April 2026 Actual Roster (Source of Truth)

Extracted from uploaded Book1.xlsx (DC_DR_April_Final):

```
Syed Ali:       A A L WO WO A A A A A WO WO A L  A A A A WO WO A A A A WO WO M A A A
Purushothaman:  M M M M WO WO M M M M WO WO CO L  M M M WO WO M M M L  M WO WO M M M M
Srinivasan:     A N WO WO M M A A WO WO A A A A A  WO WO M M A A A WO WO A A A L  L WO
Venkata Raju:   N WO WO WO A N N N NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA
Rajesh:         WO A N N N N N WO N N N N N N WO N N N N WO N N WO WO WO WO L  A N N
Srinivasulu:    WO WO A A A A WO WO A A A A M WO WO A A A A N WO WO N N N N N WO A A
Saravanan:      M WO WO M M M M M WO WO M M N N N WO WO M M A WO WO A A A A A N WO A
Dinesh:         M M M A WO WO M M M M WO WO M ES M M M M WO M M M M M M WO WO M M WO
Vijayan:        N N N N N WO WO N N N N N WO WO N N N N N WO WO N N N N N WO WO N N
VaraPrasad:     WO WO A A A WO WO L L M M M M M WO WO M A A N N WO M M M ES N N WO M
```

Shift counts (April actuals):
- Vijayan: N=22, WO=8
- Rajesh: N=19, A=2, WO=8, L=1
- Syed Ali: A=19, M=1, WO=8, L=2
- Purushothaman: M=19, WO=8, L=2, CO=1
- Dinesh: M=20, A=1, WO=8, ES=1
- Srinivasan: A=14, M=4, N=1, WO=9, L=2
- Srinivasulu: A=14, M=1, N=6, WO=9
- Saravanan: M=10, A=7, N=4, WO=9
- VaraPrasad: M=10, A=5, N=4, WO=8, L=2, ES=1
