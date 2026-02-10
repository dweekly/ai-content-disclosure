# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **W3C Community Group standards proposal** — not a software application. The canonical repo for spec work, issues, and PRs is [w3c-cg/ai-content-disclosure](https://github.com/w3c-cg/ai-content-disclosure/). This personal repo (`dweekly/ai-content-disclosure`) contains the explainer (`README.md`) and legislative tracking.

The proposal enables web authors to declare AI involvement at element-level granularity using four values: `none`, `ai-assisted`, `ai-generated`, `autonomous`. It complements (not replaces) the IETF `AI-Disclosure` HTTP header and C2PA cryptographic provenance.

## Repository Structure

- `README.md` — The spec explainer (primary deliverable; follows [W3C Explainer format](https://www.w3.org/TR/explainer-explainer/))
- `ROADMAP.md` — Stack-ranked engagement priorities
- `CHANGELOG.md` — Version history of the explainer
- `examples/` — Working HTML examples demonstrating attribute usage (basic, mixed-content, JSON-LD)

## Development Workflow

There is no build system, test suite, or CI. Work is documentation, regulatory tracking, and stakeholder engagement.

- Use git branches for major changes (e.g., `codex/` prefix for content work)
- Commit messages: imperative tense, prefixed with `docs:` for spec changes

## Key Standards Alignment

Changes to the disclosure vocabulary or attribute design **must** maintain alignment with:

- **IETF**: `AI-Disclosure` header modes (`none`, `ai-modified`, `ai-originated`, `machine-generated`) — [draft-abaris-aicdh-00](https://www.ietf.org/archive/id/draft-abaris-aicdh-00.html)
- **IPTC**: Digital Source Type taxonomy — [cv.iptc.org/newscodes/digitalsourcetype/](https://cv.iptc.org/newscodes/digitalsourcetype/)
- **WHATWG**: Page-level meta tag proposal — [html#9479](https://github.com/whatwg/html/issues/9479)
- **Schema.org**: Proposed `aiDisclosure` property on `CreativeWork` — [schemaorg#3391](https://github.com/schemaorg/schemaorg/issues/3391)

The mapping tables in README.md sections "Disclosure Values" and "Vocabulary Alignment with IPTC" are the canonical cross-references. Keep them in sync.

## Active Stakeholder Tracking

- W3C CG: [w3.org/community/ai-content-disclosure](https://www.w3.org/community/ai-content-disclosure/) (David co-chairs)
- WICG: [proposals#261](https://github.com/WICG/proposals/issues/261)
- Chromium: [ChromeStatus feature](https://chromestatus.com/feature/5078123181899776)
- Mozilla: [standards-positions#1344](https://github.com/mozilla/standards-positions/issues/1344)
- WebKit: [standards-positions#605](https://github.com/WebKit/standards-positions/issues/605)

## Content Guidelines

- The README legislative tracker is scoped to **enacted and pending U.S. state laws affecting AI-generated text/HTML disclosure**. Do not expand scope to media-only or federal-only laws unless they directly affect HTML text labeling.
- `ai-disclosure="none"` is a **positive assertion** (human-only). Absence of the attribute means "unknown." This distinction is critical throughout the spec.
- The proposal is **voluntary disclosure**, not detection. Frame accordingly — comparisons to `rel=nofollow` and Schema.org are the right analogies.
- FAQ and framing should focus on **clear labeling for reader/agent decision-making** — not editorializing about how people should feel about AI-generated content. The goal is transparent signals; value judgments are for the consumer of the signal.
