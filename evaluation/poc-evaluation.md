# PoC Evaluation: Staged Pipeline vs. One-Shot Baseline

Evaluation of `../draft/v2.md` against `../baseline/one-shot.md`, per issue
#598. Date: 2026-09-29. Method: a single structured evaluator pass (the
issue's PoC simplification), applying the review criteria already used in this
run and checking sampled technical claims against the primary references
(RFC 7208, RFC 6376, RFC 7489, RFC 8301, fetched from rfc-editor.org). This
evaluation does not modify either document.

Scope caveats, stated up front because they bound every finding below:

- `draft/v2.md` is the latest pipeline output but is **not final material**:
  stage 7 (`review/verification-review.md`) and stage 8 (`final/material.md`)
  have not run; v2's own header says "Not yet verified". The README success
  test nominally targets `final/material.md`; section 6 applies it to v2 as
  the latest output and says so.
- Technical-accuracy checks are samples (3+ claims per mechanism per
  document), not an exhaustive audit, per the issue.
- Writing-pattern counts are one evaluator's structured pass. The pattern
  labels and the pass's standard are those of `review/compression-review.md`.

## 1. Documents Compared

| Document | Pipeline stage | Words (`wc -w`) | Lines |
| --- | --- | --- | --- |
| `baseline/one-shot.md` | 0 · one-shot baseline | 3518 | 532 |
| `draft/v1.md` | 4 · initial writing | 3352 | 339 |
| `draft/v2.md` | 6 · targeted revision | 3245 | 333 |

## 2. Dimension 1 — Information Completeness

### 2.1 Topic areas present in each document

**Baseline (`baseline/one-shot.md`)** — 7 sections:

1. The problem email authentication solves (spoofing, phishing)
2. SPF: purpose, TXT record construction, mechanisms, worked example, receiver evaluation, shortfalls
3. DKIM: purpose, keys and selectors, signing, verification, shortfalls
4. DMARC: purpose, identifier alignment, policy record, worked example, receiver application, reporting
5. Real delivery path walkthrough (legitimate message and spoofing attempt)
6. Deployment pitfalls for SPF, DKIM, and DMARC
7. "Putting it all together" deployment advice

**v2 (`draft/v2.md`)** — 17 sections:

1. The problem email authentication addresses
2. SMTP identities (RFC5321.MailFrom, RFC5322.From, DKIM `d=`)
3. DNS as part of the authentication model
4. How the mechanisms layer together (overview)
5. SPF: record syntax
6. SPF: evaluation algorithm
7. DKIM: signature structure
8. DKIM: selectors
9. DKIM: key lookup and verification
10. DMARC: identifier alignment (strict vs. relaxed)
11. DMARC: policy options
12. SPF alignment
13. DKIM alignment
14. How SPF, DKIM, and DMARC are evaluated together
15. DMARC: aggregate reporting
16. Email forwarding and intermediary effects (incl. ARC, SRS)
17. Common failure modes

### 2.2 Topic areas present in one document but absent in the other

**In v2 but absent from the baseline** (with the outline item IDs the areas
trace to, per `outline/information-outline.md`):

- SMTP identities as a first-class topic: the three identities named
  (RFC5321.MailFrom / RFC5322.From / `d=`) and their distinct roles (O2);
  the baseline mentions envelope sender vs. `From` in passing but never
  builds the identity model.
- DNS as a trusted distribution channel and the DNSSEC non-requirement
  (O3); the baseline assumes DNS knowledge.
- The layering overview: SPF = path check, DKIM = signature, DMARC =
  decision layer (O14 at overview depth).
- SPF record syntax: `ptr` and `exists` mechanisms (O4.12, O4.13),
  `redirect=` and `exp=` modifiers (O4.18, O4.19), the implicit-neutral
  default (O5.5), and the void-lookup limit (O5.7).
- DKIM: canonicalization modes `simple`/`relaxed` and why transport
  changes break verification (O7.14, O7.15); `i=`, `t=`, `x=`, `l=` tags
  (O7.11–O7.13); key-record flags `t=y`, `t=s`, and `p=` revocation
  semantics (O8.8–O8.10); oversigning (O9.10); the
  multiple-signatures rule (O10.5).
