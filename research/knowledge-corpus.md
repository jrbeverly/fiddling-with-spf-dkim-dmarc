# Knowledge Corpus: SPF, DKIM, and DMARC

---

## The Problem Email Authentication Addresses

- SMTP (RFC 5321) does not authenticate the claimed sender of a message.
- Any MTA can issue `MAIL FROM:<anything@example.com>` without proving control of that domain.
- The `From:` header shown to users (RFC 5322) is also unverified and can differ from the envelope sender.
- These properties enable domain spoofing: delivering mail that appears to come from a trusted domain while originating from an attacker-controlled server.
- Common attacks enabled: phishing using brand impersonation, business email compromise (BEC), spam delivery hiding real origin.
- SPF, DKIM, and DMARC are layered mechanisms that together allow a receiving server to verify claims about message origin and to act on verification failures.

---

## SMTP Identities Relevant to Authentication

Three distinct domain identities appear in a mail transaction; they can all differ:

**RFC5321.MailFrom (envelope sender)**
- The domain in the `MAIL FROM:` SMTP command.
- Used by the SMTP infrastructure for bounce delivery (DSNs).
- Not normally visible to end users.
- This is the identity checked by SPF. [RFC 7208 §2]

**RFC5322.From (header From)**
- The `From:` header field in the message body.
- Displayed to the end user as the apparent sender.
- Not checked by SPF or DKIM directly; used by DMARC for alignment. [RFC 7489 §3.1]
- A single message can have multiple `From:` addresses (rare); DMARC evaluates against the first organizational domain found. [RFC 7489 §5.6.1]

**DKIM signing domain (d= tag)**
- The domain that generated the DKIM signature.
- Appears in the `DKIM-Signature` header as the `d=` tag.
- Independent of both RFC5321.MailFrom and RFC5322.From; a message signed by a third-party signing service carries that service's domain unless it signs with the From domain.
- DMARC alignment checks this domain against RFC5322.From. [RFC 6376 §3.5, RFC 7489 §3.1.2]

---

## DNS as Part of the Authentication Model

- SPF and DKIM both store authentication data in DNS TXT records.
- DNS serves as a trusted distribution channel for policy and public keys because the domain owner controls it.
- DNSSEC can be used to authenticate DNS responses but is not required by SPF, DKIM, or DMARC.
- A receiving server performs DNS lookups at delivery time to retrieve current policy.

---

## SPF: Record Syntax

RFC reference: RFC 7208 (obsoletes RFC 4408).

**Record location**: TXT record published at the RFC5321.MailFrom domain.
- Example lookup: `TXT example.com`
- If the MAIL FROM is empty (bounce message), the HELO/EHLO domain is checked instead. [RFC 7208 §2.3]

**Record prefix**: `v=spf1`

**Mechanisms** (evaluated left-to-right; first match wins):
- `all` — matches all IPs; used as the catch-all terminator.
- `include:<domain>` — recursively evaluates the SPF record of `<domain>`; counts toward the DNS lookup limit.
- `a[:<domain>]` — matches if the sending IP resolves from the A/AAAA records of the domain.
- `mx[:<domain>]` — matches if the sending IP is an MX host for the domain.
- `ip4:<address/cidr>` — matches an IPv4 address or CIDR block.
- `ip6:<address/cidr>` — matches an IPv6 address or CIDR prefix.
- `ptr[:<domain>]` — reverse-DNS match; deprecated; RFC 7208 §5.5 discourages use.
- `exists:<domain>` — matches if the domain has any A record.

**Qualifiers** (prefixed to a mechanism):
- `+` (Pass) — default if omitted; the sending IP is authorized.
- `-` (Fail) — the sending IP is not authorized; reject recommended.
- `~` (SoftFail) — the sending IP is probably not authorized; accept but mark.
- `?` (Neutral) — no assertion either way.

**Modifiers**:
- `redirect=<domain>` — replace the current record with the named domain's SPF record; used for shared SPF policies.
- `exp=<domain>` — provides a human-readable explanation for failures.

**Example**:
```
v=spf1 ip4:203.0.113.0/24 include:_spf.google.com ~all
```

