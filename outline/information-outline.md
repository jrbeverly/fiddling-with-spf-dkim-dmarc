# Information Outline: SPF, DKIM, and DMARC

---

## The Problem Email Authentication Addresses

- SMTP (RFC 5321) does not authenticate the claimed sender; any MTA can issue `MAIL FROM:<anything@example.com>` without proving control of the domain.
- The RFC 5322 `From:` header shown to users is also unverified and can differ from the envelope sender.
- Together these enable domain spoofing: delivering mail that appears to come from a trusted domain while originating from an attacker-controlled server.
- Common attacks enabled: phishing via brand impersonation, business email compromise (BEC), spam delivery hiding its real origin.
- SPF, DKIM, and DMARC are layered mechanisms that together let a receiving server verify claims about message origin and act on verification failures.

---

## SMTP Identities Relevant to Authentication

Three distinct domain identities appear in a mail transaction; they can all differ.

- **RFC5321.MailFrom (envelope sender)** — domain in the `MAIL FROM:` command; used by the SMTP infrastructure for bounce delivery (DSNs); not normally visible to end users; the identity SPF checks. [RFC 7208 §2]
- **RFC5322.From (header From)** — `From:` header field in the message body; displayed to the user; not checked directly by SPF or DKIM; used by DMARC for alignment. [RFC 7489 §3.1]
  - Multiple `From:` addresses possible (rare); DMARC evaluates against the first organizational domain found. [RFC 7489 §5.6.1]
- **DKIM signing domain (`d=`)** — domain that generated the DKIM signature; appears in the `DKIM-Signature` header; independent of the other two identities (a third-party signer puts its own domain in `d=` unless it signs with the From domain); DMARC alignment checks it against RFC5322.From. [RFC 6376 §3.5, RFC 7489 §3.1.2]

---

## DNS as Part of the Authentication Model

- SPF and DKIM both store authentication data in DNS TXT records.
- DNS serves as a trusted distribution channel for policy and public keys because the domain owner controls it.
- DNSSEC can authenticate DNS responses but is not required by SPF, DKIM, or DMARC.
- A receiving server performs DNS lookups at delivery time to retrieve current policy.

---

## SPF: Record Syntax

- Standard: RFC 7208.
- **Location** — TXT record published at the RFC5321.MailFrom domain; example lookup: `TXT example.com`.
- Empty MAIL FROM (bounce message) — the HELO/EHLO domain is checked instead. [RFC 7208 §2.3]
- **Prefix** — `v=spf1`.
- **Mechanisms** (evaluated left-to-right; first match wins):
  - `all` — matches all IPs; catch-all terminator.
  - `include:<domain>` — recursively evaluates the SPF record of `<domain>`; counts toward the DNS lookup limit.
  - `a[:<domain>]` — matches if the sending IP resolves from the A/AAAA records of the domain.
  - `mx[:<domain>]` — matches if the sending IP is an MX host for the domain.
  - `ip4:<address/cidr>` — matches an IPv4 address or CIDR block.
  - `ip6:<address/cidr>` — matches an IPv6 address or CIDR prefix.
  - `ptr[:<domain>]` — reverse-DNS match; deprecated; discouraged by RFC 7208 §5.5.
  - `exists:<domain>` — matches if the domain has any A record.
- **Qualifiers** (prefixed to a mechanism):
  - `+` Pass — default if omitted; sending IP authorized.
  - `-` Fail — sending IP not authorized; reject recommended.
  - `~` SoftFail — sending IP probably not authorized; accept but mark.
  - `?` Neutral — no assertion either way.
- **Modifiers**:
  - `redirect=<domain>` — replace the current record with the named domain's SPF record; used for shared SPF policies.
  - `exp=<domain>` — provides a human-readable explanation for failures.
- **Example**:
  ```
  v=spf1 ip4:203.0.113.0/24 include:_spf.google.com ~all
  ```

---

## SPF: Evaluation Algorithm

[RFC 7208 §4]

1. Extract the RFC5321.MailFrom domain (or HELO domain if MailFrom is empty).
2. Retrieve the TXT record at that domain.
3. Record must begin with `v=spf1`; multiple matching TXT records → PermError.
4. Evaluate mechanisms left-to-right; on a match, return that qualifier's result; no match → implicit Neutral.
5. Max 10 DNS-dependent lookups per evaluation (`include`, `a`, `mx`, `ptr`, `exists`, `redirect`); exceeding → PermError. [RFC 7208 §4.6.4]
6. Max 2 void lookups (zero records returned, NXDOMAIN or no results); exceeding → PermError. [RFC 7208 §4.6.4]

- Possible results: Pass, Fail, SoftFail, Neutral, None (no record), TempError, PermError.

---

## SPF Alignment

[RFC 7489 §3.1.1]

