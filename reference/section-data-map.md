# Editorial Dashboard — Section → Workbook Data Map

Read **by header name** (column letters shown for cross-check only — current Jun-2026 schema;
they shift if the template changes). All money displays as `S$`. "Owner" = the policy owner;
"insured" = each distinct Life Insured ID.

## Masthead
- Title = owner Name (`1_Client_Profile` B).
- Standfirst "N active policies across one household" = count of `2_All_Policies` rows.
- who-line: Age (`4_Coverage_Summary` E or DOB), Occupation (`1` H), Income/mo (`1` Q or
  `Basic_Info`), Expenses/mo (`1` R), Liabilities (`1` S), Joint top-up/yr (`1` V × 12).

## 01 · household  ← `1_Client_Profile` + `2_All_Policies`
- One `.person` row per profile: Name, Role, b. DOB, Age, "Life insured on N policies"
  = COUNT `2_All_Policies` where Life Insured ID (C) = this person.
- Joint-account footnote: balance (`1` U), monthly top-up (`1` V), yearly (V×12),
  "funds N of M policies" = count `2` where Source of Premiums (AR) = Joint Account.

## 02 · cost  ← `4_Coverage_Summary` + `1` + `7_Payment_Milestone`
- Big figs: Annual premiums = Σ Total Annual (`4` J); /mo = ÷12; Against own income % =
  annual ÷ owner annual income (`4` H owner); Paid to date = Σ Total Paid (`4` K);
  Joint headroom = joint capacity (V×12) − joint-funded annual premiums.
- By-purpose table: split `2` Annual Premium (R) by Policy Category (I) into Protection vs
  Wealth Accumulation; yearly, monthly (÷12), share of owner income.
- Who-pays split bar: From self (`4` X) vs From Joint Account (`4` Y), household totals.

## 03 · cover  ← `4_Coverage_Summary` + `Basic_Info`
- Snapshot card per insured: Sum assured base (`4` N), Death (`4` O), TPD (`4` P),
  CI total (`4` Q), early CI (`4` R), Accidental death (Σ `2` Accidental Death Benefit V for
  that insured), H&S plan names (`2` Plan Name where Type=H&S for that insured).
  Owner card = `.snap.owner` (teal).
- Needs calculator cards (Death / TPD / ECI / CI): Recommended vs Currently-have vs
  Surplus, % bar ← `Basic_Info` needs-calculator block (recommended levels) + `4` (have).
- Benchmark table: per-insured Have vs Recommended for each benefit.

## 04 · age  ← `2_All_Policies`
- Owner step chart (`c-syt`): three series — Death·TPD (`2` Death W), CI total (`2` CI Y),
  Early CI (`2` ECI Z). Step-down points at term-end ages and Multiplier End Age (`2` AA);
  whole-life base continues flat. Event lines at the ages cover drops (term/multiplier ends).
  **Keep the two-row staggered event labels.** Put the per-line end-values as a top-right
  "AT AGE 100" legend, not stacked at the converging chart floor.
- Each-child chart (`c-kids`): base sum assured ×5 until multiplier-end-age (`2` AA), then base.
- **Multiple-insured rule:** if ≥2 insured carry *different* coverage shapes, give each life
  insured its own dropdown-selectable cover-over-time chart (one `<select>` switching the SVG).
  Insured who share an identical shape (e.g. the two daughters here) stay combined in a single
  chart. So: all-different → per-insured dropdown; some-identical → group the matching insured.

## 05 · runway  ← `7_Payment_Milestone`
- Cash-premium chart (`c-pay`): yearly Total Cash Premium (`7` timeline col B) vs Year
  (col A), 2026→2120. Mark joint capacity line (V×12). Lifetime cash outlay = `7` total.
  (Fully CPF-Funded / Medisave portions are excluded by the workbook — note in caption.)
- Completion schedule: per policy, Premium End year (`2` Premium End Date K → YEAR) and the
  remaining yearly bill after it drops off; sort ascending by year.

## 06 · wealth & payouts  ← `5_Maturity_Schedule` + `8_Payout_Milestone` + `6_Passive_Income_Timeline`
Render only if wealth/payout policies exist (`2` Maturity Payout Plan? AU = Yes).
- Mini-figs: Projected maturity (Σ Total Illustrated `5` N or `8` maturity), Coupons/yr
  (`8` coupons / `2` AE), Lifetime inflow (coupons+maturity, `8` cumulative), Return on cash
  & IRR (`8` summary rows).
- Per-plan maturity table: Plan · insured, Yearly premium (`2` R), Finishes (`2` K year),
  Guaranteed (`5` L / `2` AB), Non-guaranteed (`5` M / `2` AC). Total row.
- Payout chart (`c-payout`): coupon bars then the maturity spike over time ← `8` timeline
  (no in-chart cumulative line). The maturity bar is always drawn on top of the coupon bars in a
  distinct colour (rust) with a thin paper outline, so an overlap in the same year reads cleanly.
- Cumulative payout table (below `c-payout`): 10-year increments to the last maturity year —
  columns By year · Coupons to date · Maturity received · Cumulative payout, total row at the
  final maturity. Replaces the old in-chart cumulative line.
- Passive-income chart (`c-pi`): monthly income if coupons drawn ← `6` (start/end age, monthly).

## 07 · notes  ← derived (advisor-refined)
Ranked list from the flags in SKILL.md Step 4. Each `.note`; use `.note.flag` for items
needing action (missing policy number, missing illustration, etc.).

## 08 · register  ← `2_All_Policies` (every column)
Accordion per insured (owner first, teal). Each `.pol` card (`.pol.wealth` gold edge for
Wealth Accumulation):
- chips: Insurer (E), Policy Type (H), CPF/Medisave (Source AR / CPF Premium? AQ).
- Plan Name (G), Policy Number (F → "no. not recorded" if blank), Status (M).
- psec rows: Annual premium (R), Pays until (Premium End K → year, or "lifetime" if 9999),
  Funded by (Source AR), Projected maturity (AD/AC) for wealth.
- benefit rows: Death/TPD (W/X), CI/ECI (Y/Z), Base sum assured (U), Multiplier ends
  (age from AA), Accidental death (V).
- Riders (AL), Remarks (AK).

## Notes on lifetime / placeholders
- Premium-end or maturity year of 9999 → display "lifetime" (memory `lifetime-date-convention`).
- Blank policy number → "no. not recorded" + a Notes flag.
- ILP guaranteed value is typically S$0 by nature; missing illustration → "pending".
