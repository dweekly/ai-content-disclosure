# AI Content Disclosure for HTML

## Authors

- [David E. Weekly](https://github.com/dweekly)

## Participate

- [GitHub Issues](https://github.com/dweekly/ai-content-disclosure/issues)
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
- [Stakeholder Feedback](#stakeholder-feedback)
- [References](#references)

---

## Introduction

Web pages increasingly contain text produced with varying degrees of AI
involvement — from light AI-assisted editing to fully autonomous generation.
There is currently no standard HTML mechanism for authors to disclose AI
involvement at element-level granularity within a page.

This explainer proposes an `aidisclosure` HTML attribute and a companion
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

An `aidisclosure` global attribute on any HTML element:

```html
<article>
  <section aidisclosure="none">
    <h2>Six-Month Investigation: City Budget Shortfall</h2>
    <p>Our reporters spent six months reviewing financial records...</p>
  </section>

  <aside aidisclosure="ai-generated" aimodel="gpt-4o" aiprovider="OpenAI">
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
| `aimodel` | Model identifier | `"claude-3.5-sonnet"` |
| `aiprovider` | Provider name | `"Anthropic"` |
| `aiprompturl` | URL to prompt/methodology documentation | `"/ai-methodology#summary"` |

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
<article aidisclosure="none">
  <h1>Exclusive: City Budget Shortfall</h1>
  <p>After six months of records review...</p>
</article>

<aside aidisclosure="ai-generated" aimodel="gpt-4o" aiprovider="OpenAI">
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
<article aidisclosure="ai-assisted" aimodel="claude-3.5-sonnet"
         aiprovider="Anthropic">
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
<div aidisclosure="autonomous" aimodel="weather-llm-v2"
     aiprovider="WeatherCorp">
  <h2>San Francisco Bay Area Forecast</h2>
  <p>Expect fog clearing by noon with highs near 62°F...</p>
</div>
```

### Scenario 4: Human-Only Publication Asserting Provenance

A literary journal positively asserts that no AI was used:

```html
<meta name="ai-disclosure" content="none">
<!-- ... -->
<article aidisclosure="none">
  <h1>The Weight of Feathers</h1>
  <p>She found the letter tucked inside a volume of Neruda...</p>
</article>
```

Note: `aidisclosure="none"` is a positive assertion. The *absence* of the
attribute means "unknown," not "none."

## Detailed Design

### Inheritance

- Element-level `aidisclosure` attributes override the page-level
  `<meta name="ai-disclosure">` tag
- Children inherit their nearest ancestor's `aidisclosure` value unless
  they specify their own
- Absence of the attribute means **unknown** (not "none") — this is an
  important distinction that avoids false negative assertions

### Relationship to HTTP AI-Disclosure Header

| Layer | Mechanism | Granularity |
|-------|-----------|-------------|
| HTTP | `AI-Disclosure` response header | Entire response |
| HTML page | `<meta name="ai-disclosure">` | Entire document |
| HTML element | `aidisclosure` attribute | Any element |

These are complementary. A CDN or reverse proxy can set the HTTP header;
a CMS can set the meta tag; an author or AI tool can set element-level
attributes. None supersedes the others.

### Vocabulary Alignment with IPTC

| `aidisclosure` value | IPTC Digital Source Type URI |
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
integration (e.g., address bar indicators, accessibility announcements) and
is consistent with how other proposals (like `containertiming`) have
proceeded.

### 5. RDFa / Microdata Only

Too verbose for common cases. RDFa requires namespace declarations and
multi-attribute markup for simple assertions. The `aidisclosure` attribute
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
- **Prompt text is not embedded in HTML.** The optional `aiprompturl`
  attribute links to an external resource, giving authors control over what
  they disclose and when they revoke access.
- **The voluntary nature means it cannot be relied upon for security
  decisions**, the same as any self-declared metadata (robots.txt,
  `rel=nofollow`, Schema.org markup).

## Accessibility Considerations

- Browsers and screen readers could optionally announce AI disclosure to
  users (e.g., "AI-generated content follows")
- The `aidisclosure` attribute should be exposed via the
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
- The `aiprompturl` attribute can link to localized methodology pages

## FAQ / Common Objections

### "This is the evil bit — bad actors won't comply."

This standard serves responsible publishers, regulated industries, and AI
tool vendors who want to be transparent. It is not a detection mechanism.
The EU AI Act makes compliance mandatory for covered entities, and major
platforms already require disclosure in their terms of service.

The analogy is `rel=nofollow`: voluntary, widely adopted because it aligns
incentives, and useful despite being ignorable by bad actors.

### "Where do you draw the line with grammar checkers?"

See [What Counts as AI?](#what-counts-as-ai) above. The boundary is
generative/inferential AI — systems trained on data that produce novel
outputs. Deterministic spell-check and thesaurus tools are excluded.

### "Metadata will be gamed like SEO dates."

True for any self-declared metadata. The standard enables honest disclosure;
verification requires pairing with C2PA or regulatory auditing. The value is
in the signal for those who choose to use it honestly — same as Schema.org
structured data, which search engines use despite its spoofability.

### "This will stigmatize AI-assisted content."

The granular levels (four values, not binary) allow publishers to distinguish
"AI helped me edit" from "AI wrote everything." The `ai-assisted` level
should carry no more stigma than acknowledging the use of a human copy
editor.

### "Everything will be AI-touched soon, making this meaningless."

That is exactly why granularity matters. Binary "AI/not-AI" is already
inadequate. The spectrum from `none` to `autonomous` reflects the reality of
modern content workflows and remains meaningful as AI tools become ubiquitous.

## Stakeholder Feedback

| Engine | Status | Link |
|--------|--------|------|
| Chromium | Pending | — |
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
