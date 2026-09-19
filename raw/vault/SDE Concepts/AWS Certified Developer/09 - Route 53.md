---
course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 (Udemy, Stéphane Maarek)
---

# Section 9 — Route 53

## DNS basics
DNS translates human hostnames into machine IPs (`www.google.com` → `172.217.18.36`) via a hierarchical structure: Root → TLD (`.com`) → SLD (`example.com`) → subdomain (`www.example.com`).

Key terms: **Domain Registrar** (Route 53, GoDaddy...), **DNS Records** (A, AAAA, CNAME, NS...), **Zone File** (holds the records), **Name Server** (resolves queries), **FQDN** (the full `protocol://sub.domain.tld.` string).

Resolution path: browser → local DNS server → root DNS server → TLD DNS server → SLD/authoritative DNS server → IP returned (each hop cacheable per its **TTL**).

## Route 53 itself
- A highly available, fully managed, **authoritative** DNS — "authoritative" meaning *you* control the records.
- Also functions as a domain **registrar** in its own right, and can health-check your resources.
- The only AWS service with a **100% availability SLA**. Name comes from port 53, the traditional DNS port.

## Records
Each record = domain/subdomain name, type (A/AAAA/CNAME/NS are the must-knows), value, routing policy, and TTL.
- **A** → IPv4. **AAAA** → IPv6. **CNAME** → points a hostname to *another hostname* (can't be used on the zone apex, e.g. can't CNAME `example.com` itself, only `www.example.com`). **NS** → the zone's name servers.

### Console walkthrough — creating and verifying a record (lecture 92)
Hosted zone → **Create record**:
1. **Record name** — e.g. `test` (under your existing zone, giving `test.yourdomain.com`).
2. **Record type** — A, for this example.
3. **Value** — the IPv4 to point at (in the demo, a made-up address just to show the mechanism, before a real server exists to point at).
4. **TTL** — left at the 300s default.
5. **Routing policy** — left as Simple (see the routing-policy lectures for the other options — not covered here since those weren't part of what was attended).
6. **Create records.**

**Verifying it actually resolves** — a browser alone won't tell you much (it'll just fail to connect to a fake IP), so the course verifies at the DNS layer directly instead of the HTTP layer:
- On the AWS side, open **CloudShell** and install DNS tooling that isn't there by default: `sudo yum install -y bind-utils` (gives you `nslookup` and `dig`).
- `nslookup test.yourdomain.com` → returns the A record's value directly.
- `dig test.yourdomain.com` → same answer, but with the **TTL and record type visible in the output**, which is why the course prefers it — you can literally see the 300s TTL you just set.
- On Windows, `nslookup` is available natively; on Mac, `dig` is.

This is the pattern worth remembering for any record you create: confirm it at the DNS level (`dig`/`nslookup`) *before* assuming an HTTP-level failure means the record itself is wrong — the two are separate failure points.

## Hosted Zones
- **Public** hosted zone — routes traffic on the public internet.
- **Private** hosted zone — routes traffic only within one or more VPCs (e.g. `db.example.internal`).
- $0.50/month per hosted zone.
- Confirmed hands-on: **registering a domain through Route 53 automatically creates its hosted zone**, pre-populated with exactly two records — an **NS record** (pointing resolution at Route 53's own name servers) and an **SOA record**. That NS record is *the* mechanism that makes Route 53 authoritative for the domain: nothing else needs configuring for Route 53 to become the source of truth.
- Domain registration walkthrough: pick a name → duration + **auto-renew** toggle (leave it on if you intend to keep the domain — otherwise someone else can register it out from under you the moment it lapses) → contact info (registrant/admin/tech contacts can all be the same person) → **privacy protection** (hides your real contact details from public WHOIS lookups) → review and pay. Real registration is not instant — can take minutes to hours before the hosted zone is fully live.

## TTL
High TTL (e.g. 24hr) = less Route 53 traffic/cost, but records can go stale longer. Low TTL (e.g. 60s) = more traffic/cost, fresher records. Every DNS record needs a TTL **except Alias records**.

## CNAME vs. Alias — the one worth remembering precisely
| | CNAME | Alias |
|---|---|---|
| Points to | Any other hostname | An AWS resource specifically |
| Root domain (`mydomain.com`) | ❌ not allowed | ✅ allowed |
| Non-root (`app.mydomain.com`) | ✅ allowed | ✅ allowed |
| Cost | — | Free |
| Health check | — | Native |

Practical upshot: if you need to point your **root domain** at an ALB, CloudFront distribution, or anything else with an AWS-generated hostname, it has to be an Alias record — CNAME simply isn't legal at the apex.
