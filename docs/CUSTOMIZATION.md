# Customization

The workflow is configured for Digikala by default, but it can be adapted to another company or market.

The current version is configurable but not fully single-node dynamic. Several downstream prompts and presentation elements still contain Digikala-specific values.

## 1. Change the company configuration

Edit:

```text
02 - OSINT Config
```

Update:

- Company English name
- Company Persian name
- Company domain
- Keyword variants
- Competitors
- Competitor keyword variants
- Lookback window
- Report title

Example:

```json
{
  "company": {
    "name": "Example Company",
    "name_fa": "شرکت نمونه",
    "domain": "example.com",
    "keywords": [
      "Example Company",
      "شرکت نمونه"
    ]
  },
  "competitors": [
    {
      "name": "Competitor A",
      "name_fa": "رقیب الف",
      "keywords": [
        "Competitor A",
        "رقیب الف"
      ]
    }
  ],
  "lookback_days": 14
}
```

Use spelling variants that actually appear in Persian and English media.

## 2. Review RSS sources for the industry

The current source set is:

```text
Mehr
KhabarOnline
Tabnak
Zoomit
```

For another industry, these may or may not provide sufficient coverage.

If you replace a feed, update both the RSS node and its normalizer:

```text
03 ↔ 04
05 ↔ 08
06 ↔ 09
07 ↔ 10
```

Keep these normalized fields accurate:

```text
source
source_domain
collection_source
collection_channel
```

## 3. Update source-quality tiers

Edit:

```text
25 - Evidence & Confidence Engine
```

Current major-news domains:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
```

Current specialist-news domain:

```text
digiato.com
```

If you add or replace collection sources, update these sets deliberately so source quality remains consistent with your methodology.

## 4. Update specialist discovery

Edit:

```text
12 - Tavily Search - Brands
```

Current specialist discovery domain:

```text
digiato.com
```

For another industry, use reputable specialist domains that actually cover that market.

Also review:

```text
13 - Split & Normalize - Tavily
```

The current node labels Tavily results as Digiato because Node 12 is restricted to Digiato.

If you allow several specialist domains, make source name/domain extraction dynamic instead of hardcoding Digiato.

## 5. Update verification domains

Both verification stages must remain aligned:

```text
28 - Tavily Cross Validation API
29 - Filter Verification Sources
```

Current allowlist:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
digiato.com
way2pay.ir
```

If you change the approved verification sources, update both nodes together.

Do not update only Node 28. Node 29 independently enforces the allowlist after search.

## 6. Keep independence rules

A verification source is not independent when it uses the same domain as the original source.

The workflow also counts only one candidate from each independent domain.

Preserve this behavior when extending verification.

## 7. Update the executive context

Edit:

```text
36 - Build Executive Brief Input
```

The current report context contains Digikala-specific values.

For another company, update:

```text
company
company_fa
```

or refactor them to read directly from Node 02.

## 8. Update the executive prompt

Edit:

```text
37 - AI Executive Analysis
```

The prompt explicitly references:

```text
Digikala
Torob
Snapp Shop
Technolife
```

Replace these with the new monitored company and competitors.

Keep the evidence constraints unchanged:

- No outside knowledge
- No invented Source IDs
- Risks only from `risk.exists = true`
- Opportunities only from `opportunity.exists = true`
- Impact and confidence must not be changed by the executive model

## 9. Update dashboard branding

Edit:

```text
40 - Generate Management Dashboard
```

Review:

- Logo URL
- Logo alt text
- Digikala-specific wording
- Company-specific section titles
- Footer text

The current Digikala logo is loaded from an external URL.

For a more reusable deployment, use an approved stable asset or remove brand-specific imagery.

## 10. Update the webhook path

Edit:

```text
41 - Dashboard Webhook
```

Current path:

```text
digikala-osint-report
```

Replace it with an appropriate path for the new deployment.

## 11. Preserve the risk-impact methodology

Do not ask the LLM to return a final Low/Medium/High impact directly.

Keep the four 0–2 factors and deterministic calculation:

```text
Business Scope
Financial Effect
Reputation Effect
Urgency
```

Mapping:

```text
0-2 = Low
3-5 = Medium
6-8 = High
```

## 12. Preserve citation controls

Keep these controls when customizing:

- Every article has a Source ID
- AI cannot invent Source IDs
- Tavily is a discovery layer
- Original article URLs remain citations
- Verification candidates pass an allowlist
- Executive citations are validated against the trusted dataset

## 13. Recommended refactor for broader reuse

If you want a truly generic version, move the following into configuration:

- RSS source names and URLs
- Source-quality tiers
- Tavily discovery domains
- Verification allowlist
- Company/competitor list
- Report context
- Dashboard logo
- Webhook path

The main nodes that currently contain reusable-but-hardcoded values are:

```text
12
13
25
28
29
36
37
40
41
```

Once these values are configuration-driven, the workflow can be adapted with much less manual editing.
