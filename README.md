# FallMortgage

**Sovereign single-file UK mortgage broker case management.** One HTML file, runs entirely in the browser, no server, no telemetry. Built for 1–10 person UK FCA MMR-regulated mortgage brokers (DA and AR firms).

Prime 859 · v1.0.0 · MIT · Anchor of the `fall-mortgage` bundle.

---

## For mortgage brokers

You opened a case for the Patel residential purchase. You ran the fact-find — Alice on £62k base, £8k bonus; £340k loan against a £425k Croydon terrace; Halifax 2-year fix at 4.49%. FallMortgage:

- Holds **every applicant, property, loan and affordability number** on the case record (FCA SS3/19 stress test built in: max of 8% floor or product rate + 3%).
- Generates the **ESIS** (European Standardised Information Sheet · MCD 2016 · MCOB 5A) as a frozen snapshot at issue.
- Captures **suitability evidence** (MCOB 4.7) — why this product, why this lender, why not alternatives — and signs it with adviser ID + SHA-256 + timestamp.
- Tracks **procuration fees** including clawback risk window (default 24 months) and lender disclosure.
- Logs **protection recommendations** (life · CIC · IP · buildings · contents) and what the client took up.
- Marks AR network handoff with network reference if you’re an AR firm.
- Keeps a **Mansoor P3 audit chain** of every change (FCA SYSC 7-year retention).
- Talks to the rest of the `fall-mortgage` bundle on `BroadcastChannel('fall-mortgage')` and to the wider client mesh on `BroadcastChannel('fall-client')`.

Everything lives in your browser’s IndexedDB. Wipe the browser → data gone. Export to JSON before any wipe.

### What FallMortgage is NOT

- Not a **sourcing engine**. You enter the lender / product / rate. Compare via Twenty7Tec / Mortgage Brain / Iress / direct lender portals.
- Not a **DIP / full-app submission system**. You submit via lender portals or via your network.
- Not **regulated advice**. It is a decision-support and record-keeping tool for the broker who *is* the regulated person.

### First launch

1. Enter firm details (name · FCA ref · DA or AR network · PI cover).
2. Enter first adviser (name · FCA ref · CeMAP 1/2/3 · CPD hours · SM&CR role).
3. Click **+ new case** or use the seeded demo to explore.

Or click **skip · use demo data** at step 1 to load the Patel demo case immediately.

### Daily flow

| Step | Tab | What it does |
|---|---|---|
| 1 | Cases | Open a case |
| 2 | Applicants | Fact-find: employment, income, bonus, other income, credit notes |
| 3 | Property | Address, value, type, tenure, key dates (exchange / completion / product end) |
| 4 | Affordability | Stress-test against FCA SS3/19; flags LTV / DTI / ICR breaches |
| 5 | Product | Lender, rate, type, fixed/intro period, ERC schedule |
| 6 | ESIS | Generate and freeze ESIS snapshot; export Markdown or hand off to FallMortgagePaper |
| 7 | Suitability | Write reason, sign with SHA-256 + adviser ID + timestamp |
| 8 | Protection | Record recommendations + what client took |
| 9 | Fees | Customer fees + procuration fee + clawback window |
| 10 | Timeline | Auto-built event log for the case file |

### Compliance gate baked in

- **MCOB 4.7** (Suitability) — signed evidence on every recommended case
- **MCOB 5A / MCD 2016** — ESIS snapshot frozen at issue time
- **MCOB 11.6 / FCA SS3/19** — stress test at max(8%, product + 3%)
- **MCOB 4.4A** — procuration fee disclosure recorded
- **PRA SS13/16** — portfolio landlord flagged for BTL ≥ 4 properties
- **Bank of England FPC** — LTI cap warning at 4.5×
- **SM&CR** — every action signed by current adviser ID in audit chain
- **FCA SYSC** — 7-year audit retention (configurable)
- **FCA FG21/1** — vulnerable customer flag on client record

### Built-in Q&A (T0 · no API key needed)

14 deterministic rules covering: MCOB 4.7 suitability · ESIS vs KFI · FCA SS3/19 stress · procuration fee disclosure · clawback · AR vs DA · MMR · high LTV · BTL portfolio landlord · Help to Buy / LISA · self-employed proof of income · contractor day-rate · equity release SHIP.

Add an Anthropic / OpenAI / Gemini / OpenRouter key in Settings to unlock T3 free-form questions (BYOK — your key never leaves your browser).

---

## For developers

Single HTML file, vanilla JS, no build step, no framework, no CDN dependencies (except optional inline manifest data URI). Target Chrome 113+, Safari 17+, Firefox 121+. Mobile-first responsive (works on a 380px phone).

### Architecture

