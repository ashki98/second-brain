# Asymmetric Cryptography — RSA, DH & Elliptic Curves

The family of crypto where you have a *mathematically linked key pair* instead of one shared secret. What one key locks, only the other unlocks. Solves the key-distribution problem: two strangers can communicate securely over a public channel without ever having met to swap a secret.

## The core idea

- **Symmetric** crypto: both parties share the same key. Fast, but how do you safely share the key in the first place? Anyone who intercepts it wins.
- **Asymmetric** crypto: a **public key** (broadcast to the world) and a **private key** (guarded). They're linked, but knowing the public key can't reveal the private key (computationally infeasible).
- **Padlock analogy**: your public key is an open padlock you hand out to everyone. Anyone can snap it shut on a box (encrypt). Only you have the key that opens it (decrypt). Handing out a thousand padlocks doesn't help anyone — they still can't open a locked one.

## The three security goals (keep these separate)

The single biggest source of confusion is collapsing these. RSA can do the first two; DH only does the third.

| Goal | Question it answers | Term |
|---|---|---|
| Confidentiality | Can I hide the message from eavesdroppers? | Encryption |
| Authenticity | Can I prove who sent it? | Digital signature |
| Key agreement | Can two strangers agree on a secret without meeting? | Key exchange |

## RSA — implementation

Security rests on a **trapdoor one-way function**: multiplying two large primes is trivial, but factoring the product back into those primes is practically impossible for big enough numbers.

```
p × q = n     ← easy (milliseconds)
n → p, q      ← hard (longer than the age of the universe for 2048-bit n)
```

### Key generation (done once)

Using tiny demo numbers (real ones are ~300-digit primes):

1. Pick two primes: `p = 61`, `q = 53`
2. Compute modulus: `n = p × q = 3233` (public; appears in both keys)
3. Compute Euler's totient: `φ(n) = (p−1)(q−1) = 60 × 52 = 3120` (needs p and q → this is why the private key is secure)
4. Pick public exponent `e`: `1 < e < φ(n)` and `gcd(e, φ(n)) = 1`. Common real-world choice: `65537`. Demo: `e = 17`
5. Compute private exponent `d = e⁻¹ mod φ(n)` (modular multiplicative inverse): `17 × d ≡ 1 (mod 3120)` → `d = 2753`

```
Public key  = (n=3233, e=17)     ← shared with everyone
Private key = (n=3233, d=2753)   ← secret forever
p, q, φ(n) are DISCARDED — never stored
```

### Encrypt / decrypt

```
Encrypt (anyone, with public key):   c = m^e mod n = 42^17 mod 3233   = 2557
Decrypt (only holder of private d):  m = c^d mod n = 2557^2753 mod 3233 = 42
```

Round-trip works because of Euler's theorem: `m^φ(n) ≡ 1 (mod n)`, and since `e × d ≡ 1 (mod φ(n))` by construction, applying `e` then `d` is equivalent to raising to `1` → original `m` back.

### Modular exponentiation trick (why it's practical)

`m^e` for large `e` would be an astronomically huge number. RSA never computes it directly — it uses **repeated squaring (square-and-multiply)**, taking `mod n` at every step so intermediate values never exceed `n²`.

```
17 in binary = 10001
m^17 = ((((m^2)^2)^2)^2) × m     ← ~5 operations, not 17
```

## RSA's two directions — the key-direction swap

Same operation (`x^key mod n`), but *which key you apply* changes the entire meaning. This is the thing that confuses people.

<div class="compare">
<div><h4>Encryption (confidentiality)</h4><pre>
Alice → Bob:
  lock with Bob's PUBLIC key
  → only Bob's PRIVATE key opens

"anyone can lock, only Bob opens"
</pre></div>
<div><h4>Signature (authenticity)</h4><pre>
Alice proves she wrote it:
  sign with Alice's PRIVATE key
  → anyone's copy of Alice's
    PUBLIC key verifies

"only Alice makes it, anyone checks"
</pre></div>
</div>

