---
name: SEC_MCP_Prompt_Off_Balance_Sheet_Liabilities
description: "Extract off-balance-sheet commitments and contingencies (leases not yet commenced, purchase commitments, guarantees & lease backstops, contingent liabilities) for the Hyper 5 hyperscalers from their most recent 10-Q/10-K, and produce a markdown report, CSV, and summary. Triggers: 'HYPER 5 off-balance-sheet liabilities extraction', 'off-balance-sheet liabilities', 'uncommenced leases report', 'hyperscaler commitments extraction'."
---

# SEC FILINGS MCP PROMPT: HYPER 5 OFF-BALANCE-SHEET LIABILITIES EXTRACTION

## OBJECTIVE

Extract ALL off-balance-sheet commitments and contingencies from Hyper 5 most recent 10-Q/10-K filings. Focus on four categories:
1. Leases Not Yet Commenced
2. Purchase Commitments
3. Guarantees & Lease Backstops
4. Contingent Liabilities

Generate markdown report with detailed tables for each category and each company.

---

## DATA SOURCE — READ FIRST (MANDATORY)

**All values MUST be extracted from the Pronto MCP financial-document corpus, NOT from the internet.**

- Do **NOT** use `Web_Search`, SEC EDGAR, news articles, or any external website for figures. Every dollar amount, date, term, and verbatim excerpt must come from a document returned by the Pronto MCP tools.
- Use the **`SEC Filings`** corpus (10-K, 10-Q, 8-K, annual/quarterly reports) on every Pronto MCP call. Set `corpus: ["SEC Filings"]`.
- Every figure in the output must carry the inline citation marker that the Pronto MCP tool returns (the `[MARKER](url)` embedded in sentence/text fields). Do not invent, reword, or drop these markers, and never construct URLs yourself.
- If a figure cannot be found in the corpus for a company, state "Not disclosed in available filing" — do NOT fall back to the web or to an estimate.

### Pronto MCP tools to use

| Purpose | Tool | Key arguments |
|---------|------|---------------|
| Resolve each company to an ID | `getCompanies` | `companyNameOrTicker`, `corpus:["SEC Filings"]` |
| Find the most recent 10-Q/10-K | `getDocuments` | `companiesIds`, `corpus:["SEC Filings"]`, `documentTypes`, `sortOrder:"desc"`, `excludeFutureDocuments:true`, `size:5` |
| Confirm valid `documentTypes` | `getFilterOptions` | `filterNames:["documentTypes","sections"]` |
| Pull the disclosure sentences (primary) | `searchSentences` | `topicSearchQuery`, `transcriptsIds`, `corpus:["SEC Filings"]`, `size:~100` |
| Summarize a category within a filing | `getDocumentSummary` | `transcriptsIds`, `focus`, `corpus:["SEC Filings"]` |
| Read a full section verbatim | `readDocuments` | `transcriptsIds`, `sections`, `corpus:["SEC Filings"]` |
| Widen context around a hit | `getSentenceContext` | `sentenceIds`, `beforeSentence`/`afterSentence` |

### Standard workflow (per company)

1. **Resolve company** → `getCompanies(companyNameOrTicker, corpus:["SEC Filings"])`. If several matches, pick the primary issuer (largest market cap / matching CIK below). Capture the `companyId`.
2. **Find the latest filing** → `getFilterOptions(["documentTypes"])` once, then `getDocuments(companiesIds:[id], corpus:["SEC Filings"], documentTypes:[10-Q or 10-K], sortOrder:"desc", excludeFutureDocuments:true)`. Take the single most recent 10-Q (or 10-K if it is newer). Capture the `transcriptId`, document type, and period/as-of date.
3. **Extract each category** with `searchSentences` scoped to that `transcriptId` (see per-category `topicSearchQuery` phrases below). Raise `size` (~100) and lower `similarityThreshold` toward "Low" for a thorough read of a single document. Use `getDocumentSummary` for a narrative pass and `readDocuments`/`getSentenceContext` when a number needs surrounding context or a verbatim excerpt.
4. **Record the citation marker** returned with each sentence next to every figure you extract.
5. Repeat for all five companies, then compile the report / CSV / summary.

