---
name: rpo-ai-lab-exposure
description: "Build a professional research note on a company's Remaining Performance Obligation (RPO) backlog and its concentration in private AI labs (OpenAI, Anthropic, etc), by cross-referencing SEC filings and earnings-call transcripts via the Pronto MCP. Use when: analyzing RPO disclosures, backlog concentration, or AI-lab counterparty exposure for a hyperscaler or cloud vendor. Triggers: RPO exposure, RPO concentration, remaining performance obligation, OpenAI backlog, Anthropic backlog, AI lab exposure, footnote 5 analysis, price of intelligence."
---

# RPO & Private AI-Lab Exposure Note

For any company and quarter, pair the SEC-filed RPO balance with the earnings-call commentary disclosing how much of that backlog is concentrated in a private AI lab (OpenAI, Anthropic), then reconstruct the lab's dollar share.

## Inputs (only two)

1. **Company** — name or ticker (e.g., "Microsoft", "MSFT", "Oracle").
2. **Quarter** — fiscal quarter + year (e.g., "Q4 FY2026", "Q2 2026"). Used to set the reporting date and a lookback window.

If either is missing, ask for it once, then proceed. No other inputs required.

## ⚠️ MANDATORY: Every sourced fact must carry a VISIBLE inline source link

This is a hard requirement, not a preference. In the final note, **every statement that comes from a filing or a transcript MUST be immediately followed by its inline source link, rendered as a clickable link the reader can actually see and click** — the exact `[MARKER](url)` string as returned in the Pronto result field (e.g. `[#1170](https://…/$SENTID_SEC000262329667-1170)`). No exceptions.

### Rendering rule — make the link visible (this is what was failing before)
- Output the link as **literal Markdown link syntax** exactly as returned: an opening `[`, the marker text (e.g. `#1170`), a closing `]`, then `(` + the full URL + `)`, with **no space between `]` and `(`**. Example that renders: `[#1170](https://spglobal.prontonlp.com/#/ref/$SENTID_SEC000262329667-1170)`.
- The bracket text MUST be the non-empty marker (e.g. `#1170`). **Never emit an empty `[]`, a bare `[#1170]` with no URL, a bare URL with no brackets, or a footnote-style `[1]` pointer.** Any of these renders as invisible or dead text — which is the failure this rule exists to prevent.
- Do NOT wrap the link in code fences/backticks, and do NOT place it inside a `<cite>` tag — it must be plain inline Markdown so the client renders it as a hyperlink.
- Place the link at the **end of the sentence it supports**, before the period is fine, e.g. `…up from $138 billion a year earlier [#1170](https://…-1170).`
- In the **verification table**, the "Pronto source" cell MUST contain the same clickable `[MARKER](url)` link(s), not a plain marker or description.

