# Alignment Review: Draft vs. Information Outline and Learner Journey

First-pass review of `../draft/v1.md` for alignment, coverage, and completeness against `../outline/information-outline.md` (what must be communicated) and `../outline/learner-journey.md` (in what order).

Scope, per the issue:

- Every information-outline item is recorded `present`, `inaccurate`, or `missing`.
- Every draft section is checked against a learner-journey entry.
- Draft ordering is checked against the journey sequence.
- Editorial quality is not evaluated, and technical accuracy against the RFCs themselves is not verified in this pass (deferred to the editorial review).

This review does not modify the draft, outline, or journey.

## Verdict

- **Outline coverage (O1–O16):** 156 items — 154 `present`, 2 `missing`, 0 `inaccurate`. The 2 missing items are section-level RFC citations (O8.1, O12.1). Every content claim in the outline is covered, and nothing in the draft contradicts the outline.
- **Excluded items (O17):** 4 of 6 remain correctly absent; 2 are re-introduced by one sentence in draft section 14 (O17.5, O17.6).
- **Journey trace:** all 17 draft sections trace to journey entries 1–17, one to one.
- **Ordering:** no conflicts. The draft follows the journey's three deliberate departures from outline order in every affected section.

Findings for the revision stage: F1, F2, F3/F4 (see [Findings](#findings)).

## Coverage Against the Information Outline

Statuses: `present` (correctly covered), `inaccurate` (present but technically wrong or misleading), `missing` (not covered in the draft). The draft section that covers each item is named (`D` = draft section number). Outline sections are numbered in outline order (`O`), which differs from draft order wherever the journey mandates a departure.

### O1 · The Problem Email Authentication Addresses — covered in D1

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O1.1] | SMTP (RFC 5321) does not authenticate the claimed sender; any MTA can issue `MAIL FROM:<anything@example.com>` without proving control | present — D1 | — |
| [O1.2] | RFC 5322 `From:` header shown to users is unverified and can differ from the envelope sender | present — D1 | — |
| [O1.3] | Together these enable domain spoofing: mail appearing to come from a trusted domain from an attacker-controlled server | present — D1 | — |
| [O1.4] | Attack: phishing via brand impersonation | present — D1 | — |
| [O1.5] | Attack: business email compromise (BEC) | present — D1 | — |
| [O1.6] | Attack: spam delivery hiding its real origin | present — D1 | — |
| [O1.7] | SPF, DKIM, and DMARC are layered mechanisms that together let a receiving server verify origin claims and act on failures | present — D1 | — |

### O2 · SMTP Identities Relevant to Authentication — covered in D2

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O2.1] | Three distinct domain identities appear in a mail transaction; they can all differ | present — D2 | — |
| [O2.2] | RFC5321.MailFrom: envelope; bounce delivery (DSNs); not visible to users; the identity SPF checks [RFC 7208 §2] | present — D2 | — |
| [O2.3] | RFC5322.From: displayed to the user; not checked directly by SPF or DKIM; used by DMARC for alignment [RFC 7489 §3.1] | present — D2 | — |
| [O2.4] | Multiple `From:` addresses possible (rare); DMARC evaluates against the first organizational domain found [RFC 7489 §5.6.1] | present — D2 | — |
| [O2.5] | DKIM signing domain (`d=`): independent of the other identities; DMARC alignment checks it against RFC5322.From [RFC 6376 §3.5, RFC 7489 §3.1.2] | present — D2 | — |

### O3 · DNS as Part of the Authentication Model — covered in D3

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O3.1] | SPF and DKIM both store authentication data in DNS TXT records | present — D3 | — |
| [O3.2] | DNS serves as a trusted distribution channel because the domain owner controls it | present — D3 | — |
| [O3.3] | DNSSEC can authenticate DNS responses but is not required by SPF, DKIM, or DMARC | present — D3 | — |
| [O3.4] | A receiving server performs DNS lookups at delivery time to retrieve current policy | present — D3 | — |

