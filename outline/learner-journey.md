# Learner Journey: SPF, DKIM, and DMARC

The information outline (`information-outline.md`) records **what** must be
communicated. This document records **in what order** a learner should
encounter it, so that every concept is used only after it has been introduced.

---

## How to Read This Document

- Each entry maps to a topic area of `information-outline.md`. The entry
  sequences and motivates the area; it does not restate its content.
- **Kind** — `introduce`: first occurrence of the topic; the writing stage
  explains it from scratch. `expand`: builds on a prior entry; the writing
  stage assumes that entry and only adds new detail.
- **Prerequisites** — the direct precedent entries, by number and concept.
- **Why here** — the sequencing rationale. Entries that connect multiple
  mechanisms say so here.
- **Expected understanding** — what the learner should hold after the entry;
  the alignment-review stage uses this to check that later entries never
  assume more.

---

## Sequence

### 1 · The Problem Email Authentication Addresses

- **Kind:** introduce
- **Prerequisites:** none
- **Outline section:** "The Problem Email Authentication Addresses"
- **Why here:** Opens the sequence because every later mechanism exists only
  as an answer to this problem. It gives the learner the failure to keep in
  mind — any MTA can claim any sender — so each mechanism is learned as a fix
  to a named hole rather than as syntax for its own sake. It also motivates
  entry 2: the problem is exploitable because two sender identities exist and
  neither is checked.
- **Expected understanding:** The learner can state that SMTP accepts any
  claimed sender without proof, and name the attacks this enables (phishing
  by brand impersonation, business email compromise, spam).

### 2 · SMTP Identities Relevant to Authentication

- **Kind:** introduce
- **Prerequisites:** 1 (the problem)
- **Outline section:** "SMTP Identities Relevant to Authentication"
- **Why here:** Immediately after the problem, because the three identities
  are the objects every later mechanism operates on: SPF checks one, DKIM
  signs from another, DMARC compares them. Deferred, "MAIL FROM" and "From"
  would surface as unexplained jargon inside the mechanism entries.
- **Expected understanding:** The learner distinguishes RFC5321.MailFrom
  (envelope; bounce path; invisible to users), RFC5322.From (displayed to
  users), and the DKIM `d=` signing domain, and knows all three can differ on
  one message.

### 3 · DNS as Part of the Authentication Model

- **Kind:** introduce
- **Prerequisites:** 1 (the problem)
- **Outline section:** "DNS as Part of the Authentication Model"
- **Why here:** Before any record syntax, because every mechanism below
  publishes its data as DNS TXT records. The premise "the domain owner
  controls DNS, so DNS is a trusted channel" is what makes a record lookup
  prove anything; taught once here, it is not re-justified per mechanism.
- **Expected understanding:** The learner can say why DNS TXT records are a
  trustworthy place to publish authentication data, and that receivers query
  DNS at delivery time.

### 4 · How the Mechanisms Layer Together (Overview)

- **Kind:** introduce
- **Prerequisites:** 2 (SMTP identities), 3 (DNS as distribution channel)
- **Outline section:** "How SPF, DKIM, and DMARC Are Evaluated Together" —
  first occurrence, at overview depth; the full seven-step mechanics arrive
  at entry 14.
- **Why here:** Before any mechanism syntax, the learner gets the end-to-end
  shape: SPF answers "did the sending IP have permission for the envelope
  domain?", DKIM answers "does a valid signature vouch for this message?",
  DMARC ties whichever results exist to the visible From domain and applies
  policy. Each mechanism entry below then has a slot to fill. This is the
  first connection point between the three technologies.
- **Expected understanding:** The learner can sketch the three layers, which
  identity each checks, and that DMARC is the decision layer rather than an
  authentication mechanism itself.

### 5 · SPF: Record Syntax

- **Kind:** introduce
- **Prerequisites:** 2 (SMTP identities), 3 (DNS as distribution channel),
  4 (layered overview)
- **Outline section:** "SPF: Record Syntax"
- **Why here:** SPF first among the mechanisms because it is the simpler one —
  a single TXT record of IP authorization, no cryptography — and because the
  receiver evaluates it first in the flow of entry 4. Syntax before
  evaluation so the learner sees the record a domain owner publishes before
  the receiver-side algorithm that reads it.
