# TLS Handshake & Secure Channels

TLS (Transport Layer Security) is the protocol that gives you an encrypted, authenticated channel between two machines over an untrusted network. It assembles the asymmetric-crypto primitives (ECDH for key agreement, RSA/ECDSA for identity) plus symmetric AES and MACs into one system. This note is the protocol layer; the primitives are in `Security/Asymmetric Cryptography - RSA, DH & Elliptic Curves.md`.

## What TLS actually is

TLS is a **protocol** with two phases:

1. **Handshake** — runs once at connection start. Agree a shared secret (ECDH), verify identity (certificate + signature), confirm integrity (Finished MAC).
2. **Record protocol** — runs continuously after. Application data (HTTP requests/responses) is chopped into "records," each encrypted with the AES session key and tagged with a MAC.

So "TLS" is the umbrella; the handshake is the setup ceremony, the record protocol is the ongoing encrypted conversation.

### Where it sits (the name is literal)

```
HTTP     ← application data
  │
TLS      ← encrypts / authenticates it   ← this layer
  │
TCP      ← reliable delivery
  │
IP       ← routing
```

`HTTPS` = `HTTP + TLS` — plain HTTP running through a TLS channel instead of raw TCP. Same pattern gives IMAPS etc.

### SSL vs TLS

- **SSL (Secure Sockets Layer)** is TLS's deprecated predecessor. Netscape, 1990s, versions 1.0/2.0/3.0 — all broken.
- Standardised and renamed **TLS** from 1999 (TLS 1.0 ≈ SSL 3.1). Current: **TLS 1.2** (2008) and **TLS 1.3** (2018).
- The name "SSL" persists culturally ("SSL certificate" is really a TLS/X.509 cert; `OpenSSL` implements modern TLS). Like calling every vacuum a "Hoover." When someone says SSL today they almost always mean TLS.

## The TLS 1.3 handshake (step by step)

```
Client (browser)                                      Server (bank.com)
  │  generates ephemeral ECDH key pair                  has long-term cert + ECDH key
  │
  │ ─────── 1. ClientHello ──────────────────────────▶ │
  │          client_pub + cipher suites                 │
  │                                                      │
  │ ◀────── 2. ServerHello ──────────────────────────── │
  │          server_pub + chosen cipher                  │
  │                                                      │
  │  3. BOTH derive the same shared secret independently │
  │     client: S = ECDH(client_priv, server_pub)        │
  │     server: S = ECDH(server_priv, client_pub)        │
  │     S → HKDF → separate AES keys per direction       │
  │                                                      │
  │  ═══ everything below is encrypted (AES) ═══         │
  │                                                      │
  │ ◀────── 4. Certificate ───────────────────────────  │
  │          server identity, signed by a CA             │
  │ ◀────── 5. CertificateVerify ───────────────────── │
  │          RSA/ECDSA signature over the transcript     │
  │ ◀────── 6. Finished (server) ─────────────────────  │
  │          MAC over whole handshake                    │
  │ ─────── 6. Finished (client) ────────────────────▶ │
  │                                                      │
  │  ═══ 1 round trip — handshake done ═══               │
  │ ─────── 7. GET /account (AES-encrypted) ─────────▶  │
  │ ◀────── 7. HTTP response (AES-encrypted) ─────────  │
```

- **Steps 1–2**: both generate a *fresh* ECDH key pair (typically X25519) just for this connection, exchange public points in the clear, agree a cipher suite (e.g. `TLS_AES_256_GCM_SHA384`).
- **Step 3**: the ECDH computation — each multiplies own private scalar by the other's public value → same shared secret → HKDF stretches it into AES keys (one per direction).
- **Steps 4–5**: server sends its certificate (long-term public key, signed by a CA) plus a **fresh signature over the entire handshake transcript so far**. This is the authenticity check.
- **Step 6**: Finished MAC over the full transcript, both ways — confirms nothing was tampered with and both sides derived matching keys.
- **Step 7**: application data flows, AES-encrypted.

