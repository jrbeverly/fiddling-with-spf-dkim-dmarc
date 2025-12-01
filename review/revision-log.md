# Revision Log: Draft v1 → v2

Targeted revision of `../draft/v1.md` into `../draft/v2.md`, applying the
findings of the three review passes — `alignment-review.md` (F), 
`compression-review.md` (C), and `editorial-review.md` (E). Each change is a
discrete edit corresponding to a specific finding; unrelated sections are
untouched, and the document was not regenerated. `../outline/information-outline.md`
is unchanged by this stage.

Dispositions:

- **`addressed`** — the suggested change was applied.
- **`deferred`** — not applied; the finding conflicts with outline-required
  content, per the issue's rule that conflicting findings are recorded with
  the reason rather than arbitrarily resolved.
- **`invalid`** — rejected; the finding does not hold.

Where a compression and an editorial finding touch the same passage, the
coordination notes in `editorial-review.md` were followed: one edit resolves
both, and both IDs are logged against it.

## Disposition of Findings

### Alignment review

| Finding | Disposition | Affected section(s) | Change applied or reason |
| --- | --- | --- | --- |
| [F1] | addressed | §8 DKIM: Selectors | Added the missing section citation `[RFC 6376 §3.1, §3.5]` to the opening sentence (outline item O8.1). |
| [F2] | addressed | §10 DMARC: Identifier Alignment | Added the missing section citation `[RFC 7489 §3.1]` to the opening sentence (outline item O12.1). |
| [F3] | addressed | §14 Combined Evaluation | Removed the closing sentence "One aligned passing result is sufficient; SPF and DKIM need not both pass." Step 6 already states the disposition rule the outline kept, and outline item O17.5 mandates the sentence's absence. Same edit as F4 and C-14. |
| [F4] | addressed | §14 Combined Evaluation | Same edit as F3; the one sentence restated both excluded items O17.5 and O17.6. |

### Compression review

| Finding | Disposition | Affected section(s) | Change applied or reason |
| --- | --- | --- | --- |
| [C-01] | addressed | §1 | Cut the trailing clause "the protocol carries the claim, nothing more"; the sentence now ends at "without proving control of that domain". |
| [C-02] | addressed | §1 | Cut the aphoristic closer "Each one closes a named part of the problem; none closes it alone."; the paragraph now ends at "act on verification failures". |
| [C-03] | addressed | §2 | Deleted the closing two sentences ("Because all three can differ on a single message, …" / "The mechanisms below are the answers."); the section ends on the `d=` bullet. |
| [C-04] | addressed | §3 | Deleted "A lookup therefore returns what the domain owner chose to say." |
| [C-05] | addressed | §3 | The final sentence now ends at "to retrieve current policy". |
| [C-06] | addressed | §4 | Collapsed the triple restatement into one statement: "DMARC is the decision layer, not an authentication mechanism itself: it decides what the SPF/DKIM results mean for the domain the user sees, and what the receiver should do when they fall short." "not an authentication mechanism itself" is kept (once) because learner-journey entry 4's expected understanding calls for it; "it does not authenticate anything new" is dropped. |
| [C-07] | addressed | §7 | Deleted the "The tags divide the work: …" paragraph. Same edit as E-12. |
| [C-08] | addressed | §8 | Merged the two selector sentences into one covering rotation, per-service selectors, and owner-published keys. |
| [C-09] | addressed | §10 | Cut "Neither claims anything about the RFC5322.From domain the user sees."; kept the concrete illustration ("an attacker can pass SPF for a domain they control while spoofing From"). |
| [C-10] | addressed | §10, §12 | The organizational-domain definition now appears once, with the `sub.example.com` → `example.com` example, in §10 (the journey-designated introduction point); §12 references it ("the organizational domains (section 10)"). |
| [C-11] | addressed | §10, §12 | Cut the bounce-address clause from §10's Implications sentence; the bounce pattern remains at §12, the home learner-journey entry 12 assigns it. Same edit as E-10. |
| [C-12] | addressed | §12 | The section now opens with the definitional half ("DMARC defines SPF alignment as the RFC5321.MailFrom domain compared against RFC5322.From."). The cut clause restates §10's central point; the framing line and citation are retained. |
| [C-13] | addressed | §11, §15 | §11's `ruf=` bullet is now a forward reference ("see section 15"); §15 keeps the fuller statement. Applied in coordination with E-09 (see below). |
| [C-14] | addressed | §14 | Deleted the closing line "One aligned passing result is sufficient; SPF and DKIM need not both pass." Same edit as F3/F4. |
| [C-15] | deferred | §16 | Not applied. Cutting the two mailing-list bullets would drop outline items O15.6 ("Subject modification breaks DKIM if `subject` appears in `h=`") and O15.7 ("Body modification breaks DKIM regardless of canonicalization"), which the alignment review records as required content (present — D16, no finding). The repetition the finding flags is the outline's own placement, not draft-level duplication. |
| [C-16] | deferred | §17 | Not applied. The three §17 entries carry outline items O16.9, O16.14, and O16.15, including clauses §16 does not contain (e.g., "aligns to the forwarder's domain"); shortening them to a §16 reference would drop required outline content. The alignment review records all three present with no finding, and its "duplicate material" check raised no item here. |