---

## SPF: Evaluation Algorithm

[RFC 7208 §4]

1. Extract the RFC5321.MailFrom domain (or HELO domain if MailFrom is empty).
2. Retrieve the TXT record at that domain.
3. Confirm the record begins with `v=spf1`; if multiple TXT records match, the result is PermError.
4. Evaluate mechanisms left-to-right:
   - For each mechanism, perform any required DNS lookups.
   - On a match, return the result determined by that mechanism's qualifier.
   - If no mechanism matches, the implicit result is Neutral.
5. DNS lookup limit: a maximum of 10 DNS-dependent lookups (include, a, mx, ptr, exists, redirect) are permitted per SPF evaluation. Exceeding this produces a PermError. [RFC 7208 §4.6.4]
6. Void lookup limit: at most 2 DNS lookups may return zero records (NXDOMAIN or no results); exceeding this produces a PermError. [RFC 7208 §4.6.4]

**Possible results**: Pass, Fail, SoftFail, Neutral, None (no record), TempError, PermError.

---

## SPF Alignment

[RFC 7489 §3.1.1]

- SPF alone does not prove alignment with the RFC5322.From domain.
- DMARC defines SPF alignment: the RFC5321.MailFrom domain must align with the RFC5322.From domain.

**Relaxed alignment** (default):
- The organizational domains of MailFrom and header From must match.
- Organizational domain: the registered domain under a public suffix (e.g., `sub.example.com` → `example.com`).
- Example: MailFrom `bounce.example.com`, From `user@example.com` → aligned (same org domain).

**Strict alignment**:
- The MailFrom domain must exactly equal the header From domain.
- Example: MailFrom `bounce.example.com`, From `user@example.com` → not aligned.

- SPF must both **pass** and **align** for a message to satisfy DMARC via the SPF path.

---

## DKIM: Signature Structure

RFC reference: RFC 6376 (obsoletes RFC 4871).

A DKIM signature is added as a `DKIM-Signature:` header field before transmission.

**Required tags**:
- `v=` — version; currently `1`.
- `a=` — signature algorithm; `rsa-sha256` (recommended) or `rsa-sha1` (deprecated in RFC 8301).
- `b=` — base64-encoded signature value.
- `bh=` — base64-encoded hash of the canonicalized message body.
- `d=` — signing domain (used for public key lookup and DMARC alignment).
- `h=` — colon-separated list of signed header field names (in the order signed).
- `s=` — selector.

**Common optional tags**:
- `c=` — canonicalization algorithm for header and body; format `<header>/<body>`; values: `simple` or `relaxed`; default `simple/simple`. [RFC 6376 §3.4]
- `i=` — agent or user identifier (AUID); often `@<d>` or a specific address; must share the domain in `d=`.
- `t=` — signature timestamp (Unix time).
- `x=` — signature expiration time (Unix time).
- `l=` — number of octets of the body covered by `bh=`; using this tag allows content to be appended without breaking the signature.

**Canonicalization** (`c=`):
- `simple`: whitespace and line endings are preserved as-is; fragile to minor reformatting.
- `relaxed`: collapses runs of whitespace, lowercases header names, strips trailing whitespace; more tolerant of minor modification by intermediaries.

**Example**:
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

- A selector is a string that identifies which public key to retrieve.
- DNS lookup path: `<selector>._domainkey.<domain>` TXT record.
- Example: `20230101._domainkey.example.com`
- Multiple selectors can be active simultaneously (key rotation, per-service keys).
- A signing service typically has its own selector; the domain owner publishes the corresponding public key.

**Key record tags** (at `<selector>._domainkey.<domain>`):
- `v=DKIM1` — version; recommended but optional.
- `k=` — key type; `rsa` (default) or `ed25519` (RFC 8463).
- `p=` — base64-encoded public key; empty value means the key has been revoked.
- `t=y` — testing mode flag; receivers should not apply policy but still verify.
- `t=s` — strict subdomaining: `i=` AUID must have `d=` as its exact domain (no subdomain match).

---