**Verify recency:** confirm the extracted figure is tied to the filing's period/as-of date. If the most recent filing predates a known subsequent event, capture the subsequent-event addition separately (Category 1, Field 5) only if it too appears in a corpus document.

---

## TARGET COMPANIES & FILINGS

Resolve each via `getCompanies`; CIKs are provided to disambiguate the correct issuer.

| Company | CIK | Filing |
|---------|-----|--------|
| **Alphabet Inc.** | 1652044 | Most recent 10-Q |
| **Meta Platforms, Inc.** | 1326801 | Most recent 10-Q |
| **Microsoft Corporation** | 789019 | Most recent 10-Q or 10-K |
| **Amazon.com, Inc.** | 1018724 | Most recent 10-Q |
| **Oracle Corporation** | 1341439 | Most recent 10-Q or 10-K |

---

## EXTRACTION CATEGORY 1: LEASES NOT YET COMMENCED (ASC 842-20-50-3)

### How to find it in the corpus
Run `searchSentences` scoped to the company's latest `transcriptId` (`corpus:["SEC Filings"]`). Also try `getDocumentSummary(focus:"leases not yet commenced")`. Suggested `topicSearchQuery` values (run several short ones, not one long string):
- "leases not yet commenced"
- "future lease payments not yet recorded"
- "operating and finance leases not commenced"
- "data center leases signed not commenced"

If a Commitments/Leases section exists, confirm section names via `getFilterOptions(["sections"])` and use `readDocuments(sections:[...])` for the full text.

### Field 1: Total Uncommenced Lease Amount

**What to capture:**
- Exact dollar amount as disclosed (e.g., "$278.99 billion")
- Currency (if not USD)
- Whether amount is discounted or undiscounted
- If multiple amounts given, capture ALL (e.g., short-term vs long-term, with/without certain facilities)

**Format output:**
```
Total Uncommenced Leases: $[EXACT AMOUNT] [as of Filing Date]
Discount Status: [Undiscounted / Discounted to PV / Not specified]

Example:
Total Uncommenced Leases: $278.99 billion (undiscounted)
As of: June 30, 2026 (10-Q)
```

### Field 2: Asset Type / Facility Description

**What to capture:**
- Primary asset class (data centers, offices, infrastructure, warehouses, etc.)
- Any geographic concentration
- Strategic purpose (if disclosed)
- Breakdown by asset type if provided (e.g., "60% data centers, 40% offices")

**Format output:**
```
Asset Type: [Description]

Examples:
- "data centers and network infrastructure"
- "office space and colocations"
```

### Field 3: Commencement Window

**What to capture:**
- Fiscal year or calendar year range (e.g., "remainder of 2026 through 2028")
- Exact commencement dates if specified (e.g., "Q4 2026 onward")
- Any phased commencement schedule
- Latest commencement date (e.g., "ending by 2036")

**Format output:**
```
Commencement Window: [EXACT RANGE from filing]

Examples:
- "Q4 2026 through 2028"
- "2026–2031"
- "remainder of 2026 through 2036"
```

### Field 4: Lease Terms

**What to capture:**
- Lease duration in years (e.g., "15–20 years")
- Range if terms vary
- Any mention of renewal options

**Format output:**
```
Lease Terms: [X–Y years] or [X years]

Example: "15–30 years"
```

### Field 5: Recent Amendments or Additions (Post-Balance Sheet)

**What to capture:**
- Any leases signed AFTER balance sheet date but BEFORE filing (as disclosed in the corpus document)
- Labeled as "Subsequent Events" or "Recent Commitments"
- Amount and commencement date
- Search with `topicSearchQuery:"subsequent events leases"` on the same `transcriptId`

**Format output:**
```
Recent Additions: $[Amount] (signed [Date], commences [Date])

Note: Meta had July 2026 addition of $68B in 10-Q dated June 30, 2026
```

### Field 6: Verbatim Footnote Excerpt

**What to capture:**
- The official footnote disclosure text as returned by `searchSentences` / `readDocuments`
- Exact quote, no paraphrase — include the Pronto MCP citation marker
- Include footnote/section reference if shown
- Use `getSentenceContext` to widen the excerpt if needed

**Format output:**
```
10-Q Disclosure (Footnote [X]) [MARKER]:
> "[Full text or substantial excerpt]"
```