### O4 · SPF: Record Syntax — covered in D5

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O4.1] | Standard: RFC 7208 | present — D5 | — |
| [O4.2] | Location: TXT record published at the RFC5321.MailFrom domain; example lookup `TXT example.com` | present — D5 | — |
| [O4.3] | Empty MAIL FROM (bounce message): the HELO/EHLO domain is checked instead [RFC 7208 §2.3] | present — D5 (merged into the Location bullet) | — |
| [O4.4] | Prefix: `v=spf1` | present — D5 | — |
| [O4.5] | Mechanisms evaluated left-to-right; first match wins | present — D5 | — |
| [O4.6] | Mechanism `all` — catch-all terminator | present — D5 | — |
| [O4.7] | Mechanism `include:<domain>` — recursive; counts toward the DNS lookup limit | present — D5 | — |
| [O4.8] | Mechanism `a[:<domain>]` | present — D5 | — |
| [O4.9] | Mechanism `mx[:<domain>]` | present — D5 | — |
| [O4.10] | Mechanism `ip4:<address/cidr>` | present — D5 | — |
| [O4.11] | Mechanism `ip6:<address/cidr>` | present — D5 | — |
| [O4.12] | Mechanism `ptr[:<domain>]` — deprecated; discouraged by RFC 7208 §5.5 | present — D5 | — |
| [O4.13] | Mechanism `exists:<domain>` — matches if the domain has any A record | present — D5 | — |
| [O4.14] | Qualifier `+` Pass — default if omitted | present — D5 | — |
| [O4.15] | Qualifier `-` Fail — reject recommended | present — D5 | — |
| [O4.16] | Qualifier `~` SoftFail — accept but mark | present — D5 | — |
| [O4.17] | Qualifier `?` Neutral — no assertion either way | present — D5 | — |
| [O4.18] | Modifier `redirect=<domain>` — shared SPF policies | present — D5 | — |
| [O4.19] | Modifier `exp=<domain>` — human-readable failure explanation | present — D5 | — |
| [O4.20] | Example record `v=spf1 ip4:203.0.113.0/24 include:_spf.google.com ~all` | present — D5 | — |

### O5 · SPF: Evaluation Algorithm — covered in D6

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O5.1] | Section citation [RFC 7208 §4] | present — D6 | — |
| [O5.2] | Step 1: extract the RFC5321.MailFrom domain (or HELO if MailFrom is empty) | present — D6 | — |
| [O5.3] | Step 2: retrieve the TXT record at that domain | present — D6 | — |
| [O5.4] | Step 3: record must begin with `v=spf1`; multiple matching TXT records → PermError | present — D6 | — |
| [O5.5] | Step 4: left-to-right evaluation; no mechanism matches → implicit Neutral | present — D6 | — |
| [O5.6] | Step 5: max 10 DNS-dependent lookups; exceeding → PermError [RFC 7208 §4.6.4] | present — D6 | — |
| [O5.7] | Step 6: max 2 void lookups (zero records returned); exceeding → PermError [RFC 7208 §4.6.4] | present — D6 | — |
| [O5.8] | Possible results: Pass, Fail, SoftFail, Neutral, None (no record), TempError, PermError | present — D6 | — |

### O6 · SPF Alignment — covered in D12

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O6.1] | Section citation [RFC 7489 §3.1.1] | present — D12 | — |
| [O6.2] | SPF alone does not prove alignment with RFC5322.From; DMARC defines SPF alignment as RFC5321.MailFrom vs. RFC5322.From | present — D12 | — |
| [O6.3] | Relaxed alignment (default): organizational domains must match; MailFrom `bounce.example.com` + From `user@example.com` → aligned | present — D12 | — |
| [O6.4] | Strict alignment: exact domain equality; the same pair → not aligned | present — D12 | — |

