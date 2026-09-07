---
name: rpo-ai-lab-exposure
description: "Build a professional research note on a company's Remaining Performance Obligation (RPO) backlog and its concentration in private AI labs (OpenAI, Anthropic), by cross-referencing SEC filings and earnings-call transcripts via the Pronto MCP. Use when: analyzing RPO disclosures, backlog concentration, or AI-lab counterparty exposure for a hyperscaler or cloud vendor. Triggers: RPO exposure, RPO concentration, remaining performance obligation, OpenAI backlog, Anthropic backlog, AI lab exposure, footnote 5 analysis, price of intelligence."
---

# RPO & Private AI-Lab Exposure Note

Generalizes the "footnote 5" analysis from *The Pay Off for the Price of Intelligence*: for any company and quarter, pair the SEC-filed RPO balance with the earnings-call commentary disclosing how much of that backlog is concentrated in a private AI lab (OpenAI, Anthropic), then reconstruct the lab's dollar share.

## Inputs (only two)

1. **Company** — name or ticker (e.g., "Microsoft", "MSFT", "Oracle").
2. **Quarter** — fiscal quarter + year (e.g., "Q4 FY2026", "Q2 2026"). Used to set the reporting date and a lookback window.

If either is missing, ask for it once, then proceed. No other inputs required.

## Prerequisites

- Pronto MCP tools available: `getCompanies`, `searchSentences`, `getDocuments` (optional), `getSentenceContext` (optional).
- All figures and quotes MUST come from Pronto results and carry inline `[MARKER](url)` citations. Never fabricate RPO numbers or percentages.

## Workflow

### Step 1: Resolve the company

- Call `getCompanies` with `companyNameOrTicker` = the company input.
- If multiple results, present ticker/exchange/sector/market cap and **STOP** — ask which one.
- Keep the `companyId`.

### Step 2: Derive the date window from the quarter

- Map the quarter to its period-end and set a `dateRange` that comfortably brackets both the filing and the call:
  - `gte` ≈ 2 quarters before the quarter's period-end; `lte` = "now" (or one month after the expected report date).
- Mind fiscal calendars (e.g., Microsoft & Oracle are not December FYE). If unsure of the exact period-end, widen the window rather than miss the disclosure.

### Step 3: Pull the filed RPO balance (SEC filings)

Call `searchSentences` with:
- `companiesIds` = [companyId], `corpus` = ["SEC Filings"], the Step 2 `dateRange`, `size` ≈ 30
- `topicSearchQuery` = "remaining performance obligation commercial contracted revenue not yet recognized"

Extract, for the target quarter's period-end:
- Total company RPO and, if disclosed, the commercial RPO portion.
- Weighted-average duration and the % expected to be recognized within 12 months.
- The ASC 606 definition sentence (useful framing quote).

If nothing lands for the exact period-end (e.g., the latest 10-K/10-Q isn't in the corpus yet), note the gap explicitly and fall back to the closest prior filing; flag that the target-quarter figure may only be available from the call (Step 4).

### Step 4: Pull RPO + AI-lab concentration commentary (transcripts)

Call `searchSentences` with:
- `companiesIds` = [companyId], `corpus` = ["S&P Transcripts"], the Step 2 `dateRange`, `size` ≈ 40
- `topicSearchQuery` = "OpenAI Anthropic share of commercial remaining performance obligation RPO backlog"

Capture:
- Any **direct** disclosure of the AI lab's share of RPO/backlog (e.g., "~45% of our commercial RPO is from OpenAI") — this is the footnote-5 anchor.
- Any **growth-rate** disclosure that lets you back into the share (e.g., "RPO increased X% excluding [lab]") when the % isn't restated.
- Contract-value mentions (e.g., "$250B of Azure services contracted"), duration, and volatility caveats.
- Equity-stake / accounting commentary if characterizing "double exposure."

If a second lab is relevant (e.g., Amazon → OpenAI + Anthropic), rerun with that lab named in the query.

### Step 5: Reconstruct the AI-lab dollar share

Two methods, in priority order:

1. **Direct** (preferred): lab $ share = disclosed % × commercial RPO for that quarter.
2. **Derived** (when % not restated): estimate ex-lab RPO = prior-year base × (1 + disclosed ex-lab growth %); lab $ ≈ total commercial RPO − ex-lab RPO; lab % = lab $ ÷ commercial RPO.

Label every derived number as **analyst-derived, not a company disclosure**. Show the arithmetic inline.

### Step 6: Assemble the note

Use the structure in **Output**. Requirements:
- Lead with a bold **Summary** stating the RPO balance and the lab's $ and % share for the quarter.
- Every RPO figure, %, and quote carries its inline `[MARKER](url)` citation exactly as returned by Pronto.
- Separate **filed** facts (SEC) from **stated-on-call** facts (transcripts) from **analyst-derived** estimates.
- Close with a verification table marking each claim ✅ Filed / ◑ Call-sourced / ◑ Analyst-derived, and a short coverage note on anything missing from the corpus.

**STOP**: Present the note. Do not chain into other companies/quarters unless asked.

## Output

```
# <Company> — RPO Disclosure & Private AI-Lab Exposure
### <Quarter>

**Summary.** <RPO balance>; <lab> ≈ $<X>B (<Y>%) of the backlog.

## 1. Filed RPO balance (SEC filings)
- Total / commercial RPO, duration, 12-month recognition %, ASC 606 definition — with citations.
- Coverage note if the target-quarter filing isn't in the corpus.

## 2. AI-lab concentration — the footnote-5 input (transcripts)
- Direct % disclosure (or the growth-rate disclosure used to derive it) — with citations.
- Footnote-5 calculation: % × commercial RPO ≈ $ share.

## 3. Derived share (if applicable)
- Arithmetic + explicit "analyst-derived" label.

## 4. Corroborating exposure (optional)
- Contract values, equity stake, accounting commentary.

## 5. Verification status
| Claim | Pronto source | Status |
```

## Stopping Points

- ✋ Step 1 if the company is ambiguous.
- ✋ Step 3/4 if no RPO disclosure is found in the window (report the gap, ask whether to widen).
- ✋ Step 6 final review.

## Notes

- This note tracks a **leading indicator** (contracted backlog), not recognized revenue — state that framing.
- Concentration figures are often estimates built from disclosed contract values ÷ total RPO; never present an estimate as a company disclosure.
- Reusable across hyperscalers/cloud vendors; the private-lab counterparty (OpenAI, Anthropic, or other) is inferred per company from the transcript hits.