---

## EXTRACTION CATEGORY 2: PURCHASE COMMITMENTS (CAPITAL & OPERATING)

### How to find it in the corpus
`searchSentences` on the latest `transcriptId` with `topicSearchQuery` such as:
- "unconditional purchase obligations"
- "purchase commitments"
- "non-cancelable contractual commitments"
- "cloud capacity commitments" / "supply and energy commitments"

Also `getDocumentSummary(focus:"purchase commitments")`. Capture amounts and timing bands (e.g., amount due in year 1, year 2) as disclosed.

### Keywords (as they appear in filings)
- "unconditional purchase obligations"
- "capital commitments"
- "purchase agreements"
- "supply commitments"
- "minimum purchase requirements"

### Purchase Commitments Table

**Format output:**

| Commitment Type | Amount ($B) | Timing | Notes | Source |
|-----------------|-------------|--------|-------|--------|
| [Type] | $[B] | [When due] | [Description] | [MARKER] |

**Examples of commitment types to look for:**
- Semiconductor/chip purchase commitments
- Cloud infrastructure/server purchases
- Real estate development agreements
- Equipment financing commitments
- Power/energy contracts

**Total to calculate:**
```
Total Purchase Commitments: $[SUM in B]
```

---

## EXTRACTION CATEGORY 3: GUARANTEES & LEASE BACKSTOPS

### How to find it in the corpus
`searchSentences` on the latest `transcriptId` with `topicSearchQuery` such as:
- "residual value guarantee"
- "lease backstop" / "credit support"
- "maximum exposure to loss" (VIE / joint venture disclosures)
- "indemnification guarantee"

Also check the non-marketable equity investments / VIE note via `getDocumentSummary(focus:"residual value guarantee maximum exposure")` — these guarantees are often disclosed there rather than in the commitments note.

### Keywords
- "guarantee"
- "residual value guarantee"
- "lease backstop"
- "indemnification"
- "credit support"
- "liquidity guarantee"
- "fair value guarantee"

### Guarantee Table

**Format output:**

| Guarantor | Guarantee Type | Max Amount ($B) | Trigger/Term | Status | Source |
|-----------|----------------|-----------------|--------------|--------|--------|
| [Name/Entity] | [Type] | $[B] | [When triggered] | [Active/Contingent] | [MARKER] |

**Special attention to:**
- Meta Hyperion/El Paso structures (RVG guarantees)
- Any third-party lease / JV / SPV guarantees
- Supplier financial guarantees or backstops

**Note format:**
```
Guarantee Name: [Official name]
Structure: [SPV / JV / direct guarantee]
Maximum Exposure: $[B]
Term: [Duration, e.g., "16-year guarantee period"]
Trigger: [What triggers payment]
```

---

## EXTRACTION CATEGORY 4: CONTINGENT LIABILITIES

### How to find it in the corpus
`searchSentences` on the latest `transcriptId` with `topicSearchQuery` such as:
- "legal proceedings" / "litigation contingency"
- "loss contingency reasonably possible"
- "tax contingency" / "environmental remediation"
Also `getDocumentSummary(focus:"contingencies and legal proceedings")`.

### What to Capture
- Any litigation with potential material obligation (>$100M exposure)
- Environmental remediation obligations
- Tax contingencies (if >$100M range)
- Warranty or product recall obligations
- Regulatory or compliance contingencies

**Format output:**
```
Contingency Type: [Type]
Estimated Range: $[X–Y million]
Probability: [Remote / Reasonably possible / Probable]
Status: [Active dispute / Settlement pending / Other]
Source: [MARKER]
```

---

## EXTRACTION CATEGORY 5: SUMMARY METRICS

For each company, calculate & extract:
- Total Off-Balance-Sheet Liabilities = Leases + Purchase Commitments + Guarantees
- Percentage breakdown of each category
- Leases as % of total off-balance-sheet
- Leases as % of reported balance-sheet debt (balance-sheet debt also from the corpus filing)

---

## PART 1 OUTPUT: MARKDOWN REPORT WITH TABLES

### Report Structure Template

