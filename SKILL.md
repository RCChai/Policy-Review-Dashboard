---
name: policy-dashboard-editorial
description: >
  Generate a client policy-review dashboard in the EDITORIAL house style (the current Song
  Ying Ting v2 design — Fraunces serif, paper/ink palette, magazine layout). Trigger on
  "/policy-dashboard-editorial", "build editorial dashboard", "editorial policy review",
  "build the v2 dashboard for [client]", or when Ric asks for a dashboard in the new style.
  This is the CURRENT preferred dashboard style; the legacy navy/teal `policy-dashboard`
  skill is kept only as a fallback during migration. When unsure which style, ask Ric.
---

# Policy Dashboard — Editorial Style

Generates one self-contained `.html` file (no build step, no external deps except Google
Fonts) reproducing the editorial design in
`reference/example-editorial-dashboard.html` — the canonical design checkpoint. **Read that
file first**; clone its structure, CSS and chart code, then swap in the new client's data.

Environment: local Windows + openpyxl (NOT the cloud `extract-text`/`/mnt/` setup the legacy
skill assumes). Use **PowerShell** for all filesystem/render work — the Bash tool's `/mnt/c`
mount is not reliable on this machine.

**Iterating on an existing dashboard** (Ric's common ask): he drops change requests as
**screenshots in `…/Client Policy Review (NEW) 2026/Client Dashboard Build/`, with the
instruction written AS the filename** — read that folder first. For each change: back up the
live HTML to its sibling `_backups`, apply to the live file, render-check with headless Edge
(`msedge --headless=new --screenshot`, crop the changed band), then **sync into this skill** —
copy live → `reference/example-editorial-dashboard.html` (kept byte-identical) and fold any
new general rule into Step 3 / section-data-map.

---

## Step 0 — Read + recalc the workbook
1. The consolidation workbook is openpyxl-written, so cached values may be stale/missing.
   **Recalculate first** (Excel COM or LibreOffice headless `--convert-to`) so every formula
   has a fresh value — recipe in memory `excel-com-recalc-available`. Check no `~$*` lock
   file (Ric not editing it in Excel) before touching it.
2. Read with `openpyxl(..., data_only=True)`. Read **by header name, not column letter** —
   the dashboard is immune to the Sheet 2 column shift, but Sheet 8 row positions moved
   (see `sheet2-column-shift-jun2026`), so locate rows by content, not fixed index.
3. Sheets to pull (full map: `reference/section-data-map.md`):
   `1_Client_Profile`, `2_All_Policies`, `4_Coverage_Summary`, `Basic_Info`,
   `5_Maturity_Schedule`, `7_Payment_Milestone`, `6_Passive_Income_Timeline`, `8_Payout_Milestone`.

## Step 1 — Determine shape
- Count household members (life-insured) from `1_Client_Profile`; one coverage-snapshot card
  and one register accordion per insured. Owner card uses the teal header (`.snap.owner`,
  `.acc.owner`).
- Wealth & payout section (06) and passive-income chart only render if there are wealth/
  payout policies; otherwise omit or grey out.
- Identify children (Is Child Client? = Yes) for the daughters/multiplier logic.

## Step 2 — Build the 8 sections
Clone section-for-section from the reference file. Sections (snum · id · source):
1. `household` — members, ages, roles, joint account ← Sheet 1
2. `cost` — premium totals, % of income, paid-to-date, joint headroom, by-purpose table,
   who-pays split ← Sheet 4 + Sheet 1 + Sheet 7
3. `cover` — coverage snapshot cards by insured + needs-calculator cards + benchmark table
   ← Sheet 4 + Basic_Info
4. `age` — coverage-over-time step charts (owner + each child) ← Sheet 2 (multiplier/term ends)
5. `runway` — cash-premium timeline chart + completion schedule ← Sheet 7
6. `wealth` — maturity/coupon mini-figs, per-plan maturity table, payout + passive-income
   charts ← Sheet 5 + Sheet 8 + Sheet 6
7. `notes` — ranked advisor notes & flags (drafted from data, see Step 4)
8. `register` — accordion of every policy grouped by insured ← Sheet 2 (all fields)

## Step 3 — Charts
Reuse the SVG-in-JS generators verbatim from the reference file (`c-syt` coverage step,
`c-kids` daughters, `c-pay` runway, `c-payout` payout, `c-pi` passive income). They are plain
SVG string builders — feed them the client's series/points. Keep the `spread()` de-overlap
helper and the **two-row staggered event labels** in the coverage chart (don't regress the
label-clump fix). Palette constants for charts: INK `#1F1B16`, TEAL `#1F6F6B`, GOLD `#C99A3F`,
OX `#8A3324`, FAINT `#8B8270`, RULE `#D9D1C0`, PAPER `#F7F3EC`.

Chart-specific rules (latest):
- **Cover-over-time (`c-syt` / `c-kids`):** put the per-line end-values as a top-right
  "AT AGE 100" legend, not stacked at the converging chart floor. **Multiple-insured:** if ≥2
  insured carry *different* coverage shapes, render a per-insured `<select>` dropdown that swaps
  the chart; insured sharing an identical shape stay combined in one chart (as the two daughters
  do here). All-different → dropdown; some-identical → group the matches.
- **Payout (`c-payout`):** coupon bars + the maturity spike only — **no in-chart cumulative
  line.** Maturity bar always drawn on top of the coupons, distinct colour (rust), thin paper
  outline. Show cumulative payout as a **table below** the chart in 10-year increments to the
  last maturity year (By year · Coupons to date · Maturity received · Cumulative payout + total).
- **Passive income (`c-pi`):** tier labels standardised to the right edge, stacked; stagger
  further right if they would collide.

## Step 4 — Narrative (data-drafted, advisor-refined)
The standfirst, each section's italic `.lede`, chart captions and the Notes list are
editorial prose. Draft them from the data and the flags below, then mark anything bespoke
`[ADVISOR: confirm]`. Always flag (mirrors policy-summary-report): lapsed policies; premium
end within 12 months; $0 CI/TPD; minor beneficiary without trustee; Owner-is-Payor=No with
blank payor; joint-account near/over capacity; premiums > income; missing policy numbers or
maturity illustrations. Tone: factual, plain English, address client by first name; never
"shortfall/insufficient" — use "met / surplus / review recommended".

## Step 5 — Output
Save `[ClientName]_Policy_Review_v2.html` (or `_Dashboard_[YYYYMMDD].html`) to the client's
folder under `…\Client Policy Review (NEW) 2026\[Client]\`. Back up any existing dashboard to
that client's `_backups` first (memory `backup-before-editing-files`). Verify by rendering
with headless Edge and screenshotting the sections (see how it was checked for the stagger
fix). Then run the **qa-check** skill (HTML sequence) — its design check now covers this
editorial palette.

---

## Design system (quick reference — full CSS in the reference file)
- Fonts: **Fraunces** (serif display/lede, italic), **Archivo** (body/caps), **Spline Sans
  Mono** (all numbers, `.num`, tabular-nums).
- Palette: paper `#F7F3EC`, card `#FBF8F2`, ink `#1F1B16`, ink2 `#57503F`, faint `#8B8270`,
  rule `#D9D1C0`; accents ox `#8A3324`, teal `#1F6F6B`, gold `#C99A3F`.
- Layout: `.page` max-width 1020px; sticky wrapping `nav`; numbered sections (`.snum`);
  big-figure rows (`.figs`), structured tables, split bars, snapshot/needs cards, schedule
  columns, mini-figs, numbered notes, and the policy-register bubble accordion.
- Print-friendly (`@media print` hides nav/buttons, opens accordions). Currency always `S$`.

## Never display — client privacy (hard rule)
The dashboard is shared with the client and may end up emailed, printed, or stored, so it
**must never contain personally identifying contact/ID data**, even if the workbook holds it:
- **No NRIC / FIN**, no email address, no mobile/handphone number, no home/mailing address.
- Read these columns only when logic needs them (e.g. age from DOB) but **never render them**.
  Show age, not full date of birth, in any client-facing figure where DOB isn't essential.
- Policy numbers ARE allowed — print them in full (`no. 1234567890`); they are not PII.
- Beneficiary entries show **role + share** (e.g. "Spouse · 50%"), not the beneficiary's NRIC.
If a section template would surface any of the above, drop the field. The included sample
(`reference/example-editorial-dashboard.html`) is fully anonymised and is the privacy baseline.

## Never auto-invent
Maturity values, NRICs, premium amounts, beneficiary shares, CPF flags — pull from the
workbook or leave the reference's `pending` / `[ADVISOR: confirm]` markers.
