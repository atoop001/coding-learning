# Chapter 9: Cloud Platforms

## Overview

"The cloud" is just other people's computers, rented by the minute — but the *way* you rent them spans a spectrum from "here's a bare Linux box, good luck" to "give us a GitHub repo and we'll handle literally everything else." Choosing a point on that spectrum is a real engineering decision with real tradeoffs in money, time, control, and career signaling, and this chapter walks the spectrum honestly: **PaaS platforms** (Render, Railway, Fly.io) where you should deploy your projects today, versus **IaaS clouds** (AWS, Azure, GCP) whose vocabulary you must speak in interviews even if you don't touch them for another year. You'll learn what "managed service" actually buys you, what free tiers really include (and their gotchas — spun-down services, expiring databases), deploy a web service with a database on a PaaS, and build a working conceptual map of core AWS terms — EC2, S3, RDS, IAM — because those four names anchor half the infrastructure conversations you'll ever be in.

## Definitions & Explanations

**IaaS (Infrastructure as a Service)** — renting raw computing primitives: virtual machines, disks, networks. You get root and total control; you owe OS updates, security hardening, process management, HTTPS, scaling — everything from Chapter 2's long checklist. AWS, Microsoft Azure, and Google Cloud are the big three; DigitalOcean/Hetzner/Linode are simpler VPS-flavored players.