```markdown
# Hyper 5 Off-Balance-Sheet Liabilities Analysis
**Source: SEC 10-Q/10-K Filings via Pronto SEC Filings corpus — Direct Extraction**

## Executive Summary: Total Off-Balance-Sheet Liabilities

| Company | Leases ($B) | Purchase Commitments ($B) | Guarantees ($B) | Contingent ($B) | Total ($B) |
|---------|-------------|--------------------------|-----------------|-----------------|-----------|
| Alphabet | $[X] | $[X] | $[X] | $[X] | $[X] |
| Meta | $[X] | $[X] | $[X] | $[X] | $[X] |
| Microsoft | $[X] | $[X] | $[X] | $[X] | $[X] |
| Amazon | $[X] | $[X] | $[X] | $[X] | $[X] |
| Oracle | $[X] | $[X] | $[X] | $[X] | $[X] |
| **TOTAL** | **$[X]** | **$[X]** | **$[X]** | **$[X]** | **$[X]** |

---

## Section 1: Leases Not Yet Commenced

### COMPANY: [Name]

**Filing Details:**
- Document: [10-Q / 10-K]
- Date: [Filing date]
- Fiscal Period: [As of date]
- Source: [Pronto document title + citation MARKER]

**Total Uncommenced Lease Obligation:**
- Amount: $[X] billion (undiscounted)
- Primary Asset: [Data centers / Offices / Mixed / etc.]
- Status: Non-cancelable binding commitments

**Commencement Schedule:**
- Window: [Exact range, e.g., "Q4 2026 through Q4 2028"]
- Earliest Commencement: [Fiscal period]
- Latest Commencement: [Fiscal period or year]
- Lease Terms: [X–Y years]

**Recent Additions (Subsequent Events):**
- [If applicable: Amount $[X]B signed [Date], commences [Date]]
- [If none: None disclosed]

**10-Q/10-K Disclosure (Verbatim):**
> "[Full or substantial excerpt from footnote]" [MARKER]

**Analysis Notes:**
- [Any unusual structure, concentrations, or conditions noted in filing]

---

[Repeat for all 5 companies]

---

## Section 2: Purchase Commitments

### COMPANY: [Name]

**Total Purchase Commitments:** $[X] billion

| Commitment Type | Amount ($B) | Timing | Description | Source |
|-----------------|-------------|--------|-------------|--------|
| [Type 1] | $[X] | [Due date] | [Details] | [MARKER] |
| [Type 2] | $[X] | [Due date] | [Details] | [MARKER] |
| **TOTAL** | **$[X]** | | | |

**Filing Reference:** [Section/footnote + citation MARKER]

**10-Q/10-K Disclosure:**
> "[Excerpt]" [MARKER]

---

## Section 3: Guarantees & Lease Backstops

### COMPANY: [Name]

**Total Guarantee Exposure:** $[X] billion

| Guarantee | Type | Maximum Exposure ($B) | Term | Status | Source |
|-----------|------|----------------------|------|--------|--------|
| [Name] | [Type] | $[X] | [Duration] | [Status] | [MARKER] |
| **TOTAL** | | **$[X]** | | | |

**Details of Significant Guarantees:**

#### [Guarantee Name 1]
- Structure: [SPV / JV / Direct / Other]
- Maximum Exposure: $[X] billion
- Trigger: [What causes payment obligation]
- Term: [Years of guarantee]
- Risk Assessment: [How likely to be triggered, if disclosed]

---

## Section 4: Contingent Liabilities

| Contingency Type | Estimated Range | Probability | Status | Source |
|------------------|-----------------|-------------|--------|--------|
| [Type] | $[X–Y]M | [Probable/Reasonably possible/Remote] | [Status] | [MARKER] |

---

## Section 5: Lease Focus - Detailed Breakdown

### Total Hyper 5 Uncommenced Lease Commitments
- **Aggregate Lease Liability:** $[X]B (undiscounted)
- **Estimated PV (at ~70%):** ~$[X]B
- **Time Horizon for Commencement:** [range from filings]
- **Heaviest Commencement Window:** [Fiscal periods and companies]
- **Primary Asset Class:** Data centers and AI infrastructure

### Lease Commencement Timeline

Based on disclosed commencement windows:

| Period | Companies Commencing | Estimated Flow-On Impact |
|--------|---------------------|--------------------------|
| Q4 2026 – Q1 2027 | [List] | $[X–Y]B |
| Q2 2027 – Q4 2027 | [List] | $[X–Y]B |
| Q1 2028 – Q4 2028 | [List] | $[X–Y]B |
| 2029–2036 | [List] | $[X–Y]B (tail) |

### Company Lease Ranking by Size

| Rank | Company | Uncommenced Leases | % of Total | Commencement Start |
|------|---------|-------------------|-----------|-------------------|
| 1 | [Company] | $[X]B | [X]% | [Period] |
| ... | ... | ... | ... | ... |
| **TOTAL** | | **$[X]B** | **100%** | |

*(Populate strictly from extracted corpus figures — do not carry over illustrative values.)*

---

## Section 6: Key Findings & Observations

1. **Total Off-Balance-Sheet Commitments:** $[X]T aggregate across Hyper 5
   - Leases: $[X]T ([X]%)
   - Purchase Commitments: $[X]T ([X]%)
   - Guarantees & Backstops: $[X]B ([X]%)
   - Contingent: $[X]B ([X]%)

2. **Lease Concentration:** [X]% of off-balance-sheet liabilities are leases

3. **Commencement Pressure:** $[X]B expected to flow onto balance sheets in [window]

4. **Full Recognition Timeline:** [X]% by [year]; 100% by [year]

5. **Data-Center Focus:** [X]% of leases are for data center / AI infrastructure

---

## Source Documentation

All figures extracted directly from the Pronto SEC Filings corpus (10-Q/10-K). Cite each company with the document title link + sentence citation markers returned by the Pronto MCP tools:

- **Alphabet:** [Pronto document + MARKER]
- **Meta:** [Pronto document + MARKER]
- **Microsoft:** [Pronto document + MARKER]
- **Amazon:** [Pronto document + MARKER]
- **Oracle:** [Pronto document + MARKER]

Report Generated: [Date]
Data As Of: [Latest filing period date]
```

