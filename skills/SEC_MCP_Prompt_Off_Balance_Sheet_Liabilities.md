# SEC FILINGS MCP PROMPT: HYPER 5 OFF-BALANCE-SHEET LIABILITIES EXTRACTION

## OBJECTIVE

Extract ALL off-balance-sheet commitments and contingencies from Hyper 5 most recent 10-Q/10-K filings. Focus on four categories: 
1. Leases Not Yet Commenced
2. Purchase Commitments
3. Guarantees & Lease Backstops
4. Contingent Liabilities

Generate markdown report with detailed tables for each category and each company.

---

## TARGET COMPANIES & FILINGS

| Company | CIK | Filing |
|---------|-----|--------|
| **Alphabet Inc.** | 1652044 | Most recent 10-Q |
| **Meta Platforms, Inc.** | 1326801 | Most recent 10-Q |
| **Microsoft Corporation** | 789019 | Most recent 10-Q or 10-K |
| **Amazon.com, Inc.** | 1018724 | Most recent 10-Q |
| **Oracle Corporation** | 1341439 | Most recent 10-Q |

---

## EXTRACTION CATEGORY 1: LEASES NOT YET COMMENCED (ASC 842-20-50-3)

### Search Locations
- Commitments & Contingencies footnote
- Lease accounting footnote (if separate from commitments)
- MD&A section on capital commitments
- Any section titled "Leases" or "Operating Leases"

### Keywords to Find
- "leases that have not yet commenced"
- "future lease commitments"
- "operating leases not yet commenced"
- "finance leases not yet commenced"
- "lease obligations" (where status is "not yet commenced" or "future")

### Field 1: Total Uncommenced Lease Amount

**What to capture:**
- Exact dollar amount as disclosed (e.g., "$278.99 billion")
- Currency (if not USD)
- Whether amount is discounted or undiscounted
- If multiple amounts given, capture ALL (e.g., with/without certain facilities)

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
- Any leases signed AFTER balance sheet date but BEFORE filing
- Labeled as "Subsequent Events" or "Recent Commitments"
- Amount and commencement date

**Format output:**
```
Recent Additions: $[Amount] (signed [Date], commences [Date])

Note: Meta had July 2026 addition of $68B in 10-Q dated June 30, 2026
```

### Field 6: Verbatim Footnote Excerpt

**What to capture:**
- First 200-300 characters of the official footnote disclosure
- Exact quote, no paraphrase
- Include footnote number reference

**Format output:**
```
10-Q Disclosure (Footnote [X]):
> "[Full text or substantial excerpt]"
```

---

## EXTRACTION CATEGORY 2: PURCHASE COMMITMENTS (CAPITAL & OPERATING)

### Search Locations
- Commitments & Contingencies footnote
- "Contractual Obligations" table (if present)
- MD&A section on "Liquidity and Capital Resources"
- Supply agreements or equipment purchase commitments

### Keywords
- "unconditional purchase obligations"
- "capital commitments"
- "purchase agreements"
- "supply commitments"
- "minimum purchase requirements"

### Purchase Commitments Table

**Format output:**

| Commitment Type | Amount ($B) | Timing | Notes |
|-----------------|-------------|--------|-------|
| [Type] | $[B] | [When due] | [Description] |

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

### Search Locations
- Commitments & Contingencies footnote
- "Guarantees" or "Indemnifications" subsection
- Equity-method investment disclosures
- Related party guarantees

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

| Guarantor | Guarantee Type | Max Amount ($B) | Trigger/Term | Status |
|-----------|----------------|-----------------|--------------|--------|
| [Name/Entity] | [Type] | $[B] | [When triggered] | [Active/Contingent] |

**Special attention to:**
- Meta Hyperion/El Paso structures (RVG guarantees)
- Alibaba/Ant Group guarantees (if applicable)
- Any third-party lease guarantees

**Note format:**
```
Guarantee Name: [Official name]
Structure: [SPV or direct guarantee]
Maximum Exposure: $[B]
Term: [Duration, e.g., "16-year guarantee period"]
Trigger: [What triggers payment]
```

---

## EXTRACTION CATEGORY 4: CONTINGENT LIABILITIES

### Search Locations
- Commitments & Contingencies footnote
- Legal proceedings section
- Environmental contingencies
- Tax contingencies
- Insurance-related contingencies

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
```

---

## EXTRACTION CATEGORY 5: SUMMARY METRICS

For each company, calculate & extract:

- Total Off-Balance-Sheet Liabilities = Leases + Purchase Commitments + Guarantees
- Percentage breakdown of each category
- Leases as % of total off-balance-sheet
- Leases as % of reported balance-sheet debt

---

## PART 1 OUTPUT: MARKDOWN REPORT WITH TABLES

### Report Structure Template

```markdown
# Hyper 5 Off-Balance-Sheet Liabilities Analysis
**Source: SEC 10-Q/10-K Filings — Direct Extraction**

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
- Source Footnote: [Footnote reference, e.g., "Note 6 - Commitments and Contingencies"]

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
> "[Full or substantial excerpt from footnote]"