### Correctly-cited example (copy this pattern)
> Total RPO was **$638 billion** as of May 31, 2026, up from $138 billion a year earlier [#1170](https://spglobal.prontonlp.com/#/ref/$SENTID_SEC000262329667-1170), of which "approximately 12%" is expected to convert within twelve months [#1396](https://spglobal.prontonlp.com/#/ref/$SENTID_SEC000262329667-1396).

### Rules
- **Filings (SEC) and transcripts (calls) alike** — every RPO figure, percentage, duration, growth rate, contract value, and any quoted or paraphrased sentence gets its own visible inline link, placed right next to the claim it supports.
- **One link per source.** If several sources back one claim, attach all of them; never collapse many claims under a single trailing citation.
- **Copy links verbatim.** Never invent, reword, shorten, or renumber a marker, and never construct a URL yourself. Use only links that appear in the tool results.
- **Never substitute** a sentence-level link with a document-title link or vice versa.
- **Self-check before presenting (Step 5):** scan the rendered draft line by line; for every filing/transcript sentence confirm there is an adjacent, non-empty, clickable `[MARKER](url)`. If any is missing, empty-bracketed, un-URL'd, backtick-wrapped, or a bare marker, fix it before responding. A note with an uncited or invisible-link sourced fact is incomplete — do not deliver it.

## Prerequisites

- Pronto MCP tools available: `getCompanies`, `searchSentences`, `getDocuments` (optional), `getSentenceContext` (optional).
- All figures and quotes MUST come from Pronto results and carry the visible inline source links defined above. Never fabricate RPO numbers or percentages.

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

Extract, for the target quarter's period-end — **retaining each result's inline source link for use in the note**:
- Total company RPO and, if disclosed, the commercial RPO portion.
- Weighted-average duration and the % expected to be recognized within 12 months.
- The ASC 606 definition sentence (useful framing quote).

If nothing lands for the exact period-end (e.g., the latest 10-K/10-Q isn't in the corpus yet), note the gap explicitly and fall back to the closest prior filing; flag that the target-quarter figure may only be available from the call (Step 4).

### Step 4: Pull RPO + AI-lab concentration commentary (transcripts)

Call `searchSentences` with:
- `companiesIds` = [companyId], `corpus` = ["S&P Transcripts"], the Step 2 `dateRange`, `size` ≈ 40
- `topicSearchQuery` = "OpenAI Anthropic share of commercial remaining performance obligation RPO backlog"

Capture — **retaining each result's inline source link for use in the note**:
- Any **direct** disclosure of the AI lab's share of RPO/backlog (e.g., "~45% of our commercial RPO is from OpenAI").
- Any **growth-rate** disclosure that lets you back into the share (e.g., "RPO increased X% excluding [lab]") when the % isn't restated.
- Contract-value mentions (e.g., "$250B of Azure services contracted"), duration, and volatility caveats.
- General transcripts for identifying any language around Private AI Labs Anthropic and OpenAI. 

If a second lab is relevant (e.g., Amazon → OpenAI + Anthropic), rerun with that lab named in the query.

### Step 5: Assemble the note

Use the structure in **Output**. Requirements:
- Lead with a bold **Summary** stating the RPO balance and the lab's $ and % share for the quarter — each sourced number carrying its visible inline link.
- **Apply the MANDATORY visible-source-link rule above to every filing/transcript fact.** Run the Step 5 self-check (scan the rendered draft; confirm every sourced sentence has an adjacent, non-empty, clickable `[MARKER](url)`) before presenting.
- Separate **filed** facts (SEC) from **stated-on-call** facts (transcripts) from **analyst-derived** estimates.
- Close with a verification table whose "Pronto source" column holds the clickable `[MARKER](url)` link(s), marking each claim ✅ Filed / ◑ Call-sourced / ◑ Analyst-derived, and a short coverage note on anything missing from the corpus.

**STOP**: Present the note. Do not chain into other companies/quarters unless asked.

## Output

```
# <Company> — RPO Disclosure & Private AI-Lab Exposure
### <Quarter>

**Summary.** <RPO balance + visible link>; <lab> ≈ $<X>B (<Y>%) of the backlog.

## 1. Filed RPO balance (SEC filings)
- Total / commercial RPO, duration, 12-month recognition %, ASC 606 definition — each with its visible inline source link.
- Coverage note if the target-quarter filing isn't in the corpus.

## 2. Executive Commentaries (Earnings Call Transcripts)
- Direct % disclosure (or the growth-rate disclosure used to derive it) — each with its visible inline source link.
- Footnote-5 calculation: % × commercial RPO ≈ $ share.

## 3. Verification status
| Claim | Pronto source (clickable [MARKER](url)) | Status |
```

## Stopping Points

- ✋ Step 1 if the company is ambiguous.
- ✋ Step 3 final review — including the visible-source-link self-check.

## Notes

- This note tracks a **leading indicator** (contracted backlog), not recognized revenue — state that framing.
- Concentration figures are often estimates built from disclosed contract values ÷ total RPO; never present an estimate as a company disclosure.
- Reusable across hyperscalers/cloud vendors; the private-lab counterparty (OpenAI, Anthropic, or other) is inferred per company from the transcript hits.