**PaaS (Platform as a Service)** — renting the *outcome*: "run this app at this URL." The platform provisions machines, builds from your repo (or runs your Docker image), injects config, terminates HTTPS, restarts crashes, streams logs. Current learner-friendly trio — **as of mid-2026** (free tiers change; verify current terms before committing to any of them, and see capstones/README.md's "Keeping It Free" section for the maintained summary):
- **Render** — the gentlest on-ramp; connect repo → web service; free static sites; a genuinely free-tier web service that sleeps after ~15 idle minutes (30–60s cold start on the next request); managed Postgres exists but its free tier now expires after 30 days — pair Render's free web service with a free Neon or Turso database (below) if you want a stack that stays at $0 indefinitely.
- **Railway** — slick, fast, usage-metered pricing (a monthly credit you burn down); excellent for app+database stacks; as of mid-2026 no longer offers a true free tier — expect to pay from day one (or shortly after a trial credit runs out).
- **Fly.io** — closest to real infrastructure: takes your Dockerfile, runs true VMs in regions you pick, CLI-first (`flyctl`). More to learn, more control; as of mid-2026 also no longer offers a true free tier. The natural "graduating" PaaS once you're paying for something anyway.

Honest comparison in one line each: Render optimizes for *easy* (and is currently the only one of the three with a real free tier), Railway for *pleasant*, Fly for *powerful*. Any of them can host every project in this track; pick one, learn it well, and know the other two exist.

**Managed service** — any component where the provider operates the software and you consume it: managed Postgres (they patch, back up, monitor the database server), managed Redis, object storage, load balancers. The trade is always the same: **money for undifferentiated toil**. A managed database costs more than running Postgres on a VM yourself — and is worth it approximately always at your scale, because the alternative's true cost includes you becoming a competent database administrator with a tested backup strategy (Chapter 10 shows what that entails).

**Serverless/FaaS (in the spectrum)** — sits beyond PaaS: no always-on process at all; functions run per-request (AWS Lambda, Cloudflare Workers). Chapter 1 placed it; it stays out of scope here.

**Buildpacks vs Dockerfile deploys** — the two ways a PaaS turns your repo into a running thing: auto-detection ("looks like Node; I'll `npm ci` and run your start script" — zero config, some magic) or your own Dockerfile (Chapter 5's work, honored exactly — portable, predictable). Start with detection for speed; switch to your Dockerfile when you want the same artifact locally, in CI, and in prod.

**Free tier** — real, useful, and gotcha-laden, and shrinking: **as of mid-2026**, Railway and Fly.io have both dropped their free tiers entirely; Render is the PaaS that still has a genuine one. Free-tier facts drift fast — the categories below are stable, but the specifics WILL be out of date by the time you read this; capstones/README.md's "Keeping It Free" section is kept current and is the thing to trust over this paragraph. The patterns to know: **(1) Sleep-on-idle** — Render's free web services spin down after ~15 idle minutes; the next visitor waits through a cold start of 30–60 seconds. Fine for demos; know it exists before a recruiter clicks your link. **(2) Time-limited or pausing resources** — Render's free Postgres expires after 30 days; Supabase's free projects pause after 7 idle days. The databases that *don't* expire: **Neon** (free Postgres, scales to zero when idle, no expiry) and **Turso** (free SQLite, generous limits) — the safe $0 database picks right now. **(3) Credit-based trials** (historically Railway's model) that convert to metered billing once the credit is gone. **(4) IaaS "free tiers"** (AWS) that are free only within precise limits and bill real money past them — a famous source of surprise-bill horror stories. Universal defense: read the current pricing page (these change yearly, sometimes overnight), set billing alerts anywhere a card exists, and treat any free database as disposable.

**Availability zone (AZ)** — subdivision of a region: physically separate data centers close enough for fast networking. "Multi-AZ" means your service survives one building's outage — the redundancy tier below "multi-region." PaaS platforms handle this invisibly; the term matters because AWS asks you to choose.

**Region** — the physical location of your rented computers (`us-east`, `frankfurt`). Two rules cover you for years: put app and database in the *same* region (cross-region database calls add latency to every query), and put both near your users.

**Infrastructure as Code (IaC)** — describing infrastructure (servers, databases, networks, permissions) in versioned config files instead of clicking through a dashboard, so it's reviewable, diffable, and reproducible like any other code. **Terraform** (HashiCorp, cloud-agnostic, its own HCL language) and **Pulumi** (same idea, written in a real programming language like TypeScript or Python) are the two names you'll hear most; both work by describing the *desired end state* and letting the tool figure out how to get there ("apply"), and both can tear infrastructure down as cleanly as they built it. You've already met a miniature version of this idea: Fly's `fly.toml` is dashboard-settings-as-a-committed-file. At vocabulary level for now — recognize the names and the concept for interviews; hands-on Terraform is a natural next step whenever raw cloud work becomes real.

**Core AWS vocabulary** — conceptual level; each maps to something you already understand:
- **EC2 (Elastic Compute Cloud)** — virtual machines. Chapter 2's VPS, at AWS scale. When a PaaS runs your app, EC2-class machines are what's underneath.
- **S3 (Simple Storage Service)** — object storage: put files (any size) at keys in "buckets," get them by URL, effectively-infinite, pay-per-GB. *Not* a disk drive — no folders really, no in-place editing; it's a giant key→file dictionary. The industry's default place for uploads, backups, and static assets; every cloud has a clone (Azure Blob Storage, GCS), and many services speak the "S3-compatible" API.
- **RDS (Relational Database Service)** — managed databases (Postgres, MySQL...). Render/Railway's managed Postgres is the same idea, retailed smaller.
- **IAM (Identity and Access Management)** — who/what may do which action on which resource. Users, roles, and policies; the reason "the app can read this one bucket and nothing else" is expressible. The **principle of least privilege** — grant the minimum that works — lives here, and it's the part of AWS most tested in interviews because it's the part that prevents (or causes) breaches.

Rounding out to the names you'll hear alongside: **Lambda** (functions), **CloudFront** (CDN), **Route 53** (DNS — Chapter 12), **VPC** (Virtual Private Cloud — your own isolated network inside AWS, where you decide which machines are internet-reachable and which, like databases, are private-only; the compose-network idea from Chapter 6, scaled to a data center). Azure/GCP have equivalents of everything (Azure VMs/GCE ≈ EC2, etc.); learn concepts once, map spellings per cloud.

**Load balancer** — a traffic distributor in front of two or more copies of your app, spreading requests and skipping unhealthy instances (your Chapter 11 health check is what it consults). On a PaaS this appears as a slider ("instances: 3") with the balancing invisible; on AWS it's a product you configure (ALB). The concept matters before the tool does: *scaling out means running copies, and something must divide the traffic.*

**Horizontal vs vertical scaling** — vertical: a bigger machine (more RAM/CPU — simple, has a ceiling). Horizontal: more machines (no ceiling, but now your app must tolerate copies — no in-memory sessions, no local-file state; notice how the 12-factor rules you've already adopted quietly made your apps horizontal-ready). **Autoscaling** adds/removes copies automatically based on load.

**Egress** — data *leaving* a cloud, which is where clouds famously charge: cheap to bring data in, costly to send it out. The reason "move all my images out of S3 to another provider" makes people wince, and a real (if cynical) ingredient in lock-in. At learner scale you'll pay pennies; the concept belongs in your vocabulary anyway.

**When to graduate from PaaS to raw cloud** — the honest triggers: a PaaS bill an IaaS setup would clearly beat *including your time*; a need the PaaS can't express (private networking to other systems, unusual hardware, compliance rules about data location); or — most likely for you — *an employer's stack*. Not a trigger: vibes, résumé anxiety, or "real developers use AWS." Plenty of profitable companies run entirely on PaaS. Résumé truth: "deployed and operated a production service with CI/CD, managed Postgres, monitoring, and rollbacks" beats "clicked around the AWS console once" — and AWS-the-vocabulary can be learned in a weekend when a job demands it, on top of concepts this track already gave you.

The whole IaaS-vs-PaaS tradeoff, compressed into the table you'll redraw in interviews:

| Question | PaaS (Render/Railway/Fly) | IaaS (AWS/Azure/GCP) |
|---|---|---|
| Time to first deploy | Minutes | Days (honestly) |
| Who patches the OS? | Them | You |
| Who restarts crashes? | Them | You (systemd you configured) |
| HTTPS | Automatic | You (or their managed LB, configured by you) |
| Cost at hobby scale | $0 (Render web + Neon/Turso db) – a few $/month (Railway/Fly) | $5–30/month + your evenings |
| Cost at big scale | Premium per unit | Cheapest per unit, if operated well |
| Control / unusual needs | Limited | Total |
| What it teaches | Application delivery | Systems administration |
| Junior-job relevance | "Have deployed and operated something real" | Vocabulary + concepts (this chapter) |

## Code Examples

Deploying a web service + Postgres on Render — the flow is dashboard-driven; here it is as a checklist you can follow beside the browser:

```text
1. render.com → sign in with GitHub (this grants repo access for auto-deploys)
2. New → PostgreSQL → name it, pick a region (REMEMBER IT), create.
   → copy the "Internal Database URL" (works for services in the same region/account;
     the External URL is for your laptop's psql — Chapter 10 uses both)
3. New → Web Service → pick your repo →
     Build command:  npm ci            (or: pip install -r requirements.txt)
     Start command:  node server.js    (or: gunicorn app:app)  — SAME REGION as the DB
4. Environment tab → add DATABASE_URL = <the internal URL> plus your app's other
   contract variables (this is Chapter 3's dashboard-injection, live)
5. Deploy. Watch the build log stream. Your app is at https://<name>.onrender.com
6. Push a commit → watch it auto-deploy. (Then decide who owns deploys — Chapter 8.)
```

Reading a PaaS deploy log — the platforms all narrate the same five acts, and knowing the acts tells you where a failure lives:

```text
==> Cloning from https://github.com/you/yourapp...        ACT 1: fetch your code
==> Detected Node.js app                                  ACT 2: decide how to build
==> Running build command: npm ci                         ACT 3: build
==> Uploading build...                                    ACT 4: package the artifact
==> Starting service with: node server.js                 ACT 5: run + health check
==> Your service is live 🎉

Failure in Act 3  -> your code/dependencies (reproduce with a clean local install)
Failure in Act 5  -> config: missing env var, wrong PORT/binding, crashed process —
                     the app's OWN log lines just above the failure are the evidence
"Live" but errors -> the app runs; the bug is yours. Chapter 11's territory.
```

Two platform-portability notes that save real debugging time:

```js
// 1. Listen on the platform's port. Every PaaS injects PORT; hardcoding 3000 = instant failure.
const port = process.env.PORT || 3000;

// 2. Bind to 0.0.0.0 (Chapter 5's container lesson applies to PaaS too —
//    several platforms run your app in containers whether you dockerized or not).
app.listen(port, "0.0.0.0");
```

The same deployment on Fly.io, CLI-first — worth seeing for the contrast in control:

```powershell
# Install flyctl (PowerShell installer from fly.io docs), then authenticate:
fly auth login

# From your project root — detects your app/Dockerfile and writes fly.toml (its config-as-code):
fly launch

# Create a Postgres and wire it: sets DATABASE_URL on the app automatically
fly postgres create
fly postgres attach <your-db-name>

# Deploy, inspect, read logs, open in browser:
fly deploy
fly status
fly logs
fly open
```

Notice what `fly.toml` represents: the *dashboard settings as a committed file*. That idea — infrastructure described in versioned files instead of clicked-through UIs — scales all the way up to Terraform on AWS; you're meeting it in miniature.

S3's mental model, as pseudocode (no AWS account needed today):

```text
bucket: "myapp-user-uploads"                     # a globally-named container of objects
PUT    "avatars/user-42.png"  <bytes>            # store bytes at a key
GET    "avatars/user-42.png"  -> <bytes>         # retrieve by key
# "Folders" are a display trick on key prefixes. No disk, no filesystem — a key->file dictionary.
# IAM policy sketch (conceptual): "role app-server MAY s3:GetObject, s3:PutObject
#   ON myapp-user-uploads/* AND NOTHING ELSE" — least privilege, spelled out.
```

Measuring what "region" means with your own connection — latency is geography, observable from PowerShell:

```powershell
# Time a request to the same platform's endpoints in different regions
# (many platforms expose region-named hosts; or just compare any US vs EU site):
Measure-Command { curl.exe -s -o NUL https://us-east-example.example.com } | Select-Object TotalMilliseconds
Measure-Command { curl.exe -s -o NUL https://eu-central-example.example.com } | Select-Object TotalMilliseconds

# Rule of thumb you'll observe: each Atlantic crossing costs ~80-120ms.
# Now imagine your app in Virginia querying its database in Frankfurt —
# EVERY query pays that toll. That's the co-location rule, measured.
```

Free-tier defense kit — the settings to flip the day you create any account with a card attached:

```text
Any PaaS:      note the free allowances (hours, GB, services) and what happens at
               the edge — hard stop (service pauses) vs soft overflow (billing).
Railway:       usage dashboard + configure a usage limit/alert on the account.
AWS (someday): Billing console -> Budgets -> create a $5 budget with email alerts
               BEFORE launching anything. The classic horror stories all begin
               with "I didn't know it was still running."
Everywhere:    calendar reminder, monthly, five minutes: "what am I running, and
               what did it cost?" Orphaned experiments are the #1 leak.
```

And the one command to run before trusting any free tier:

```text
Open the platform's current pricing page and read the free row yourself.
This chapter's specifics WILL drift; the gotcha CATEGORIES (sleep, expiry,
credits, overage) are stable. Verify which apply this year.
```

## Common Pitfalls

1. **Hardcoding the port / binding to localhost on a PaaS.** Deploy "succeeds," platform health checks fail or the URL times out, logs look clean. Correction: `process.env.PORT` and `0.0.0.0`, always — make it your template default so you never think about it again.

2. **Treating a sleeping free service as broken (or letting a recruiter discover it).** Thirty-second cold start after idle is *the documented free-tier behavior*, not a bug. Correction: know which of your deployments sleep; for anything you'll show humans, either accept the warm-up, pay the few dollars for always-on, or warm it before demos.

3. **Losing a project's data to free-database expiry.** Render's free Postgres expires after 30 days; the deletion email goes to spam; the demo dies quietly. Correction: read the expiry terms when you create it, calendar the date, and practice Chapter 10's backup/restore *before* it matters — or sidestep the whole problem by pointing Render's free web service at a free Neon or Turso database instead, neither of which expires.

4. **App in one region, database in another.** Every query crosses an ocean; the app is mysteriously slow though nothing is "wrong." Correction: co-locate app and DB at creation time (moving later is a chore). When users are far away, move the *pair*.

5. **Starting on raw AWS because job listings say AWS.** Weeks in IAM policies and VPC docs, nothing shipped, momentum dead — the Chapter 2 VPS trap at 10x complexity, now with a credit card attached. Correction: PaaS for building the actual skills (deploys, config, DBs, monitoring — all transferable); AWS vocabulary from this chapter for interviews; hands-on AWS when a job or a concrete need demands it — set billing alerts on day one when you do go.

6. **Confusing S3 with a hard drive.** Trying to "mount" it, expecting file locking or in-place appends, being surprised by per-request semantics. Correction: object storage is a key→file dictionary over HTTP. Uploads go *to* it, apps link *to* it; your database stores the key, not the file.

7. **Platform lock-in via unnecessary proprietary features.** Deep-integrating a platform's bespoke cron/queue/storage API when a portable alternative existed, then facing a rewrite at migration time. Correction: at your scale prefer the portable form (Dockerfile over buildpack magic, standard Postgres, S3-compatible APIs). You don't need to be paranoid — just notice each time you grab a proprietary handle, and know your exit cost.

## Practice Exercises

1. **Spectrum placement.** Draw the IaaS→PaaS→FaaS spectrum and place: a Hetzner VPS, EC2, Fly.io, Render, Railway, Lambda, and GitHub Pages. For each: who patches the OS? Who restarts crashes? Who owns HTTPS? Who decides when it scales?

2. **Pricing-page recon.** For Render, Railway, Fly.io, Neon, and Turso, read the *current* pricing/free-tier pages and fill a table: free web service (or database) behavior (sleep? scale-to-zero? expiry?), what a hobby project realistically costs monthly, and one gotcha per platform in your own words. Date the table, then compare it against capstones/README.md's "Keeping It Free" section — note any drift you find; you'll enjoy how fast this ages.

3. **First PaaS deploy.** Deploy any backend of yours to Render or Railway with a managed Postgres attached and config via dashboard env vars (a dry run for Project 5 — keep it rough). Capture: the PORT/0.0.0.0 lesson observed live, cold-start timing after 20 idle minutes, and where in the dashboard logs live.

4. **AWS translation table.** From memory, define EC2, S3, RDS, and IAM in one sentence each, then map each to (a) its Azure or GCP equivalent by name and (b) what fills that role in your PaaS deployment from exercise 3. This double mapping — across clouds, and down the abstraction ladder — is exactly the mental move interviews probe.

5. **Least-privilege thought experiment.** Your app stores user uploads in an S3-style bucket and reads/writes a Postgres. Write, in plain English, the minimal permission set the app server needs — then list two permissions it must NOT have (e.g., delete-bucket, create-users) and one concrete harm each excluded permission would have enabled if the app were compromised.

6. **Graduation memo.** Invent a plausible future project (e.g., "my tutoring-business app has 500 paying users and a 3-person team") and write a one-page memo: stay on PaaS or move to AWS? Costs, migration effort, what breaks, what improves, and the specific trigger event that would change your answer. There's no right conclusion — the graded artifact is the quality of the tradeoff reasoning.
