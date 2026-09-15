# SEC FILINGS MCP PROMPT: HYPER 5 OFF-BALANCE-SHEET LIABILITIES EXTRACTION

## OBJECTIVE

Extract ALL off-balance-sheet commitments and contingencies from Hyper 5 most recent 10-Q/10-K filings. Focus on four categories:
1. Leases Not Yet Commenced
2. Purchase Commitments
3. Guarantees & Lease Backstops
4. Contingent Liabilities

Generate a markdown report with detailed tables for each category and each company, a raw-data CSV, a summary-stats file, AND a cross-sectional visualization of the Hyper 5.

---

## DATA SOURCE — READ FIRST (MANDATORY)

**All values MUST be extracted from the Pronto MCP financial-document corpus, NOT from the internet.**

- Do **NOT** use `Web_Search`, SEC EDGAR, news articles, or any external website for figures. Every dollar amount, date, term, and verbatim excerpt must come from a document returned by the Pronto MCP tools.
- Use the **`SEC Filings`** corpus (10-K, 10-Q, 8-K, annual/quarterly reports) on every Pronto MCP call. Set `corpus: ["SEC Filings"]`.
- Every figure in the output must carry the inline citation marker that the Pronto MCP tool returns (the `[MARKER](url)` embedded in sentence/text fields). Do not invent, reword, or drop these markers, and never construct URLs yourself.
- **Cite the Pronto MCP website as the source.** Every citation link points to the Pronto MCP website (`https://spglobal.prontonlp.com/...`) — the document link (`$DOCID_...`) and the sentence link (`$SENTID_...`) returned by the tools. Always surface these Pronto MCP links inline next to each figure, and list the per-company Pronto MCP document link in the Source Documentation section, so every number is traceable back to the Pronto MCP source. Do not substitute any other website (EDGAR, company IR, news) for the citation.
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
5. Repeat for all five companies, then compile the report / CSV / summary / visualization.

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

All figures extracted directly from the Pronto SEC Filings corpus (10-Q/10-K). Cite each company with the Pronto MCP document link (`https://spglobal.prontonlp.com/...` `$DOCID_...`) plus the sentence citation markers (`$SENTID_...`) returned by the Pronto MCP tools. The Pronto MCP website is the cited source of record — do not substitute EDGAR, company IR pages, or news links:

- **Alphabet:** [Pronto MCP document link + MARKER]
- **Meta:** [Pronto MCP document link + MARKER]
- **Microsoft:** [Pronto MCP document link + MARKER]
- **Amazon:** [Pronto MCP document link + MARKER]
- **Oracle:** [Pronto MCP document link + MARKER]

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

## PART 3 OUTPUT: CROSS-SECTIONAL VISUALIZATION (MANDATORY)

After the report and CSV are built, produce a single cross-sectional comparison figure of the Hyper 5 using the extracted figures. Use `matplotlib` in the sandbox.

**Rules:**
- Feed the function ONLY values extracted from the Pronto MCP corpus. Pass `None` for any figure not disclosed / not aggregated in that company's 10-Q — the function renders these as "n/d" (not disclosed), never as `0` or an estimate.
- Write the PNG under `/tmp` first, then copy it into `/workspace/` (Office/zip writers fail directly on the stage mount; PNG via matplotlib is fine either way, but keep the copy step for consistency).
- Surface the file to the user with an `<asset>` tag pointing at the `/workspace/` path.
- Keep the source note on the figure crediting the **Pronto SEC Filings corpus** as the source (matching the citation rule above).

### Reusable function (`scripts/plot_hyper5_cross_section.py`)