### O7 · DKIM: Signature Structure — covered in D7

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O7.1] | Standard: RFC 6376 | present — D7 | — |
| [O7.2] | Signature added as a `DKIM-Signature:` header field before transmission | present — D7 | — |
| [O7.3] | Required tag `v=` — version, currently `1` | present — D7 | — |
| [O7.4] | Required tag `a=` — `rsa-sha256` (recommended) or `rsa-sha1` (deprecated in RFC 8301) | present — D7 | — |
| [O7.5] | Required tag `b=` — base64-encoded signature value | present — D7 | — |
| [O7.6] | Required tag `bh=` — base64-encoded hash of the canonicalized body | present — D7 | — |
| [O7.7] | Required tag `d=` — signing domain (public key lookup; DMARC alignment) | present — D7 | — |
| [O7.8] | Required tag `h=` — colon-separated signed header names, in order signed | present — D7 | — |
| [O7.9] | Required tag `s=` — selector | present — D7 | — |
| [O7.10] | Optional tag `c=` — canonicalization `<header>/<body>`; default `simple/simple` [RFC 6376 §3.4] | present — D7 | — |
| [O7.11] | Optional tag `i=` — AUID; must share the `d=` domain | present — D7 | — |
| [O7.12] | Optional tags `t=` (signature timestamp) and `x=` (expiration) | present — D7 (split into two bullets) | — |
| [O7.13] | Optional tag `l=` — octets of the body covered by `bh=` | present — D7 | — |
| [O7.14] | Canonicalization `simple` — fragile to minor reformatting | present — D7 | — |
| [O7.15] | Canonicalization `relaxed` — tolerant of minor modification by intermediaries | present — D7 | — |
| [O7.16] | Example `DKIM-Signature` header | present — D7 | — |

### O8 · DKIM: Selectors — covered in D8

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O8.1] | Section citation [RFC 6376 §3.1, §3.5] | missing — D8 carries no citation | See F1 |
| [O8.2] | A selector is a string identifying which public key to retrieve | present — D8 | — |
| [O8.3] | Lookup name `<selector>._domainkey.<domain>`; example `20230101._domainkey.example.com` | present — D8 | — |
| [O8.4] | Multiple selectors can be active simultaneously (key rotation, per-service keys) | present — D8 | — |
| [O8.5] | A signing service has its own selector; the domain owner publishes the corresponding public key | present — D8 | — |
| [O8.6] | Key record tag `v=DKIM1` — recommended but optional | present — D8 | — |
| [O8.7] | Key record tag `k=` — `rsa` (default) or `ed25519` (RFC 8463) | present — D8 | — |
| [O8.8] | Key record tag `p=` — empty value means the key has been revoked | present — D8 | — |
| [O8.9] | Key record tag `t=y` — testing mode | present — D8 | — |
| [O8.10] | Key record tag `t=s` — strict subdomaining | present — D8 | — |

### O9 · DKIM: Key Lookup and Verification — covered in D9

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O9.1] | Section citation [RFC 6376 §6] | present — D9 | — |
| [O9.2] | Step 1: locate `DKIM-Signature` header(s) | present — D9 | — |
| [O9.3] | Step 2: for each signature, extract `d=` and `s=` | present — D9 | — |
| [O9.4] | Step 3: query DNS for the TXT record at `<s>._domainkey.<d>` | present — D9 | — |
| [O9.5] | Step 4: retrieve the public key from the `p=` tag | present — D9 | — |
| [O9.6] | Step 5: recompute the body hash using `a=` and the body part of `c=`; compare to `bh=` | present — D9 | — |
| [O9.7] | Step 6: reconstruct the canonicalized headers in `h=`; verify `b=` against them and the body hash | present — D9 | — |
| [O9.8] | Step 7: signature verifies → DKIM pass for domain `d=` | present — D9 | — |
| [O9.9] | Failing signature ≠ fraud (modification in transit); missing signature = no DKIM result | present — D9 | — |
| [O9.10] | Oversigning — signing an absent header name prevents intermediaries inserting it [RFC 6376 §8.15] | present — D9 | — |

### O10 · DKIM Alignment — covered in D13

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O10.1] | Section citation [RFC 7489 §3.1.2] | present — D13 | — |
| [O10.2] | DMARC DKIM alignment compares the `d=` tag of a passing signature with the RFC5322.From domain | present — D13 | — |
| [O10.3] | Relaxed alignment (default): organizational domain of `d=` equals organizational domain of From; `d=mail.example.com` + `user@example.com` → aligned | present — D13 | — |
| [O10.4] | Strict alignment: exact domain equality; the same pair → not aligned | present — D13 | — |
| [O10.5] | Multiple signatures: DMARC passes via DKIM if any one passing signature is aligned [RFC 7489 §4.1] | present — D13 | — |

