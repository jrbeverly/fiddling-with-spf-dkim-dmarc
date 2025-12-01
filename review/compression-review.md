# Compression and Information-Density Review — Draft v1

Review of `../draft/v1.md` (17 sections) against the pattern list in
`../../VISION.md` § "Compression and Information-Density Review", per issue
#602. Date: 2026-09-28.

The draft's status header (line 3) is pipeline metadata, not learner material;
it is not assessed. This review does not modify the draft and does not estimate
word-count savings. Per the issue, completeness and technical accuracy belong
to the alignment and editorial reviews.

## Pattern Types

Labels used below, mapped to the issue's pattern list:

| Label | Pattern |
| --- | --- |
| `repeated-explanation` | The same concept explained more than once within a section or across sections |
| `formulaic-introduction` | Generic or formulaic introduction |
| `formulaic-conclusion` | Generic or formulaic conclusion |
| `empty-transition` | Transition sentence that conveys no information |
| `over-explanation` | Explanation substantially longer than the concept requires |
| `artificial-emphasis` | Artificial emphasis on an otherwise straightforward statement |

## Findings

| ID | Draft section | Pattern | Suggested action |
| --- | --- | --- | --- |
| C-01 | §1 | `artificial-emphasis` | "the protocol carries the claim, nothing more" restates the preceding clause ("without proving control of that domain") for effect. Cut the trailing clause, or keep it only if the generalization to all protocol-carried claims is wanted. Borderline. |
| C-02 | §1 | `formulaic-conclusion` | "Each one closes a named part of the problem; none closes it alone." is an aphoristic closer whose second half restates the first. Keep the layering fact in one clause, e.g. end the paragraph at "act on verification failures", or keep only "none closes it alone" if the takeaway is needed. |
| C-03 | §2 | `empty-transition` | The closing pair — "Because all three can differ on a single message, ... has to be asked separately for each. The mechanisms below are the answers." — restates the section opener ("they can all differ", "Every mechanism in this document operates on one of them") and the last sentence conveys nothing. Delete the closing two sentences. |
| C-04 | §3 | `repeated-explanation` | "A lookup therefore returns what the domain owner chose to say." restates the owner-control point of the previous sentence. Delete the sentence or merge it into the previous one. |
| C-05 | §3 | `repeated-explanation` | "so the records are re-read for every message" restates "performs DNS lookups at delivery time". End the sentence at "to retrieve current policy". Minor. |
| C-06 | §4 | `repeated-explanation` | "not an authentication mechanism itself", "it does not authenticate anything new", and "it decides what the SPF/DKIM results mean" say the same thing three times. Keep one: "DMARC is the decision layer: it decides what the SPF/DKIM results mean for the domain the user sees, and what the receiver should do when they fall short." |
| C-07 | §7 | `repeated-explanation` | "The tags divide the work: `d=` names the signing identity (section 2), `h=` fixes exactly which headers the signature covers, and `s=` names the public key to fetch (section 8)." restates the tag bullets immediately above. Delete the paragraph, or reduce it to the section cross-references if those links matter more than the restatement. |
| C-08 | §8 | `repeated-explanation` | "A signing service typically has its own selector; the domain owner publishes the corresponding public key under it." repeats "each signing service can have its own selector" from the preceding sentence. Merge into one sentence covering rotation, per-service selectors, and owner-published keys. |
| C-09 | §10 | `repeated-explanation` | "Neither claims anything about the RFC5322.From domain the user sees" and "Authentication without alignment does not protect the visible From domain" state the same point within one paragraph. Cut one; keep the concrete illustration "an attacker can pass SPF for a domain they control while spoofing From". |
| C-10 | §10, §12 | `repeated-explanation` | The organizational-domain definition appears in §10 ("the registered domain under a public suffix, determined using the Public Suffix List") and again in §12 ("the registered domain under a public suffix (`sub.example.com` → `example.com`)"). The learner journey (entry 10) specifies this concept is introduced once. Define once — with the example — at §10, and have §12 reference it. |
| C-11 | §10, §12 | `repeated-explanation` | The bounce-address implication appears in §10 ("relaxed SPF alignment accommodates common patterns such as bounce addresses at a subdomain") and again as §12's payoff paragraph ("The bounce-domain example is the payoff..."). The learner journey (entry 12) makes §12 the example's home. Cut the bounce clause from §10's Implications sentence; keep the strict-mode implication there. |
| C-12 | §12 | `repeated-explanation` | "SPF alone does not prove alignment with the RFC5322.From domain" restates §10's central point. Start the section at the definitional half: "DMARC defines SPF alignment as the RFC5321.MailFrom domain compared against RFC5322.From." |
| C-13 | §11, §15 | `repeated-explanation` | `ruf=` adoption stated twice: §11 "widely unsupported by receivers" and §15 "Major receivers rarely generate them due to privacy concerns". Keep the fuller §15 statement; shorten §11's bullet to a forward reference. |
| C-14 | §14 | `repeated-explanation` | The closing line "One aligned passing result is sufficient; SPF and DKIM need not both pass." duplicates step 6 ("either path passes with alignment → the message passes DMARC"). Delete the closing line; step 6 already states the rule. |
| C-15 | §16 | `repeated-explanation` | The mailing-list bullets ("Subject modification breaks DKIM if `subject` appears in `h=`", "Body modification breaks DKIM regardless of canonicalization") repeat the "DKIM fails if:" list two paragraphs earlier in the same section. Keep the list-specific facts (`[List]` prefixes, footers, `From:` rewriting) and drop the duplicated failure statements. |
| C-16 | §16, §17 | `repeated-explanation` | §17's "SPF alignment fails on forwarding" and "DKIM breaks in mailing lists" restate §16's forwarding and mailing-list mechanics, and "Signed headers modified" repeats §16's subject-modification bullet. The learner journey (entry 17) assumes entry 16. Shorten these entries to the diagnosis framing (which step broke, which record choice caused it) and reference §16 for the mechanism. |