- DMARC: the `fo=` tag (O11.13); the cross-domain `rua=` authorization
  record (O13.6); per-source-IP aggregate-report contents (O13.5).
- Forwarding/intermediary effects: ARC and SRS (O15.9–O15.12).
- A diagnosis-framed failure-modes catalog (O16): void-limit and
  split-brain SPF failures, unsigned-`From`, key-revocation gap,
  organizational-domain determination error, `pct=`-below-100, subdomain
  policy, cross-domain `rua=` without authorization.

**In the baseline but absent from v2**:

- CNAME-based DKIM key delegation ("Some providers use a CNAME instead of
  a TXT record", baseline §3). No outline item; the RFC basis is checked
  in section 4.3.
- A worked legitimate-delivery walkthrough with named actors and concrete
  records (baseline §5) and an explicit step-by-step spoofing attempt with
  an attacker IP. v2 §14 gives the abstract algorithm; it has no narrative
  worked message.
- Pitfall/advice items: "Using `+all` by accident", and "Aggregate reports
  can be large and complex — use a reporting tool". (The baseline's
  graduated-rollout advice has a reduced counterpart in v2 §11: `p=none`
  "used during initial deployment" and `pct=` "used for gradual rollout".)

**Coverage measured against the outline.** The alignment review
(`review/alignment-review.md`, Verdict) checked v1 against all 156 outline
items: 154 present, 2 missing (section citations O8.1, O12.1), 0
inaccurate. The revision log records F1 and F2 as addressed, restoring both
citations, and F3/F4 as addressed, removing the two excluded items
(O17.5/O17.6) that v1 had re-introduced. v2 is therefore at 156/156 outline
items present with the exclusions honored. The baseline was not produced
against the outline; measured against it directly, the baseline lacks the
outline items listed above (at minimum O3, O4.12–O4.13, O4.18–O4.19, O5.5,
O5.7, O7.10–O7.15, O8.8–O8.10, O9.10, O10.5, O11.13, O13.6, O15.9–O15.12,
and O16 items O16.3, O16.5, O16.6, O16.12, O16.20, O16.21).

## 3. Dimension 2 — AI Writing-Pattern Prevalence

Criteria: the six pattern labels of `review/compression-review.md`
(`repeated-explanation`, `formulaic-introduction`, `formulaic-conclusion`,
`empty-transition`, `over-explanation`, `artificial-emphasis`), applied
with the same standard — a passage counts when it restates a fact already
stated nearby or conveys framing rather than information.

### 3.1 Findings present in v2

v1 carried 16 compression findings (compression review, Summary). The
revision log records 14 addressed and 2 deferred (C-15, C-16); both
deferrals remain present in v2:

| ID | v2 section | Pattern | Why it remains |
| --- | --- | --- | --- |
| [C-15] | §16 | `repeated-explanation` | Cutting the mailing-list bullets would drop outline items O15.6 and O15.7, which the alignment review records as required content (revision log) |
| [C-16] | §17 | `repeated-explanation` | The three §17 entries carry outline items O16.9, O16.14, O16.15, including clauses §16 does not contain (revision log) |

Spot checks confirm the 14 addressed findings are gone from v2 (e.g., §1
now ends at "act on verification failures" per C-02; §14 no longer carries
the closing restatement per C-14/F3/F4). No new compression-category
finding was introduced by the revision. **v2 residual count: 2.**

### 3.2 Findings in the baseline under the same criteria