```
index.html (single file, ~92KB)
├── HTML shell (header, sidebar, main view, palette, modal, toast)
├── CSS (oxblood/brass/cream/void · serif/sans/mono triad)
└── <script>
    ├── KONOMI shim (sovereign storage abstraction)
    ├── IDB (firms · advisers · clients · cases · lenderProducts · audit · settings)
    ├── LS fallback (lsPut/lsGet for environments without IDB)
    ├── Mesh:
    │   ├── fall-client (shared with IFA bundle)
    │   ├── fall-mortgage (this bundle: case.created/updated/offered/completed/declined, procurationFee.recorded, esis.issued, sync)
    │   └── fall-signal (estate-wide hello/ping/pong)
    ├── Schema factories (newBlankFirm / newBlankAdviser / newBlankClient / newBlankCase)
    ├── Affordability engine (gross→net→committed→disposable→stress-test→max-loan; BTL ICR variant)
    ├── ESIS builder (12 sections per MCD 2016)
    ├── Suitability sign-and-hash (SHA-256 over reason + adviser id + timestamp)
    ├── T0 rule base (14 MCOB/FCA Q&A entries)
    ├── Render dispatch (15 tab renderers)
    ├── Onboarding flow (2-step firm + adviser, or skip-to-demo)
    ├── Audit chain (Mansoor P3 extended: i, ts, tool, adviserId, clientId, caseId, action, reasoning, configVersion, prevHash, docHash, payload)
    ├── Palette (Ctrl+K · combined T0/cases/actions)
    ├── Settings modal (BYOK, audit toggle, export/import/wipe)
    └── Boot (openDB → loadAllStores → initMesh → render → fingerprint)
```

### Schema conformance

Conforms to:
- `IFA-BUNDLE-SHARED-SCHEMA.md` (Client / Adviser / Firm shapes)
- `MORTGAGE-BUNDLE-SHARED-SCHEMA.md` (MortgageCase extends with property/loan/affordability/applicants/protectionRecommended/fees/procurationFee/status/product/esisIssuedAt/suitabilityReason/network/timeline)

Stores: `firms` · `advisers` · `clients` · `cases` · `lenderProducts` · `audit` · `settings` (7).

### BroadcastChannel mesh

- `fall-client` — shared IFA-bundle client/adviser/firm sync (heritage from FallAdviser v2)
- `fall-mortgage` — case lifecycle events:
  - `case.created` / `case.updated` / `case.offered` / `case.completed` / `case.declined`
  - `procurationFee.recorded`
  - `esis.issued` (broadcasts the snapshot)
  - `esis.handoff` (target: fallmortgagepaper)
  - `sync.request` / `sync.snapshot` (boot-time merge by `updatedAt`)
- `fall-signal` — estate-wide hello / ping / pong / si-didy query relay

### Audit chain

Mansoor P3 extended. Every write appends an entry signed with:
```
docHash = sha256(prevHash + ts + action + adviserId + clientId + caseId + payload)
```
Tamper-evident: change any entry, the chain breaks at the next docHash. Exportable as JSON for compliance audit.

### 14-point sovereign gate compliance

| # | Gate | Status |
|---|---|---|
| 1 | Single HTML file | ✅ |
| 2 | < 500 KB | ✅ (~92 KB) |
| 3 | IDB primary | ✅ |
| 4 | localStorage fallback | ✅ |
| 5 | KONOMI shim | ✅ (`window.KONOMI`) |
| 6 | fall-signal mesh | ✅ |
| 7 | fall-mortgage mesh | ✅ |
| 8 | PWA manifest (data URI) | ✅ |
| 9 | T0 deterministic offline | ✅ (14 rules) |
| 10 | T3 BYOK optional | ✅ (4 providers) |
| 11 | Two-audience README + MIT | ✅ |
| 12 | Mobile-first responsive | ✅ |
| 13 | Disclaimer always visible | ✅ |
| 14 | Audit chain on by default | ✅ |

### Deploy as GitHub Pages

```sh
# from this directory
git init
git add index.html README.md LICENSE .nojekyll
git commit -m "FallMortgage v1.0.0"
git remote add origin git@github.com:<you>/fallmortgage.git
git push -u origin main
# Then: Settings → Pages → Source: main → / (root) → Save
```

The `.nojekyll` file is required so GitHub Pages serves files starting with `_` (and skips Jekyll preprocessing). Pages must be set to **legacy build** (not Actions workflow) for the simple static deploy to work reliably.

### License

MIT — see [LICENSE](LICENSE).

---

## Disclaimer

> FallMortgage is a tool for UK FCA MMR-regulated mortgage brokers (directly authorised and appointed-representative firms). It assists with fact-find, affordability, ESIS issuance, suitability evidence, and procuration-fee accounting. It is not a sourcing engine — lender product data + DIP/full-app submission remain the broker’s responsibility. Sovereign — client data never leaves the device.

Research and decision support, not regulated advice. The broker remains the regulated person on every recommendation.
