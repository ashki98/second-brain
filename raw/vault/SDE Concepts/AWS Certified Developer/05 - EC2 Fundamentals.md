---
course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 (Udemy, Stéphane Maarek)
---

# Section 5 — EC2 Fundamentals

## What EC2 actually bundles
EC2 = Elastic Compute Cloud = Infrastructure as a Service. "Knowing EC2" really means knowing four things together: renting VMs (EC2 itself), virtual drives (EBS), distributing load (ELB), and scaling (ASG — see [[07 - ELB and ASG]]).

## Console walkthrough — launching your first instance, in order (lecture 34)
EC2 → Instances → Launch instances. This is the exact sequence the console presents, and what each field actually does:
1. **Name and tags** — e.g. `My First Instance`. Just a `Name` tag under the hood; you can add more tags here but don't have to.
2. **AMI (the OS image)** — pick from "Quick Start" (Amazon Linux 2 is the free-tier-eligible one used throughout the course) or search the full catalog. You can build and reuse your own AMIs later, but quick-start images are enough to begin.
3. **Instance type** — `t2.micro` is the one that's free-tier eligible (750 hrs/month for the first year); `t3.micro` is the substitute in regions where `t2.micro` isn't offered. This is where the General/Compute/Memory/Storage-optimized families below come into play once you're past the free tier.
4. **Key pair** — needed only if you plan to SSH in. Creating one: name it, choose **RSA**, then a format — `.pem` for Mac/Linux/Windows 10+, `.ppk` for PuTTY on Windows 7/8. You can also explicitly "proceed without a key pair" and rely on EC2 Instance Connect instead (see below).
5. **Network settings** — leave default VPC/subnet with a public IP, and this is where the **security group actually gets created**, not as a separate up-front step: the console auto-creates one named `launch-wizard-1` and lets you tick which inbound rules to add right here (e.g. SSH from anywhere, HTTP from anywhere) — see the Security Groups section below for what these rules actually mean.
6. **Storage** — default 8GB gp2 root volume (free tier covers up to 30GB). Under "Advanced," the one setting worth knowing is **Delete on Termination** (default: yes) — terminating the instance also deletes this volume unless you turn that off.
7. **Advanced details → User Data** (right at the bottom) — paste a bootstrap script here; see the User Data section below for what it actually does.
8. **Launch instance.**

**What you get back, and how to actually use it**: the instance takes ~10–15s to leave "pending" and go "running." It has both a **public IPv4** (what you connect to it *with*, from outside) and a **private IPv4** (how it's addressed *inside* the AWS network — this is what a User Data script sees if it reports its own address, and it never changes). Access the web server with `http://<public-ip>` — **not https**, since nothing set up a certificate; using https here just hangs on an infinite loading screen. Stopping an instance keeps its EBS volume (so state persists) but stops billing for compute; starting it again will very likely hand you a **different public IPv4** than before (the private IPv4 never changes) — so don't hardcode the public IP anywhere you expect to reuse.

## EC2 User Data
- Runs commands automatically **once**, only at first boot — installing updates/software, downloading files, anything scriptable.
- Runs as the **root** user.
- Confirmed hands-on: a typical demo script updates the OS, installs the `httpd` web server, and writes an `index.html` that echoes back the instance's own private IP — this is exactly the "hello world from `<ip>`" page you get after launch, and it's why the IP shown on that page is the *private* one even though you reached it via the *public* one.
- This is the EC2-native equivalent of the container startup patterns from [[ECS]] — same idea (bootstrap on launch), much simpler (one shell script instead of a task definition).

## Instance types
Naming convention example: `m5.2xlarge` → `m` = instance class, `5` = generation, `2xlarge` = size within the class.

| Family | Best for |
|---|---|
| General Purpose (e.g. `t2.micro`) | Balanced compute/memory/networking — web servers, code repos |
| Compute Optimized | Batch processing, media transcoding, HPC, ML, gaming servers |
| Memory Optimized | In-memory/relational DBs, BI workloads, real-time big-data processing |
| Storage Optimized | High-throughput OLTP, NoSQL, Redis-style caches, data warehousing |

## Security Groups
- The **firewall** for EC2 instances — regulate inbound and outbound traffic by port and IP range (IPv4/IPv6), or by referencing another security group.
- Live **outside** the instance — if a security group blocks traffic, the EC2 instance never even sees the packet.
- All inbound traffic **blocked** by default; all outbound traffic **allowed** by default (confirmed hands-on: a fresh SG's outbound rule is "allow all traffic, IPv4, anywhere").
- Can attach to multiple instances; locked to a region+VPC combination.
- **An instance can have several security groups attached at once, and the rules simply union together** — and the reverse is true too: one security group can be attached to many different instances. Each SG has its own ID, same as an instance does.
- Diagnostic rule of thumb, confirmed directly in the console: deleting the port-80 inbound rule and refreshing the browser produces an **infinite loading / timeout** — not an error page. That's the tell: connection **times out** → security group issue (port isn't open, or wrong direction). Connection **refused** → the app itself isn't running or has its own error, not a SG problem.
- Classic ports worth memorizing: 22 SSH, 21 FTP, 22 SFTP, 80 HTTP, 443 HTTPS, 3389 RDP.
- "Anywhere" in the console is shorthand for the CIDR `0.0.0.0/0`. The "My IP" option scopes a rule to your current IP specifically — convenient, but it'll silently break access (another timeout) the next time your IP changes.

## EC2 Instance Connect
Browser-based SSH with no downloaded key file — AWS uploads a temporary key to the instance for you. Only works out-of-the-box on Amazon Linux 2, and port 22 still has to be open in the security group.

## Cost control (AWS Budgets)
Practical housekeeping, not exam-critical, but worth having on file:
- Billing data is only visible from the **root account** by default — an IAM user (even an administrator) gets access-denied until the root account explicitly turns on "IAM user and role access to billing information" under Account settings.
- **AWS Budgets**: a **zero-spend budget** alerts the moment you cross $0.01 — the simplest tripwire for "did anything unexpected start billing." A **monthly cost budget** (e.g. $10/month) can alert at multiple thresholds — actual spend at 85%, actual at 100%, and forecasted at 100% — each to an email.
- The **Free Tier** dashboard shows current + forecasted usage against the free allowance per service, and flags anything projected to exceed it.
- **Bills → [month] → charges by service** is the fastest way to find *which* service is generating cost — it breaks a line item like "EC2" down further into things like NAT Gateway hourly + per-GB charges, EBS snapshot storage, unattached Elastic IP charges, etc.