## Section Coverage

| Draft section | Patterns found |
| --- | --- |
| §1 | C-01, C-02 |
| §2 | C-03 |
| §3 | C-04, C-05 |
| §4 | C-06 |
| §5 | none — dense tables, one fact per row |
| §6 | none — the numbered algorithm is the section's purpose; restating §5's "the first match wins" *is* the algorithm, and the result vocabulary is deliberately established here (learner journey entry 6) |
| §7 | C-07 |
| §8 | C-08 |
| §9 | none — each verification step is a distinct fact; the pass/fail vs. missing distinction is new |
| §10 | C-09, C-10, C-11 |
| §11 | C-13 |
| §12 | C-10, C-11, C-12 |
| §13 | none — deliberately symmetric to §12 and compact; correctly avoids re-defining the organizational domain |
| §14 | C-14 |
| §15 | C-13 |
| §16 | C-15, C-16 |
| §17 | C-16 |

No findings for the `formulaic-introduction` and `over-explanation` patterns in
any section: no section opens with generic framing, and no paragraph is
substantially longer than its concept requires — the length problems found are
all restatement, not over-elaboration.

## Intentionally Long

Sections that are lengthy but should remain as-is:

| Section | Why it should remain |
| --- | --- |
| §5 SPF: Record Syntax | One distinct fact per table row; the length is content (a syntax reference), not prose. |
| §11 DMARC: Policy Options | Each tag controls distinct behavior a learner must be able to read from a record. |
| §14 Combined Evaluation | The seven-step walkthrough is the section's purpose; each step names a distinct evaluation stage, and the section is self-contained by design (learner journey entry 14). |
| §16 Forwarding | Each mechanism fails differently under forwarding, and ARC and SRS each introduce a new mechanism. |
| §17 Common Failure Modes | Catalog format; each entry is a distinct diagnosis — but entries restating §16 mechanics should be trimmed per C-16, not the catalog itself. |

## Summary

The draft's dominant compression problem is restatement: 16 findings, 13 of
which are `repeated-explanation` — the same fact stated twice within a sentence
(C-01, C-02, C-06), within a section (C-04, C-05, C-07, C-08, C-09, C-12, C-15),
or across sections (C-10, C-11, C-13, C-14, C-16). Cross-section repeats in
C-10, C-11, and C-16 work against the learner journey's explicit
introduce-once sequencing (entries 10, 12, 17). The draft is otherwise clean:
no formulaic introductions, no one-fact paragraphs, and only one redundant
example (the bounce-address pair, C-11).

Coordination note for the revision stage: C-10, C-11, and C-16 are
cross-section duplications that the alignment review's "duplicate material"
checklist may also flag; deduplicate once rather than applying both reviews'
edits independently.