### O11 · DMARC: Policy Options — covered in D11

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O11.1] | Standard: RFC 7489 | present — D11 | — |
| [O11.2] | Location: TXT record at `_dmarc.<domain>`; example lookup `TXT _dmarc.example.com` | present — D11 | — |
| [O11.3] | Prefix: `v=DMARC1` | present — D11 | — |
| [O11.4] | `p=` — policy applied to the organizational domain of the RFC5322.From [RFC 7489 §6.3] | present — D11 | — |
| [O11.5] | `p=none` — reporting only; initial deployment | present — D11 | — |
| [O11.6] | `p=quarantine` — deliver to spam/junk or flag for review | present — D11 | — |
| [O11.7] | `p=reject` — reject at the SMTP level (or discard after acceptance) | present — D11 | — |
| [O11.8] | `sp=` — subdomain policy; same values as `p=` [RFC 7489 §6.3] | present — D11 | — |
| [O11.9] | `pct=` — integer 0–100; fraction to which `p=` applies [RFC 7489 §6.3] | present — D11 | — |
| [O11.10] | Alignment mode tags `adkim=r`/`s` (default `r`) and `aspf=r`/`s` (default `r`) | present — D11 | — |
| [O11.11] | `rua=` — comma-separated URIs for aggregate report delivery | present — D11 | — |
| [O11.12] | `ruf=` — failure (forensic) report delivery; widely unsupported by receivers | present — D11 | — |
| [O11.13] | `fo=` — failure reporting options `0`, `1`, `d`, `s` | present — D11 | — |
| [O11.14] | Example record `v=DMARC1; p=quarantine; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@example.com` | present — D11 | — |

### O12 · DMARC: Identifier Alignment (Strict vs. Relaxed) — covered in D10

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O12.1] | Section citation [RFC 7489 §3.1] | missing — D10 carries no citation | See F2 |
| [O12.2] | Alignment ties SPF and DKIM results to the RFC5322.From domain visible to users | present — D10 | — |
| [O12.3] | Relaxed alignment: subdomains allowed; organizational-domain comparison uses the Public Suffix List; default mode | present — D10 | — |
| [O12.4] | Strict alignment: exact match required; no subdomain tolerance | present — D10 | — |
| [O12.5] | Implications: relaxed accommodates bounce addresses at a subdomain; strict requires full control of each identity field | present — D10 | — |

### O13 · DMARC: Aggregate Reporting — covered in D15

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O13.1] | Section citation [RFC 7489 §7.2] | present — D15 | — |
| [O13.2] | Receivers send aggregate reports to the address(es) in `rua=` | present — D15 | — |
| [O13.3] | Reports summarize authentication results over a time interval (typically 24 hours) | present — D15 | — |
| [O13.4] | Format: XML, described by the DMARC XML schema [RFC 7489 Appendix C] | present — D15 | — |
| [O13.5] | Contents: policy applied at time of delivery; per-source-IP rows (source IP, message count, disposition, SPF result + aligned domain, DKIM result + aligned domain, override reasons) | present — D15 | — |
| [O13.6] | Cross-domain reporting authorization record `<policy-domain>._report._dmarc.<reporting-domain>` with value `v=DMARC1` [RFC 7489 §7.1] | present — D15 | — |
| [O13.7] | Forensic (`ruf=`) reports: per-message; ARF (RFC 5965); rarely generated by major receivers due to privacy | present — D15 | — |

### O14 · How SPF, DKIM, and DMARC Are Evaluated Together — covered at overview depth in D4, in full in D14

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O14.1] | Section citation [RFC 7489 §4, §5] | present — D14 | — |
| [O14.2] | Step 1: SPF evaluation — RFC5321.MailFrom (or HELO if empty); seven results [RFC 7208 §4] | present — D14 | — |
| [O14.3] | Step 2: DKIM evaluation — verify each `DKIM-Signature`; independent pass or fail [RFC 6376 §6] | present — D14 | — |
| [O14.4] | Step 3: DMARC record lookup — `_dmarc.<from-domain>` TXT; no record → processing stops [RFC 7489 §5.6] | present — D14 | — |
| [O14.5] | Step 4: SPF alignment check — if SPF passed, compare under `aspf=` [RFC 7489 §4.1] | present — D14 | — |
| [O14.6] | Step 5: DKIM alignment check — each DKIM pass under `adkim=` [RFC 7489 §4.1] | present — D14 | — |
| [O14.7] | Step 6: disposition — either path passes with alignment → pass, policy not applied; neither → fail, `p=` applied (subject to `pct=`) | present — D14 | — |
| [O14.8] | Step 7: reporting — results aggregated for the next report to `rua=` | present — D14 | — |