---

## PART 2 OUTPUT: RAW DATA TABLE (CSV/MARKDOWN)

Produce a simple CSV-style table with key metrics for each company. **Populate every value from the extracted corpus figures — the row below is a column-layout example only, not real data.**

```csv
Company,Leases_Undiscounted_B,Lease_Asset_Type,Commencement_Start_FY,Commencement_End_FY,Lease_Terms_Years,Purchase_Commitments_B,Guarantees_B,Contingent_B,Total_OBS_B
[Name],[from corpus],[from corpus],[from corpus],[from corpus],[from corpus],[from corpus],[from corpus],[from corpus],[computed]
```

---

## VERIFICATION CHECKLIST

Before finalizing extraction, confirm:

- [ ] Every figure was pulled from the Pronto **SEC Filings** corpus (no web/EDGAR/news sources)
- [ ] Each figure carries its Pronto MCP citation marker
- [ ] All 5 companies have a leases-not-yet-commenced figure extracted from their latest corpus filing
- [ ] Each figure tied to a specific filing date and fiscal period (not estimated)
- [ ] Commencement window taken verbatim from the filing (not inferred)
- [ ] Asset type clearly identified
- [ ] Lease terms captured (if disclosed)
- [ ] Verbatim footnote excerpt (with marker) included for each company
- [ ] Any post-balance-sheet amendments captured separately (only if in a corpus document)
- [ ] Purchase commitments table complete for each company
- [ ] Guarantees identified with max exposure amounts
- [ ] Contingent liabilities >$100M flagged
- [ ] Summary totals calculated correctly
- [ ] All figures in billions (convert from millions if needed)
- [ ] Filing dates and document types noted throughout
- [ ] Any figure not found is marked "Not disclosed in available filing" (never guessed or web-sourced)

---

## OUTPUT DELIVERABLES

1. **markdown_report.md** — Full detailed report with all tables and narrative
2. **raw_data_table.csv** — Simple table with key metrics (for Excel/model import)
3. **summary_stats.txt** — Executive totals and key observations
   - Total uncommenced leases: $[X]B
   - Total purchase commitments: $[X]B
   - Total guarantees: $[X]B
   - Total contingent: $[X]B
   - Grand total off-balance-sheet: $[X]B
   - Earliest commencement: [Date]
   - Latest commencement: [Year]

---
