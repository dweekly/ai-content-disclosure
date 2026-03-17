# Roadmap

Stack-ranked priorities for the AI Content Disclosure proposal.

---

- ~~**File WICG Proposals issue**~~ — Done:
  [WICG/proposals#261](https://github.com/WICG/proposals/issues/261)

- ~~**Comment on WHATWG HTML #9479**~~ — Done: commented linking to the
  explainer as the element-level solution requested by thread participants.

- ~~**Request browser standards positions**~~ — Done:
  - [ChromeStatus](https://chromestatus.com/feature/5078123181899776)
  - [mozilla/standards-positions#1344](https://github.com/mozilla/standards-positions/issues/1344)
  - [WebKit/standards-positions#605](https://github.com/WebKit/standards-positions/issues/605)

- ~~**Engage W3C AI & the Web Interest Group**~~ — Done: posted to
  `public-webai@w3.org` mailing list introducing the proposal.

- ~~**Create W3C Community Group**~~ — Done:
  [W3C AI Content Disclosure Community Group](https://www.w3.org/community/ai-content-disclosure/)
  established. This is now the primary venue for discussion and participation.

- ~~**File Schema.org proposal**~~ — Done: proposed `aiDisclosure` property
  on `CreativeWork` as a comment on
  [schemaorg/schemaorg#3391](https://github.com/schemaorg/schemaorg/issues/3391).

- ~~**Connect with Doğu Abaris**~~ — Done: Reached out to the author of the
  expired [IETF AI-Disclosure header draft](https://datatracker.ietf.org/doc/draft-abaris-aicdh/).
  He is supportive of this proposal but cannot commit to sustained collaboration
  right now. Open to light-touch help (reviewing, sanity-checking terminology).
  The HTML proposal can proceed independently of reviving his IETF draft.

- **Connect with IPTC** — Verify vocabulary alignment with IPTC Digital
  Source Type maintainers. Ensure the mapping table is accurate and
  future-compatible. Discuss whether they'd endorse an HTML binding and
  understand their roadmap for text content (vs. images).

- **Monitor Schema.org** — Watch [schemaorg/schemaorg#3391](https://github.com/schemaorg/schemaorg/issues/3391)
  and [#3392](https://github.com/schemaorg/schemaorg/issues/3392) for direction
  on `digitalSource` property adoption. Schema.org is aligning with IPTC
  terminology.

- **Connect with C2PA working group** — Discuss how HTML-level disclosure
  could be cryptographically bound to C2PA manifests for verified provenance.

- **Request W3C TAG review** — Once WICG incubation is underway, request a
  [TAG review](https://github.com/w3ctag/design-reviews/issues) for
  architectural feedback.

- **Build a polyfill / demo** — Create a small JavaScript library and browser
  extension that surfaces `ai-disclosure` attributes visually, demonstrating
  the user experience without browser-native support.

- **Engage regulatory stakeholders** — Position this standard as a technical
  mechanism for EU AI Act Article 50 compliance. Connect with the EU Code of
  Practice on AI-Generated Content working group.

- **Explore compact attribution notation** — He, Houde & Weisz (CHI '25)
  propose an "AIA" compact attribution statement format (modeled on Creative
  Commons license chooser) that encodes model, contribution types, proportion,
  initiative, and human review status. Evaluate whether a similar notation
  could complement `ai-disclosure` attributes for richer attribution — e.g.,
  as a structured value for `ai-prompt-url` or a future `ai-attribution`
  attribute. See their [AI Attribution toolkit](https://aiattribution.github.io)
  and [w3c-cg/ai-content-disclosure#11](https://github.com/w3c-cg/ai-content-disclosure/issues/11).

- **Collaborate with AI Attribution researchers** — Justin Weisz (formerly
  IBM Research), Jessica He (IBM Research), Hyo Jin Do (Gina), and Min Kyung
  Lee (UT Austin) have expressed interest in contributing expertise to the
  W3C specification. Their empirical work on attribution perceptions (CHI '25)
  and disclosure barriers/stigma (FAccT '26) directly validates the four-level
  taxonomy and granular disclosure approach. Engage them as CG participants
  or invited experts.