### O15 · Email Forwarding and Intermediary Effects — covered in D16

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O15.1] | SPF and forwarding: intermediary sends from its own IP, not authorized; SPF fails at the final destination [RFC 7208 §11.4] | present — D16 | — |
| [O15.2] | Forwarders rewriting MAIL FROM to their own domain (bounce address): passes SPF for their domain, loses the original domain's authentication | present — D16 | — |
| [O15.3] | DKIM and forwarding: signatures survive unmodified relay; DKIM is the more robust mechanism | present — D16 | — |
| [O15.4] | DKIM fails if the body is re-encoded/reformatted, signed headers modified, or oversigning-covered headers inserted | present — D16 | — |
| [O15.5] | Mailing lists modify messages: `[List]` prefixes, footers, `List-*` headers, `From:` rewritten | present — D16 | — |
| [O15.6] | Subject modification breaks DKIM if `subject` appears in `h=` | present — D16 | — |
| [O15.7] | Body modification breaks DKIM regardless of canonicalization | present — D16 | — |
| [O15.8] | `From:` rewriting changes the alignment domain entirely | present — D16 | — |
| [O15.9] | ARC (RFC 8617): records authentication state; adds `ARC-Seal`, `ARC-Message-Signature`, `ARC-Authentication-Results` | present — D16 | — |
| [O15.10] | ARC lets a final receiver know an upstream hop validated SPF/DKIM | present — D16 | — |
| [O15.11] | ARC is a trust mechanism, not an authentication mechanism; receivers may choose whether to honor it | present — D16 | — |
| [O15.12] | SRS: rewrites MailFrom to a forwarder-controlled domain; SPF passes at the next hop; does not address DKIM or DMARC alignment | present — D16 | — |

### O16 · Common Failure Modes — covered in D17

SPF (O16.1–O16.6), DKIM (O16.7–O16.13), DMARC (O16.14–O16.21).

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O16.1] | SPF: unlisted sending IP — new ESP/CRM/marketing platform not in the record; Fail or SoftFail | present — D17 | — |
| [O16.2] | SPF: DNS lookup limit exceeded — PermError, treated as Fail by many receivers [RFC 7208 §4.6.4] | present — D17 | — |
| [O16.3] | SPF: void lookup limit exceeded — more than 2 zero-record mechanisms; PermError [RFC 7208 §4.6.4] | present — D17 | — |
| [O16.4] | SPF: SoftFail instead of Fail — `~all` provides no enforcement | present — D17 | — |
| [O16.5] | SPF: split-brain SPF — different records at different DNS authorities | present — D17 | — |
| [O16.6] | SPF: missing MAIL FROM handling — no rule covers automated bounce paths or subdomain senders | present — D17 | — |
| [O16.7] | DKIM: key rotation without overlap — in-flight mail signed with the removed key fails | present — D17 | — |
| [O16.8] | DKIM: body modification by intermediary — invalidates the body hash | present — D17 | — |
| [O16.9] | DKIM: signed headers modified — e.g. `Subject` rewrite by a mailing list | present — D17 | — |
| [O16.10] | DKIM: unsigned From header — From replaceable without breaking the signature | present — D17 | — |
| [O16.11] | DKIM: weak key size — RFC 8301 minimum 1024-bit RSA; best practice 2048 | present — D17 | — |
| [O16.12] | DKIM: key revocation gap — empty `p=` revokes immediately; in-flight mail fails | present — D17 | — |
| [O16.13] | DKIM: missing or incorrect key TXT record — typos, wrong name, truncation | present — D17 | — |
| [O16.14] | DMARC: SPF alignment fails on forwarding — forwarder IP; SPF fails or aligns to the forwarder's domain | present — D17 | — |
| [O16.15] | DMARC: DKIM breaks in mailing lists — both paths fail; DMARC fails for legitimately originated mail | present — D17 | — |
| [O16.16] | DMARC: policy at `none` indefinitely — reports generated but no action | present — D17 | — |
| [O16.17] | DMARC: no `rua=` set — no aggregate reports; no visibility | present — D17 | — |
| [O16.18] | DMARC: subdomain policy not considered — explicit `sp=` prevents policy gaps [RFC 7489 §6.3] | present — D17 | — |
| [O16.19] | DMARC: cross-domain `rua=` without authorization record — reports may not be accepted [RFC 7489 §7.1] | present — D17 | — |
| [O16.20] | DMARC: organizational domain determination error — Public Suffix List implementations disagree | present — D17 | — |
| [O16.21] | DMARC: `pct=` below 100 — retryable until landing in the unenforceable percentage | present — D17 | — |

