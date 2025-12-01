# Email Authentication with SPF, DKIM, and DMARC

Email authentication answers a simple but critical question: **is this message really from the domain it claims to be from?**

The core email protocols were designed decades ago when the internet was much more trusting. SMTP lets a sender write almost any `From` address and almost any envelope sender address, much like putting a false return address on a physical envelope. Attackers exploit this to impersonate banks, executives, government agencies, and brands in phishing campaigns.

Email authentication gives domain owners a way to say:

- which machines are allowed to send mail for a domain,
- whether a message carries cryptographic proof that the domain authorized it,
- and what receiving systems should do if authentication fails.

The three main mechanisms are **SPF**, **DKIM**, and **DMARC**. They work together, and together they are now expected by major mailbox providers such as Gmail, Microsoft, and Yahoo. Without them, legitimate email is more likely to land in spam or be rejected.

This tutorial assumes you already know DNS records such as A, MX, TXT, and CNAME. It will explain how SPF, DKIM, and DMARC work, how to read and create their records, and how they interact during real mail delivery.

---

## 1. The Problem Email Authentication Solves

Consider a message that appears in your inbox like this:

```
From: support@example-bank.com
Subject: Your account has been suspended
```

If the message is actually sent by a criminal from a hacked server, it may still show `support@example-bank.com` in the `From` header. SMTP does not verify that the `From` header matches the real sender.

Email authentication helps answer three questions:

1. **Is the sending server authorized for the envelope sender domain?**  
   That is SPF.

2. **Does the message carry a valid cryptographic signature from a domain?**  
   That is DKIM.

3. **Do those checks actually match the domain in the visible `From` header, and what should the receiver do if they do not?**  
   That is DMARC.

In short:

- SPF authenticates the **envelope sender**.
- DKIM authenticates the **message content and a signing domain**.
- DMARC authenticates the **visible `From` domain** by tying SPF and DKIM to it and publishing a policy.

---

## 2. SPF: Authorizing Sending IP Addresses

### What SPF Is For

SPF stands for **Sender Policy Framework**. It allows a domain owner to publish a list of IP addresses and hostnames that are allowed to send email using that domain in the **envelope sender address**.

The envelope sender address is the address used in the SMTP `MAIL FROM` command. It is also called the **Return-Path** or bounce address. This is usually not visible to the person reading the message, but it is the address that bounces would return to.

SPF is a path-based mechanism. It does not sign the message and does not care about the contents. It only answers:

> Did this message arrive from an IP address that the domain owner authorized?

### How an SPF TXT Record Is Constructed

SPF records are published as TXT records at the domain used in the envelope sender.

A basic SPF record looks like this:

```txt
example.com.  TXT  "v=spf1 ip4:192.0.2.10 ip4:198.51.100.0/24 include:_spf.google.com -all"
```

The record always starts with `v=spf1`, meaning “SPF version 1”.

SPF records are evaluated left to right. The main mechanisms are:

| Mechanism | Meaning |
|-----------|---------|
| `ip4:192.0.2.10` | Match the connecting IPv4 address. |
| `ip4:198.51.100.0/24` | Match any IPv4 address in the given CIDR range. |
| `ip6:2001:db8::/32` | Match an IPv6 address or range. |
| `a` | Match any IP address found in the A record of the domain. |
| `a:mail.example.com` | Match any IP address found in the A record of the given hostname. |
| `mx` | Match any IP address found in the MX records of the domain. |
| `include:_spf.example.net` | Check the SPF record of another domain, and if that record passes, treat this mechanism as a match. |
| `all` | Match everything. This is normally placed at the end as a default. |

Mechanisms can have qualifiers:

| Qualifier | Result |
|-----------|--------|
| `+` | Pass (this is the default) |
| `-` | Hard fail |
| `~` | Soft fail |
| `?` | Neutral |

For example:

- `+ip4:192.0.2.10` is the same as `ip4:192.0.2.10` and means “pass”.
- `-all` means “hard fail for anything not already matched”.
- `~all` means “soft fail for anything not already matched”.
- `?all` means “neutral for anything not already matched”.

### A Short Worked Example

Suppose `example.com` sends email only from:

- its own mail server at `192.0.2.10`,
- an office network `198.51.100.0/24`,
- and Google Workspace.

A suitable SPF record might be:

```txt
example.com.  TXT  "v=spf1 ip4:192.0.2.10 ip4:198.51.100.0/24 include:_spf.google.com -all"
```

This means:

1. If the connecting IP is `192.0.2.10`, SPF passes.
2. If the connecting IP is inside `198.51.100.0/24`, SPF passes.
3. If the connecting IP is allowed by Google’s SPF record, SPF passes.
4. Otherwise, SPF hard fails because of `-all`.

### How Receivers Evaluate SPF

When an email arrives, the receiving mail server performs these steps:

1. Look at the domain in the envelope sender address, for example `bounce@example.com`.
2. Query the DNS TXT record for `example.com`.
3. Compare the IP address of the machine that connected to the receiving server with the SPF mechanisms.
4. Stop as soon as a mechanism matches.
5. Return a result such as `pass`, `fail`, `softfail`, `neutral`, `none`, `temperror`, or `permerror`.

If no SPF record exists at all, the result is `none`. If the record has syntax or DNS lookup problems, the result may be `permerror` or `temperror`.

Important: SPF uses the **envelope sender domain**, not the `From` header domain. A malicious sender can set the envelope sender to a domain they control and still put a fake domain in the visible `From` header. SPF alone will not catch that.

### Where SPF Alone Falls Short

SPF is valuable but limited.

1. **It does not authenticate the visible `From` header.**  
   A phisher can send with an envelope sender of `evil.com` and a `From` header of `bank.com`. SPF passes for `evil.com`, but the recipient sees `bank.com`.

2. **It breaks when mail is forwarded.**  
   Suppose `alice@example.com` sends to `bob@forwarder.net`, and `forwarder.net` forwards the message to `carol@example.net`. The envelope sender may still be `alice@example.com`, but the message now arrives from the forwarder’s IP address, not from an IP authorized by `example.com`. SPF then fails.

3. **SPF has a DNS lookup limit.**  
   SPF processing is limited to 10 DNS lookups. Complex records with too many `include`, `a`, or `mx` mechanisms can cause SPF to fail with a `permerror`.

4. **SPF provides no policy of its own.**  
   It tells the receiver whether the IP was authorized, but it does not tell the receiver what to do with a failure. Different receivers handle SPF failure differently.

---

## 3. DKIM: Cryptographically Signing Messages

### What DKIM Is For

DKIM stands for **DomainKeys Identified Mail**. It gives a domain a way to digitally sign a message so that receiving systems can verify the message was genuinely authorized by the signing domain and that its contents were not changed in transit.

Unlike SPF, DKIM is not based on the sending IP address. It is a cryptographic signature attached to the message itself. This means DKIM can survive forwarding, as long as the signed parts of the message are not altered.

### Keys and Selectors: The DKIM DNS Record

DKIM uses public-key cryptography.

- A sender signs emails with a **private key**.
- The public key is published in DNS as a TXT record.
- Receivers use the public key to verify the signature.

Because a domain may use multiple mail systems or rotate keys over time, DKIM uses a **selector** to identify which key was used.

The DKIM DNS record is published at:

```txt
<selector>._domainkey.<signing-domain>  TXT  "..."
```

For example:

```txt
mail2025._domainkey.example.com.  TXT  "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA..."
```

The record contains:

- `v=DKIM1` — the version.
- `k=rsa` — the key type, usually `rsa`; `ed25519` also exists but RSA is still the most widely supported.
- `p=` — the base64-encoded public key.

Long public keys are often split across multiple quoted TXT strings. DNS automatically concatenates them.

Some providers use a CNAME instead of a TXT record, for example:

```txt
mail2025._domainkey.example.com.  CNAME  mail2025._domainkey.provider.net.
```

This lets the provider manage the actual key record.

### How DKIM Signing Works

When a mail system signs a message with DKIM, it creates a `DKIM-Signature` header and adds it to the message.

A simplified DKIM-Signature header looks like this:

```txt
DKIM-Signature: v=1; a=rsa-sha256; c=relaxed/relaxed;
 d=example.com; s=mail2025;
 h=from:to:subject:date;
 bh=47DEQpj8HBSa+/TImW+5JCeuQeRkm5NMpJWZG3hSuFU=;
 b=Tw1g5lY9aB2q7xR8zVhGk3qTcL2BdQk7uFJXZ5cD9yM6r5sPtX...
```

The important parts are:

