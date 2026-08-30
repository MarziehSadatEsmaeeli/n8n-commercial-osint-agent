# Troubleshooting

This document covers common issues when importing or running the Commercial OSINT Agent.

## 1. RSS node returns 403

A `403` normally means the publisher or infrastructure is blocking the automated request.

Current configured feeds:

```text
Mehr
https://www.mehrnews.com/rss

KhabarOnline
https://www.khabaronline.ir/rss

Tabnak
https://www.tabnak.ir/fa/rss/allnews

Zoomit
https://www.zoomit.ir/feed/
```

Test each feed node independently.

If one publisher starts blocking n8n, replace that feed and update its matching normalizer and source-quality configuration.

## 2. RSS node returns 404

A `404` generally means the publisher changed or removed the RSS endpoint.

Confirm the URL manually before changing workflow logic.

## 3. RSS node works but returns no relevant items

Check:

- `lookback_days` in Node 02
- Company keywords
- Competitor keywords
- Publication dates
- Whether the feed contains the target brand
- Output of the matching Filter & Normalize node

A valid RSS feed can legitimately produce zero target-brand items.

## 4. Missing credentials after import

This is expected in the sanitized GitHub workflow.

Assign your own credentials to:

```text
12 - Tavily Search - Brands
19A - OpenRouter Model - Relevance
23A - OpenRouter Model - Analysis
28 - Tavily Cross Validation API
32A - OpenRouter Model - Evidence
37A - OpenRouter Model - Executive
```

## 5. Tavily community node is not recognized

Node 12 uses:

```text
@tavily/n8n-nodes-tavily.tavily
```

Your n8n environment must support that community node.

If it is unavailable, install/enable the compatible package or replace the discovery stage with a standard HTTP Request implementation.

## 6. Tavily returns 401 or 403

Check the credential assigned to:

```text
28 - Tavily Cross Validation API
```

It must send:

```text
Authorization: Bearer <YOUR_TAVILY_API_KEY>
```

Also confirm that the Tavily key is active.

## 7. Tavily verification returns unexpected domains

The workflow uses two allowlist controls:

```text
28 - Tavily Cross Validation API
29 - Filter Verification Sources
```

Both should contain:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
digiato.com
way2pay.ir
```

If you add a domain to only one node, results can be filtered unexpectedly.

## 8. OpenRouter model fails or is unavailable

The current workflow uses free OpenRouter routes.

Model availability can change.

Check:

- OpenRouter credential
- Model availability
- Account access
- Rate limits
- Provider status

If necessary, select another compatible model while preserving the expected JSON output schema.

## 9. AI returns invalid JSON

Nodes such as 20, 24, 33, and 38 deliberately fail when AI output is malformed.

Common causes:

- Extra prose around JSON
- Missing required fields
- Invalid enum values
- Model truncation
- Unreliable free-model routing

Recommended actions:

- Re-run once
- Inspect the preceding model output
- Keep temperature low
- Use a model with stronger structured-output reliability
- Preserve the validation node instead of bypassing it

## 10. `Invalid or invented Source ID`

An AI model returned a Source ID that is not present in the validated input.

This is rejected by design.

Check that the prompt still says:

```text
Preserve the exact Source ID.
Never invent a new Source ID.
```

## 11. `Invalid category`

Node 24 only accepts approved categories or aliases.

If a useful synonym appears repeatedly, map it through the controlled alias table.

Do not accept arbitrary categories without validation if consistent reporting matters.

## 12. `Invalid opportunity type`

The opportunity type must match the approved schema or a known alias.

Keep the alias mapping controlled rather than disabling validation.

## 13. Risk impact appears wrong

The final impact is calculated deterministically in Node 24.

Check the four factors:

```text
business_scope
financial_effect
reputation_effect
urgency
```

Then verify the sum:

```text
0-2 = Low
3-5 = Medium
6-8 = High
```

The executive LLM is not allowed to recalculate or change this result.

## 14. Many items have Low confidence

Inspect:

```text
25 - Evidence & Confidence Engine
```

The current major-news domains are:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
```

The current specialist domain is:

```text
digiato.com
```

If you add a new source without adding it to the appropriate tier, it falls back to `other_public_source`.

## 15. Verification candidate is rejected

Node 29 can reject a Tavily result for:

```text
invalid_url
domain_not_allowed
same_as_original_source
duplicate_domain
```

Inspect:

```text
rejected_results
```

for the exact reason.

## 16. Independent confirmation is not found

This does not automatically mean the original claim is false.

The workflow uses:

```text
verification_status = no_independent_evidence_found
```

and preserves the initial confidence.

## 17. Conflicting evidence forces Low confidence

Strong contradictory evidence causes:

```text
verification_status = conflicting_evidence
final_confidence_level = Low
needs_human_review = true
```

This is expected behavior.

## 18. Executive report fails citation validation

Node 38 validates the final executive output.

Typical causes:

- Invented Source ID
- Risk cites evidence where `risk_exists` is false
- Executive model changed impact score or level
- Opportunity cites evidence where `opportunity_exists` is false
- Invalid recommended-action priority

Fix the upstream output instead of bypassing Node 38.

## 19. No competitor insight appears

The executive prompt is deliberately conservative.

If reliable evidence for a competitor is missing, it should return:

```text
evidence_level = insufficient
source_ids = []
```

This is preferable to inventing a competitor strategy.

## 20. Dashboard logo does not load

Node 40 uses an external Digikala logo URL.

If the external host is unavailable or blocked, the rest of the dashboard can still render.

Replace the logo URL with another approved asset if needed.

## 21. Dashboard webhook is slow or times out

The current webhook path can cause the complete OSINT pipeline to run before HTML is returned.

The browser may therefore wait for:

- RSS collection
- Tavily
- Multiple LLM calls
- Verification
- Executive synthesis

For testing, inspect the HTML directly from:

```text
40 - Generate Management Dashboard
```

For production, a better pattern is:

```text
Scheduled workflow generates and stores latest report
+
Lightweight endpoint serves the stored HTML
```

This avoids rebuilding the report on every page request.

## 22. Workflow works in one n8n instance but not another

Check compatibility for:

- n8n core version
- LangChain nodes
- OpenRouter node
- Tavily community node
- Credential types

Reassign all credentials after import.

## 23. Debugging order

Inspect the earliest stage where output becomes wrong:

```text
02 Config
→ 03/05/06/07 RSS
→ 04/08/09/10 Normalize
→ 12/13 Tavily Discovery
→ 15 Merge
→ 16 Dedup
→ 17 Source IDs
→ 20 Relevance
→ 24 Analysis Validation
→ 25 Confidence
→ 29 Verification Candidates
→ 33 Final Confidence
→ 35 Trusted Dataset
→ 38 Citation Validation
→ 40 Dashboard
```

Fix the first incorrect stage rather than modifying later nodes to compensate.