- **Expected understanding:** The learner can read an SPF record (`v=spf1`,
  mechanisms, qualifiers, modifiers) and state what each part authorizes,
  including the record example in the outline.

### 6 · SPF: Evaluation Algorithm

- **Kind:** expand
- **Prerequisites:** 5 (SPF record syntax)
- **Outline section:** "SPF: Evaluation Algorithm"
- **Why here:** Immediately after the syntax, because evaluation is the
  receiver's execution of the record just learned; every step lands on known
  vocabulary (mechanisms, qualifiers). The result vocabulary (Pass, Fail,
  SoftFail, Neutral, None, TempError, PermError) is established here because
  entries 10, 14, and 17 name results without redefining them.
- **Expected understanding:** The learner can walk the left-to-right
  evaluation, state the 10-DNS-lookup and 2-void-lookup limits and their
  PermError consequence, and list the seven possible results.

### 7 · DKIM: Signature Structure

- **Kind:** introduce
- **Prerequisites:** 2 (SMTP identities), 3 (DNS as distribution channel),
  4 (layered overview)
- **Outline section:** "DKIM: Signature Structure"
- **Why here:** With SPF complete, DKIM opens with the signature — the
  artifact the sender produces. This entry and entry 8 together introduce the
  key-pair model behind DKIM: the sender signs with a private key (the
  signature in `b=`), and the receiver verifies with the public key published
  under the selector. Signature structure before selectors and verification
  because those two explain the public-key side of an artifact the learner
  has already seen.
- **Expected understanding:** The learner can read the required and optional
  `DKIM-Signature` tags, state the roles of `d=`, `h=`, and `s=`, and explain
  canonicalization as "why minor transport changes can break verification".

### 8 · DKIM: Selectors

- **Kind:** expand
- **Prerequisites:** 7 (DKIM signature structure)
- **Outline section:** "DKIM: Selectors"
- **Why here:** Selectors are the bridge from the `s=` tag just learned to the
  key record the receiver fetches; "which key to retrieve" only means
  something now that the learner knows what a signature is. The key-record
  tags (`v=DKIM1`, `k=`, `p=`, `t=y`, `t=s`) land here as the second half of
  the key-pair model opened in entry 7.
- **Expected understanding:** The learner can build the
  `<selector>._domainkey.<domain>` lookup name from a signature, and explain
  why selectors enable key rotation and per-service keys.

### 9 · DKIM: Key Lookup and Verification

- **Kind:** expand
- **Prerequisites:** 7 (DKIM signature structure), 8 (DKIM selectors)
- **Outline section:** "DKIM: Key Lookup and Verification"
- **Why here:** Completes DKIM with the receiver-side procedure, using the
  lookup name from entry 8 and the tags from entry 7. Oversigning is kept
  here rather than in the signature structure because it only makes sense
  once both sides of verification are in view.
- **Expected understanding:** The learner can walk the seven verification
  steps, and state the difference between a failing signature (modification
  or fraud) and a missing signature (no result).

### 10 · DMARC: Identifier Alignment (Strict vs. Relaxed)

- **Kind:** introduce
- **Prerequisites:** 2 (SMTP identities), 6 (SPF evaluation results),
  9 (DKIM verification results)
- **Outline section:** "DMARC: Identifier Alignment (Strict vs. Relaxed)"
- **Why here:** This is DMARC's first appearance, and alignment — not the
  record — is its central idea: SPF and DKIM each authenticate an identity of
  their own choosing; DMARC asks whether either result vouches for the domain
  the user sees (RFC5322.From). All its inputs now exist (identities from 2,
  results from 6 and 9). The strict/relaxed distinction and the
  organizational-domain (Public Suffix List) concept are introduced once
  here, rather than three times as the outline repeats them.
- **Expected understanding:** The learner can state why authentication
  without alignment does not protect the visible From domain, and define
  relaxed vs. strict comparison in general terms.

### 11 · DMARC: Policy Options

