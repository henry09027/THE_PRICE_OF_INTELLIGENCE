---
name: rpo-cross-sectional-analysis
description: "Produce a sourced, cross-sectional comparison of Remaining Performance Obligation (RPO) / revenue-backlog balances across a peer set of cloud vendors and AI hyperscalers, pairing SEC-filed balances with earnings-call commentary on private AI-lab concentration, and render the results as a comparative chart. Use for: RPO peer comparison, backlog benchmarking, AI-lab exposure across hyperscalers, cross-company backlog concentration. Triggers: cross sectional RPO, RPO comparison, hyperscaler backlog, compare RPO, backlog benchmarking, AI lab exposure comparison."
---

# Cross-Sectional RPO / Backlog & AI-Lab Exposure

## Purpose

For a peer set of cloud vendors / AI hyperscalers, retrieve each company's latest
filed backlog balance (RPO, "revenue backlog", or "commitments not yet recognized"),
pair it with earnings-call commentary on private AI-lab concentration (OpenAI,
Anthropic, xAI, and peers), and present the comparison graphically alongside a
sourced note.

**Framing to state in every note:** RPO / revenue backlog is a *leading indicator*
of contracted demand, **not** recognized revenue. Balances are **not** directly
comparable across companies — fiscal-year ends differ, the disclosure label differs
by issuer, and some companies do not disclose an RPO figure at all. Any single-lab
dollar share is typically **analyst-derived** (disclosed % × backlog); never present
a derived figure as a company disclosure.

## Inputs

| Input | Description | Example |
|---|---|---|
| Peer set | List of companies (names or tickers) | "Oracle, Microsoft, Alphabet, Amazon, Meta" |
| Period (optional) | Reporting window of interest | "latest", "Q2 2026" |

If the peer set is missing, request it once, then proceed. Default to each company's
**latest filed** balance when no period is specified.

## Prerequisites

- Pronto MCP tools: `getCompanies`, `searchSentences` (required);
  `getDocuments`, `getSentenceContext` (optional).
- A code sandbox with `matplotlib` for the comparative chart.
- All figures and quotations must originate from Pronto results. Fabricated
  balances, percentages, or contract values are a hard failure.

---

## Citation rule (mandatory)

Every statement derived from a filing or transcript must be immediately followed by a
visible, clickable inline source link, reproduced verbatim from the Pronto result field.

**Required format:** literal Markdown link syntax — `[MARKER](URL)` — with a non-empty
marker and no space between `]` and `(`.

```
Valid:   [#1170](https://spglobal.prontonlp.com/#/ref/$SENTID_SEC000262329667-1170)
```

**Invalid forms — all render as dead or invisible text:**

| Invalid | Reason |
|---|---|
| `[](url)` | Empty marker |
| `[#1170]` | No URL |
| Bare URL | Not a link in the client |
| `[1]` footnote pointer | Not resolvable inline |
| `` `[#1170](url)` `` | Backticks suppress rendering |

**Placement and integrity:**

- One link per source, at the end of the sentence it supports.
- Where multiple sources support one claim, attach all of them. Never collapse
  several claims under a single trailing citation.
- Copy markers and URLs verbatim. Never invent, reword, shorten, renumber, or
  construct a URL.
- Verification-table cells must contain the same clickable `[MARKER](url)`.

---

## Workflow

### Step 1 — Resolve every company

Call `getCompanies` once per name/ticker in the peer set.

- Single match: retain `companyId` and continue.
- Multiple matches (e.g. "Oracle" → Oracle Corp, Oracle Japan, Oracle Power):
  select the primary listing by market cap / expected exchange, or present the
  candidates and **stop** if genuinely ambiguous.

### Step 2 — Derive the date window

Map each company to its fiscal calendar; RPO is disclosed at period-end and these
differ (e.g. Oracle FY ends May, Microsoft June, most others December). Use a window
wide enough to capture the target period's 10-K/10-Q **and** the matching call:

- `gte` ≈ two quarters before the target period-end (or ~6 months back for "latest")
- `lte` = current date

Widen the window rather than risk missing a disclosure.

### Step 3 — Retrieve the filed backlog balance (per company)

Call `searchSentences` with `corpus: ["SEC Filings"]`, `companiesIds: [companyId]`,
the Step-2 `dateRange`, `size: 20`, and a `topicSearchQuery` adapted to the issuer's
vocabulary — the label is **not** uniform:

| Issuer style | Disclosure term | Suggested query |
|---|---|---|
| Oracle, Microsoft | "remaining performance obligations" | `remaining performance obligations total RPO balance` |
| Alphabet | "revenue backlog" | `revenue backlog remaining performance obligations cloud` |
| Amazon | "commitments not yet recognized" (primarily AWS) | `commitments not yet recognized AWS backlog remaining performance obligation` |

