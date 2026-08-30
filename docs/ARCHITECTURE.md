# Architecture

The Commercial OSINT Agent is an n8n workflow that converts public news signals into a citation-aware management dashboard.

The central design principle is:

> Use AI where meaning matters; use deterministic rules where control, validation, scoring, and traceability matter.

## High-level flow

```text
Public RSS + Specialist Discovery
            |
            v
     Collect & Normalize
            |
            v
     Merge & Deduplicate
            |
            v
   Validate + Source IDs
            |
            v
     Semantic Relevance
            |
            v
      Business Analysis
            |
            v
   Rule-Based Risk Impact
            |
            v
 Confidence + Verification
            |
            v
 Independent Evidence Check
            |
            v
     Trusted Dataset
            |
            v
    Executive Synthesis
            |
            v
     Citation Validation
            |
            v
 Persian RTL HTML Dashboard
```

## 1. Public-source collection

### RSS

The default RSS sources are:

- Mehr
- KhabarOnline
- Tabnak
- Zoomit

Nodes:

```text
03 - RSS - Mehr
05 - RSS - KhabarOnline
06 - RSS - Tabnak
07 - RSS - Zoomit
```

Each feed is normalized by a dedicated Code node:

```text
04 - Filter & Normalize - Mehr
08 - Filter & Normalize - KhabarOnline
09 - Filter & Normalize - Tabnak
10 - Filter & Normalize - Zoomit
```

The normalizers:

- Apply the configured lookback window
- Require a title, URL, and valid publication date
- Match company/competitor keywords
- Produce a common article schema
- Attach source and source-domain metadata

### Specialist discovery

Tavily is used as a discovery layer in:

```text
11 - Build Tavily Search Jobs
12 - Tavily Search - Brands
13 - Split & Normalize - Tavily
14 - Relevance Pre-Filter
```

The current discovery domain is:

```text
digiato.com
```

Tavily is not treated as the final citation. The original article URL is retained as the evidence source.

## 2. Why Google News is not part of the current pipeline

During design testing, Google News searches for a marketplace brand produced too many shopping and product-result pages for this use case.

The selected collection strategy is therefore:

```text
RSS + curated specialist discovery + controlled cross-validation
```

This is a project design choice, not a general statement that Google News is unusable for OSINT.

## 3. Merge and deduplication

All normalized inputs are appended in:

```text
15 - Merge News Sources
```

Deduplication occurs in:

```text
16 - Deduplicate News
```

The node:

- Normalizes URLs
- Removes tracking parameters
- Removes URL fragments
- Rejects duplicate normalized URLs
- Rejects titles with high token-set similarity

## 4. Source ID assignment

Node:

```text
17 - Validate & Assign Source IDs
```

checks required fields and assigns report-local IDs:

```text
SRC-001
SRC-002
SRC-003
```

These IDs are carried through every AI and validation stage.

An AI model is never allowed to invent a new Source ID.

## 5. Semantic relevance

Articles are batched in:

```text
18 - Build Relevance Batches
```

The LLM evaluates actual business relevance in:

```text
19 - AI Semantic Relevance Check
19A - OpenRouter Model - Relevance
```

The model returns:

- `is_relevant`
- `relevance_score`
- `relevance_reason`

Node 20 validates the response and maps it to:

```text
relevant
needs_review
irrelevant
```

Node 21 routes the result.

The relevance prompt explicitly rejects incidental purchase links, retailer mentions, affiliate-style references, and other weak brand mentions.

## 6. Business analysis

Relevant articles are batched in:

```text
22 - Build Analysis Batches
```

They are interpreted by:

```text
23 - AI Business Analysis
23A - OpenRouter Model - Analysis
```

The model extracts:

- Business category
- Secondary tags
- Business sentiment
- Short summary
- Key claim
- Risk existence/type/description
- Four risk-impact factors
- Opportunity existence/type/description
- Management signal

## 7. Deterministic schema validation and impact

Node:

```text
24 - Validate Analysis & Calculate Impact
```

validates allowed values and normalizes known aliases.

Risk impact is not directly chosen by the LLM.

The LLM scores four factors from 0 to 2:

```text
business_scope
financial_effect
reputation_effect
urgency
```

The workflow sums them:

```text
0-2 = Low
3-5 = Medium
6-8 = High
```

If `risk.exists = false`, all four factors are forced to zero and impact becomes `None`.

This makes the final impact level deterministic and auditable.

## 8. Source quality and initial confidence

