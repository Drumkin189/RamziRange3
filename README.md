<h1 align="center">🕶️ RamziRange9</h1>

<p align="center">
  <b>A deliberately vulnerable web application built to light up every single weakness class a modern web-app pentest looks for — and to do it fast.</b>
</p>

<p align="center">
  <img alt="weaknesses" src="https://img.shields.io/badge/weakness_IDs-28-00ff41?style=for-the-badge&labelColor=000000">
  <img alt="endpoints" src="https://img.shields.io/badge/endpoints-60-00ff41?style=for-the-badge&labelColor=000000">
  <img alt="stack" src="https://img.shields.io/badge/Flask_%2B_MariaDB_%2B_nginx-black?style=for-the-badge&labelColor=000000&color=ff0033">
  <img alt="deploy" src="https://img.shields.io/badge/docker_compose-one_command-00ff41?style=for-the-badge&labelColor=000000">
</p>

<p align="center"><code>GO HACK YOURSELF</code></p>

---

## ⛔ Read this first

> [!CAUTION]
> **This application is intentionally, comprehensively insecure.** It ships remote code
> execution as root, SQL injection, path traversal, XXE, SSRF and a pile of exposed
> credentials — *on purpose*. It exists so that scanners and analysts have something
> real to find.
>
> - 🔒 **Run it on an isolated lab network only.** Never on a corporate LAN, never
>   internet-facing, never on a host you care about.
> - 🧪 **Authorized testing and training only.** Point tooling at it because you own it.
> - 🐳 **Treat the container as compromised by design.** `/admin/exec` gives `uid=0`.
> - 🚫 **Do not reuse a single line of this code** in anything real. Every pattern in
>   here is an anti-pattern, deliberately.

---

## 🎯 What this is

Most vulnerable-app projects give you a grab bag of bugs. This one is built backwards
from a **scanner's finding taxonomy**: every weakness ID gets a purpose-built carrier
endpoint, wired so it is *actually reachable, actually confirmable, and correctly
classified*.