Signatures in practice sign the **hash** of the message, not the whole message: `sig = hash(m)^d mod n`. Verifier hashes the received message independently, decrypts the signature with the public key, and checks the two hashes match.

Mental model: **private key = "only I can do this"; public key = "anyone can do this."**

## Diffie-Hellman (DH) — key exchange

DH does NOT encrypt anything. It lets two parties each contribute randomness and independently arrive at the *same* secret number, which neither ever transmitted.

- **Colour-mixing analogy**: both start with a shared public colour. Each mixes in their own secret colour and exchanges the result. Each then adds their own secret again → both arrive at the same final mix. An observer who saw the exchanged mixes can't un-mix them to recover the secret.
- **The math**:

```
Public params: prime p, generator g
Alice secret a → sends A = g^a mod p
Bob   secret b → sends B = g^b mod p

Alice computes: B^a mod p = g^(ab) mod p
Bob   computes: A^b mod p = g^(ab) mod p
                            ↑ same shared secret S, never transmitted
```

- **Hard problem = discrete logarithm**: given `g^a mod p`, `g`, and `p`, find `a`. Believed infeasible for large `p`. (RSA's equivalent is factoring; DH's is discrete log.)
- **Critical weakness — no identity**: DH tells you *someone* is on the other end, not *who*. An attacker can do a DH exchange with you and you'd have no idea it wasn't the intended party. This is why DH always needs a signature layer (RSA/ECDSA) on top for authentication — see the TLS note.

## Why DH ≠ RSA

Same mathematical *family* (modular arithmetic, one-way trapdoor functions), different jobs.

| | RSA | Diffie-Hellman |
|---|---|---|
| Job | Encrypt data; sign messages | Establish a shared secret |
| Authentication | Yes (via private-key signatures) | No (needs a separate signature layer) |
| Forward secrecy | No (same key pair reused) | Yes (fresh a, b per session) |
| Hard problem | Integer factorisation | Discrete logarithm |

## Elliptic Curve (EC) — just a faster substrate

EC is **not a new idea** — it's a different mathematical surface to run the same DH/signature ideas on. The prefix tells you the algorithm; **EC = the implementation technique underneath.**

- **ECDH** = EC + Diffie-Hellman (same key agreement)
- **ECDSA** = EC + DSA (same job as RSA signatures)

Why bother: EC's trapdoor is *harder to reverse per bit*, so you get equivalent security with much smaller keys and faster math.

| Security level | RSA / classic DH | Elliptic Curve |
|---|---|---|
| ~128-bit | 3072-bit keys | 256-bit keys |

Named curves seen in the wild: **X25519** (key agreement in TLS 1.3, Signal, WireGuard, SSH), **Ed25519** (signatures). Everything modern has moved to EC variants for the size/speed win. That's all you need to know about EC — the purpose and guarantees are identical to the non-EC versions; only the underlying number-crunching differs.

## Hybrid encryption (how it's actually used)

RSA/asymmetric is never used to encrypt the real message body:
1. It's ~1000× slower than symmetric ciphers (AES).
2. The message must be shorter than `n` (a few hundred bytes max).

So real systems (TLS, PGP, SSH) use **hybrid encryption**:

```
1. Asymmetric (RSA or ECDH) → securely establish a small symmetric key (e.g. 256-bit AES key)
2. Symmetric (AES)          → encrypt the actual bulk data at full speed
```

<div class="info">This is exactly the TLS pattern: the handshake uses ECDH/RSA to agree a session key, then AES encrypts the traffic. See <code>Security/TLS Handshake & Secure Channels.md</code>.</div>

## Connections

- **JWT (`JWT.md`)**: RS256 signing is RSA in signature mode — backend signs the token with a private key, any service verifies with the public key. Exactly the "only I can make it, anyone can check" direction above.
- **Identity vs confidentiality split**: recurs everywhere — a signature proves *who*, encryption/DH provides *secrecy*. They are separate guarantees and real systems layer both.