- SPF alone does not prove alignment with the RFC5322.From domain; DMARC defines SPF alignment as RFC5321.MailFrom vs RFC5322.From.
- **Relaxed alignment** (default) — organizational domains must match; organizational domain = registered domain under a public suffix (e.g., `sub.example.com` → `example.com`). MailFrom `bounce.example.com` + From `user@example.com` → aligned.
- **Strict alignment** — exact domain equality required. MailFrom `bounce.example.com` + From `user@example.com` → not aligned.

---

## DKIM: Signature Structure

- Standard: RFC 6376.
- A DKIM signature is added as a `DKIM-Signature:` header field before transmission.
- **Required tags**:
  - `v=` — version; currently `1`.
  - `a=` — signature algorithm; `rsa-sha256` (recommended) or `rsa-sha1` (deprecated in RFC 8301).
  - `b=` — base64-encoded signature value.
  - `bh=` — base64-encoded hash of the canonicalized message body.
  - `d=` — signing domain (public key lookup; DMARC alignment).
  - `h=` — colon-separated signed header field names, in the order signed.
  - `s=` — selector.
- **Common optional tags**:
  - `c=` — canonicalization for header/body; format `<header>/<body>`; values `simple` or `relaxed`; default `simple/simple`. [RFC 6376 §3.4]
  - `i=` — agent or user identifier (AUID); often `@<d>` or a specific address; must share the `d=` domain.
  - `t=` — signature timestamp (Unix time); `x=` — signature expiration (Unix time).
  - `l=` — octets of the body covered by `bh=`; allows content to be appended without breaking the signature.
- **Canonicalization**:
  - `simple` — whitespace and line endings preserved as-is; fragile to minor reformatting.
  - `relaxed` — collapses whitespace runs, lowercases header names, strips trailing whitespace; tolerant of minor modification by intermediaries.
- **Example**:
  ```
  DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
    d=example.com; s=20230101;
    h=from:to:subject:date:message-id;
    bh=<body-hash>;
    b=<signature>
  ```

---

## DKIM: Selectors

[RFC 6376 §3.1, §3.5]

- A selector is a string identifying which public key to retrieve.
- DNS lookup path: `<selector>._domainkey.<domain>` TXT record; example: `20230101._domainkey.example.com`.
- Multiple selectors can be active simultaneously (key rotation, per-service keys).
- A signing service typically has its own selector; the domain owner publishes the corresponding public key.
- **Key record tags** (at `<selector>._domainkey.<domain>`):
  - `v=DKIM1` — version; recommended but optional.
  - `k=` — key type; `rsa` (default) or `ed25519` (RFC 8463).
  - `p=` — base64-encoded public key; empty value means the key has been revoked.
  - `t=y` — testing mode; receivers should not apply policy but still verify.
  - `t=s` — strict subdomaining; the `i=` AUID must have `d=` as its exact domain (no subdomain match).

---

## DKIM: Key Lookup and Verification

[RFC 6376 §6]

1. Locate `DKIM-Signature` header(s) in the received message.
2. For each signature, extract `d=` and `s=`.
3. Query DNS for the TXT record at `<s>._domainkey.<d>`.
4. Retrieve the public key from the `p=` tag.
5. Recompute the body hash using `a=` and the body part of `c=`; compare to `bh=`.
6. Reconstruct the canonicalized headers listed in `h=`; verify `b=` against those headers and the body hash using the public key.
7. Signature verifies → DKIM pass for domain `d=`.

- A failed signature does not always mean fraud; it may indicate modification in transit. A missing signature means no DKIM result, not a failure.
- **Oversigning** — signing a header name that is absent in the message prevents intermediaries from inserting that header. [RFC 6376 §8.15]

---

## DKIM Alignment

[RFC 7489 §3.1.2]

- DMARC DKIM alignment compares the `d=` tag of a passing DKIM signature with the RFC5322.From domain.
- **Relaxed alignment** (default) — organizational domain of `d=` must equal organizational domain of the header From. `d=mail.example.com` + From `user@example.com` → aligned.
- **Strict alignment** — exact domain equality required. `d=mail.example.com` + From `user@example.com` → not aligned.
- A message may carry multiple DKIM signatures; DMARC passes via DKIM if any one passing signature is aligned. [RFC 7489 §4.1]

---

## DMARC: Policy Options

- Standard: RFC 7489.
- **Location** — TXT record at `_dmarc.<domain>`; example lookup: `TXT _dmarc.example.com`.
- **Prefix** — `v=DMARC1`.
- **`p=`** — policy applied to the organizational domain of the RFC5322.From. [RFC 7489 §6.3]
  - `none` — no action on failing messages; reporting only; used during initial deployment to observe without consequences.
  - `quarantine` — deliver to spam/junk or otherwise flag for review.
  - `reject` — reject at the SMTP level (or discard after acceptance).