```python
import matplotlib
matplotlib.use("Agg")
import matplotlib.pyplot as plt
import numpy as np


def plot_hyper5_cross_section(leases, purchase, guarant, conting, addition,
                              out_path="/tmp/hyper5_cross_section.png",
                              companies=("Oracle", "Meta", "Microsoft", "Alphabet", "Amazon")):
    """Render a 4-panel cross-sectional comparison of Hyper 5 off-balance-sheet liabilities.

    Each argument is a dict keyed by company name -> value in $B (float), or None
    when the figure is NOT disclosed / not aggregated in that company's 10-Q.
    None is drawn as "n/d" (never 0, never estimated).
      leases   : leases not yet commenced (undiscounted, $B)
      purchase : purchase commitments ($B)
      guarant  : guarantees / lease backstops ($B)
      conting  : contingent liabilities accrued/exposure ($B)
      addition : subsequent-event lease additions ($B), 0 if none
    Returns the output path.
    """
    plt.rcParams.update({"font.size": 10, "axes.titlesize": 12, "axes.titleweight": "bold",
                         "figure.facecolor": "white", "axes.facecolor": "#fbfbfd"})
    companies = list(companies)
    colors = {"Oracle": "#C74634", "Meta": "#0866FF", "Microsoft": "#00A4EF",
              "Alphabet": "#EA4335", "Amazon": "#FF9900"}
    ccol = [colors.get(c, "#555555") for c in companies]
    x = np.arange(len(companies))

    fig = plt.figure(figsize=(15, 10))
    gs = fig.add_gridspec(2, 2, hspace=0.38, wspace=0.22)

    def barlabels(ax, xs, vals, fmt="${:,.0f}B", na="n/d"):
        for xi, v in zip(xs, vals):
            if v is None or (isinstance(v, float) and np.isnan(v)):
                ax.text(xi, 2, na, ha="center", va="bottom", fontsize=8, color="#888")
            else:
                ax.text(xi, v, fmt.format(v), ha="center", va="bottom", fontsize=8.5, fontweight="bold")

    # Panel A: leases not yet commenced (+ subsequent-event additions stacked/hatched)
    axA = fig.add_subplot(gs[0, 0])
    base = [leases[c] or 0 for c in companies]
    add = [addition.get(c, 0) or 0 for c in companies]
    axA.bar(x, base, color=ccol, edgecolor="white")
    axA.bar(x, add, bottom=base, color=ccol, alpha=0.4, hatch="//", edgecolor="white",
            label="Subsequent-event addition")
    for xi, c in zip(x, companies):
        if leases[c] is None:
            axA.text(xi, 4, "n/d in 10-Q", ha="center", va="bottom", fontsize=8, color="#888")
        else:
            tot = (leases[c] or 0) + (addition.get(c, 0) or 0)
            lbl = f"${leases[c]:,.0f}B" + (f"\n(+${addition[c]:,.0f}B)" if addition.get(c) else "")
            axA.text(xi, tot, lbl, ha="center", va="bottom", fontsize=8.5, fontweight="bold")
    axA.set_title("A. Leases Not Yet Commenced (undiscounted, $B)")
    axA.set_xticks(x); axA.set_xticklabels(companies)
    axA.set_ylim(0, max([b + a for b, a in zip(base, add)] + [1]) * 1.25)
    axA.legend(loc="upper right", fontsize=8, frameon=False)
    axA.spines[["top", "right"]].set_visible(False)

    # Panel B: purchase commitments
    axB = fig.add_subplot(gs[0, 1])
    pv = [purchase[c] for c in companies]
    axB.bar(x, [v or 0 for v in pv], color=ccol, edgecolor="white")
    barlabels(axB, x, pv, na="not agg. in 10-Q")
    axB.set_title("B. Purchase Commitments ($B)")
    axB.set_xticks(x); axB.set_xticklabels(companies)
    axB.set_ylim(0, max([v or 0 for v in pv] + [1]) * 1.2)
    axB.spines[["top", "right"]].set_visible(False)

    # Panel C: guarantees & contingent (grouped)
    axC = fig.add_subplot(gs[1, 0])
    w = 0.38
    gv = [guarant[c] for c in companies]; cv = [conting[c] for c in companies]
    axC.bar(x - w / 2, [v or 0 for v in gv], w, color="#6C5CE7", edgecolor="white", label="Guarantees / backstops")
    axC.bar(x + w / 2, [v or 0 for v in cv], w, color="#00B894", edgecolor="white", label="Contingent (accrued/exposure)")
    for xi, v in zip(x - w / 2, gv):
        axC.text(xi, (v or 0), "" if v is None else f"${v:,.1f}", ha="center", va="bottom", fontsize=7.5, fontweight="bold")
    for xi, v in zip(x + w / 2, cv):
        axC.text(xi, (v or 0), "" if v is None else f"${v:,.1f}", ha="center", va="bottom", fontsize=7.5, fontweight="bold")
    axC.set_title("C. Guarantees & Contingent Liabilities ($B)")
    axC.set_xticks(x); axC.set_xticklabels(companies)
    axC.set_ylim(0, max([v or 0 for v in gv + cv] + [1]) * 1.25)
    axC.legend(loc="upper right", fontsize=8, frameon=False)
    axC.spines[["top", "right"]].set_visible(False)

    # Panel D: total disclosed off-balance-sheet stack
    axD = fig.add_subplot(gs[1, 1])
    cats = [("Leases", leases, "#0984E3"), ("+Subseq. lease", addition, "#74B9FF"),
            ("Purchase", purchase, "#E17055"), ("Guarantees", guarant, "#6C5CE7"),
            ("Contingent", conting, "#00B894")]
    bottoms = np.zeros(len(companies))
    for name, d, col in cats:
        vals = np.array([(d.get(c) or 0) for c in companies], dtype=float)
        axD.bar(x, vals, bottom=bottoms, color=col, edgecolor="white", label=name)
        bottoms += vals
    for xi, tot in zip(x, bottoms):
        axD.text(xi, tot, f"${tot:,.0f}B", ha="center", va="bottom", fontsize=8.5, fontweight="bold")
    axD.set_title("D. Total Disclosed Off-Balance-Sheet Stack ($B)")
    axD.set_xticks(x); axD.set_xticklabels(companies)
    axD.set_ylim(0, max(bottoms.tolist() + [1]) * 1.2)
    axD.legend(loc="upper right", fontsize=7.5, frameon=False)
    axD.spines[["top", "right"]].set_visible(False)

    fig.suptitle("Hyper 5 Off-Balance-Sheet Liabilities — Cross-Sectional Comparison",
                 fontsize=16, fontweight="bold", y=0.975)
    fig.text(0.5, 0.005,
             "Source: most recent 10-Q per company (Pronto SEC Filings corpus). "
             "n/d = not disclosed / not aggregated as a single figure in the 10-Q.",
             ha="center", fontsize=8, color="#666")
    fig.savefig(out_path, dpi=150, bbox_inches="tight")
    return out_path


if __name__ == "__main__":
    # Example: replace ALL values with figures extracted from the Pronto MCP corpus.
    # Use None wherever the 10-Q does not disclose / aggregate the figure.
    leases   = {"Oracle": 288.0, "Meta": 278.99, "Microsoft": 196.6, "Alphabet": 85.2, "Amazon": None}
    purchase = {"Oracle": 34.15, "Meta": 349.31, "Microsoft": None, "Alphabet": 707.0, "Amazon": None}
    guarant  = {"Oracle": None, "Meta": 41.0, "Microsoft": None, "Alphabet": 51.4, "Amazon": None}
    conting  = {"Oracle": None, "Meta": 7.14, "Microsoft": 0.647, "Alphabet": 17.4, "Amazon": 7.1}
    addition = {"Oracle": 0, "Meta": 68.0, "Microsoft": 0, "Alphabet": 5.8, "Amazon": 0}
    print(plot_hyper5_cross_section(leases, purchase, guarant, conting, addition))
```

**Panels produced:**
- **A.** Leases not yet commenced, with subsequent-event additions shown as a hatched cap on each bar.
- **B.** Purchase commitments.
- **C.** Guarantees / backstops vs. contingent liabilities (grouped).
- **D.** Total disclosed off-balance-sheet stack (composition per company).

**Interpretation note to include in the writeup:** `n/d` bars are disclosure gaps, not zeros, so the Panel D totals are NOT apples-to-apples across companies that disclose fewer discrete aggregates (typically Amazon and Microsoft). State this caveat whenever the stacked totals are compared.

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
4. **hyper5_cross_section.png** — Cross-sectional visualization (Part 3), surfaced to the user via an `<asset>` tag

---

## VERIFICATION CHECKLIST

Before finalizing extraction, confirm:

- [ ] Every figure was pulled from the Pronto **SEC Filings** corpus (no web/EDGAR/news sources)
- [ ] Each figure carries its Pronto MCP citation marker
- [ ] Citations link to the Pronto MCP website (`https://spglobal.prontonlp.com/...`); no other website is substituted as the source
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
- [ ] Cross-sectional visualization generated with `None` for undisclosed figures (rendered "n/d", never 0), saved to `/workspace/`, and surfaced via `<asset>`
