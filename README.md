[README.md](https://github.com/user-attachments/files/29020298/README.md)
# Policy Dashboard Generator — Editorial Style

A **dashboard generator**: feed it a filled policy-consolidation Excel workbook and it
produces one self-contained client policy-review **HTML file** in the editorial house style
(Fraunces serif, paper/ink palette, magazine layout).

**Excel in → HTML out.** Runs locally as a Claude Code / Claude skill — no build step, no
dependencies except Google Fonts. The generated HTML is delivered to the client directly
(email/file); nothing is hosted publicly. This repo stores the **generator**, not client data.

## What's in this repo

```
policy-dashboard-editorial/
├── SKILL.md                              # the skill: triggers + 5-step build process
└── reference/
    ├── section-data-map.md               # which workbook cell feeds each dashboard section
    └── example-editorial-dashboard.html  # canonical design checkpoint — clone this
```

`example-editorial-dashboard.html` is the **source of truth** for the design. The skill
tells Claude to read that file first, clone its structure / CSS / chart code, then swap in
the new client's data.

## Install

### Option A — personal skill (Claude Code, local)

Copy the `policy-dashboard-editorial` folder into your skills directory:

- **Windows:** `%USERPROFILE%\.claude\skills\policy-dashboard-editorial\`
- **macOS / Linux:** `~/.claude/skills/policy-dashboard-editorial/`

The folder **must** be named `policy-dashboard-editorial` and contain `SKILL.md` at its
root (folder-based skills only — a loose `.md` won't register).

### Option B — pull from this repo

```bash
git clone https://github.com/<you>/policy-dashboard-editorial.git
# Windows
copy /E policy-dashboard-editorial "%USERPROFILE%\.claude\skills\policy-dashboard-editorial"
# macOS / Linux
cp -r policy-dashboard-editorial ~/.claude/skills/
```

Restart Claude Code (or reload skills) so it picks up the new folder.

## Use

In a session, trigger with any of:

- `/policy-dashboard-editorial`
- "build editorial dashboard"
- "editorial policy review"
- "build the v2 dashboard for [client]"

Then point it at the client's filled consolidation workbook. Output:
`[ClientName]_Policy_Review_v2.html`.

## Requirements

- A filled `policy-consolidation-template` workbook (v2 schema).
- Python with **openpyxl** (workbook read).
- A recalc path so cached formula values are fresh — **Excel COM** (Windows) or
  **LibreOffice headless** (`--convert-to`). The skill reads `data_only=True`, so stale
  caches show blank.
- Headless **Edge / Chromium** for the render-check screenshot step (optional but
  recommended).

## Keeping the design in sync

`reference/example-editorial-dashboard.html` and the live client dashboard are kept
**byte-identical**. When the design changes, update the live file, then copy it back over
the reference file and fold any new general rule into `SKILL.md` Step 3 / `section-data-map.md`.
Commit that change so the repo always holds the current canonical design.

## Privacy

The dashboard is a **client-facing** document and never renders personally identifying
contact/ID data, by design (see `SKILL.md` → *Never display — client privacy*):

- No NRIC/FIN, no email, no mobile/handphone, no home address — ever.
- DOB is used only for age maths; client-facing figures show age, not full DOB.
- Policy numbers are masked to the last 4 digits (`no. •••• 1234`).
- The bundled sample (`reference/example-editorial-dashboard.html`) is **fully fictional** —
  the "Jordan Lee Wei Han" family, figures, and masked policy numbers contain no real data.
- `.gitignore` blocks `*.xlsx`, generated client dashboards, and `_backups/` so real client
  files can never be committed by accident.

## Notes

- Self-contained output: one HTML file, inline CSS + SVG-in-JS charts, Google Fonts only.
- Never auto-invents maturity values, NRICs, premiums, beneficiary shares, or CPF flags —
  those come from the workbook or stay marked `pending` / `[ADVISOR: confirm]`.
- Currency always displays as `S$`.