- **`sp=`** — subdomain policy; overrides `p=` for subdomains of the From domain without their own DMARC record; same values as `p=`. [RFC 7489 §6.3]
- **`pct=`** — integer 0–100; fraction of messages to which `p=` applies; the rest receive `none` treatment; used for gradual rollout. [RFC 7489 §6.3]
- **Alignment mode tags** — `adkim=r` (default) or `s` for DKIM; `aspf=r` (default) or `s` for SPF.
- **Reporting tags**:
  - `rua=` — comma-separated URIs for aggregate report delivery (e.g., `mailto:dmarc-reports@example.com`).
  - `ruf=` — comma-separated URIs for failure (forensic) report delivery; widely unsupported by receivers.
  - `fo=` — failure reporting options: `0` (default; report if both SPF and DKIM fail), `1` (either fails), `d` (DKIM failure), `s` (SPF failure).
- **Example**:
  ```
  v=DMARC1; p=quarantine; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@example.com
  ```

---

## DMARC: Identifier Alignment (Strict vs. Relaxed)

[RFC 7489 §3.1]

- DMARC alignment ties SPF and DKIM results to the RFC5322.From domain visible to users.
- **Relaxed alignment** — subdomains allowed (`mail.example.com` aligns with `example.com`); organizational domain comparison uses the Public Suffix List to determine the registered domain; default mode.
- **Strict alignment** — exact match required between the checked identity domain and the RFC5322.From domain; no subdomain tolerance.
- **Implications** — relaxed SPF alignment accommodates common patterns such as bounce addresses at a subdomain (e.g., `bounce.example.com`); strict alignment requires full control over exactly which domain appears in each identity field.

---

## DMARC: Aggregate Reporting

[RFC 7489 §7.2]