TLS 1.3 does this in **1 round trip** (vs TLS 1.2's 2).

### See it live

```bash
openssl s_client -connect google.com:443 -tls1_3 2>&1 | head -30
# Protocol : TLSv1.3
# Cipher   : TLS_AES_256_GCM_SHA384    ← symmetric cipher for data
# Server Temp Key: X25519, 253 bits    ← ephemeral ECDH key ("Temp" = forward secrecy)
```

## Certificate Authority (CA) — the chain of trust

**Problem**: when bank.com sends its public key, how do you know it's really bank.com's and not an attacker's? You've never met bank.com. You need someone you already trust to vouch.

**A CA is a trusted third party (DigiCert, Let's Encrypt) that vouches for the link between a domain and a public key.** It verifies the requester controls bank.com, then *signs* a certificate saying "this public key belongs to bank.com" (same digital-signature concept — private signs, public verifies).

Your browser/OS ships with a built-in list of trusted CA root keys. So:

```
CA's trusted root key (built into your browser)
   └─ signs bank.com's certificate
        └─ certificate contains bank.com's public key
             └─ that public key signs the ECDH exchange (CertificateVerify)
```

Short version: **CA = trusted notary that pre-vouches for which public key belongs to which domain**, so the browser doesn't trust the server blindly.

## MAC — Message Authentication Code

A short tag proving two things at once: the message wasn't tampered with, **and** it came from someone who knows the shared secret key.

```
MAC = hash(secret_key + message)
```

Sender computes the tag, sends it alongside the message. Receiver (who has the same key) recomputes from the received message and checks it matches. One bit changed → mismatch → tampering detected. An attacker can't forge a valid tag because they lack the key.

### MAC vs digital signature

| | Digital signature | MAC |
|---|---|---|
| Key type | Asymmetric (private signs, public verifies) | Symmetric (same secret both sides) |
| Who can verify | Anyone with the public key | Only someone with the shared key |
| Proves identity to third parties | Yes | No (both sides share the key, so neither can prove *which* made it) |

Signature = "only I could have made this, anyone can check." MAC = "one of us two made this, only we two can check." TLS uses both: signatures (ECDSA/RSA) for the one-time identity proof, MACs for fast per-message integrity once the shared key exists (symmetric = much cheaper).

## Attack 1 — Downgrade, and why Finished covers the transcript

The ClientHello/ServerHello (steps 1–2) are sent in **plaintext, before** the certificate is verified. That ordering is a vulnerability window.

```
1. Browser ClientHello: "I support AES-256, AES-128, and (legacy) weak cipher"
2. Attacker on wire EDITS it to: "I support only the weak cipher"
3. Server picks the weak cipher, proceeds
4. Certificate verification STILL PASSES — server really is bank.com;
   the cert check says nothing about which cipher was chosen
5. Session uses a weak cipher the attacker can break
```

The attacker impersonated no one — they quietly weakened the negotiation, and the identity check has no visibility into that.

**Fix**: the Finished MAC is computed over the **entire handshake transcript** — every byte, including the original ClientHello.

```
Browser's transcript: [ClientHello as SENT]     + ...
Server's transcript:  [ClientHello as RECEIVED] + ...
```

If the attacker edited it in transit, the two transcripts differ → the two Finished MACs differ → connection aborts. The attacker can't forge a correct MAC (no shared key — it came from ECDH, never transmitted).

<div class="info">The split: <strong>CA/certificate verify</strong> answers "is this really bank.com?" (authenticates the endpoint). The <strong>Finished MAC</strong> answers "did both sides see the exact same handshake, byte for byte?" (authenticates the conversation). You need both — identity and integrity are separate guarantees.</div>

## Attack 2 — MITM with the attacker's own keys, and why it fails

The natural attack: attacker M runs *two* ECDH exchanges — one with the client (secret 1, M's keys), one with the server (secret 2). M decrypts from one side, re-encrypts to the other, relaying transparently. Then M just forwards the server's certificate to the client.

Forwarding the certificate is fine — **it's public, anyone can copy it.** The attack dies at **CertificateVerify**, because that signature covers the transcript, which includes the server's ECDH public key:

```
Server signs a transcript containing:  server's ECDH key (secret 2)
Client verifies against its transcript: M's ECDH key      (secret 1)  ← what client actually saw
                                        ↑ mismatch → signature fails
```

M is stuck choosing between two losing options:

```
Option A: relay server's real ECDH key unchanged
          → but M doesn't know that private key → can't decrypt → M is just a dumb wire
Option B: substitute M's own ECDH key so M CAN decrypt
          → server's signature no longer covers M's key → client verification fails
```

To repair the mismatch, M would need to forge a signature over its *own* ECDH key that still verifies against bank.com's certificate → requires bank.com's **certificate private key**, which never leaves the server. Copying the public cert doesn't grant the ability to sign with it.

<div class="good">This is why CertificateVerify signs the transcript rather than presenting a static cert. A static "here is bank.com's certificate" WOULD be forwardable — the transcript signature closes that hole by cryptographically stapling the verified identity to the specific key-exchange the client actually saw. Identity is bound to <em>this session</em>, not a reusable stamp.</div>

## Forward secrecy

Both ECDH keys are **ephemeral** — generated fresh per connection, discarded after. Even if bank.com's long-term certificate private key is stolen a year later, past sessions can't be decrypted: their session keys came from private scalars that existed only in memory for one handshake, then vanished. (TLS 1.2's RSA key-exchange mode lacked this — the same RSA key decrypted every session, so one theft = all past sessions compromised. TLS 1.3 removed it.)

## Connections

- **`Networking/What happens when you visit a website.md`** — already has a TLS handshake section in the URL-to-page journey; this note is the deep dive on the crypto.
- **`Security/Asymmetric Cryptography - RSA, DH & Elliptic Curves.md`** — the primitives (ECDH, RSA/ECDSA, hybrid encryption) that TLS assembles.
- **Auth work (bearer tokens / IDOR)** — same identity-vs-integrity split one level up: a token proves *who* the caller is, but the proof must be bound to *this specific request/session*, or a relay can lift the credential and replay it elsewhere. TLS provides the confidentiality underneath so tokens can't be intercepted in transit in the first place.