Extract for the target period-end, retaining each result's inline source link:

- Total backlog balance (and any commercial/cloud subtotal disclosed).
- Weighted-average duration and/or % recognized within 12–24 months.
- The definition sentence (useful framing quotation).

**No-disclosure case.** Some issuers (e.g. Meta) explicitly decline to disclose
unsatisfied performance obligations and are net **buyers** of compute rather than
hyperscale sellers. Record them as "not disclosed", capture the sentence where they
decline, and exclude them from the ranked bars (show as an annotated gap).

### Step 4 — Retrieve backlog & AI-lab concentration commentary (per company)

Call `searchSentences` with `corpus: ["S&P Transcripts"]`, `companiesIds: [companyId]`,
the Step-2 `dateRange`, `size: 15`, and `topicSearchQuery` such as
`RPO backlog AI cloud contracts OpenAI Anthropic concentration`.

Capture, retaining each result's inline source link:

- **Direct concentration disclosure** — a lab's stated share of RPO/backlog
  (e.g. "approximately 45% of our commercial RPO is from [lab]").
- **Named-counterparty commitments** — dollar-sized lab contracts disclosed on the
  call or in the filing (e.g. "+$100B over 8 years").
- Diversification framing (some issuers stress "a broad mix of customers").
- Volatility / lumpiness caveats management attaches to large lab contracts.

Where a company frames its backlog as diversified rather than lab-concentrated,
report that framing rather than forcing a concentration figure.

### Step 5 — Build the comparative chart

In the sandbox, produce a single bar chart of latest filed balances:

- One bar per company, ranked descending by balance.
- Value labels on bars; a distinct greyed/annotated placeholder for any
  "not disclosed" company.
- X-axis labels must include the **period-end date** so unequal fiscal calendars are
  visible; footnote any label mismatch (e.g. Amazon's "commitments not yet recognized").
- Write to `/tmp` then `cp` to `/workspace/`; surface with an `<asset>` tag.

Chart hygiene: title, axis units ("US$ billions"), source footnote, no chartjunk.

### Step 6 — Assemble and self-check

Draft the note per the **Output** template, then complete both checks before presenting:

1. **Citation check.** Every filing- or transcript-derived sentence carries an
   adjacent, non-empty, clickable `[MARKER](url)`.
2. **Provenance & comparability check.** Each fact is classified as filed,
   stated-on-call, or analyst-derived; no derived figure is shown as a disclosure;
   the comparability caveat (differing period-ends, differing labels, non-disclosers)
   is stated explicitly.

---

## Output template

```
# AI Hyperscalers — Cross-Sectional RPO / Backlog & AI-Lab Exposure

**Framing.** RPO / revenue backlog is contracted backlog, not recognized revenue.
Balances are not directly comparable (fiscal calendars and disclosure labels differ;
some issuers do not disclose). Figures are each company's latest filed balance.

[comparative chart asset]

## 1. Filed backlog balances (latest SEC filing)
- One bullet per company: balance + period-end + definition/label + inline link.
- Explicit "not disclosed" bullet for non-disclosers, with the declining sentence.

| Company | Backlog measure | Latest balance | Period-end | AI-lab counterparty |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

## 2. Management commentary — AI-lab concentration
- Direct % disclosure, named-counterparty commitments, or diversification framing —
  each with its inline source link.
- Any single-lab $ share: show arithmetic (disclosed % × backlog) and label DERIVED.

## 3. Read-through
- Rank the peer set by backlog size and by lab concentration; note volatility caveats.

## Verification status
| Claim | Pronto source | Status |
|---|---|---|
| <claim> | [#NNNN](url) | Filed / Call-sourced / Analyst-derived |

**Coverage notes.** Differing period-ends, non-disclosers, and any derived figures.
```

## Stopping points

- **Step 1** — halt if a company is genuinely ambiguous.
- **Step 6** — halt for citation, provenance, and comparability self-check before presenting.
- **On delivery** — present the note and chart only. Do not chain into additional
  peer sets or periods unless asked.

## Generalization notes

- The peer set is arbitrary — any cloud vendors or hyperscalers. The private-lab
  counterparty is **inferred per company** from filing/transcript hits, not hard-coded.
- The disclosure label is issuer-specific; always adapt the `topicSearchQuery`
  (Step 3 table) rather than assuming the literal string "RPO".
- For a single-company deep dive (one issuer's RPO + lab share) use the companion
  single-name RPO exposure workflow; this skill is the multi-company comparison layer.
