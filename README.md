# AI Content Disclosure for HTML

## Authors

- [David E. Weekly](https://github.com/dweekly)

## Participate

- [W3C AI Content Disclosure Community Group](https://www.w3.org/community/ai-content-disclosure/)
  ([GitHub repo](https://github.com/w3c-cg/ai-content-disclosure/)) — the
  canonical home for spec work, issues, and pull requests
- [WICG Proposals Issue #261](https://github.com/WICG/proposals/issues/261)

## Table of Contents

- [Introduction](#introduction)
- [Problem / Motivation](#problem--motivation)
- [Goals](#goals)
- [Non-Goals](#non-goals)
- [Proposed Solution](#proposed-solution)
- [Key Scenarios](#key-scenarios)
- [Detailed Design](#detailed-design)
- [What Counts as AI?](#what-counts-as-ai)
- [Alternatives Considered](#alternatives-considered)
- [Privacy and Security Considerations](#privacy-and-security-considerations)
- [Accessibility Considerations](#accessibility-considerations)
- [Internationalization Considerations](#internationalization-considerations)
- [FAQ / Common Objections](#faq--common-objections)
- [U.S. State Legislative Tracker (Text/HTML Focus)](#us-state-legislative-tracker-texthtml-focus)
- [Legislative Gap Analysis](#legislative-gap-analysis)
- [Stakeholder Feedback](#stakeholder-feedback)
- [References](#references)

---

## Introduction

Web pages increasingly contain text produced with varying degrees of AI
involvement — from light AI-assisted editing to fully autonomous generation.
There is currently no standard HTML mechanism for authors to disclose AI
involvement at element-level granularity within a page.

This explainer proposes an `ai-disclosure` HTML attribute and a companion
`<meta name="ai-disclosure">` tag, enabling authors to declare the degree
of AI involvement in any section of a web page.

## Problem / Motivation

A modern news article page might contain a human-written investigation
alongside an AI-generated summary sidebar and AI-moderated user comments.
Today, there is no standard way to label these sections differently.

Existing approaches operate at coarser granularity:

- **[WHATWG HTML #9479](https://github.com/whatwg/html/issues/9479)** proposes
  a page-level `<meta>` tag with four values. It does not support marking
  individual elements. Commenters on that issue (42+) identified element-level
  granularity as the critical missing capability.

- **[IETF draft-abaris-aicdh-00](https://www.ietf.org/archive/id/draft-abaris-aicdh-00.html)**
  defines an `AI-Disclosure` HTTP response header. It applies to entire
  HTTP responses and cannot distinguish mixed content within a page.

- **[C2PA 2.2](https://spec.c2pa.org/)** provides cryptographic provenance
  for media files (images, video, audio). It does not support HTML text
  content and is designed for file-level, not element-level, assertions.

### Regulatory Context

The [EU AI Act Article 50](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
(effective August 2026) requires that AI-generated text content be "marked in
a machine-readable format and detectable as artificially generated or
manipulated." Major platforms (YouTube, Meta, TikTok) already require AI
disclosure in their policies. A standard mechanism would serve both regulatory
compliance and voluntary transparency.

## Goals

1. Enable authors to declare AI involvement at **element-level granularity**
   in HTML
2. Provide a **machine-readable signal** usable by browsers, search engines,
   accessibility tools, and research crawlers
3. **Align with existing vocabularies** — specifically the
   [IETF AI-Disclosure](https://www.ietf.org/archive/id/draft-abaris-aicdh-00.html)
   header modes and
   [IPTC Digital Source Type](https://cv.iptc.org/newscodes/digitalsourcetype/)
   taxonomy — for cross-standard consistency
4. **Complement** (not replace) HTTP-level and cryptographic provenance
   mechanisms

## Non-Goals

- **Detecting** AI content (this is voluntary declaration, not detection)
- Replacing [C2PA](https://c2pa.org/) cryptographic provenance
- Preventing AI content creation or prescribing editorial policy
- Defining how browsers should **render** the disclosure (that is
  UA-specific)
- Covering non-text media (images, video, audio — handled by C2PA and IPTC)
- Solving metadata integrity (spoofing is possible with any self-declared
  metadata; verified provenance requires cryptographic systems like C2PA)

## Proposed Solution

### Page-Level Declaration (Meta Tag)

For pages with uniform AI involvement:

```html
<meta name="ai-disclosure" content="none">
<!-- or "ai-assisted", "ai-generated", "autonomous", "mixed" -->
```

The value `mixed` signals that different sections have different levels;
inspect element-level attributes for detail.

### Element-Level Declaration (HTML Attribute)

An `ai-disclosure` global attribute on any HTML element:

```html
<article>
  <section ai-disclosure="none">
    <h2>Six-Month Investigation: City Budget Shortfall</h2>
    <p>Our reporters spent six months reviewing financial records...</p>
  </section>

  <aside ai-disclosure="ai-generated" ai-model="gpt-4o" ai-provider="OpenAI">
    <h3>AI Summary</h3>
    <p>The investigation found a $4.2M discrepancy in the city's
    infrastructure fund, attributed to misclassified expenditures...</p>
  </aside>
</article>
```

### Disclosure Values

Four values, aligned with the IETF AI-Disclosure header and IPTC Digital
Source Type vocabulary:

| Value | Meaning | IETF Equivalent | IPTC Digital Source Type |
|-------|---------|-----------------|------------------------|
| `none` | No AI involvement | `none` | `digitalCapture` |
| `ai-assisted` | Human-authored, AI edited or refined | `ai-modified` | `compositeWithTrainedAlgorithmicMedia` |
| `ai-generated` | AI-generated with human prompting and/or review | `ai-originated` | `trainedAlgorithmicMedia` |
| `autonomous` | AI-generated without human oversight | `machine-generated` | `trainedAlgorithmicMedia` |

### Optional Metadata Attributes

| Attribute | Purpose | Example |
|-----------|---------|---------|
| `ai-model` | Model identifier | `"claude-3.5-sonnet"` |
| `ai-provider` | Provider name | `"Anthropic"` |
| `ai-prompt-url` | URL to prompt/methodology documentation | `"/ai-methodology#summary"` |

### Schema.org Integration (JSON-LD)

For search engine discoverability, the same information can be expressed as
structured data. The simplest form is just the level string:

```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Investigation: City Budget Shortfall",
  "author": { "@type": "Person", "name": "Jane Doe" },
  "aiDisclosure": "ai-assisted"
}
```

An expanded form supports optional metadata. All fields except `level` are
strictly optional — publishers may have legitimate reasons not to disclose
specific tools or providers:

```json
{
  "@context": "https://schema.org",
  "@type": "NewsArticle",
  "headline": "Investigation: City Budget Shortfall",
  "author": { "@type": "Person", "name": "Jane Doe" },
  "aiDisclosure": {
    "level": "ai-assisted",
    "tool": "Claude 3.5 Sonnet",
    "provider": "Anthropic",
    "description": "AI used for copy editing and fact-checking assistance",
    "methodologyUrl": "https://example.com/ai-methodology"
  }
}
```

*(Proposed as a comment on
[schemaorg/schemaorg#3391](https://github.com/schemaorg/schemaorg/issues/3391).)*

## Key Scenarios

### Scenario 1: News Article with AI Summary Sidebar

A newsroom publishes an investigative piece with a human-written article
and an AI-generated summary:

```html
<meta name="ai-disclosure" content="mixed">
<!-- ... -->
<article ai-disclosure="none">
  <h1>Exclusive: City Budget Shortfall</h1>
  <p>After six months of records review...</p>
</article>

<aside ai-disclosure="ai-generated" ai-model="gpt-4o" ai-provider="OpenAI">
  <h3>Key Takeaways (AI-Generated)</h3>
  <ul><li>$4.2M discrepancy found...</li></ul>
</aside>
```

### Scenario 2: Blog Post Written with AI Editing

A blogger writes a post and uses an LLM for grammar, style, and clarity
improvements:

```html
<meta name="ai-disclosure" content="ai-assisted">
<!-- ... -->
<article ai-disclosure="ai-assisted" ai-model="claude-3.5-sonnet"
         ai-provider="Anthropic">
  <h1>My Trip to Kyoto</h1>
  <p>The bamboo grove felt otherworldly at dawn...</p>
</article>
```

### Scenario 3: Fully Automated Content Feed

An automated system generates weather reports without per-instance human
oversight:

```html
<meta name="ai-disclosure" content="autonomous">
<!-- ... -->
<div ai-disclosure="autonomous" ai-model="weather-llm-v2"
     ai-provider="WeatherCorp">
  <h2>San Francisco Bay Area Forecast</h2>
  <p>Expect fog clearing by noon with highs near 62°F...</p>
</div>
```

### Scenario 4: Human-Only Publication Asserting Provenance

A literary journal positively asserts that no AI was used:

```html
<meta name="ai-disclosure" content="none">
<!-- ... -->
<article ai-disclosure="none">
  <h1>The Weight of Feathers</h1>
  <p>She found the letter tucked inside a volume of Neruda...</p>
</article>
```

Note: `ai-disclosure="none"` is a positive assertion. The *absence* of the
attribute means "unknown," not "none."

## Detailed Design

### Inheritance

- Element-level `ai-disclosure` attributes override the page-level
  `<meta name="ai-disclosure">` tag
- Children inherit their nearest ancestor's `ai-disclosure` value unless
  they specify their own
- Absence of the attribute means **unknown** (not "none") — this is an
  important distinction that avoids false negative assertions

### Relationship to HTTP AI-Disclosure Header

| Layer | Mechanism | Granularity |
|-------|-----------|-------------|
| HTTP | `AI-Disclosure` response header | Entire response |
| HTML page | `<meta name="ai-disclosure">` | Entire document |
| HTML element | `ai-disclosure` attribute | Any element |

These are complementary. A CDN or reverse proxy can set the HTTP header;
a CMS can set the meta tag; an author or AI tool can set element-level
attributes. None supersedes the others.

### Vocabulary Alignment with IPTC

| `ai-disclosure` value | IPTC Digital Source Type URI |
|---------------------|----------------------------|
| `none` | `http://cv.iptc.org/newscodes/digitalsourcetype/digitalCapture` |
| `ai-assisted` | `http://cv.iptc.org/newscodes/digitalsourcetype/compositeWithTrainedAlgorithmicMedia` |
| `ai-generated` | `http://cv.iptc.org/newscodes/digitalsourcetype/trainedAlgorithmicMedia` |
| `autonomous` | `http://cv.iptc.org/newscodes/digitalsourcetype/trainedAlgorithmicMedia` |

## What Counts as AI?

To address a frequently raised concern ("where do you draw the line with
grammar checkers?"), here is boundary guidance:

**Not AI** (no disclosure needed):
- Deterministic spell-check (dictionary lookup)
- Thesaurus / synonym suggestions
- Search engines used for research
- Calculators and formula solvers
- Template-based mail merge

**`ai-assisted`**:
- LLM-based grammar and style tools (Grammarly AI, etc.)
- AI summarization of human-written text
- AI-powered translation (DeepL, Google Translate neural mode)
- AI rewriting of human-provided drafts

**`ai-generated`**:
- LLM-generated drafts from human prompts
- AI expanding bullet points into prose
- Chatbot-generated responses with human review before publication

**`autonomous`**:
- Agent-generated content without per-instance human prompting
- Automated reporting systems (weather, sports scores, financial summaries)
- Content produced by persistent AI agents with standing instructions

The boundary is **generative/inferential AI** — systems trained on data
that produce novel outputs. Deterministic tools that apply fixed rules are
not covered.

## Alternatives Considered

### 1. Page-Level Meta Tag Only ([WHATWG #9479](https://github.com/whatwg/html/issues/9479))

Does not handle mixed-content pages — the single most requested feature in
that issue's 42+ comments.

### 2. HTTP Header Only ([IETF AI-Disclosure](https://www.ietf.org/archive/id/draft-abaris-aicdh-00.html))

No element-level granularity. Not accessible to client-side tools processing
the DOM. Cannot distinguish mixed content within a page.

### 3. C2PA Manifest for HTML

C2PA is file-based cryptographic provenance. HTML pages are dynamically
assembled from templates, databases, and user input — they are not single
files with stable hashes. C2PA and this proposal are complementary, not
competing.

### 4. `data-*` Attributes Only

Using `data-ai-disclosure` instead of a dedicated attribute was considered.
Trade-off: `data-*` attributes have no semantic meaning to browsers or
assistive technology. A dedicated attribute signals intent for future browser
integration (e.g., address bar indicators, accessibility announcements).

### 5. RDFa / Microdata Only

Too verbose for common cases. RDFa requires namespace declarations and
multi-attribute markup for simple assertions. The `ai-disclosure` attribute
provides a lightweight default; RDFa or JSON-LD can supplement it for richer
structured data needs.

## Privacy and Security Considerations

*Responses to the [W3C Security and Privacy Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/):*

- **No new information about the user (reader) is exposed.** The attribute
  describes the content, not the person viewing it.
- **No new fingerprinting surface.** The attribute is author-declared
  metadata, not a browser API.
- **Disclosure is voluntary and advisory.** It carries no integrity
  protection. For verified provenance, pair with C2PA.
- **Model/provider metadata is optional** to avoid requiring disclosure of
  trade secrets or proprietary tooling.
- **Prompt text is not embedded in HTML.** The optional `ai-prompt-url`
  attribute links to an external resource, giving authors control over what
  they disclose and when they revoke access.
- **The voluntary nature means it cannot be relied upon for security
  decisions**, the same as any self-declared metadata (robots.txt,
  `rel=nofollow`, Schema.org markup).

## Accessibility Considerations

- Browsers and screen readers could optionally announce AI disclosure to
  users (e.g., "AI-generated content follows")
- The `ai-disclosure` attribute should be exposed via the
  [Accessibility Object Model](https://wicg.github.io/aom/)
- No change to existing content rendering — the attribute is purely
  informational
- Useful for accessibility research: understanding how AI-generated content
  affects comprehension for users with cognitive disabilities

## Internationalization Considerations

- Attribute values (`none`, `ai-assisted`, `ai-generated`, `autonomous`)
  are English-language tokens intended for machine consumption, not display
- Human-readable presentation of disclosure status is a User Agent
  responsibility and can be localized
- The `ai-prompt-url` attribute can link to localized methodology pages

## FAQ / Common Objections

### "This is the evil bit — bad actors won't comply."

This is not a detection mechanism. It is a labeling mechanism for publishers,
regulated industries, and AI tool vendors. The EU AI Act makes compliance
mandatory for covered entities, and major platforms already require
disclosure in their terms of service.

The analogy is `rel=nofollow`: voluntary, widely adopted because it aligns
incentives, and useful despite non-universal adoption.

### "Where do you draw the line with grammar checkers?"

See [What Counts as AI?](#what-counts-as-ai) above. The boundary is
generative/inferential AI — systems trained on data that produce novel
outputs. Deterministic spell-check and thesaurus tools are excluded.

### "Metadata will be gamed like SEO dates."

True for any self-declared metadata. Verification requires pairing with
C2PA or regulatory auditing. The signal is useful to consumers who encounter
it — same as Schema.org structured data, which search engines use despite
its spoofability.

### "Different readers will react differently to AI disclosure."

Yes, and that is expected. The purpose of this proposal is not to prescribe a
single value judgment about AI use; it is to provide transparent,
machine-readable authorship signals so readers can apply their own standards.

### "Everything will be AI-touched soon, making this meaningless."

That is exactly why granularity matters. Binary "AI/not-AI" is already
inadequate. The spectrum from `none` to `autonomous` reflects the reality of
modern content workflows and remains meaningful as AI tools become ubiquitous.

## U.S. State Legislative Tracker (Text/HTML Focus)

*Last updated: 2026-02-06. Scope is intentionally limited to enacted and
pending state laws that materially affect disclosure for AI-generated text,
HTML content, or conversational web interactions. This is not legal advice.*

### Enacted and Pending (High Relevance)

| Jurisdiction | Bill / Law | Status (as of 2026-02-06) | Text/HTML-Relevant Requirement | Intersection with this proposal |
|--------------|------------|---------------------------|--------------------------------|---------------------------------|
| New York | S3008C (Part U, GBL Article 47) | **Enacted** (signed 2025-05-09; effective 180 days after enactment) | AI companion systems must give clear/conspicuous notice that user is not communicating with a human at interaction start (no more than once per day) and at least every 3 hours for continuing interactions. | Strong overlap on disclosure intent; adds runtime UX cadence requirements beyond static HTML metadata. |
| New York | S6748 | **Pending** (in Senate Consumer Protection) | Would require print/electronic publications to conspicuously mark page/webpage content created wholly or partly with generative AI. | Strong overlap for publication text/web content labeling; suggests need for mapping from machine-readable metadata to visible labels. |
| New York | S8874 | **Pending** (in Senate Internet and Technology) | Would require point-of-interaction AI disclosure, plain-English explanation of AI role, and human-assistance instructions (if applicable). | Overlaps on disclosure presence; introduces "human handoff" requirement outside current attribute set. |
| Utah | SB226 + SB332 (amending/replacing SB149 regime) | **Enacted** (SB226 signed 2025-03-27; SB332 signed 2025-03-25) | Requires disclosures in consumer interactions when asked, plus stronger disclosure requirements in high-risk regulated-service interactions. | Overlap for interactive AI disclosure; legal trigger can depend on interaction type and profession, not only HTML content authorship. |
| California | BPC §17941 (Bots), as amended over time | **Enacted** | Prohibits deceptive bot interactions in online commercial/election contexts unless bot identity is clearly disclosed. | Partial overlap; legal focus is anti-deception and bot identity in specific contexts, not general AI-content provenance. |
| California | AB3030 (Health & Safety Code §1339.75 et seq.) | **Enacted** (Chapter 848, 2024) | AI-generated patient communications must include a disclaimer and human-contact instructions, including in continuous online interactions. | Sector-specific overlap; disclosure wording/placement and human escalation requirements exceed current spec scope. |
| California | SB243 (Companion chatbots) | **Enacted** (Chapter 677, 2025) | Requires clear notice when user could believe they are interacting with a human, plus recurring disclosures for minors in continuing interactions. | Similar to NY Part U pattern; runtime interaction obligations exceed static element-level metadata. |
| Colorado | SB24-205, as amended by SB25B-004 | **Enacted** (effective-date changes passed in 2025 special session) | Requires consumer disclosure when interacting with AI systems; broader high-risk AI obligations phased to 2026. | Overlap for machine-readable disclosure signal; compliance also depends on risk-management and enforcement frameworks beyond markup. |

### Adjacent (Lower Relevance for Text/HTML Disclosure)

| Jurisdiction | Bill / Law | Status | Why adjacent |
|--------------|------------|--------|--------------|
| California | SB942 + AB853 (California AI Transparency Act changes) | **Enacted** | Primarily focuses on media provenance/detection/disclosures for image/video/audio ecosystems and platform handling, not element-level HTML text labeling. |

## Legislative Gap Analysis

### Gap Matrix

| Gap | What laws require or trend toward | Implication for this proposal | Primary owner |
|-----|-----------------------------------|-------------------------------|---------------|
| Human-visible disclosure | "Clear and conspicuous" notice to readers/users (not just machine-readable metadata) | `ai-disclosure` alone is insufficient for some statutes; needs explicit bridging guidance | Spec + publisher + UA |
| Interaction timing/cadence | Notices at start of interaction and periodically during ongoing chat | Static HTML attribute cannot express temporal cadence by itself | Browser/app/runtime layer |
| Human escalation/handoff | Instructions for contacting a human in some sectors and proposals | Outside current vocabulary (`ai-model`, `ai-provider`, etc.) | Application policy / sector regulation |
| Context-triggered duties | Different rules for healthcare, minors, elections, commerce, regulated occupations | A single generic content-provenance attribute cannot encode all legal contexts | Out-of-scope for base spec |
| Enforcement/audit duties | Risk assessments, logging, AG notifications, civil penalties | Not representable as HTML-only metadata | Organizational compliance layer |
| Non-text provenance | Watermarks/latent metadata/provenance in audio-video-image pipelines | Already complementary with C2PA/IPTC layer | Keep out-of-scope for this HTML text proposal |

## Stakeholder Feedback

| Stakeholder | Status | Link |
|-------------|--------|------|
| W3C Community Group | **Formed** (2026-02-03) | [w3.org/community/ai-content-disclosure](https://www.w3.org/community/ai-content-disclosure/) |
| Chromium | Feature Entry Filed | [ChromeStatus](https://chromestatus.com/feature/5078123181899776) |
| Gecko (Mozilla) | Position Requested | [mozilla/standards-positions#1344](https://github.com/mozilla/standards-positions/issues/1344) |
| WebKit | Position Requested | [WebKit/standards-positions#605](https://github.com/WebKit/standards-positions/issues/605) |

## References

1. [WHATWG HTML #9479: Proposal: Meta Tag for AI Generated Content](https://github.com/whatwg/html/issues/9479)
2. [IETF draft-abaris-aicdh-00: AI Content Disclosure Header](https://www.ietf.org/archive/id/draft-abaris-aicdh-00.html)
3. [WICG #104: Content Provenance and Authenticity](https://github.com/WICG/proposals/issues/104)
4. [WICG #141: Advancing Web Metadata](https://github.com/WICG/proposals/issues/141)
5. [W3C: AI & the Web — Understanding and managing the impact of Machine Learning models on the Web](https://www.w3.org/reports/ai-web-impact/)
6. [W3C Explainer Explainer](https://www.w3.org/TR/explainer-explainer/)
7. [C2PA Specification 2.2](https://spec.c2pa.org/specifications/specifications/2.2/specs/C2PA_Specification.html)
8. [IPTC Digital Source Type Vocabulary](https://cv.iptc.org/newscodes/digitalsourcetype/)
9. [EU AI Act — Regulation (EU) 2024/1689, Article 50](https://eur-lex.europa.eu/eli/reg/2024/1689/oj)
10. [EU Code of Practice on Transparency of AI-Generated Content (draft)](https://digital-strategy.ec.europa.eu/en/policies/code-practice-ai-generated-content)
11. [Bloomberg Container Timing Explainer](https://github.com/nicolo-ribaudo/container-timing) (structural model for this proposal)
12. [New York S3008C (2025) — budget bill including AI companion model disclosure requirements (Part U)](https://www.nysenate.gov/legislation/bills/2025/S3008/amendment/C)
13. [New York S6748 (2025-2026) — publication AI-use disclosure proposal](https://www.nysenate.gov/legislation/bills/2025/S6748)
14. [New York S8874 (2025-2026) — customer-service AI disclosure proposal](https://www.nysenate.gov/legislation/bills/2025/S8874)
15. [Utah SB149 (2024) — Artificial Intelligence Amendments (historical baseline)](https://le.utah.gov/~2024/bills/static/SB0149.html)
16. [Utah SB226 (2025) — Artificial Intelligence Consumer Protection Amendments](https://le.utah.gov/Session/2025/bills/enrolled/SB0226.pdf)
17. [Utah SB332 (2025) — Artificial Intelligence Revisions](https://le.utah.gov/Session/2025/bills/enrolled/SB0332.pdf)
18. [California BPC §17941 — Bots: disclosure](https://leginfo.legislature.ca.gov/faces/codes_displaySection.xhtml?sectionNum=17941.&lawCode=BPC)
19. [California AB3030 (2023-2024) — health care AI communication disclaimers](https://leginfo.legislature.ca.gov/faces/billStatusClient.xhtml?bill_id=202320240AB3030)
20. [California SB243 (2025-2026) — companion chatbot disclosure requirements](https://leginfo.legislature.ca.gov/faces/billStatusClient.xhtml?bill_id=202520260SB243)
21. [California SB942 (2023-2024) — California AI Transparency Act](https://leginfo.legislature.ca.gov/faces/billStatusClient.xhtml?bill_id=202320240SB942)
22. [California AB853 (2025-2026) — amendments to California AI Transparency Act](https://leginfo.legislature.ca.gov/faces/billStatusClient.xhtml?bill_id=202520260AB853)
23. [Colorado SB24-205 — consumer protections in interactions with AI systems](https://leg.colorado.gov/bills/sb24-205)
24. [Colorado SB25B-004 — effective date changes tied to SB24-205 implementation](https://leg.colorado.gov/bills/sb25b-004)
