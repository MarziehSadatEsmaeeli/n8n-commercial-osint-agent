# Documentation

This folder contains the technical documentation for the Commercial OSINT Agent.

## Documents

- [SETUP.md](SETUP.md) — import, credentials, RSS checks, and first run
- [ARCHITECTURE.md](ARCHITECTURE.md) — end-to-end workflow design and guardrails
- [CUSTOMIZATION.md](CUSTOMIZATION.md) — adapt the workflow to another company or industry
- [SECURITY.md](SECURITY.md) — credential hygiene, public-source boundaries, and webhook considerations
- [TROUBLESHOOTING.md](TROUBLESHOOTING.md) — common runtime and validation issues

## Current default case study

The repository is currently configured for:

- Company: Digikala
- Competitors: Snapp Shop, Torob, Technolife
- Lookback window: 14 days
- Primary RSS sources: Mehr, KhabarOnline, Tabnak, Zoomit
- Specialist Tavily discovery source: Digiato
- Cross-validation domains: Mehr, KhabarOnline, Tabnak, Zoomit, Digiato, Way2Pay

The public workflow file is sanitized and does not contain real API keys or private n8n credential identifiers.