## DKIM: Key Lookup and Verification

[RFC 6376 §6]

Verification procedure:
1. Locate `DKIM-Signature` header(s) in the received message.
2. For each signature, extract `d=` and `s=`.
3. Query DNS for the TXT record at `<s>._domainkey.<d>`.
4. Retrieve the public key from the `p=` tag.
5. Recompute the body hash using the algorithm in `a=` and `c=` (body part), and compare to `bh=`.
6. Reconstruct the canonicalized headers listed in `h=`, then verify the signature in `b=` against those headers and the body hash using the public key.
7. If the signature verifies, the DKIM result is a pass for domain `d=`.

**Important**: A failed DKIM signature does not always mean the message is fraudulent; it may indicate modification in transit. A missing DKIM signature means no DKIM result, not a failure.

**Oversigning**: Signing a header name that is absent in the message prevents insertion of that header by intermediaries. [RFC 6376 §8.15]

---

## DKIM Alignment

[RFC 7489 §3.1.2]

- DMARC DKIM alignment compares the `d=` tag of a passing DKIM signature with the RFC5322.From domain.

**Relaxed alignment** (default):
- The organizational domain of `d=` must equal the organizational domain of the header From domain.
- Example: `d=mail.example.com`, From `user@example.com` → aligned.

**Strict alignment**:
- The `d=` domain must exactly equal the header From domain.
- Example: `d=mail.example.com`, From `user@example.com` → not aligned.

- A message may carry multiple DKIM signatures; DMARC passes via DKIM if any one passing signature is aligned. [RFC 7489 §4.1]

---

## DMARC: Policy Options

RFC reference: RFC 7489.

**Record location**: TXT record at `_dmarc.<domain>`.
- Example lookup: `TXT _dmarc.example.com`

**Record prefix**: `v=DMARC1`

**Policy tag** (`p=`): applied to the organizational domain of the RFC5322.From. [RFC 7489 §6.3]
- `none` — take no action on failing messages; reporting only. Used during initial deployment to observe without consequences.
- `quarantine` — deliver to spam/junk folder or otherwise flag for review.
- `reject` — reject the message at the SMTP level (or discard after acceptance).

**Subdomain policy** (`sp=`): overrides `p=` for subdomains of the From domain that do not have their own DMARC record. Same values as `p=`. [RFC 7489 §6.3]

**Percentage** (`pct=`): integer 0–100; fraction of messages to which the `p=` policy is applied; non-policy messages receive `none` treatment. Used for gradual rollout. [RFC 7489 §6.3]

**Alignment mode tags**:
- `adkim=r` (default) or `adkim=s` — relaxed or strict DKIM alignment.
- `aspf=r` (default) or `aspf=s` — relaxed or strict SPF alignment.

**Reporting tags**:
- `rua=` — comma-separated list of URIs for aggregate report delivery (e.g., `mailto:dmarc-reports@example.com`).
- `ruf=` — comma-separated list of URIs for failure (forensic) report delivery. Widely unsupported by receivers.
- `fo=` — failure reporting options; `0` (default, report if both SPF and DKIM fail), `1` (report if either fails), `d` (DKIM failure), `s` (SPF failure).

**Example**:
```
v=DMARC1; p=quarantine; adkim=r; aspf=r; pct=100; rua=mailto:dmarc@example.com
```

---

## DMARC: Identifier Alignment (Strict vs. Relaxed)

[RFC 7489 §3.1]

- DMARC alignment is the mechanism that ties SPF and DKIM results to the RFC5322.From domain visible to users.
- A message satisfies DMARC if it achieves an authenticated identifier alignment on at least one path:
  - SPF pass + SPF alignment, OR
  - DKIM pass + DKIM alignment.
- Both paths need not pass; one is sufficient.

**Relaxed alignment**:
- Allows subdomains: `mail.example.com` aligns with `example.com`.
- Organizational domain comparison uses the Public Suffix List to determine the registered domain.
- Default mode.

**Strict alignment**:
- Exact match required between the checked identity domain and the RFC5322.From domain.
- No subdomain tolerance.

