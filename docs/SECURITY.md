# Security

The Commercial OSINT Agent is designed for public-source intelligence.

The public repository should not contain real credentials, internal company information, private documents, or production secrets.

## 1. Never commit credentials

Do not commit:

- OpenRouter API keys
- Tavily API keys
- Bearer tokens
- Passwords
- Private webhook secrets
- Private n8n instance identifiers
- Real n8n credential IDs

Use the n8n Credentials UI.

## 2. Sanitized workflow

The repository version of the workflow should contain placeholders instead of real credential bindings.

Typical placeholders:

```text
REPLACE_WITH_OPENROUTER_CREDENTIAL
REPLACE_WITH_TAVILY_CREDENTIAL
REPLACE_WITH_TAVILY_BEARER_CREDENTIAL
REPLACE_ME
```

The public workflow should remain:

```json
"active": false
```

until the person importing it configures their own environment.

## 3. External services and data flow

The current workflow sends public-source information to external services.

### OpenRouter receives

- Article excerpts
- Structured article metadata
- Key claims
- Verification-candidate excerpts
- Aggregated evidence used for executive synthesis

### Tavily receives

- Company/competitor search terms
- Verification queries built from public key claims

For this reason, do not insert confidential or internal company information into the workflow unless the external data flow has been explicitly approved.

## 4. Public-source boundary

The intended inputs are:

- Public RSS feeds
- Public news articles
- Public specialist media
- Public web-search results

Do not place the following into prompts or Code nodes:

- Customer data
- Employee secrets
- Access tokens
- Internal reports
- Confidential financial data
- Private incident details
- Authentication material

## 5. Treat web content as untrusted input

News content is external text and should be treated as untrusted data.

The AI prompts should not follow instructions contained inside collected articles.

Preserve prompt constraints such as:

- Use only supplied evidence
- Do not use outside knowledge
- Do not invent Source IDs
- Return the required JSON schema
- Do not create unsupported risks or opportunities

The deterministic validators are part of the security and integrity model.

## 6. Source allowlists

Cross-validation is restricted to:

```text
mehrnews.com
khabaronline.ir
tabnak.ir
zoomit.ir
digiato.com
way2pay.ir
```

This allowlist is enforced in:

```text
28 - Tavily Cross Validation API
29 - Filter Verification Sources
```

Keep both nodes synchronized when the list changes.

## 7. Social media

The current verification request excludes common social-media domains.

Social content should be treated as a signal rather than independent confirmation unless your methodology explicitly defines and validates it as evidence.

## 8. Citation integrity

Tavily is used for discovery.

The final report should retain original article URLs as citations.

A candidate should not be considered supporting evidence just because it:

- Mentions the same company
- Uses similar keywords
- Discusses the same general topic

The evidence relationship must be validated.

## 9. Webhook exposure

The dashboard can be served by:

```text
41 - Dashboard Webhook
42 - Serve Dashboard
```

Current path:

```text
digikala-osint-report
```

An unprotected public webhook can expose the generated report to anyone who can reach the URL.

For non-public deployments, use controls available in your environment, such as:

- Authenticated reverse proxy
- Private network/VPN
- Application-layer authentication
- Access-control gateway

Do not rely on an obscure webhook path as the only access control.

## 10. External logo dependency

The current HTML dashboard loads the Digikala logo from an external website.

A viewer's browser can contact that external host when the report is opened.

If that is not acceptable, replace it with an approved internal/static asset or remove the external image.

## 11. GitHub release checklist

Before publishing a workflow export:

```text
[ ] No API keys
[ ] No Bearer tokens
[ ] No passwords
[ ] No real credential IDs
[ ] No private instance IDs
[ ] No confidential information
[ ] Workflow inactive by default
[ ] Sample output contains only public information
[ ] Webhook exposure documented
[ ] External data flows documented
[ ] Source allowlists reviewed
```

## 12. Analytical safety

Public-source reporting can contain incomplete or conflicting claims.

The workflow therefore preserves:

- Source traceability
- Confidence levels
- Verification status
- Human review for conflicting evidence
- Explicit limitations

The dashboard should support management review, not replace human judgment.
