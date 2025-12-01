# Editorial and Learner-Experience Review — Draft v1

Review of `../draft/v1.md` (17 sections), read section by section as a learner
encountering the material, against the four dimensions in issue #603:
terminology consistency, technical precision, logical progression, and
professional tone. Date: 2026-09-28.

The draft's status header (line 3) is pipeline metadata, not learner material;
it is not assessed. This review does not modify the draft. Per the issue,
completeness against the outline belongs to the alignment review, compression
to the compression review, and rewriting to the revision stage. Per the issue's
PoC simplification, technical claims are not verified against primary
references; claims that look uncertain are flagged for follow-up rather than
verified.

## Dimensions

| Dimension | What is checked |
| --- | --- |
| Terminology | The same concept is named the same way throughout the draft |
| Precision | No vague or hedged statements where a precise one is available; examples and lookup names stated so a learner can use them |
| Progression | Within a section, each paragraph follows from the previous one |
| Tone | Direct statements, no filler, no meta-commentary about the content |

## Findings

| ID | Draft section | Dimension | Finding | Suggested action |
| --- | --- | --- | --- | --- |
| E-01 | §2 | Terminology | "DMARC evaluates against the first organizational domain found" uses the term *organizational domain* before §10, the section the learner journey (entry 10) designates as its introduction and definition; a learner first meets the term here with nothing to attach it to. | Defer the term: say "the first address's domain" and add a forward pointer, or add a one-line parenthetical definition "(defined in section 10)". |
| E-02 | §15 | Terminology | "The receiving domain must publish a DNS authorization record" — "receiving domain" is ambiguous: the same section opens by calling mail receivers "receivers", and the domain that must publish is the one named in `rua=`, not the mail receiver. §17 repeats the same phrase. | Replace with "the domain named in `rua=`" (or "the report-receiving domain") in both §15 and §17. |
| E-03 | §14 | Terminology | Step 3 queries `_dmarc.<from-domain>`, a third naming form for the same concept: §11 gives the record location as `_dmarc.<domain>`, and everywhere else the identity is RFC5322.From. Whether the lookup domain is the From domain itself (RFC 7489's tree-walk) or the organizational domain is also left unstated. | Match §11's `_dmarc.<domain>` placeholder. Separately, confirm with the alignment review whether the tree-walk detail is deliberately omitted for depth; this review flags only the naming drift. |
| E-04 | §5 | Precision | "Example lookup: `TXT example.com`" (and §11's `TXT _dmarc.example.com`) is not a runnable command; a learner who copies it into a shell gets a command not found. The baseline material this pipeline replaces gave the working form (`dig TXT example.com`). | Either give the runnable form (`dig TXT example.com`, `dig TXT _dmarc.example.com`) or phrase it as a description, not a command: "the TXT record at example.com". |
| E-05 | §9 | Precision | The oversigning explanation — "an inserted header becomes part of a signed set that does not verify" — uses a nonce term ("signed set") and never says why verification fails. | State the mechanism: a header named in `h=` but absent at signing is not covered by the signature; if an intermediary inserts it, verification recomputes over a header set that now includes it, and the signature fails. |
| E-06 | §10 | Precision | "Strict alignment requires full control over exactly which domain appears in each identity field" is opaque: it does not say what strict alignment requires of the domains, and "full control over exactly which domain appears" buries the point. The preceding bullet already says it plainly ("exact match required"). | State the consequence directly: "strict alignment requires the authenticated domain to equal the From domain exactly, so senders must present the same domain in both identity fields". |
| E-07 | §17 | Precision | "Without `sp=`, subdomains of a `p=reject` domain default to `none` in some interpretations" is a hedged claim where a precise rule is available, and it contradicts the draft's own §11 ("`sp=` — subdomain policy; overrides `p=` for subdomains … Same values as `p=`", which implies `p=` applies absent `sp=`). The hedge "in some interpretations" reads as uncertainty about the material. | Verify against RFC 7489 §6.3 and state the rule definitively (absent `sp=`, the receiver applies `p=` to subdomains without their own record), or drop the clause if it cannot be sourced. |
| E-08 | §17 | Precision | "512-bit and 768-bit RSA keys are considered broken" is an unsourced assertion — considered broken by whom, and since when — next to a properly cited claim (RFC 8301's 1024-bit floor). | Attach the reference the claim comes from (e.g., NIST guidance on 1024-bit retirement, or the knowledge corpus), or state only the cited RFC 8301 requirement. |
| E-09 | §15 | Precision | "Major receivers rarely generate them due to privacy concerns" (§15) and "widely unsupported by receivers" (§11) are unsourced adoption claims about `ruf=`; neither cites a receiver's documented behaviour. | Verify against receiver documentation or the knowledge corpus and cite the source, or soften to a structural statement ("RFC 7489 notes privacy considerations for forensic reports"). Applies to both §11 and §15, in one edit with C-13. |
| E-10 | §12 | Professional tone | "The bounce-domain example is the payoff:" is meta-commentary announcing the example's value instead of letting it demonstrate, and "payoff" is a casual idiom out of register with the rest of the draft. | Cut the lead-in and state the fact: "Bounce addresses commonly sit on a subdomain of the sending domain, and relaxed mode — the default — accepts exactly that pattern." |
| E-11 | §16 | Professional tone | The section opener — `Each effect below is "which step of section 14 breaks when an intermediary touches the message."` — is a sentence fragment wrapping the organizing idea in quotation marks; as the frame for the whole section it should be a direct statement. | Convert to a complete sentence and drop the quotes: "Each effect below names which step of section 14 breaks when an intermediary touches the message." |
| E-12 | §7 | Logical progression | The "tags divide the work" paragraph re-explains `d=`, `h=`, and `s=` after the canonicalization discussion has moved the section on, so the closing thread jumps from canonicalization back to the tag lists. | Delete the paragraph (the same fix as C-07), or move it directly after the optional-tags list if the section cross-references are wanted; either way the canonicalization discussion should close the section. |
| E-13 | §15 | Logical progression | The cross-domain authorization block sits between the aggregate-report contents and the forensic-report paragraph, separating the two report types and leaving forensic reports as a dangling second kind after an `rua=`-specific detail. | Move the forensic (`ruf=`) paragraph before the cross-domain block, so the two report types are paired and the section closes with the `rua=`-specific authorization detail. Low severity. |
| E-14 | §5 | Precision | Two mechanism table rows phrase matching loosely: "The sending IP resolves from the A/AAAA records" (IPs do not resolve from records; the records contain the IP) and "The sending IP is an MX host" (the IP is not a host). | Rephrase precisely: "The sending IP is listed in the domain's A/AAAA records" and "The sending IP is among the A/AAAA addresses of the domain's MX hosts". |

## Section-by-Section Evaluation

| Draft section | Terminology | Precision | Progression | Tone |
| --- | --- | --- | --- | --- |
| §1 | Consistent | OK | Problem → attacks → layered answer; flows | Aphoristic closer (C-02) adds effect where a direct statement would do |
| §2 | E-01 | OK; identities precisely delimited with RFC citations | OK | Closer "The mechanisms below are the answers." is generated bridging (C-03) |
| §3 | Consistent | OK; "chose to say" is informal but unambiguous | OK | OK |
| §4 | Consistent; "decision layer" introduced once | OK | OK; promises §14's mechanics, delivered later | Triple restatement of "not an authentication mechanism" (C-06) |
| §5 | Consistent; HELO/EHLO matches §6/§14 | E-04, E-14 | Mechanisms → qualifiers → modifiers → example; each builds on the last | Dense reference style; no filler |
| §6 | Consistent; result vocabulary established once, as journey entry 6 intends | OK; both limits cited | Steps in RFC order | OK |
| §7 | Consistent | OK (see summary note on the `a=`/`k=` ed25519 seam) | E-12 | OK apart from the E-12 paragraph |
| §8 | Consistent | OK | Selector → lookup name → rotation → key tags | Repeated per-service sentence (C-08) |
| §9 | Consistent | E-05 | Steps → conflation caveat → oversigning; OK | OK; "Two outcomes that are easy to conflate" is teaching, not filler |
| §10 | OK; organizational domain and Public Suffix List defined once, per journey entry 10 | E-06 | Motivation → modes → implications; OK | Within-paragraph restatement (C-09) |
| §11 | Consistent | E-04, E-09 | `p=` → `sp=` → `pct=` → mode tags → reporting tags; OK | OK |
| §12 | OK; organizational domain restated with example (C-10) | OK; worked example is exact | OK | E-10 |
| §13 | Consistent; deliberately symmetric | OK; any-one-signature rule stated | OK | OK |
| §14 | E-03 | OK; steps reuse established vocabulary and results | OK; known pieces run in order | Closing line restates step 6 (C-14) |
| §15 | E-02 | E-09 | E-13 | OK |
| §16 | Consistent | OK; each effect names the broken step | SPF → DKIM → lists → ARC → SRS; OK | E-11 |
| §17 | E-02 (repeats "the receiving domain") | E-07, E-08 | OK; grouped by mechanism, each entry a diagnosis | OK; catalog format, functional opener |

## Generated-Passage Assessment

For each major section: whether any passage reads as generated rather than
authored (formulaic hedging, repeated sentence structures, meta-commentary
about the content).

| Draft section | Assessment |
| --- | --- |
| §1 | Present — "the protocol carries the claim, nothing more" (emphasis for effect) and the aphoristic closer (C-02) |
| §2 | Present — the closing "The mechanisms below are the answers." announces what follows (C-03) |
| §3 | None detected — short, direct prose; the C-04/C-05 redundancy is restatement, not formulaic filler |
| §4 | None beyond C-06's repetition; the one-clause opener ("Before any record syntax, the shape of the whole system:") is functional |
| §5 | None detected — reference-style tables, one fact per row |
| §6 | None detected |
| §7 | Present — "The tags divide the work:" restates the bullets above it (C-07, E-12) |
| §8 | None detected beyond the repeated selector sentence (C-08) |
| §9 | None detected |
| §10 | Present — the same point ("SPF/DKIM results say nothing about From") stated twice within one paragraph (C-09) |
| §11 | None detected |
| §12 | Present — "The bounce-domain example is the payoff:" is meta-commentary about the content (E-10) |
| §13 | None detected — the "Symmetric to section 12" opener is meta but load-bearing; the body is clean |
| §14 | Present — the closing line restates step 6 (C-14) |
| §15 | None detected |
| §16 | Borderline — the quoted fragment opener is self-announcing framing (E-11), though its content is the section's intended organizing principle (learner journey entry 16) |
| §17 | Borderline — the "diagnosis exercise" opener announces the section's pedagogy, as journey entry 17 intends; the catalog body is clean |

## Unaddressed Findings from Concurrent Reviews

**Alignment review.** At the time this review was conducted, no alignment
review exists under `review/` (the directory contains only
`compression-review.md`). There are no alignment findings to list; if the
alignment review produces IDs after this review, the revision stage must fold
them in with the lists below.

**Compression review.** `draft/v1.md` is unchanged since the compression
review was written (draft mtime 2026-09-28 17:01 vs compression review
2026-09-28 17:07; the compression review explicitly does not modify the
draft). All sixteen findings remain unaddressed: C-01, C-02, C-03, C-04, C-05,
C-06, C-07, C-08, C-09, C-10, C-11, C-12, C-13, C-14, C-15, C-16.

Coordination notes for the revision stage, where an editorial finding and a
compression finding touch the same passage and should be fixed in one edit:

| Editorial | Compression | Shared passage |
| --- | --- | --- |
| E-09 | C-13 | The `ruf=` adoption claim, stated in §11 and §15 |
| E-10 | C-11 | The bounce-address pattern, §10 and §12 |
| E-12 | C-07 | The "tags divide the work" paragraph, §7 |

## Summary

The draft's dominant editorial trait is strong: dense, direct, precisely cited
prose with consistent identity vocabulary (RFC5321.MailFrom, RFC5322.From,
`d=`) throughout. Most prose issues are the compression review's restatements;
this review's distinct findings concentrate in §15 and §17 (E-02, E-07, E-08,
E-09, E-13 — ambiguous or hedged reporting statements), in two
meta-commentary passages (E-10, E-11), and in a handful of precision fixes
(E-04, E-05, E-06, E-14). E-07 is the only finding that surfaces a substantive
technical question (the `sp=` default), and it is also the draft's only
internal contradiction.

Two notes for the alignment review, which this review does not act on:

- §7's `a=` lists only `rsa-sha256` and `rsa-sha1` while §8's `k=` lists
  `ed25519` (RFC 8463). The draft matches the outline here, so this is not an
  outline-completeness gap, but a learner crossing from §7 to §8 meets an
  unexplained asymmetry.
- §14 step 3's `_dmarc.<from-domain>` naming (E-03) and whether the
  RFC 7489 tree-walk to the organizational domain belongs in the learner
  material are depth decisions for the alignment review.