**Implications**:
- Relaxed SPF alignment accommodates common patterns such as bounce addresses at a subdomain (e.g., `bounce.example.com`).
- Strict alignment requires full control over exactly which domain appears in each identity field.

---

## DMARC: Aggregate Reporting

[RFC 7489 §7.2]

- Receivers that implement DMARC reporting send aggregate reports to the address(es) in `rua=`.
- Reports summarize authentication results for a sending domain over a time interval (typically 24 hours).
- Report format: XML, described by the DMARC XML schema. [RFC 7489 Appendix C]

**Report contents**:
- Policy applied at time of delivery (the receiver's view of the DMARC record).
- Per-source-IP rows, each with:
  - Source IP address.
  - Message count.
  - Disposition applied (`none`, `quarantine`, `reject`).
  - SPF result and aligned domain.
  - DKIM result and aligned domain.
  - Override reasons (e.g., local policy, forwarding).

**Cross-domain reporting** (`rua=mailto:reports@otherdomain.com`):
- If the `rua=` address is on a different domain from the DMARC policy domain, the receiving domain must publish a DNS authorization record: `<policy-domain>._report._dmarc.<reporting-domain>` with value `v=DMARC1`. [RFC 7489 §7.1]

**Forensic (failure) reports** (`ruf=`):
- Sent per-message on individual failures; format similar to ARF (RFC 5965).
- Rarely generated by major receivers due to privacy concerns.

---

## How SPF, DKIM, and DMARC Are Evaluated Together

[RFC 7489 §4, §5]

Evaluation on a received message proceeds as follows:

1. **SPF evaluation**: The receiving MTA performs an SPF check on the RFC5321.MailFrom domain (or HELO if MailFrom is empty). Result: Pass, Fail, SoftFail, Neutral, None, TempError, or PermError. [RFC 7208 §4]

2. **DKIM evaluation**: The receiving MTA verifies each `DKIM-Signature` header. Each produces an independent pass or fail. [RFC 6376 §6]

3. **DMARC record lookup**: The receiver queries `_dmarc.<from-domain>` TXT. If no record is found, DMARC processing stops with no enforcement. [RFC 7489 §5.6]

4. **SPF alignment check**: If SPF passed, the receiver checks whether the RFC5321.MailFrom domain aligns with the RFC5322.From domain under the configured mode (`aspf=`). [RFC 7489 §4.1]

5. **DKIM alignment check**: For each DKIM pass, the receiver checks whether the `d=` domain aligns with the RFC5322.From domain under the configured mode (`adkim=`). [RFC 7489 §4.1]

6. **DMARC disposition**:
   - If any SPF pass + alignment OR any DKIM pass + alignment: message **passes DMARC**; policy is not applied.
   - If neither path passes: message **fails DMARC**; the receiver applies the policy in `p=` (subject to `pct=`).

7. **Reporting**: Regardless of pass/fail, authentication results are aggregated for inclusion in the next aggregate report sent to `rua=`.

**Order matters**: DMARC does not require SPF and DKIM to pass simultaneously. A single aligned passing result is sufficient.

---

## Email Forwarding and Intermediary Effects

**SPF and forwarding**:
- When a message is forwarded by an intermediary (e.g., a .forward alias or mailing list server), the intermediary sends the message from its own IP address.
- The original RFC5321.MailFrom domain's SPF record does not authorize the forwarder's IP.
- Result: SPF fails at the final destination for a legitimately forwarded message. [RFC 7208 §11.4]
- Some forwarders rewrite the MAIL FROM to their own domain (bounce address), which passes SPF for their domain but loses the original domain's authentication.

**DKIM and forwarding**:
- DKIM signatures survive forwarding if the signed headers and body content are not modified.
- Pure forwarding (relay) typically preserves DKIM; DKIM is the more robust mechanism for forwarded mail.
- DKIM fails if: the message body is re-encoded or reformatted, signed headers are modified, or new headers are inserted that are covered by the signature's oversigning.

**Mailing lists**:
- List servers commonly modify messages: adding `[List]` prefixes to `Subject:`, appending footers to the body, adding `List-*` headers, or changing the `From:` to the list address.
- Subject modification breaks DKIM if `subject` appears in `h=`.
- Body modification breaks DKIM regardless of canonicalization.
- `From:` rewriting (common for DMARC compliance of p=reject domains) changes the alignment domain entirely.

**ARC (Authenticated Received Chain)**:
- RFC 8617 defines ARC as a mechanism for intermediaries to record the authentication state at the point they received the message.
- ARC adds three headers: `ARC-Seal`, `ARC-Message-Signature`, and `ARC-Authentication-Results`.
- Allows a final receiver to know that an upstream hop validated SPF/DKIM even if those results are no longer verifiable.
- ARC is a trust mechanism, not an authentication mechanism; receivers may choose whether to honor it.

**SRS (Sender Rewriting Scheme)**:
- Rewrites the RFC5321.MailFrom to a domain controlled by the forwarder, preserving the original address encoded in the local part.
- Allows SPF to pass at the next hop.
- Does not address DKIM or DMARC alignment.

---

## Common Failure Modes

### SPF

- **Unlisted sending IP**: A new email service provider (ESP), CRM, or marketing platform sends mail for the domain but its IP range is not in the SPF record. Result: SPF Fail or SoftFail.
- **DNS lookup limit exceeded**: Deeply nested `include:` chains or many mechanisms consuming all 10 permitted DNS lookups. Result: PermError, which many receivers treat as Fail. [RFC 7208 §4.6.4]
- **Void lookup limit**: More than 2 DNS mechanisms returning zero records. Result: PermError. [RFC 7208 §4.6.4]
- **SoftFail instead of Fail**: Using `~all` means receivers accept and mark rather than reject; provides no enforcement.
- **Split-brain SPF**: Different SPF records at different DNS authorities during propagation.
- **Missing MAIL FROM handling**: The domain has an SPF record but no rule covers the MAIL FROM used by automated bounce paths or subdomain senders.

### DKIM

- **Key rotation without overlap**: Removing the old selector from DNS before in-flight messages are delivered causes verification to fail for messages signed with the old key.
- **Body modification by intermediary**: Any change to the message body after signing (line wrapping, footer addition, encoding changes) invalidates the body hash.
- **Signed headers modified**: Modifications to headers listed in `h=` (e.g., `Subject` rewrite by a mailing list) break the signature.
- **Unsigned From header**: If `from` is not in `h=`, the From can be replaced without breaking the signature, defeating the purpose of the signature for DMARC.
- **Weak key size**: 512-bit and 768-bit RSA keys are considered broken; RFC 8301 deprecates SHA-1 and requires minimum 1024-bit RSA keys; best practice is 2048 bits.
- **Key revocation gap**: Setting `p=` to empty revokes the key immediately; previously signed mail in flight will fail.
- **Missing or incorrect DNS TXT record**: Typos in the key record, publishing at the wrong name, or TXT record truncation.

### DMARC

- **SPF alignment fails on forwarding**: The forwarded message arrives from a forwarder IP; SPF fails or aligns to the forwarder's domain, not the From domain.
- **DKIM breaks in mailing lists**: Both SPF and DKIM fail for the From domain; DMARC fails even if the message originated legitimately.
- **Policy at `none` indefinitely**: No enforcement; failures generate reports but receivers take no action.
- **No `rua=` set**: No aggregate reports delivered; no visibility into which sources are failing.
- **Subdomain policy not considered**: Without `sp=`, subdomains of a `p=reject` domain default to `none` in some interpretations; explicit `sp=` prevents unexpected policy gaps. [RFC 7489 §6.3]
- **Cross-domain `rua=` without authorization record**: If `rua=` points to a domain other than the policy domain, the receiving domain must publish an authorization record or reports may not be accepted. [RFC 7489 §7.1]
- **Organizational domain determination error**: Different implementations of the Public Suffix List can disagree on organizational domain boundaries, causing alignment results to differ across receivers.
- **`pct=` below 100**: Only a fraction of failing messages receive enforcement; attackers can retry until they land in the unenforceable percentage.
