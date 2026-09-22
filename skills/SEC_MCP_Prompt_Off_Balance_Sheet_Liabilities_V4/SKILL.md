---
name: sec-off-balance-sheet-liabilities-hyper5
description: "Extract all off-balance-sheet commitments and contingencies (leases not yet commenced, purchase commitments, guarantees/backstops, contingent liabilities) for the Hyper 5 (Alphabet, Meta, Microsoft, Amazon, Oracle) from their most recent periodic filings, and produce a markdown report, CSV, summary-stats file, and cross-sectional chart. Use when: analyzing off-balance-sheet liabilities, data-center/AI lease build-out, uncommenced leases, purchase commitments, RVG/backstop guarantees, or contingent liabilities for the megacap tech names. Triggers: off-balance-sheet, off balance sheet, leases not yet commenced, uncommenced leases, purchase commitments, residual value guarantee, RVG, lease backstop, contingent liabilities, Hyper 5, HYPER 5 extraction."
---

# HYPER 5 Off-Balance-Sheet Liabilities Extraction

## OBJECTIVE

Extract ALL off-balance-sheet commitments and contingencies from the Hyper 5's most recent periodic filings, across four categories:
1. Leases Not Yet Commenced (ASC 842-20-50-3)
2. Purchase Commitments (capital & operating)
3. Guarantees & Lease Backstops (RVG, credit derivatives, backstops)
4. Contingent Liabilities (legal, tax, environmental)

Produce: a markdown report with per-category tables, a raw-data CSV, a summary-stats file, and a cross-sectional 4-panel chart. Every figure MUST be pulled live from the filings at extraction time and carry (a) a source-basis citation naming the specific filing and (b) a source marker/link. **This skill deliberately stores NO figures — all numbers are extracted fresh each run from the primary sources below, because they roll forward every quarter.**

## TARGET COMPANIES

| Company | CIK | Primary filing basis |
|---------|-----|----------------------|
| Alphabet Inc. | 1652044 | Most recent 10-Q |
| Meta Platforms, Inc. | 1326801 | Most recent 10-Q |
| Microsoft Corporation | 789019 | Most recent 10-Q OR 10-K — whichever is newer |
| Amazon.com, Inc. | 1018724 | Most recent 10-Q |
| Oracle Corporation | 1341439 | Most recent 10-Q OR 10-K — whichever is newer |

## DATA SOURCE — READ FIRST (MANDATORY)

Every figure comes from primary filings, pulled at extraction time. Two sources, in this order:

1. **Pronto MCP SEC Filings corpus** (`corpus:["SEC Filings"]`) — the primary retrieval path. Extract each figure from a document returned by the Pronto MCP tools and keep the inline citation marker (`[MARKER](url)` / `$SENTID_...` / `$DOCID_...`). Cite the Pronto MCP website (`https://spglobal.prontonlp.com/...`) as the source of record for corpus-sourced figures.
2. **The official SEC website / EDGAR** (`sec.gov`, `data.sec.gov`) — the authoritative cross-check and the fallback whenever the corpus is stale or a figure is missing. The sandbox may be blocked from reaching `data.sec.gov` directly; use `Web_Search` restricted to `allowed_domains:["sec.gov"]` to retrieve filing text. Reputable third-party readers (Bloomberg, WSJ, Reuters) may only CONFIRM a primary-source number, never originate one.

**CRITICAL — always cross-check against the SEC website.** The Pronto corpus can lag the newest filing, and a first-pass "not disclosed" can be wrong. For EVERY company you MUST verify the extracted figures against the official SEC filing before finalizing — do not rely on the corpus alone. Two recurring failure modes to guard against:
- **A newer filing exists that the corpus has not ingested** (e.g. an interim 10-Q is superseded by a just-filed fiscal-year-end 10-K). Always check EDGAR for a newer 10-K/10-Q than the corpus's latest document, especially around fiscal year-ends and earnings season, and use the newest filing.
- **A figure that IS disclosed gets wrongly marked `n/d`.** Before concluding a number is absent, confirm against the actual filing text on the SEC website.

Never substitute a web estimate for a number that is genuinely absent from disclosure — mark it `n/d` (never `0`, never estimated). Record the source basis (which filing, filed when, corpus vs. EDGAR) for every figure.

### Pronto MCP tools

| Purpose | Tool | Key arguments |
|---------|------|---------------|
| Resolve company -> ID | `getCompanies` | `companyNameOrTicker`, `corpus:["SEC Filings"]` |
| Confirm doc types | `getFilterOptions` | `filterNames:["documentTypes","sections"]` |
| Find most recent filing | `getDocuments` | `companiesIds`, `corpus:["SEC Filings"]`, `documentTypes:["10-Q","10-K"]`, `sortOrder:"desc"`, `excludeFutureDocuments:true`, `size:5` |
| Pull disclosure sentences | `searchSentences` | `topicSearchQuery`, `transcriptsIds`, `corpus:["SEC Filings"]`, `size:~40`, `similarityThreshold:"Low"` |
| Narrative summary | `getDocumentSummary` | `transcriptsIds`, `focus` |
| Read a section verbatim | `readDocuments` | `transcriptsIds`, `sections` |
| Widen context | `getSentenceContext` | `sentenceIds`, `beforeSentence`/`afterSentence` |
| SEC/EDGAR cross-check | `Web_Search` | `allowed_domains:["sec.gov"]` |