- Receivers that implement DMARC reporting send aggregate reports to the address(es) in `rua=`.
- Reports summarize authentication results for a sending domain over a time interval (typically 24 hours).
- Format: XML, described by the DMARC XML schema. [RFC 7489 Appendix C]
- **Contents** — policy applied at time of delivery (the receiver's view of the DMARC record); per-source-IP rows with: source IP address, message count, disposition applied (`none`, `quarantine`, `reject`), SPF result and aligned domain, DKIM result and aligned domain, override reasons (e.g., local policy, forwarding).
- **Cross-domain reporting** — if the `rua=` address is on a different domain from the DMARC policy domain, the receiving domain must publish a DNS authorization record: `<policy-domain>._report._dmarc.<reporting-domain>` with value `v=DMARC1`. [RFC 7489 §7.1]
- **Forensic (failure) reports** (`ruf=`) — sent per-message on individual failures; format similar to ARF (RFC 5965); rarely generated by major receivers due to privacy concerns.

---

## How SPF, DKIM, and DMARC Are Evaluated Together

[RFC 7489 §4, §5]

1. **SPF evaluation** — check the RFC5321.MailFrom domain (or HELO if MailFrom is empty); result: Pass, Fail, SoftFail, Neutral, None, TempError, or PermError. [RFC 7208 §4]
2. **DKIM evaluation** — verify each `DKIM-Signature` header; each produces an independent pass or fail. [RFC 6376 §6]
3. **DMARC record lookup** — query `_dmarc.<from-domain>` TXT; no record → DMARC processing stops, no enforcement. [RFC 7489 §5.6]
4. **SPF alignment check** — if SPF passed, compare RFC5321.MailFrom with RFC5322.From under the configured mode (`aspf=`). [RFC 7489 §4.1]
5. **DKIM alignment check** — for each DKIM pass, compare the `d=` domain with RFC5322.From under the configured mode (`adkim=`). [RFC 7489 §4.1]
6. **DMARC disposition** — either path passes with alignment → message passes DMARC, policy not applied; neither path passes → message fails DMARC, receiver applies `p=` (subject to `pct=`).
7. **Reporting** — regardless of pass/fail, authentication results are aggregated for the next aggregate report to `rua=`.

---

## Email Forwarding and Intermediary Effects

- **SPF and forwarding** — an intermediary (e.g., a .forward alias or mailing list server) sends from its own IP, which the original RFC5321.MailFrom domain's SPF record does not authorize; result: SPF fails at the final destination for a legitimately forwarded message. [RFC 7208 §11.4]
- Some forwarders rewrite MAIL FROM to their own domain (bounce address): passes SPF for their domain, loses the original domain's authentication.
- **DKIM and forwarding** — signatures survive if signed headers and body content are unmodified; pure forwarding (relay) typically preserves DKIM; DKIM is the more robust mechanism for forwarded mail.
- DKIM fails if: the body is re-encoded or reformatted, signed headers are modified, or new headers covered by oversigning are inserted.
- **Mailing lists** — list servers commonly modify messages: `[List]` prefixes on `Subject:`, appended body footers, added `List-*` headers, or `From:` rewritten to the list address.
  - Subject modification breaks DKIM if `subject` appears in `h=`.
  - Body modification breaks DKIM regardless of canonicalization.
  - `From:` rewriting (common for DMARC compliance of `p=reject` domains) changes the alignment domain entirely.
- **ARC** (RFC 8617) — lets intermediaries record the authentication state at the point they received the message; adds three headers: `ARC-Seal`, `ARC-Message-Signature`, `ARC-Authentication-Results`.
  - Lets a final receiver know an upstream hop validated SPF/DKIM even if those results are no longer verifiable.
  - Trust mechanism, not an authentication mechanism; receivers may choose whether to honor it.
- **SRS** — rewrites RFC5321.MailFrom to a domain controlled by the forwarder, preserving the original address encoded in the local part; lets SPF pass at the next hop; does not address DKIM or DMARC alignment.

---

## Common Failure Modes

### SPF

- **Unlisted sending IP** — a new ESP, CRM, or marketing platform sends mail but its IP range is not in the SPF record; result: SPF Fail or SoftFail.
- **DNS lookup limit exceeded** — deeply nested `include:` chains or many mechanisms consume all 10 permitted lookups; result: PermError, which many receivers treat as Fail. [RFC 7208 §4.6.4]
- **Void lookup limit exceeded** — more than 2 DNS mechanisms returning zero records; result: PermError. [RFC 7208 §4.6.4]
- **SoftFail instead of Fail** — `~all` means receivers accept and mark rather than reject; provides no enforcement.
- **Split-brain SPF** — different SPF records at different DNS authorities during propagation.
- **Missing MAIL FROM handling** — an SPF record exists but no rule covers the MAIL FROM used by automated bounce paths or subdomain senders.

### DKIM

- **Key rotation without overlap** — removing the old selector from DNS before in-flight messages are delivered makes verification fail for messages signed with the old key.
- **Body modification by intermediary** — any post-signing change (line wrapping, footer addition, encoding changes) invalidates the body hash.
- **Signed headers modified** — modifications to headers in `h=` (e.g., `Subject` rewrite by a mailing list) break the signature.
- **Unsigned From header** — if `from` is not in `h=`, the From can be replaced without breaking the signature, defeating the signature's purpose for DMARC.
- **Weak key size** — 512- and 768-bit RSA keys are considered broken; RFC 8301 deprecates SHA-1 and requires minimum 1024-bit RSA keys; best practice is 2048 bits.
- **Key revocation gap** — setting `p=` to empty revokes the key immediately; previously signed mail in flight will fail.
- **Missing or incorrect key TXT record** — typos in the key record, publishing at the wrong name, or TXT record truncation.

### DMARC

- **SPF alignment fails on forwarding** — the forwarded message arrives from a forwarder IP; SPF fails or aligns to the forwarder's domain, not the From domain.
- **DKIM breaks in mailing lists** — both SPF and DKIM fail for the From domain; DMARC fails even if the message originated legitimately.
- **Policy at `none` indefinitely** — no enforcement; failures generate reports but receivers take no action.
- **No `rua=` set** — no aggregate reports delivered; no visibility into which sources are failing.
- **Subdomain policy not considered** — without `sp=`, subdomains of a `p=reject` domain default to `none` in some interpretations; explicit `sp=` prevents unexpected policy gaps. [RFC 7489 §6.3]
- **Cross-domain `rua=` without authorization record** — if `rua=` points to another domain, the receiving domain must publish an authorization record or reports may not be accepted. [RFC 7489 §7.1]
- **Organizational domain determination error** — different implementations of the Public Suffix List can disagree on organizational domain boundaries, causing alignment results to differ across receivers.
- **`pct=` below 100** — only a fraction of failing messages receive enforcement; attackers can retry until they land in the unenforceable percentage.

---

## Excluded

Items from `research/knowledge-corpus.md` not carried into the outline:

- "RFC 7208 (obsoletes RFC 4408)" — standards-lineage note; the outline keeps the current standard reference, not superseded history.
- "RFC 6376 (obsoletes RFC 4871)" — standards-lineage note; as above.
- "SPF must both **pass** and **align** for a message to satisfy DMARC via the SPF path." — duplicate of the disposition rule; kept once in "How SPF, DKIM, and DMARC Are Evaluated Together" step 6.
- "A message satisfies DMARC if it achieves an authenticated identifier alignment on at least one path: SPF pass + SPF alignment, OR DKIM pass + DKIM alignment." — duplicate of the disposition rule; kept once in evaluation step 6.
- "Both paths need not pass; one is sufficient." — duplicate of the disposition rule; kept once in evaluation step 6.
- "Order matters: DMARC does not require SPF and DKIM to pass simultaneously. A single aligned passing result is sufficient." — restatement of the disposition rule in step 6.