### Editorial review

| Finding | Disposition | Affected section(s) | Change applied or reason |
| --- | --- | --- | --- |
| [E-01] | addressed | §2 | First use of "organizational domain" now carries the parenthetical "(defined in section 10)", pointing at the section the learner journey designates as its introduction. |
| [E-02] | addressed | §15, §17 | "the receiving domain" replaced with "the domain named in `rua=`" in both sections. |
| [E-03] | addressed | §14 | Step 3 now uses `_dmarc.<domain>`, matching §11's placeholder, qualified by "for the From domain" so v1's association is kept. The tree-walk depth question is left as-is: the outline (O14.4) omits it and the alignment review made no finding, so the omission is deliberate. |
| [E-04] | addressed | §5, §11 | Example lookups now use the runnable form: `dig TXT example.com` and `dig TXT _dmarc.example.com`. (Verified during revision: `dig TXT example.com` returns the domain's SPF record. Note: the finding attributes this form to the baseline, but `baseline/one-shot.md` uses zone-file style, not `dig`; the dig form was verified directly instead.) |
| [E-05] | addressed | §9 | Oversigning now states why verification fails ("the signature does not cover it, so … verification recomputes over a header set that now includes it and fails"); the nonce term "signed set" is removed. |
| [E-06] | addressed | §10 | The Implications sentence now states the consequence directly: "strict alignment requires the authenticated domain to equal the From domain exactly, so senders must present the same domain in both identity fields." |
| [E-07] | addressed | §17 | Verified against RFC 7489 §6.3 ("If absent, the policy specified by the 'p' tag MUST be applied for subdomains."). The hedged clause "default to `none` in some interpretations" is replaced with the definitive rule; this also resolves v1's internal contradiction with §11's `sp=` bullet. The outline's O16.18 wording carries the same hedge, but the outline is a frozen reference and is not edited in this stage; the correction is recorded here. |
| [E-08] | addressed | §17 | Verified RFC 8301: it sets a 1024-bit signing floor and requires verifiers to "MUST NOT consider signatures using RSA keys of less than 1024 bits as valid", but does not itself call 512- or 768-bit keys "broken" (no in-repo source does). The entry now states only the RFC 8301 requirement, keeping the 512/768 mention as the concrete instance of the floor. |
| [E-09] | deferred | §11, §15 | Not applied beyond the deduplication in C-13. The adoption claim ("rarely generated by major receivers due to privacy concerns") cannot be cited — no source exists in the repository, and the knowledge corpus carries it unsourced — and cannot be removed without dropping outline items O11.12 and O13.7, which the alignment review records as present. Left for the verification stage with the claim stated once, in §15. |
| [E-10] | addressed | §12 | Cut the "The bounce-domain example is the payoff:" lead-in; the fact is stated directly: "Bounce addresses commonly sit on a subdomain of the sending domain, and relaxed mode — the default — accepts exactly that pattern." Same edit as C-11. |
| [E-11] | addressed | §16 | The opener is now a complete sentence without quotes: "Each effect below names which step of section 14 breaks when an intermediary touches the message." |
| [E-12] | addressed | §7 | Deleted the "The tags divide the work: …" paragraph; the canonicalization discussion now closes the section prose (before the example). Same edit as C-07. |
| [E-13] | addressed | §15 | The forensic (`ruf=`) paragraph moved before the cross-domain authorization block, so the two report types are paired and the section closes with the `rua=`-specific detail. |
| [E-14] | addressed | §5 | Both mechanism rows rephrased precisely: "The sending IP is listed in the domain's A/AAAA records" and "The sending IP is among the A/AAAA addresses of the domain's MX hosts". (Diverges from outline O4.8/O4.9 phrasing; recorded here as a deliberate precision correction.) |

## Notes Carried Over Without Finding IDs

Items raised outside the finding tables, with the disposition chosen here:

- **Editorial review, notes to the alignment review** (the §7 `a=`/§8 `k=` ed25519 asymmetry; whether the RFC 7489 tree-walk belongs at §14 step 3): the alignment review raised no finding on either, and the draft matches the outline in both places. No change.
- **Alignment review, presentation notes** (O4.3 merged into §5's Location bullet; O7.12 split into two bullets; extra journey-called-for sentences in D1/D2/D4/D7/D12): explicitly "not findings". No change.
- **Compression review, coordination note** (C-10/C-11/C-16 deduplicated once, not twice): followed — the alignment review produced no overlapping findings, so each finding was applied or deferred exactly once above.

## Summary

- 4 alignment findings: 4 addressed.
- 16 compression findings: 14 addressed, 2 deferred (C-15, C-16).
- 14 editorial findings: 13 addressed, 1 deferred (E-09).
- Total: 34 findings, 31 addressed, 3 deferred, 0 invalid.

The deferrals all share one reason: the suggested cut would drop outline items the
alignment review records as required content. Per the issue, they are logged
here with the reason rather than resolved arbitrarily.