- **Kind:** expand
- **Prerequisites:** 10 (alignment concept)
- **Outline section:** "DMARC: Policy Options"
- **Why here:** The record that configures DMARC only means something after
  the concept it configures exists: `p=` is "what the receiver does when
  neither path aligns", `adkim=`/`aspf=` select the modes defined in
  entry 10, and `sp=`/`pct=` refine them. See Ordering Decision 3.
- **Expected understanding:** The learner can read a DMARC record and state
  what `p=`, `sp=`, `pct=`, `adkim=`, `aspf=`, `rua=`, `ruf=`, and `fo=` each
  control, including the record example in the outline.

### 12 · SPF Alignment

- **Kind:** expand
- **Prerequisites:** 6 (SPF evaluation), 10 (alignment concept)
- **Outline section:** "SPF Alignment"
- **Why here:** An expansion of entry 10's alignment concept onto the SPF
  path. Deferred from the outline's placement — immediately after SPF
  evaluation — because alignment is defined by DMARC and motivates nothing
  until the learner knows DMARC exists to ask "does the SPF pass vouch for
  the From domain?"; taught there, it is an unmotivated rule. See Ordering
  Decision 2. The outline's bounce-domain example is the payoff here.
- **Expected understanding:** The learner can decide whether a given
  MailFrom/From pair aligns under relaxed and strict modes, and knows the
  default is relaxed (`aspf=r`).

### 13 · DKIM Alignment

- **Kind:** expand
- **Prerequisites:** 9 (DKIM verification), 10 (alignment concept)
- **Outline section:** "DKIM Alignment"
- **Why here:** Symmetric to entry 12 for the DKIM path (`d=` vs. From).
  Placed immediately after SPF alignment so the learner sees the two paths
  side by side before the combined evaluation. The multiple-signature rule
  (any one aligned passing signature suffices) is kept here, next to the
  `d=` identity it uses.
- **Expected understanding:** The learner can decide whether a passing
  signature's `d=` aligns with From under both modes, and state the
  any-one-aligned-signature rule.

### 14 · How SPF, DKIM, and DMARC Are Evaluated Together

- **Kind:** expand
- **Prerequisites:** 6 (SPF evaluation), 9 (DKIM verification),
  11 (DMARC record), 12 (SPF alignment), 13 (DKIM alignment)
- **Outline section:** "How SPF, DKIM, and DMARC Are Evaluated Together" —
  full mechanics, promised at entry 4.
- **Why here:** The capstone of the mechanism material. Every step of the
  seven-step walkthrough (SPF check, DKIM verification, DMARC record lookup,
  both alignment checks, disposition, reporting hook) is now a known piece,
  so this entry runs them in order without stopping to explain any. The
  second connection point between the three technologies.
- **Expected understanding:** The learner can trace a message end-to-end
  through the seven steps and predict the disposition from given SPF/DKIM
  results and a DMARC record.

### 15 · DMARC: Aggregate Reporting

- **Kind:** expand
- **Prerequisites:** 11 (DMARC record tags), 14 (full evaluation)
- **Outline section:** "DMARC: Aggregate Reporting"
- **Why here:** After the full evaluation, because report rows summarize
  exactly those outputs (per-source IP, SPF result, DKIM result,
  disposition). See Ordering Decision 4. The cross-domain `rua=`
  authorization record lands here with `rua=` already known from entry 11.
- **Expected understanding:** The learner can state what an aggregate report
  contains, where it is sent, and what the cross-domain authorization record
  is for.

### 16 · Email Forwarding and Intermediary Effects

- **Kind:** introduce
- **Prerequisites:** 14 (full evaluation)
- **Outline section:** "Email Forwarding and Intermediary Effects"
- **Why here:** Assumes the complete evaluation, because every effect it
  describes is "which step of entry 14 breaks when an intermediary touches
  the message": SPF fails at the forwarder's IP, DKIM survives unmodified
  relay, mailing lists modify the message, DMARC fails on the From domain.
  It also introduces the two concepts that patch the gaps — ARC and SRS — so
  it is an introduction, not a recap.
- **Expected understanding:** The learner can predict what happens to a
  legitimately forwarded or list-processed message under each mechanism, and
  state what ARC and SRS each do and do not fix.

### 17 · Common Failure Modes