### O17 · Excluded — items the outline deliberately omits

The compliant draft state for these items is absence. `missing` here means correctly absent; `present` means re-introduced against the outline's exclusion.

| ID | Outline item | Status | Suggested action |
| --- | --- | --- | --- |
| [O17.1] | "RFC 7208 (obsoletes RFC 4408)" — standards-lineage note | missing — correctly absent | — |
| [O17.2] | "RFC 6376 (obsoletes RFC 4871)" — standards-lineage note | missing — correctly absent | — |
| [O17.3] | "SPF must both pass and align …" — duplicate of the disposition rule (kept once in evaluation step 6) | missing — correctly absent | — |
| [O17.4] | "A message satisfies DMARC … on at least one path …" — duplicate of the disposition rule (kept once in step 6) | missing — correctly absent | — |
| [O17.5] | "Both paths need not pass; one is sufficient." — duplicate of the disposition rule | present — restated by D14's closing sentence | See F3 |
| [O17.6] | "Order matters … a single aligned passing result is sufficient." — restatement of step 6 | present — restated by D14's closing sentence | See F4 |

## Draft Sections: Trace to Learner-Journey Entries

Every draft section and whether it traces to a learner-journey entry.

| Draft section | Position | Journey entry | Traces? |
| --- | --- | --- | --- |
| 1 · The Problem Email Authentication Addresses | 1 | 1 | yes |
| 2 · SMTP Identities Relevant to Authentication | 2 | 2 | yes |
| 3 · DNS as Part of the Authentication Model | 3 | 3 | yes |
| 4 · How the Mechanisms Layer Together (Overview) | 4 | 4 | yes |
| 5 · SPF: Record Syntax | 5 | 5 | yes |
| 6 · SPF: Evaluation Algorithm | 6 | 6 | yes |
| 7 · DKIM: Signature Structure | 7 | 7 | yes |
| 8 · DKIM: Selectors | 8 | 8 | yes |
| 9 · DKIM: Key Lookup and Verification | 9 | 9 | yes |
| 10 · DMARC: Identifier Alignment (Strict vs. Relaxed) | 10 | 10 | yes |
| 11 · DMARC: Policy Options | 11 | 11 | yes |
| 12 · SPF Alignment | 12 | 12 | yes |
| 13 · DKIM Alignment | 13 | 13 | yes |
| 14 · How SPF, DKIM, and DMARC Are Evaluated Together | 14 | 14 | yes |
| 15 · DMARC: Aggregate Reporting | 15 | 15 | yes |
| 16 · Email Forwarding and Intermediary Effects | 16 | 16 | yes |
| 17 · Common Failure Modes | 17 | 17 | yes |

The draft's preamble (title and "First-pass draft …" note) is not a section and has no journey entry; that is expected.

## Draft Ordering vs. the Learner Journey

Actual position of each draft section against its expected position in the journey sequence.

| Draft section | Actual position | Expected position (journey entry) | Conflict? |
| --- | --- | --- | --- |
| 1 · The Problem Email Authentication Addresses | 1 | 1 | none |
| 2 · SMTP Identities Relevant to Authentication | 2 | 2 | none |
| 3 · DNS as Part of the Authentication Model | 3 | 3 | none |
| 4 · How the Mechanisms Layer Together (Overview) | 4 | 4 | none |
| 5 · SPF: Record Syntax | 5 | 5 | none |
| 6 · SPF: Evaluation Algorithm | 6 | 6 | none |
| 7 · DKIM: Signature Structure | 7 | 7 | none |
| 8 · DKIM: Selectors | 8 | 8 | none |
| 9 · DKIM: Key Lookup and Verification | 9 | 9 | none |
| 10 · DMARC: Identifier Alignment (Strict vs. Relaxed) | 10 | 10 | none |
| 11 · DMARC: Policy Options | 11 | 11 | none |
| 12 · SPF Alignment | 12 | 12 | none |
| 13 · DKIM Alignment | 13 | 13 | none |
| 14 · How SPF, DKIM, and DMARC Are Evaluated Together | 14 | 14 | none |
| 15 · DMARC: Aggregate Reporting | 15 | 15 | none |
| 16 · Email Forwarding and Intermediary Effects | 16 | 16 | none |
| 17 · Common Failure Modes | 17 | 17 | none |