**Analysis Notes:**
- [Any unusual structure, concentrations, or conditions noted in filing]

---

[Repeat for all 5 companies]

---

## Section 2: Purchase Commitments

### COMPANY: [Name]

**Total Purchase Commitments:** $[X] billion

| Commitment Type | Amount ($B) | Timing | Description |
|-----------------|-------------|--------|-------------|
| [Type 1] | $[X] | [Due date] | [Details] |
| [Type 2] | $[X] | [Due date] | [Details] |
| **TOTAL** | **$[X]** | | |

**Filing Reference:** [Footnote # and page]

**10-Q/10-K Disclosure:**
> "[Excerpt]"

---

## Section 3: Guarantees & Lease Backstops

### COMPANY: [Name]

**Total Guarantee Exposure:** $[X] billion

| Guarantee | Type | Maximum Exposure ($B) | Term | Status |
|-----------|------|----------------------|------|--------|
| [Name] | [Type] | $[X] | [Duration] | [Status] |
| **TOTAL** | | **$[X]** | | |

**Details of Significant Guarantees:**

#### [Guarantee Name 1]
- Structure: [SPV / Direct / Other]
- Maximum Exposure: $[X] billion
- Trigger: [What causes payment obligation]
- Term: [Years of guarantee]
- Risk Assessment: [How likely to be triggered, if disclosed]

---

## Section 4: Contingent Liabilities

| Contingency Type | Estimated Range | Probability | Status |
|------------------|-----------------|-------------|--------|
| [Type] | $[X–Y]M | [Probable/Reasonably possible/Remote] | [Status] |

---

## Section 5: Lease Focus - Detailed Breakdown

### Total Hyper 5 Uncommenced Lease Commitments
- **Aggregate Lease Liability:** $[1,164]B (undiscounted)
- **Estimated PV (at 70%):** ~$[815]B
- **Time Horizon for Commencement:** 2026–2036
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
| 1 | Meta | $347B | 29.8% | Q4 2026 |
| 2 | Microsoft | $329B | 28.3% | FY2027 |
| 3 | Oracle | $260B | 22.3% | Q4 FY2026 |
| 4 | Alphabet | $91B | 7.8% | 2026 |
| 5 | Amazon | $137B | 11.8% | 2026 |
| **TOTAL** | | **$1,164B** | **100%** | |

---

## Section 6: Key Findings & Observations

1. **Total Off-Balance-Sheet Commitments:** $[2.8]T aggregate across Hyper 5
   - Leases: $[1.1]T (39%)
   - Purchase Commitments: $[1.5]T (54%)
   - Guarantees & Backstops: $[81]B (3%)
   - Contingent: $[X]B (4%)

2. **Lease Concentration:** [X]% of off-balance-sheet liabilities are leases

3. **Commencement Pressure:** $[300–400]B expected to flow onto balance sheets in Q4 2026 – Q2 2027

4. **Full Recognition Timeline:** 70–80% by end of 2028; 100% by 2036

5. **Data-Center Focus:** [X]% of leases are for data center / AI infrastructure

---

## Source Documentation

All figures extracted directly from SEC EDGAR filings:

- **Alphabet:** [Filing link or reference]
- **Meta:** [Filing link or reference]
- **Microsoft:** [Filing link or reference]
- **Amazon:** [Filing link or reference]
- **Oracle:** [Filing link or reference]

Report Generated: [Date]
Data As Of: [Latest filing date]
```

---

## PART 2 OUTPUT: RAW DATA TABLE (CSV/MARKDOWN)

Produce a simple CSV-style table with key metrics for each company:

```csv
Company,Leases_Undiscounted_B,Lease_Asset_Type,Commencement_Start_FY,Commencement_End_FY,Lease_Terms_Years,Purchase_Commitments_B,Guarantees_B,Contingent_B,Total_OBS_B
Alphabet,$91,Data Centers,2026,2031,15-20,$811,$37.4,$50,$989
Meta,$347,Data Centers/Infra,Q4 2026,2036,18-30,$349,$41,$75,$812
Microsoft,$329,Data Centers,FY2027,FY2033,1-20,$194,$25,$40,$588
Amazon,$137,Data Centers,2026,2028,15-20,$148,$15,$30,$330
Oracle,$260,Data Centers,Q4 FY2026,FY2028,15-19,$35.6,$3.3,$20,$319
```

---

## VERIFICATION CHECKLIST

Before finalizing extraction, confirm:

- [ ] All 5 companies have leases not yet commenced figure extracted from SEC filing
- [ ] Each figure tied to a specific filing date and fiscal period (not estimated)
- [ ] Commencement window taken verbatim from filing (not inferred)
- [ ] Asset type clearly identified
- [ ] Lease terms captured (if disclosed)
- [ ] Verbatim footnote excerpt included for each company
- [ ] Any post-balance-sheet amendments captured separately
- [ ] Purchase commitments table complete for each company
- [ ] Guarantees identified with max exposure amounts
- [ ] Contingent liabilities >$100M flagged
- [ ] Summary totals calculated correctly
- [ ] All figures in billions (convert from millions if needed)
- [ ] Filing dates and document types noted throughout

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