- `d=` — the signing domain. This is the domain that claims responsibility for the message.
- `s=` — the selector. This tells the receiver which DNS key record to query.
- `h=` — the list of header fields that are signed.
- `bh=` — the hash of the body.
- `b=` — the digital signature over the listed headers.

The signer can sign a chosen set of headers, usually including `From`, `To`, `Subject`, and `Date`. It also hashes the message body.

### How Receivers Verify DKIM

When a message arrives with a `DKIM-Signature` header, the receiver:

1. Reads the signature header and extracts `d=`, `s=`, `h=`, `bh=`, and `b=`.
2. Queries DNS for `<selector>._domainkey.<d>` to retrieve the public key.
3. Uses the public key to verify the cryptographic signature `b=` over the listed headers.
4. Computes the body hash independently and compares it with `bh=`.
5. If both checks pass, DKIM returns `pass`.

If the public key is missing, the result is `permerror` or `none`. If the signature is invalid or the body hash does not match, DKIM returns `fail`.

### Where DKIM Alone Falls Short

DKIM is powerful, but it also has limitations.

1. **DKIM does not authenticate the visible `From` header by itself.**  
   A phisher can sign a message with `d=evil.com` and put `From: support@bank.com`. DKIM will pass for `evil.com`, but the visible `From` domain is still spoofed.

2. **DKIM does not specify any policy.**  
   It tells the receiver whether the signature matched. It does not say what the receiver should do if the signature fails or is missing.

3. **Mail that modifies signed content breaks DKIM.**  
   If a mailing list adds a footer to the body, changes the subject, or rewrites a signed header, the signature may no longer verify. DKIM canonicalization settings can help, but some modifications will still break the signature.

4. **Key management is required.**  
   Domains must publish and rotate keys carefully. A lost or compromised key can cause either bounced mail or forged mail.

---

## 4. DMARC: Aligning SPF and DKIM with the From Header

### What DMARC Is For

DMARC stands for **Domain-based Message Authentication, Reporting, and Conformance**.

DMARC solves the biggest remaining problem with SPF and DKIM: neither of them directly authenticates the domain that the recipient sees in the `From` header.

DMARC builds on SPF and DKIM by requiring **identifier alignment**. It says:

> If a message claims to be from `example.com`, then at least one of the following must be true:  
> - SPF passes **and** the SPF-authenticated domain aligns with `example.com`,
> - OR DKIM passes **and** the DKIM signing domain aligns with `example.com`.

If neither condition is true, the DMARC check fails, and the receiver applies the domain’s published policy.

### Identifier Alignment

DMARC defines two alignment modes: **relaxed** and **strict**.

#### SPF Alignment

SPF authentication returns a domain from the envelope sender. DMARC compares that domain with the domain in the visible `From` header.

- **Relaxed SPF alignment**: The organizational domains must match.  
  For example, envelope sender `bounce@mail.example.com` and `From: user@example.com` align because both belong to `example.com`.

- **Strict SPF alignment**: The domains must be exactly identical.  
  For example, envelope sender `bounce@mail.example.com` and `From: user@example.com` do **not** align because `mail.example.com` is not exactly `example.com`.

#### DKIM Alignment

DKIM authentication returns the signing domain from `d=` in the `DKIM-Signature` header.

- **Relaxed DKIM alignment**: Organizational domains must match.  
  For example, `d=mail.example.com` and `From: user@example.com` align.

- **Strict DKIM alignment**: The `d=` domain must exactly match the `From` domain.  
  For example, only `d=example.com` aligns with `From: user@example.com`.

By default, DMARC treats both SPF and DKIM alignment as **relaxed**. Domain owners can choose strict mode by adding `aspf=s` for SPF or `adkim=s` for DKIM.

### The DMARC Policy Record

DMARC records are TXT records published at:

```txt
_dmarc.example.com.  TXT  "..."
```

A typical record looks like:

```txt
_dmarc.example.com.  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@example.com; ruf=mailto:forensics@example.com; pct=100; adkim=relaxed; aspf=relaxed; sp=none"
```

The important tags are:

| Tag | Meaning |
|-----|---------|
| `v=DMARC1` | Protocol version. |
| `p=` | Policy for the domain: `none`, `quarantine`, or `reject`. |
| `sp=` | Policy for subdomains if they do not publish their own DMARC record. If omitted, `p=` applies. |
| `pct=` | Percentage of failing messages the policy applies to, from 1 to 100. The default is 100. |
| `rua=` | URIs for aggregate reports, usually `mailto:`. |
| `ruf=` | URIs for forensic/failure reports, usually `mailto:`. |
| `adkim=` | DKIM alignment mode, `r` for relaxed or `s` for strict. |
| `aspf=` | SPF alignment mode, `r` for relaxed or `s` for strict. |