Node:

```text
25 - Evidence & Confidence Engine
```

currently classifies the RSS domains as major news:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
```

and the specialist source as:

```text
digiato.com
```

Source quality contributes to the initial confidence score.

An official company-domain source receives additional confidence weight.

## 9. Verification routing

Node:

```text
26 - Route Verification
```

sends an article to cross-validation when the deterministic rules decide additional evidence is useful.

Examples include:

- A meaningful risk supported by only one source
- `management_signal = investigate`
- `management_signal = consider_action` with only one source
- Low initial confidence

## 10. Verification query generation

Node:

```text
27 - Build Verification Queries
```

builds a query using:

```text
brand + validated key claim
```

This is more specific than searching only for the brand name.

## 11. Tavily cross-validation

Node:

```text
28 - Tavily Cross Validation API
```

queries Tavily using the approved verification domains:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
digiato.com
way2pay.ir
```

Social domains are explicitly excluded from this verification request.

## 12. Deterministic verification-source filter

Node:

```text
29 - Filter Verification Sources
```

enforces the same domain allowlist again after Tavily returns results.

It rejects:

- Invalid URLs
- Domains outside the allowlist
- The same domain as the original source
- Duplicate candidate domains

Only one candidate per independent domain is retained.

## 13. Evidence relationship analysis

When candidates exist, the LLM runs in:

```text
32 - AI Evidence Check
32A - OpenRouter Model - Evidence
```

Each candidate is classified as exactly one of:

```text
supports
contradicts
related_not_verifying
unrelated
```

Node 33 validates:

- Candidate URL
- Candidate domain
- Allowed relation
- Evidence score
- Source ID

Strong independent support can raise confidence.

Strong contradictory evidence forces Low confidence and human review.

## 14. No-evidence behavior

If no independent candidate is found:

```text
31 - Finalize No Evidence
```

keeps the original article and preserves its initial confidence.

The workflow does not interpret absence of independent confirmation as proof that a claim is false.

If verification is not required:

```text
34 - Finalize No Verification
```

preserves the initial confidence directly.

## 15. Trusted dataset

The three verification paths are merged in:

```text
35 - Merge Final Trusted Dataset
```

This becomes the evidence base used by the executive stage.

## 16. Executive brief input

Node:

```text
36 - Build Executive Brief Input
```

builds:

- Aggregate statistics
- Brand counts
- Category counts
- Sentiment counts
- Risk counts
- Opportunity counts
- Confidence counts
- Verification-status counts
- Compact evidence records

## 17. Executive synthesis

Nodes:

```text
37 - AI Executive Analysis
37A - OpenRouter Model - Executive
```

generate a Persian executive brief containing:

- Executive summary
- Main Digikala media topics
- Competitor focus
- Top risks
- Opportunities
- Non-binding management considerations
- Limitations

The prompt explicitly forbids outside knowledge and unsupported risk/opportunity creation.

## 18. Citation validation

Node:

```text
38 - Validate Executive Citations
```

checks the executive output against the trusted dataset.

It verifies:

- Every cited Source ID exists
- Risks cite actual `risk_exists = true` evidence
- Risk impact level and score match the validated article
- Opportunities cite actual `opportunity_exists = true` evidence
- Action priority uses the approved values

This validator is one of the main anti-hallucination controls in the workflow.

## 19. Dashboard

Node:

```text
39 - Build Dashboard Payload
```

creates the final report payload.

Node:

```text
40 - Generate Management Dashboard
```

creates a responsive Persian RTL HTML dashboard with:

- KPIs
- Executive summary
- Media topics
- Competitor section
- Risks
- Opportunities
- Recommended considerations
- Human-review queue
- Limitations
- Clickable source table

The report is optionally served by:

```text
41 - Dashboard Webhook
42 - Serve Dashboard
```

## AI responsibilities

AI is used for:

- Semantic relevance
- Business interpretation
- Evidence-relationship classification
- Executive synthesis

## Deterministic responsibilities

Code and workflow rules handle:

- Time-window filtering
- Brand keyword matching
- Deduplication
- Schema validation
- Source ID assignment
- Category/type validation
- Risk impact calculation
- Source-quality tiers
- Verification routing
- Domain allowlisting
- Confidence rules
- Citation validation

## Traceability model

A management-level statement should be traceable through:

```text
Dashboard statement
→ Source ID
→ Trusted dataset
→ Original article URL
```

That traceability is a core project requirement.