The draft order departs from the outline's section order wherever the journey mandates it (journey ordering decisions 2, 3, and 4). These are not conflicts — the journey is the sequence authority and the draft matches it:

| Section | Draft position | Outline position | Journey entry | Ordering decision |
| --- | --- | --- | --- | --- |
| How the Mechanisms Layer Together (Overview) | 4 | outline has no overview section (overview depth of "Evaluated Together", 14th) | 4 | Decision 1 — overview before mechanism detail |
| DMARC: Identifier Alignment (Strict vs. Relaxed) | 10 | 12th | 10 | Decision 3 — alignment before the policy record |
| DMARC: Policy Options | 11 | 11th | 11 | Decision 3 — policy record after the alignment concept |
| SPF Alignment | 12 | 6th | 12 | Decision 2 — alignment deferred until DMARC exists |
| DKIM Alignment | 13 | 10th | 13 | Decision 2 — as above |
| DMARC: Aggregate Reporting | 15 | 13th | 15 | Decision 4 — reporting after the combined evaluation |

## Cross-Reference and Prerequisite Checks

The journey uses "Expected understanding" to guarantee later entries never assume more than earlier ones taught. Spot checks against the draft:

- D4 promises the full mechanics in section 14; D14 delivers them — the split the journey's entry 4 designs.
- D11's alignment-mode tags say "the strict/relaxed modes of section 10" — correct under the journey order (10 before 11); the draft updated the reference to match the reversal, rather than leaving the outline's forward order.
- D14 step 7 names `rua=` after D11 defines it; journey entry 14's prerequisite (entry 11) holds.
- D12 and D13 use only the organizational-domain concept from D10 and the result vocabulary from D6/D9; journey prerequisites for entries 12 and 13 hold.
- D16 frames every effect as "which step of section 14 breaks"; journey entry 16's prerequisite (entry 14) holds.
- D17's SPF/DKIM/DMARC groupings map onto journey entries 6/9/11, and its forwarding-caused failures assume D16, matching journey entry 17's prerequisites.
- No draft section assumes a concept its journey entry lists as not yet taught.

## Findings

Discrete, addressable items for the revision stage.

| ID | Outline item or draft section | Status | Suggested action |
| --- | --- | --- | --- |
| [F1] | O8.1 — "DKIM: Selectors" section citation [RFC 6376 §3.1, §3.5] | missing | Add the citation line to draft section 8 (DKIM: Selectors); every other outline section's citation is present in the draft. |
| [F2] | O12.1 — "DMARC: Identifier Alignment (Strict vs. Relaxed)" section citation [RFC 7489 §3.1] | missing | Add the citation line to draft section 10 (DMARC: Identifier Alignment). |
| [F3] | O17.5 — excluded item "Both paths need not pass; one is sufficient." | present (non-compliant — outline excludes it as a duplicate of the disposition rule) | Remove the closing sentence of draft section 14 ("One aligned passing result is sufficient; SPF and DKIM need not both pass."). Step 6 already states the disposition rule the outline kept. |
| [F4] | O17.6 — excluded item "Order matters … a single aligned passing result is sufficient." | present (non-compliant — outline excludes it as a restatement of step 6) | Same edit as F3; the one sentence restates both excluded items. |

## Presentation Notes (Not Findings)

Differences in layout that do not change coverage:

- O4.3 (empty MAIL FROM → HELO/EHLO) is covered inside D5's Location bullet rather than as a separate bullet.
- O7.12 (`t=` / `x=`) is split into two bullets in D7.
- D1, D2, D4, D7, and D12 carry sentences beyond the outline that the journey's "Expected understanding" fields call for (DMARC as decision layer, roles of `d=`/`h=`/`s=`, canonicalization as the cause of verification breaks, the bounce-domain payoff). Checked against the journey; intentional, not findings.