The three main policy values are:

- `p=none`: Deliver mail normally; use DMARC only for monitoring.
- `p=quarantine`: Treat failing mail as suspicious; typically deliver to spam or quarantine.
- `p=reject`: Reject failing mail at SMTP time or discard it.

### A Short Worked Example

Suppose `example.com` publishes:

```txt
_dmarc.example.com.  TXT  "v=DMARC1; p=reject; rua=mailto:dmarc-reports@example.com; adkim=relaxed; aspf=relaxed; pct=100"
```

This means:

- The domain wants strict protection.
- If neither SPF nor DKIM passes with alignment to `example.com`, the receiver should reject the message.
- Aggregate reports should be sent to `dmarc-reports@example.com`.
- DKIM and SPF alignment are relaxed.

### How Receivers Apply DMARC

During delivery, a receiver performs SPF and DKIM checks as usual. Then it checks the `From` header domain’s DMARC record and evaluates alignment.

The DMARC result is `pass` if at least one of these is true:

1. SPF passes **and** the SPF-authenticated domain aligns with the `From` domain.
2. DKIM passes **and** the `DKIM-Signature` `d=` domain aligns with the `From` domain.

If both fail, the DMARC result is `fail`, and the receiver applies the policy from the `p=` tag.

### Reporting

DMARC also provides visibility. Domain owners can receive two main kinds of reports:

- **Aggregate reports** (`rua=`)  
  These are XML documents sent daily by participating receivers. They contain statistics about how many messages were seen from a domain, from which IP addresses, and how the SPF, DKIM, and DMARC checks performed. These reports help domain owners find legitimate senders that are failing authentication.

- **Forensic reports** (`ruf=`)  
  These may include copies or headers of individual messages that failed DMARC. They can be useful for debugging but can also expose sensitive information. Many organizations choose not to request forensic reports initially.

Aggregate reports are essential for moving from `p=none` to `p=quarantine` or `p=reject` safely.

---

## 5. How the Three Mechanisms Interact in a Real Delivery Path

Let’s walk through a legitimate message from `alice@example.com` to `bob@example.net`.

### Sending Side

The sending mail server at `example.com`:

1. Connects to the receiving server from IP address `192.0.2.10`.
2. Uses the envelope sender `MAIL FROM:<alice@example.com>`.
3. Adds a DKIM signature with `d=example.com`, `s=mail2025`.
4. The message has `From: alice@example.com`.

### Receiving Side

The receiving server for `example.net` performs several checks.

#### SPF Check

1. The server looks at the envelope sender domain: `example.com`.
2. It queries the SPF TXT record for `example.com`.
3. The SPF record is:

```txt
v=spf1 ip4:192.0.2.10 -all
```

4. The connecting IP `192.0.2.10` matches, so SPF passes for the domain `example.com`.

#### DKIM Check

1. The server sees the `DKIM-Signature` header with `d=example.com; s=mail2025`.
2. It queries `mail2025._domainkey.example.com` and retrieves the public key.
3. It verifies the signature and the body hash.
4. DKIM passes for the domain `example.com`.

#### DMARC Check

1. The server looks at the visible `From` header: `alice@example.com`.
2. It queries `_dmarc.example.com` and finds:

```txt
v=DMARC1; p=reject; aspf=relaxed; adkim=relaxed
```

3. SPF passed with domain `example.com`, which aligns with the `From` domain `example.com`.
4. DKIM also passed with domain `example.com`, which also aligns.
5. At least one aligned authentication passed, so DMARC passes.
6. The receiver delivers the message normally.

### Spoofing Attempt

Now suppose an attacker tries to spoof the same domain.

- The attacker connects from IP `203.0.113.5`.
- The envelope sender is `bounce@attacker.com`.
- The visible `From` header is `support@example.com`.
- There is no valid DKIM signature with `d=example.com`.

The receiver checks SPF:

- The envelope sender domain is `attacker.com`, not `example.com`.
- The SPF record for `attacker.com` may pass, but it authenticates `attacker.com`.

The receiver checks DKIM:

- Either there is no signature, or the signature is for `attacker.com`.

The receiver checks DMARC for the `From` domain `example.com`:

- SPF pass is not aligned with `example.com`.
- DKIM pass is not aligned with `example.com`.
- DMARC fails.

Because `example.com` published `p=reject`, the receiver rejects the spoofed message.

This is the core purpose of DMARC: it forces SPF and DKIM authentication to match the domain the recipient actually sees.

---

## 6. Common Pitfalls When Deploying SPF, DKIM, and DMARC

### SPF Pitfalls

- **Too many DNS lookups.**  
  SPF records that use many `include`, `a`, or `mx` mechanisms can exceed the 10-DNS-lookup limit and produce a `permerror`.

- **Using `+all` by accident.**  
  A record ending in `+all` or `?all` essentially lets anyone send. End with `-all` for strict protection or `~all` for a transitional soft fail.

- **Multiple SPF records.**  
  A domain should only have one SPF TXT record. Multiple records can cause a `permerror`.

- **Forgetting third-party senders.**  
  If you use a bulk email provider, helpdesk system, or CRM, their sending IPs must be included in SPF, or mail may fail.

- **Using SPF as a substitute for DKIM/DMARC.**  
  SPF alone does not protect the visible `From` header.

- **Forwarding breaks SPF.**  
  If mail is forwarded through a host that does not rewrite the envelope sender, SPF often fails.

### DKIM Pitfalls

- **Selector mismatch.**  
  The selector in the `DKIM-Signature` header must exactly match the DNS record name.

- **Key not published or not propagated.**  
  If the public key is missing or newly published, verification can fail.

- **Using weak keys.**  
  1024-bit RSA keys are increasingly discouraged. 2048-bit or stronger keys are recommended.

- **Rotating keys too aggressively.**  
  When rotating DKIM keys, keep the old selector active for a period after introducing the new one, because some messages may still be in transit.

- **Breaking signatures through message modification.**  
  Mailing lists that add footers or rewrite subjects can break DKIM. Using relaxed canonicalization can help, but some modifications still break signatures.

- **Signing too few headers.**  
  If critical headers such as `From` or `Subject` are not signed, someone might alter them without breaking DKIM.

### DMARC Pitfalls

- **Moving too fast to `p=reject`.**  
  Start with `p=none`, analyze aggregate reports, fix legitimate senders, then move to `quarantine` and later `reject`.

- **Alignment mismatch with subdomains or third-party senders.**  
  If your SPF-authenticated domain or DKIM `d=` domain differs from your `From` domain, strict alignment may fail even though the message is legitimate. Use relaxed alignment if subdomains are involved.

- **Ignoring subdomain policy.**  
  A main domain record with `p=reject` can affect subdomains if they do not publish their own DMARC records. Use `sp=` to set an appropriate subdomain policy.

- **Forgetting that DMARC requires alignment, not just SPF or DKIM pass.**  
  A message from a third-party sender must either use an envelope sender domain aligned with your `From` domain, or sign with a DKIM `d=` domain aligned with your `From` domain.

- **Aggregate reports can be large and complex.**  
  Use a DMARC reporting tool or service to parse and understand them.

- **Forensic reports can leak sensitive data.**  
  Do not enable `ruf=` casually.

- **DMARC pass can happen through SPF alone or DKIM alone.**  
  That is by design, but if your SPF fails due to forwarding yet DKIM passes, DMARC can still pass. This is why having both is valuable.

---

## 7. Putting It All Together

A strong email authentication setup typically involves:

1. **SPF** to authorize the servers that may send email for your domain.
2. **DKIM** to cryptographically sign outgoing messages and survive forwarding better than SPF.
3. **DMARC** to require alignment between SPF or DKIM and the visible `From` domain, and to publish a policy.

A practical deployment often looks like this:

- Publish SPF records for your sending domains.
- Enable DKIM signing on your mail systems and publish the corresponding public keys.
- Start DMARC at `p=none` and monitor aggregate reports.
- Identify legitimate third-party senders and ensure they pass SPF and DKIM with alignment.
- Gradually move to `p=quarantine` and then `p=reject`.

Used together, SPF, DKIM, and DMARC significantly reduce domain spoofing, improve mailbox providers’ confidence in your email, and protect both your brand and your recipients. They do not solve every email security problem, but they are now a baseline requirement for reliable, trustworthy email delivery.