## WORKFLOW (per company)

1. **Resolve** via `getCompanies` (match the CIK above to pick the correct issuer).
2. **Find latest filing**: `getFilterOptions(["documentTypes"])` once, then `getDocuments(...)`. Take the single most recent 10-Q, OR the 10-K if it is newer. Capture `transcriptId`, doc type, period/as-of date, and filing date. **Flag any company whose latest corpus doc predates its most recent fiscal period-end — that company needs an EDGAR recency check for a newer filing.**
3. **Extract each category** via `searchSentences` scoped to that `transcriptId` (short queries; raise `size`, lower `similarityThreshold` to "Low"). Use `getDocumentSummary`/`readDocuments`/`getSentenceContext` as needed.
4. **Record source marker + source basis** next to every figure.
5. **Cross-check on the SEC website (EDGAR)** — for every company, and mandatorily for every `n/d` or possibly-stale figure. Replace stale corpus numbers with the newest EDGAR figure and note the source basis.
6. Repeat for all five, then compile report / CSV / summary / chart from the freshly-pulled numbers.

## SEARCH QUERIES PER CATEGORY

- **Leases not yet commenced**: "leases not yet commenced", "future lease payments not yet recorded", "data center leases signed not commenced"
- **Purchase commitments**: "unconditional purchase obligations", "purchase commitments", "non-cancelable contractual commitments", "cloud capacity commitments"
- **Guarantees/backstops**: "residual value guarantee", "lease backstop credit support", "maximum exposure to loss", "credit derivatives backstop"
- **Contingent**: "legal proceedings litigation contingency", "loss contingency reasonably possible", "uncertain tax positions", "income tax contingencies"
- **Subsequent events**: "subsequent events leases"

## FIELDS TO CAPTURE PER CATEGORY

**Cat 1 (Leases not yet commenced):** total uncommenced amount (undiscounted); asset type; commencement window (verbatim); lease terms (years); subsequent-event additions (captured separately); verbatim footnote excerpt + marker.
**Cat 2 (Purchase commitments):** each commitment type, amount, timing, notes, source; total. Distinguish what the company OWES (purchase obligations) from remaining performance obligations that customers owe the company (do not conflate).
**Cat 3 (Guarantees):** guarantor, type, max exposure, trigger/term, status, source. Tie every guarantee/RVG to the issuer whose filing discloses it (see pitfalls).
**Cat 4 (Contingent):** type, estimated range, probability (remote/reasonably possible/probable), status, source. Flag exposures >$100M.

## PITFALLS TO AVOID (process learnings — do NOT hardcode figures)

1. **Stale interim filing vs. newer year-end filing.** An interim 10-Q's uncommenced-lease number can be superseded by a much larger figure in a subsequently-filed fiscal-year-end 10-K. Always check EDGAR for the newest filing near fiscal year-ends; the corpus may lag. Use the newest disclosed figure.
2. **Wrongly marking a disclosed figure `n/d`.** A first-pass extraction (or an over-eager "not disclosed" conclusion) can miss an aggregate that is in fact in the filing. Verify any `n/d` against the actual filing text on the SEC website before finalizing.
3. **Cross-company misattribution.** Guarantees/RVGs and venture-specific commitments must be attributed to the issuer whose filing discloses them (e.g. a data-center-venture residual value guarantee belongs to the company that reports it, not a peer). Never move a figure across companies.
4. **Conflating commitments with performance obligations.** "Commitments not yet recognized" that customers owe the company (remaining performance obligations / backlog) are NOT the company's purchase commitments. Keep them distinct.

## OUTPUT DELIVERABLES

Write to `/workspace/hyper5/` (write Office/zip formats to `/tmp` first, then copy; PNG/CSV/MD can be written directly). Populate every value from the freshly-extracted figures — pass `None`/`n/d` for anything not disclosed (never `0`, never estimated):
1. `markdown_report.md` — full report with per-category tables, a **source-basis column**, verbatim excerpts + markers, and a caveats note (n/d gaps mean totals are not apples-to-apples).
2. `raw_data_table.csv` — one row per company incl. a `Source_Basis` column and a computed `Total_OBS_Disclosed_B`.
3. `summary_stats.txt` — category totals, ranked leases, per-company filing basis, any revisions applied during cross-check, caveats.
4. `hyper5_cross_section.png` — 4-panel matplotlib chart (A: leases + subsequent additions hatched; B: purchase commitments; C: guarantees vs contingent grouped; D: total stacked). `None` -> "n/d" (never 0). The figure's source note must name the per-company filing basis. Surface via an `<asset>` tag.

See `references/plot_hyper5_cross_section.py` for the reusable chart function.

## Stopping Points

- After resolving companies + latest filings: confirm none are stale vs. their fiscal period-end (else EDGAR-check for a newer filing).
- After extraction, before finalizing: confirm every figure was cross-checked on the SEC website, every `n/d` verified against filing text, and no cross-company misattribution exists.
- After generating deliverables: present totals and surface the chart asset.

## Output

Present the ranked uncommenced-lease table and the disclosed grand total across categories (with the apples-to-apples caveat), list the four deliverable files via `<asset>` tags, and offer to schedule a recurring (quarterly, post-earnings) automation. Do NOT expose internal file paths or implementation details in the user-facing summary.