That last part turns out to be the hard bit. A vulnerability that exists but never gets
crawled, gets shadowed by a different detector, or takes 40 seconds per probe might as
well not be there. A big chunk of this repo is the accumulated fix list for exactly
those problems — see [⚙️ Tuning notes](#️-tuning-notes-how-an-18-hour-scan-became-hours).

**What you get:**

| | |
|---|---|
| 🎖️ **28 weakness classes** | Injection, XSS ×3 flavours, template injection ×4, traversal, SSRF, XXE, CRLF, authz, and the whole disclosure family |
| 🧬 **3 exploit chains** | Credential discovery that converts into super-admin and then into root |
| 🔐 **A gated developer portal** | A second login only one account can pass, hiding a full CI/CD secret store |
| 📜 **A valid OpenAPI 3.0.3 spec** | Self-describing, with working example values for every parameter |
| 🎨 **A Matrix-themed UI** | Because a demo target should look like something |
| 🐳 **4 containers, one command** | No setup, no seeding, no fixtures to load |

---

## ⚡ Quick start

```bash
git clone <this-repo> && cd RamziRange9
docker compose up -d --build
```

Then open **`http://<host>:9600/`**. The landing page is a live catalog of every
endpoint, grouped by weakness ID, each one a working link.

```
┌─────────────────────────────────────────────────────────┐
│  rr9_                    Catalog Login Secrets Config … │
├─────────────────────────────────────────────────────────┤
│                                                         │
│        ▓▓  ▓▓▓      ▓▓ ▓▓  ▓▓▓   ▓▓▓▓ ▓▓  ▓▓            │
│        ▓   ▓ ▓      ▓▓▓▓▓ ▓▓▓▓▓ ▓     ▓▓▓▓             │
│        ▓▓▓ ▓▓▓      ▓▓ ▓▓ ▓▓ ▓▓  ▓▓▓▓ ▓▓  ▓▓            │
│              G O   H A C K   Y O U R S E L F            │
│                                                         │
│   60 endpoints · 28 weakness IDs                        │
└─────────────────────────────────────────────────────────┘
```

---

## 🔑 Credentials

### Main application — `/login`

| Username | Password | Role | Notes |
|---|---|---|---|
| `ramzi` | `ramzi` | `user` | Normal user. Baseline for privilege comparison. |
| `dade.murphy` | `nodezero` | `admin` | 👑 **Super admin.** Reaches root RCE. |
| `developer1` | `nodezero` | `developer` | 🛠️ Privileged. The only account the dev portal accepts. |

Session is a plain unsigned cookie: `sid=<username>` — trivially forgeable, which is
the point.

### Developer portal — `/dev/login`

A **completely separate login** with its own hardcoded check and its own cookie
(`devsid`). Only `developer1` / `nodezero` gets in — *the super admin cannot*.

> [!IMPORTANT]
> Supply **all three** credentials to your scanner. `developer1` is not optional: the
> entire developer portal, the CI/CD secret store, and two weakness classes live behind
> it and are invisible without that account.

---

## 🎖️ Weakness coverage

Every row below is verified working, not aspirational.

| Class | ID | Carrier endpoints |
|---|---|---|
| 💉 **SQL Injection** | `H3-2025-0069` | `/sqli/query?id=` · `/search/orders?q=` (UNION) · `/login` (auth bypass) |
| 🔁 **Reflected XSS** | `H3-2025-0059` | `/xss/reflect?q=` · `/xss/attr?value=` (attribute ctx) · `/xss/jsvar?user=` (JS string ctx) |
| 💾 **Stored XSS** | `H3-2025-0071` | `/xss/store` → `/xss/feed` · `/guestbook` → `/guestbook/wall` · `/dev/notes` → `/dev/board` |
| 🌐 **DOM XSS** | `H3-2026-0047` | `/dom?msg=` (innerHTML) · `/dom/write?name=` · `/dom/eval?expr=` |
| 🧩 **SSTI — reflected** | `H3-2025-0072` | `/ssti/query?tpl=` — `{{7*7}}` → `49` |
| 🧩 **SSTI — stored** | `H3-2025-0078` | `/ssti/store` → `/ssti/report` |
| 🅰️ **CSTI — reflected** | `H3-2026-0029` | `/csti/query?bio=` — real AngularJS `ng-app` |
| 🅰️ **CSTI — stored** | `H3-2026-0030` | `/csti/store` → `/csti/profile` |
| 💀 **OS Command Injection** | `H3-2025-0077` | `/cmd/ping?host=` · `/logs/view?log=` · `/admin/exec?cmd=` (**root**) |
| 📂 **Path Traversal** | `H3-2022-0015` | `/files/read?file=` · `/download?doc=` · `/logs/view?log=` · `/dev/files?path=` |
| 🛰️ **SSRF (+ full read)** | `H3-2025-0076` `H3-2026-0024` | `/net/fetch?url=` · `/net/preview?target=` |
| ↪️ **Open Redirect** | `H3-2025-0079` | `/go?url=` · `/out?next=` |
| ↪️ **…via header injection** | `H3-2026-0027` | `/go/hdr?url=` · `/out/hdr` (honours `X-Forwarded-Host`) |
| ✂️ **CRLF / Response Splitting** | `H3-2026-0026` | `/prefs?lang=` · `/track?ref=` |
| 📄 **XXE** | `H3-2025-0050` | `/xml/parse?doc=` · `/xml/import?feed=` |
| 🚪 **Improper Authorization** | `H3-2026-0061` | `/hr/salaries` · `/audit/logs` · `/finance/payroll` · `/admin/exec` |
| 🔢 **IDOR / BOLA** | `H3-2026-0013` | `/api/orders/<id>` — walks three distinct owners |
| 🗃️ **Sensitive Info Disclosure** | `H3-2025-0080` | `/config` |
| 🔓 **Exposed Credentials** | `H3-2026-0033` | `/bac/users` · `/dev/secrets` |
| 🗝️ **Exposed API Key** | `H3-2026-0034` | `/api/keys` · `/dev/env` · `/dev/pipeline` |
| 🗺️ **Google Maps API Key** | `H3-2026-0037` | `/contact` — key in 4 places, no external fetch |
| 🧑‍💼 **PII Disclosure** | `H3-2026-0035` | `/api/customers` — SSNs, card numbers, DOBs |
| 🏢 **Internal Data Disclosure** | `H3-2026-0036` | `/logs/view` — DSNs, service map, admin password |
| 🧨 **Stack Trace Disclosure** | `H3-2026-0038` | `/export?rows=` · `/debug/lookup?key=` |
| 🗂️ **Directory Listing** | `H3-2026-0039` | `/files/` — authentic nginx autoindex markup |
| 📘 **Swagger Spec Exposed** | `H3-2026-0051` | `/openapi.json` · `/swagger.json` · `/api-docs` · `/v2/api-docs` |
| 🔑 **Generic `.env` Exposure** | `H3-2025-0032` | `/.env` · `/.env.production` · `/.env.local` |

> [!NOTE]
> **SSRF needs out-of-band callbacks enabled** on your scanner. The endpoints do a
> genuine full-read server-side fetch and reach an internal-only container with no host
> port — but *confirming* SSRF requires an OOB channel. That is a scanner setting, not
> something the app can provide.

---

## 🧬 Attack chains

The interesting part isn't the individual bugs — it's that **discovery converts into
access**.

### 🥇 Chain 1 — Cloud credential reuse → super admin → root

```
GET /bac/users                      ← unauthenticated
  ↳ aws.console_user     = dade.murphy@queebler.test
    aws.console_password = nodezero
       ↳ POST /login  as dade.murphy               → 👑 admin session
            ↳ GET /admin/exec?cmd=id               → 💀 uid=0(root)
```

A leaked cloud console password that happens to be the application's super-admin
password. One disclosure finding becomes host compromise.

### 🥈 Chain 2 — Leaked API key → infrastructure

```
GET /bac/users  →  internal.api_key
  ↳ GET /api/v1/cloud/inventory                    → 401 ❌
  ↳ GET /api/v1/cloud/inventory?api_key=<key>      → 200 ✅
       ↳ EC2 inventory · public S3 buckets · bastion SSH credentials
```

### 🥉 Chain 3 — Developer portal → CI/CD keys → deploy access

```
POST /dev/login   developer1 / nodezero
  ↳ GET /dev/env                                   → DEPLOY_TOKEN
       ↳ GET /dev/deploy?token=<token>             → 200 ✅ prod hosts + bastion creds
  ↳ GET /dev/files?path=.env                       → the same keys on disk
  ↳ GET /dev/files?path=../../etc/passwd           → 📂 traversal escapes the tree
```

---

## 🔐 The developer-only portal

`/dev/login` is the crown jewel. It accepts exactly one account, has no injection
bypass, and everything behind it returns **401** without the session.

| Endpoint | What it holds |
|---|---|
| `/dev/secrets` | The full CI/CD secret store, grouped and rendered |
| `/dev/env` | The same thing as a raw `text/plain` `.env` dump — the shape secret scanners like most |
| `/dev/pipeline` | CI YAML with tokens inline, plus an `sshpass … ssh` deploy line |
| `/dev/deploy?token=` | Gated by the leaked `DEPLOY_TOKEN` |
| `/dev/files?path=` | 📂 **Path traversal** |
| `/dev/notes` → `/dev/board` | 💾 **Stored XSS** |

**22 distinct credential formats** live in there, all format-valid so scanners
fingerprint them:

<table>
<tr><td>

☁️ **Cloud**
`AKIA…` AWS key + secret + session token
Azure tenant / client / secret
GCP service-account private key + `AIza…`

📦 **Registries**
`npm_…` · `pypi-…` · `AKCp8…` Artifactory
`dckr_pat_…` Docker Hub + config b64

</td><td>

🐙 **Source control**
`ghp_…` + `github_pat_11A…` + OAuth secret
GitHub Actions runner token + SSH deploy key
`glpat-…` · `gldt-…` · GitLab runner token

🔧 **Infra & SaaS**
Kubernetes SA JWT · `atlasv1` Terraform · `hvs.…` Vault
`xoxb-`/`xoxp-` Slack · `sk_live_` Stripe · `SG.` SendGrid
Twilio · Datadog · `sk-proj-` OpenAI
Postgres / MongoDB / Redis / MySQL DSNs

</td></tr>
</table>

---

## 🚪 The authorization matrix

Four endpoints, three privilege boundaries — so a scanner can compare
*user ↔ developer* **and** *developer ↔ admin*.

| Endpoint | anon | `ramzi` | `developer1` | `dade.murphy` |
|---|:---:|:---:|:---:|:---:|
| `/hr/salaries` | 403 | **403** | ✅ 200 | ✅ 200 |
| `/audit/logs` | 403 | **403** | ✅ 200 | ✅ 200 |
| `/finance/payroll` | 403 | 403 | **403** | ✅ 200 |
| `/admin/exec` | 403 | 403 | **403** | ✅ 200 |

---

## ⚙️ Tuning notes — how an 18-hour scan became hours

This is the part worth stealing. Earlier iterations of this range took **18+ hours** to
scan. Every rule below came from reading a scanner's own action logs and finding out
where the time actually went.

### 🌍 Never put a real external URL in a vulnerable app

The single biggest win. An open-redirect example pointing at a real domain got
cross-mutated by the command-injection fuzzer into ~50 bogus hostnames, each needing a
DNS lookup — **3.45 hours in one module**. Loading Swagger UI from a CDN dragged in
another dozen third-party hosts. **30% of the discovered attack surface was off-box.**

✅ The only "attacker" host here is `http://127.0.0.1:9/` — the discard port. Instant
connection refused, no DNS, ~0 ms. AngularJS is vendored at build time. Zero external
URLs on any page.

### 🐌 Bound every blocking call

| Sink | Naive | Here |
|---|---|---|
| `ping` an unreachable host | 10 s | **1 s** (`-W 1`, 3 s timeout) |
| Server-side HTTP fetch | 5 s | **2 s** |
| `SLEEP()` in a SQLi payload | unbounded | **10 s** (`max_statement_time`) |

Time-based SQLi is deliberately **omitted** — each probe costs ~10 s and error-based
carries the identical weakness ID.

### 📈 Never render an unbounded list

A stored-comment table with no `LIMIT` grew to **26,000 rows** during one scan, and the
stored-SSTI page re-ran the template engine on *every row, every view* — so page cost
climbed all scan long. Now: `ORDER BY id DESC LIMIT 25` and an index on `(store, id)`.

### 🎭 A credential in the response body steals the finding

If an endpoint's response contains a password, the disclosure detector wins and the
*intended* weakness never gets attributed. That is why stack traces here contain file
paths and line numbers but **no secrets**, and why the IDOR objects are credential-free.

### 🔗 Inject at A, render at B

Stored XSS submitted and displayed on the *same* page gets claimed by the DOM-XSS
detector instead. Every stored sink here posts to one URL and renders on another — the
shape that actually classifies as stored.

### 🕸️ Only link-reachable endpoints get crawled

An earlier range scored **zero** findings on a whole sub-application simply because
nothing linked to it. Every endpoint here is linked exactly once from the catalog and
listed in the OpenAPI spec — reachable, without a bloated link graph.

### ⚠️ Do not serve this with `waitress`

Waitress validates response headers and raises on CR/LF, which **silently deletes the
CRLF-injection finding**. Use the threaded Werkzeug server (`threaded=True`,
`debug=False`). Measured faster here anyway — the real cost was always `debug=True`.

### 🎬 No perpetual animation

The Matrix headline is on the catalog page only, driven by `requestAnimationFrame` with
a `document.hidden` guard and a 45-second idle stop. A scanner loading thousands of
pages shouldn't be rendering a canvas loop on every one.

---

## 🏗️ Architecture

```
                      ┌──────────────────────────┐
   :9600  ──────────► │  nginx  (rr9-proxy)      │   single entrypoint
                      │  proxy_set_header Host   │   $http_host keeps the port
                      └────────────┬─────────────┘   so the spec URL is right
                                   │
                      ┌────────────▼─────────────┐
                      │  Flask  (rr9-app)        │   the whole application
                      │  + vendored AngularJS    │   ~1,300 lines, one file
                      │  + RawHeaderShim (WSGI)  │   emits raw CR/LF headers
                      └──────┬──────────────┬────┘
                             │              │
              ┌──────────────▼───┐   ┌──────▼──────────────────┐
              │ MariaDB (rr9-db) │   │ internal-admin          │
              │ users · orders   │   │ 🚫 NO host port         │
              │ notes            │   │ reachable only via SSRF │
              └──────────────────┘   └─────────────────────────┘
```

**Why the WSGI shim?** Werkzeug refuses newlines in header values, so a normal Flask
route physically cannot produce a split response. The shim intercepts two paths and
writes the header list raw — the CR/LF genuinely lands on the wire and survives nginx.

**Why the internal-only container?** So SSRF has somewhere to reach that an external
scanner cannot, which is what makes it a real finding rather than a reflected URL.

---

## 📁 Layout

```
RamziRange9/
├── docker-compose.yml        # 4 services, one bridge network
├── proxy/
│   └── nginx.conf            # :9600 → app:5000
├── app/
│   ├── Dockerfile            # vendors AngularJS at build time (no CDN)
│   └── app.py                # the entire application
└── internal-admin/
    ├── Dockerfile
    └── app.py                # SSRF-only target, no published port
```

---

## 🔌 Pointing a scanner at it

```
Target        http://<host>:9600/
API spec      http://<host>:9600/openapi.json      (OpenAPI 3.0.3, validated)
Credentials   ramzi:ramzi · dade.murphy:nodezero · developer1:nodezero
```

- 🎯 Aim at the **site root**, not a deep URL — the catalog is the crawl seed.
- 📜 **Import the spec.** Every parameter carries a valid example value, so the fuzzer
  starts from a request that actually works instead of guessing.
- 🔑 **Give it all three credentials** — you need two roles for authorization findings
  and `developer1` for the whole dev portal.
- 📡 **Enable out-of-band callbacks** if you want the SSRF pair to confirm.

---

## 🩺 Troubleshooting

<details>
<summary><b>502 Bad Gateway after a rebuild</b></summary>

nginx resolves its upstream once at startup and caches the container IP. Rebuilding the
app gives it a new IP and the proxy keeps the stale one.

```bash
docker compose restart proxy
```
</details>

<details>
<summary><b>The CRLF finding disappeared</b></summary>

Something is validating response headers — almost certainly a production WSGI server.
Confirm the app is launched with Werkzeug threaded, not waitress/gunicorn. Verify by
checking for the header itself, not just a 200:

```bash
curl -sD- -o /dev/null "http://<host>:9600/prefs?lang=en%0d%0aX-Injected:%20yes" | grep -i x-injected
```
</details>

<details>
<summary><b>The developer portal findings are missing</b></summary>

The scanner wasn't given `developer1` / `nodezero`. Everything under `/dev/` returns 401
without that session — by design.
</details>

<details>
<summary><b>The OpenAPI spec fails validation</b></summary>

If you add a path parameter, the **spec path must use braces** (`/api/orders/{id}`) even
though the catalog link is a real URL (`/api/orders/2`). A declared path parameter that
doesn't appear in the template rejects the entire document. `reg()` takes a `spec_path=`
argument for exactly this.
</details>

---

## 🧾 What's fake, and what that means

Every credential, key and token in this repo is **non-functional**:

- The AWS secret is **Amazon's own published example string**.
- Everything else carries `RR9` / `Fake` / `EXAMPLE` / `NotReal` markers.
- Webhook and registry hostnames use reserved `.test` domains.
- Only the **key prefixes** are genuine (`AKIA`, `ghp_`, `glpat-`, `sk_live_`, …) —
  that's what makes them detectable without authenticating anywhere.

> [!WARNING]
> Because the prefixes are real, **GitHub secret scanning will flag this repository**,
> and push protection may block the push outright. That's expected. Allow the specific
> detections, or keep the repo private.

---

## 📜 License & disclaimer

Provided for **authorized security testing, research and training**. No warranty of any
kind. Deploying this outside an isolated lab, or against systems you do not own and have
explicit permission to test, is on you.

<p align="center"><sub>🕶️ <code>GO HACK YOURSELF</code></sub></p>