| ID | Baseline section | Pattern | Finding |
| --- | --- | --- | --- |
| BC-01 | Intro | `formulaic-introduction` | "Email authentication answers a simple but critical question…" + "The core email protocols were designed decades ago when the internet was much more trusting." — generic importance-and-history framing before any content |
| BC-02 | §1 | `repeated-explanation` | The "In short:" list restates the three numbered questions immediately above it, which already assign SPF/DKIM/DMARC to the three checks |
| BC-03 | §2 | `repeated-explanation` | The example record appears twice verbatim (record-construction section and worked example), and the worked example's four steps restate the mechanism table |
| BC-04 | §2 | `empty-transition` | "SPF is valuable but limited." conveys nothing the heading does not |
| BC-05 | §2 | `repeated-explanation` | "SPF provides no policy of its own. It tells the receiver whether the IP was authorized, but it does not tell the receiver what to do with a failure." — one fact stated twice |
| BC-06 | §3 | `empty-transition` | "DKIM is powerful, but it also has limitations." — same pattern as BC-04 |
| BC-07 | §3 | `repeated-explanation` | "DKIM does not specify any policy. It tells the receiver whether the signature matched. It does not say what the receiver should do…" — same double statement as BC-05 |
| BC-08 | §4 | `repeated-explanation` | The DMARC pass rule is stated twice in §4 itself: "at least one of the following must be true" (What DMARC Is For) and the numbered pass conditions (How Receivers Apply DMARC) |
| BC-09 | §4 | `repeated-explanation` | The worked example's "This means:" bullets restate the tag table rows, and its record duplicates the record shown earlier in the section |
| BC-10 | §5 | `repeated-explanation` | "At least one aligned authentication passed, so DMARC passes." — third statement of the pass rule |
| BC-11 | §5 | `formulaic-conclusion` | "This is the core purpose of DMARC: it forces SPF and DKIM authentication to match the domain the recipient actually sees." — aphoristic closer restating §4's purpose paragraph |
| BC-12 | §6 | `repeated-explanation` | SPF pitfalls "Too many DNS lookups", "Using SPF as a substitute for DKIM/DMARC", and "Forwarding breaks SPF" restate §2's three shortfall items |
| BC-13 | §6 | `repeated-explanation` | DKIM pitfall "Breaking signatures through message modification" restates §3's shortfall item 3; "Rotating keys too aggressively" echoes §3's item 4 |
| BC-14 | §6 | `repeated-explanation` | DMARC pitfalls restate §4 three times: "Forgetting that DMARC requires alignment…", "DMARC pass can happen through SPF alone or DKIM alone" (fourth statement of the pass rule), and "Forensic reports can leak sensitive data" (restating §4's `ruf=` warning) |
| BC-15 | §7 | `formulaic-conclusion` | "Used together, SPF, DKIM, and DMARC significantly reduce domain spoofing, improve mailbox providers' confidence…" — generic positive-summary closing |
| BC-16 | §7 | `repeated-explanation` | The three role bullets ("SPF to authorize the servers…", "DKIM to cryptographically sign…", "DMARC to require alignment…") restate §1's "In short:" list |

**Baseline count: 16** (11 `repeated-explanation`, 2 `formulaic-conclusion`,
2 `empty-transition`, 1 `formulaic-introduction`).

### 3.3 Pattern-mix comparison

- Same raw count (16) for the baseline and for v1 — but different mixes:
  v1's findings were 13 of 16 restatement inside or between dense technical
  sections, with no formulaic framing anywhere (compression review,
  Summary: "No findings for the `formulaic-introduction` and
  `over-explanation` patterns in any section"). The baseline's 16 include 3
  findings of generic framing/closers (BC-01, BC-11, BC-15) — the
  "recognizable AI writing habits" VISION.md names — and 6 findings of
  cross-section repetition where earlier material is re-served later
  (BC-12–BC-14 restate the shortfall lists as pitfalls; BC-08/BC-10
  restate the pass rule across §4–§5).
- After the revision stage, v2 carries 2 findings, both retained
  deliberately against outline requirements. On this dimension the
  pipeline output is well ahead of the baseline: **2 vs. 16**.

## 4. Dimension 3 — Technical Accuracy

Primary references: RFC 7208 (SPF), RFC 6376 (DKIM), RFC 7489 (DMARC),
RFC 8301 (DKIM algorithm/key updates). Verdicts: *correct*, *correct with
qualification*, *partially correct*, *incorrect*.

### 4.1 Checks against v2 (`draft/v2.md`)

| ID | Location | Claim checked | Reference | Verdict |
| --- | --- | --- | --- | --- |
| S1 | §5 | Empty MAIL FROM (bounce): the HELO/EHLO domain is checked instead [RFC 7208 §2.3] | RFC 7208 §2.4 | Correct — §2.4 defines the null-reverse-path MAIL FROM identity as `postmaster`@HELO-identity, so the HELO domain is what gets checked; §2.4 is the more precise citation |
| S2 | §6 step 3 | Record must begin `v=spf1`; multiple matching TXT records → PermError | RFC 7208 §4.5 | Correct |
| S3 | §6 step 5 | 10 DNS-dependent lookups per evaluation; exceeding → PermError | RFC 7208 §4.6.4 | Correct ("MUST limit the total number of those terms to 10… MUST return permerror") |
| S4 | §6 step 6 | Void lookup limit: 2 zero-record lookups; exceeding → PermError | RFC 7208 §4.6.4 | Correct with qualification — the two-void limit is a SHOULD; "Exceeding the limit produces a permerror result" when imposed |
| S5 | §5 | `ptr` — deprecated and discouraged | RFC 7208 §5.5 | Correct ("ptr (do not use)", "SHOULD NOT be published") |
| D1 | §8 | `p=` empty value means the key has been revoked | RFC 6376 §3.6.1 | Correct |
| D2 | §8 | `t=s` — strict subdomaining; `i=` must use `d=` exactly | RFC 6376 §3.6.1 | Correct ("the 'i=' domain MUST NOT be a subdomain of 'd='") |
| D3 | §9 | Oversigning: signing an absent header name prevents intermediaries inserting it — "the signature does not cover it" | RFC 6376 §3.5, §8.15 | Correct in substance, imprecise in the causal clause — §3.5 says the signature covers a signed-but-absent header "as the null string", which is exactly why inserting one later breaks verification; "the signature does not cover it" misstates the mechanism. §8.15 exists and is apt |
| D4 | §7 | `c=` default is `simple/simple` | RFC 6376 §3.5 | Correct |
| D5 | §7 | `l=` — body octets covered by `bh=`; appended content does not break the signature | RFC 6376 §3.5, §6.1.3 | Correct ("All data beyond that limit is not validated by DKIM") |
| M1 | §2 | Multiple `From:` addresses: "DMARC evaluates against the organizational domain of the first address" [RFC 7489 §5.6.1] | RFC 7489 | **Incorrect** — RFC 7489 has no §5.6.1 (§5 has no subsections), and §6.6.1 says multiple-address `From:` is typically rejected; for a multi-valued `From:` field each domain is checked and "the most strict policy selected among the checks that fail" is applied. There is no "first address" rule |
| M2 | §11 | `pct=` integer 0–100; the rest receive `none` treatment | RFC 7489 §6.3, §6.6.4 | Partially correct — the 0–100 range matches §6.3; "the rest receive `none` treatment" is wrong for `p=reject`: §6.6.4 says unsampled failing mail "SHOULD treat the email as though the 'quarantine' policy applies" |
| M3 | §14 step 3 | No DMARC record → processing stops, no enforcement [RFC 7489 §5.6] | RFC 7489 §6.6.3 | Claim correct ("policy discovery terminates and DMARC processing is not applied to this message"); citation wrong — §5.6 does not exist; the rule is §6.6.2 step 2 / §6.6.3 |
| M4 | §12, §13 | SPF alignment cited to [RFC 7489 §3.1.1]; DKIM alignment cited to [§3.1.2] | RFC 7489 §3.1.1, §3.1.2 | Claims correct; citations **swapped** — §3.1.1 is "DKIM-Authenticated Identifiers" and §3.1.2 is "SPF-Authenticated Identifiers" |
| M5 | §13 | Multiple DKIM signatures: DMARC passes via DKIM if any one passing signature is aligned [§4.1] | RFC 7489 §3.1.1 | Claim correct ("it is considered to be a DMARC 'pass' if any DKIM signature is aligned and verifies"); citation imprecise — §4.1 is "Authentication Mechanisms" |
| M6 | §15 | Cross-domain `rua=` authorization record `<policy-domain>._report._dmarc.<reporting-domain>` with value `v=DMARC1` | RFC 7489 §7.1 | Correct |
| M7 | §10 | Relaxed alignment compares organizational domains, determined via the public suffix list; default mode | RFC 7489 §3.2 | Correct |

**v2: 13/14 substantive claims correct** (1 incorrect — M1; 1 partially
correct — M2), with 4 citation errors (M1, M3, M4, M5).

### 4.2 Checks against the baseline (`baseline/one-shot.md`)

| ID | Location | Claim checked | Reference | Verdict |
| --- | --- | --- | --- | --- |
| bS1 | §2 | Result vocabulary: pass/fail/softfail/neutral/none/temperror/permerror; no record → `none`; syntax or DNS problems → permerror/temperror | RFC 7208 §2.6, §4.5 | Correct |
| bS2 | §2 | "SPF processing is limited to 10 DNS lookups… can cause SPF to fail with a permerror" | RFC 7208 §4.6.4 | Correct |
| bS3 | §2 | Records evaluated left to right; stop as soon as a mechanism matches | RFC 7208 §4.6 | Correct |
| bS4 | §2 | Qualifiers: `+` pass (default), `-` hard fail, `~` soft fail, `?` neutral | RFC 7208 §4.6.2 | Correct |
| bD1 | §3 | "Long public keys are often split across multiple quoted TXT strings. DNS automatically concatenates them." | RFC 6376 §3.6.2.2 | Correct ("Strings in a TXT RR MUST be concatenated together before use") |
| bD2 | §3 | "If the public key is missing, the result is `permerror` or `none`." | RFC 6376 §2.7, §6.1.2 | **Incorrect** — RFC 6376 has no `permerror` result (that vocabulary belongs to SPF/DMARC); a nonexistent key record returns PERMFAIL ("no key for signature") per §6.1.2, and `none` means no signature was present — a different condition |
| bD3 | §3 | `k=rsa` usual; `ed25519` also exists | RFC 6376 §3.6.1, RFC 8463 | Correct |
| bD4 | §3 | "Some providers use a CNAME instead of a TXT record" for key delegation | RFC 6376 §3.6.2.2 | Not supported by the primary reference — the RFC specifies TXT RR storage and never mentions CNAME. The claim reflects industry practice but has no RFC basis |
| bM1 | §4 | Example records contain `adkim=relaxed; aspf=relaxed` (both example records) | RFC 7489 §6.3 | **Incorrect** — §6.3 defines only `r`/`s` as valid values; the records as printed are invalid syntax, and they contradict the baseline's own tag table, which correctly says "`r` for relaxed or `s` for strict" |
| bM2 | §4 | `pct=` "from 1 to 100" | RFC 7489 §6.3 | **Incorrect** — §6.3 says "integer between 0 and 100, inclusive" |
| bM3 | §4 | `sp=` omitted → `p=` applies to subdomains | RFC 7489 §6.3 | Correct ("If absent, the policy specified by the 'p' tag MUST be applied for subdomains") |
| bM4 | §4 | Aggregate reports: XML documents sent daily | RFC 7489 §7.2 | Correct ("daily (or more frequent)"; format in Appendix C) |
| bM5 | §4 | Forensic reports "may include copies or headers of individual messages that failed DMARC" | RFC 7489 §7.3 | Correct ("SHOULD include as much of the message and message header as is reasonable") |
| bM6 | §6 | "1024-bit RSA keys are increasingly discouraged. 2048-bit or stronger keys are recommended." | RFC 8301 §3.2 | Partially correct — RFC 8301 makes ≥1024 bits a MUST for signers and SHOULD ≥2048; "discouraged" understates that 1024 is a hard floor |

**Baseline: 9/14 fully correct**, 3 incorrect (bD2, bM1, bM2), 1 partially
correct (bM6), 1 uncited-practice (bD4).

### 4.3 Comparison

Both documents contain real errors, of different kinds:

- The baseline's errors are load-bearing: two example DMARC records with
  invalid `adkim=`/`aspf=` values (a learner copying them publishes a
  broken record) and a wrong DKIM result vocabulary. It also asserts a
  practice (CNAME delegation) with no primary-reference support.
- v2's substantive claims are nearly all correct and checkable, but the
  pipeline introduced a new failure mode the baseline cannot exhibit:
  **confidently wrong citations** — two citations to RFC sections that do
  not exist (§5.6, §5.6.1), one swapped pair (§3.1.1/§3.1.2), and one
  imprecise (§4.1). One of these carries a substantively wrong rule (M1,
  the "first address" claim).

The citation errors were verified to originate in
`research/knowledge-corpus.md` (lines 30, 106, 201, 313), propagate
verbatim through `outline/information-outline.md` (O2.4, O6.1, O10.1,
O14.4) into v1 and v2, and pass all three review passes unchanged: the
alignment review scopes out "technical accuracy against the RFCs
themselves" (alignment review, Scope), and the editorial review, per its
PoC simplification, flags uncertain claims "for follow-up rather than
verified" (editorial review, Scope) — and flagged none of these.

## 5. The README Questions

Findings for the seven questions in `README.md` § Questions.

### Q1. Does building the information outline before writing materially improve the final result?

**Yes, on coverage.** The outline functioned as a coverage contract: the
alignment review found v1 at 154/156 outline items present, 0 inaccurate,
2 missing (section-level citations only), and the revision restored the 2
(F1/F2 → 156/156). Meanwhile the baseline, produced without an outline,
lacks at minimum 20+ outline items (section 2.2 of this evaluation). The
outline-gated output also achieves this coverage in 273 fewer words than
the baseline. Causation caveat: the research corpus and the outline were
produced together; this run isolates neither. *(alignment-review.md
Verdict; revision-log.md F1/F2; §1 and §2.2 here.)*

### Q2. Does retaining the outline prevent information loss during compression?

**Yes, as a guardrail — with a cost.** The two compression findings that
were not applied (C-15, C-16) were deferred specifically because the cuts
would drop outline items the alignment review records as required (O15.6,
O15.7, O16.9, O16.14, O16.15). The outline blocked lossy cuts. But the
outline also froze in corpus-level errors: revision is barred from editing
it ("the outline is a frozen reference and is not edited in this stage",
revision-log E-07 note), so the §5.6.1 "first address" error and the
swapped §3.1.1/§3.1.2 citations (section 4.3) are locked into the
pipeline's reference model and every document checked against it.
*(revision-log.md C-15, C-16 dispositions and the E-07 note; §4.3 here.)*

### Q3. Can AI design a sensible learning sequence from a technical knowledge outline?

**Yes, evidenced.** The learner journey made four deliberate departures
from outline order (overview before mechanism detail; SPF/DKIM alignment
deferred until DMARC exists; reporting after the combined evaluation), and
the alignment review verified the result end to end: all 17 draft sections
trace one-to-one to journey entries, no ordering conflicts, and
cross-reference spot checks found no section assuming a concept its
journey entry lists as not yet taught. *(learner-journey.md entries 1–17
"Why here"/"Expected understanding"; alignment-review.md "Journey
Trace", "Draft Ordering vs. the Learner Journey", "Cross-Reference and
Prerequisite Checks".)*

### Q4. How much useful compression can be achieved without reducing technical completeness?

**Modest within the run; the larger win is density versus the baseline.**
v1 → v2 removed 107 words (3352 → 3245, 3.2%) while coverage moved from
154/156 to 156/156 and the two excluded items were removed. The small
number is explained by what the compression review found: "the length
problems found are all restatement, not over-elaboration" — v1 was already
dense, so there was little prose to cut. Against the baseline, v2 delivers
measurably more topic areas at 92.2% of the baseline's word count (3245
vs. 3518). *(compression-review.md Summary; revision-log.md Summary; §1
and §2 here.)*

### Q5. Do specialized judge passes produce better results than generic editing prompts?

**Indicatively yes, with a demonstrated blind spot.** The three passes
produced 34 discrete findings across non-overlapping responsibilities with
only three shared passages (editorial-review coordination table), and each
pass found things the others structurally could not: alignment caught the
coverage gaps, compression the restatements, editorial the
learner-facing defects — including the draft's only internal
contradiction (E-07). The blind spot: no pass was authorized to verify
claims or citations against primary references, so the four citation
errors in section 4.1 passed all three reviews into v2. Specialized
passes divide the work well; the pipeline still needs a pass whose
specialty is "check it against the source". *(editorial-review.md
"Coordination notes" and Summary; §4.1, §4.3 here.)*

### Q6. Are three review stages useful, excessive, or insufficient?

**Useful and roughly right-sized; insufficient as the only checks.**
The three passes were non-duplicative (34 findings, 3 overlapping
passages) and each earned its place (Q5). But two gaps remain: no stage
verifies against primary references, and the verification stage
(`review/verification-review.md`, stage 7) has not run — the accuracy
findings in section 4.1 are exactly what that stage should own. The
deferral mechanism also exposed a missing authority: when a review finding
conflicts with the outline, the finding is deferred (C-15, C-16) and there
is no stage that may correct the outline. *(compression-review.md
coordination note; revision-log.md deferrals; §4.3 here.)*

### Q7. Does targeted ticket-based revision outperform complete document regeneration?

**Supported, with a caveat: no regeneration control was run.** All 34
findings were applied as discrete edits with per-finding dispositions;
"unrelated sections are untouched, and the document was not regenerated"
(revision-log.md, opening). v2 preserves v1's structure, ordering, and
coverage while resolving 31/34 findings, and the documented risks of
regeneration — damage to already-good content, loss of traceability —
did not materialize. "Outperform" is therefore supported only by the
absence of damage plus full traceability, not by a head-to-head run; a
control should be part of a second iteration. *(revision-log.md
dispositions and Summary; v1 vs. v2 structure comparison in
alignment-review.md trace tables, which still hold for v2.)*

## 6. Verdict

**The staged pipeline produced materially different output from the
one-shot baseline.**

| Dimension | Verdict | Evidence |
| --- | --- | --- |
| Information completeness | **Better** | v2 covers every outline item (156/156 after F1/F2) and the baseline lacks 20+ of them (§2.2); v2-only areas include the identity model, canonicalization, ARC/SRS, `fo=`, cross-domain reporting |
| AI writing-pattern prevalence | **Better** | 2 residual findings in v2 (both outline-mandated deferrals) vs. 16 in the baseline under the same criteria, including 3 formulaic framing/closers the baseline retains (§3) |
| Word economy / information density | **Better** | 3245 vs. 3518 words (7.8% fewer) with more topic areas |
| Technical accuracy of claims | **Better, narrowly** | 13/14 sampled v2 claims correct vs. 9/14 baseline; baseline errors include invalid example records a learner would copy (bM1) and wrong result vocabulary (bD2) |
| Citation reliability | **Worse** | v2 carries citations to nonexistent RFC sections (§5.6, §5.6.1) and a swapped pair; the baseline avoids this by citing nothing (§4.1) |
| Worked narrative examples | **Worse** | Baseline §5 walks a full legitimate delivery and a spoofing attempt with concrete records; v2 §14 is an abstract algorithm with no narrative message |
| Logical structure | **Equivalent** | Both are logically ordered; v2's order is additionally proven journey-aligned by the alignment review |

**README success test, applied to v2 as the latest pipeline output:**
condition 1 (fewer words) holds — 3245 < 3518; condition 2 (no outline
concept absent) holds — 156/156 per the alignment review plus F1/F2. The
binary verdict is therefore `true` for v2. Caveats: the test nominally
targets `final/material.md`, which does not exist yet (stages 7–8
pending), and the baseline was not evaluated against the outline (the
outline postdates it; the topic-area comparison of §2 stands in).

## 7. What Should Change in a Second Iteration

1. **Add a primary-reference verification pass — and make it run on the
   outline before anything is written against it.** This is the single
   change best grounded in this run's artefacts: every citation error
   found by this evaluation (§4.1: M1, M3, M4, M5) was born in
   `research/knowledge-corpus.md`, froze into the outline, and survived
   all three review passes because none of them verifies against primary
   references (their scopes say so explicitly). A verification pass with
   a primary-reference mandate — ideally stage 7's
   `review/verification-review.md`, extended to check a sample of
   citations against the RFCs — would have caught all four before v2.
2. **Give the revision stage authority to correct the outline (or the
   corpus).** Revision-log E-07 shows the outline's own error ("subdomains
   default to `none` in some interpretations") could only be fixed in the
   draft, leaving the pipeline's reference model wrong and the material
   and outline in disagreement. The "frozen reference" rule prevented
   information loss (Q2) but also locked in error; a correction path with
   a recorded disposition keeps the first benefit without the second.
3. **Run a regeneration control.** Q7's "outperform" verdict rests on
   absence of damage, not a comparison. Regenerating the same v1 with a
   single revision prompt and running both outputs through the same
   criteria would turn it into a measurable claim.
4. **Complete stages 7–8 before the next evaluation.** This evaluation
   was forced to assess v2, a document that says "Not yet verified",
   because `final/material.md` does not exist.