- **Kind:** expand
- **Prerequisites:** 6 (SPF evaluation), 9 (DKIM verification),
  11 (DMARC record), 15 (aggregate reporting), 16 (forwarding effects)
- **Outline section:** "Common Failure Modes"
- **Why here:** Last, as application of everything above: each failure mode is
  a diagnosis exercise ("which step of the evaluation, and which record
  choice, produced this outcome?"). The SPF/DKIM/DMARC groupings in the
  outline map onto entries 6, 9, and 11 respectively; the forwarding-caused
  failures additionally assume entry 16. Kept as one entry rather than three
  per-mechanism ones because the learner's job here is seeing the system as a
  whole.
- **Expected understanding:** The learner can, for a described symptom, name
  the likely failing step and the record or configuration choice that caused
  it.

---

## Ordering Decisions

The sequence deliberately departs from how the primary reference
documentation and the outline order the material.

### 1. Overview before mechanism detail

The references describe each mechanism in full within its own specification
(RFC 7208 for SPF, RFC 6376 for DKIM, RFC 7489 for DMARC); the combined
picture surfaces only inside RFC 7489 (cited by the outline as §4–§5) —
three specifications in. The journey instead opens with the three-layer
picture at overview depth (entry 4). Why: a learner needs a place to file
each new fact; without the overview, SPF syntax is learned before the
learner knows what problem SPF occupies in the chain, or that DMARC will
later tie the pieces together. The full mechanics (entry 14) are postponed
until every step is individually known.

### 2. Alignment deferred until DMARC's purpose exists

RFC 7489 introduces SPF alignment (§3.1.1, cited by the outline) before DKIM
alignment, and the outline places "SPF Alignment" immediately after SPF
evaluation — before DKIM is even introduced. In the RFC this works because
the reader is already inside the DMARC standard; in a learner-facing
sequence it does not: alignment is defined by DMARC and exists to answer
"does this authentication result vouch for the visible From domain?", a
question with no meaning until the learner knows DMARC will ask it. Taught
at the outline's position, alignment is an unmotivated rule from a standard
the learner has not yet met. The journey introduces the alignment concept
once, as DMARC's first appearance (entry 10), and teaches the SPF and DKIM
cases as expansions (entries 12 and 13). This also removes the outline's
triple restatement of strict/relaxed.

### 3. Policy record after the alignment concept

The outline places "DMARC: Policy Options" before "DMARC: Identifier
Alignment"; the journey reverses them (entries 10 then 11). The record's
`adkim=`/`aspf=` configure the alignment modes and `p=` describes action on
misaligned mail, so reading the syntax first means reading configuration for
a concept that does not exist yet. RFC 7489 itself orders alignment (§3.1)
before its record fields (§6.3); the journey follows the RFC here, against
the outline.

### 4. Reporting after the combined evaluation

The outline places "DMARC: Aggregate Reporting" before "How SPF, DKIM, and
DMARC Are Evaluated Together"; the journey moves it after (entries 14 then
15). Report contents are a summary of the evaluation's outputs — SPF result,
DKIM result, disposition — so presenting them first makes the row fields
reference results the learner has not yet seen computed. RFC 7489 orders the
evaluation (§4–§5) before reporting (§7); the journey follows the RFC here,
against the outline.

---

## Topic Coverage

Every topic area of `information-outline.md` appears in the sequence above.

| Outline topic area | Journey entry |
| --- | --- |
| The Problem Email Authentication Addresses | 1 |
| SMTP Identities Relevant to Authentication | 2 |
| DNS as Part of the Authentication Model | 3 |
| SPF: Record Syntax | 5 |
| SPF: Evaluation Algorithm | 6 |
| SPF Alignment | 12 |
| DKIM: Signature Structure | 7 |
| DKIM: Selectors | 8 |
| DKIM: Key Lookup and Verification | 9 |
| DKIM Alignment | 13 |
| DMARC: Policy Options | 11 |
| DMARC: Identifier Alignment (Strict vs. Relaxed) | 10 |
| DMARC: Aggregate Reporting | 15 |
| How SPF, DKIM, and DMARC Are Evaluated Together | 4, 14 |
| Email Forwarding and Intermediary Effects | 16 |
| Common Failure Modes | 17 |
