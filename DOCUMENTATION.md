# CImEx Compliance: Platform Documentation & Feature Specification

## 1. Executive Summary & Mission
**CImEx Compliance (Carbon Import Export)** is an industrial carbon compliance platform engineered for Indian metal manufacturers and exporters facing the European Union’s Carbon Border Adjustment Mechanism (EU CBAM). 

The platform’s architectural philosophy is:
> **"AI reads the documents. People confirm. Deterministic rules decide the number."**

While generative AI is leveraged for multimodal extraction of unstructured paperwork (low-light photos, scanned mill certificates, fuel bills), **zero AI is used in calculations**. All emissions calculations, mass balancing, route allocations, and ledger deductions are strictly executed by deterministic Python algorithms compliant with EU Implementing Regulation (EU) 2023/1773 and Annex V methodology.

---

## 2. The Core Problem & Differentiation from Traditional ERPs

### Why Generic ERPs Fail at CBAM
Traditional ERPs (SAP, Oracle, Tally, Zoho) are built for financial accounting, inventory tracking, and tax invoicing. They fail at EU CBAM for five fundamental reasons:
1. **Lack of Metallurgical Mass Balancing:** ERPs track finished goods in/out, but CBAM Annex V requires tracking physical yield loss, scrap generation, and specific precursor consumption (e.g., wire rod consumed per tonne of finished bolts).
2. **No Sub-Installation Process Routing:** ERPs aggregate electricity and fuel bills into overall factory overheads. CBAM requires attributing continuous heat-treatment furnace fuel *only* to heat-treated lines, while attributing `0.0 tCO₂/t` direct fuel to cold-formed lines.
3. **Double-Claiming Vulnerability:** When an exporter buys a 50 MT batch from a certified mill and exports across multiple quarters, standard ERPs lack an active carbon depletion ledger to ensure the certified heat isn't double-allocated to different shipments.
4. **Trader vs. Melting Mill Legal Distinction:** Under EU CBAM Art. 3(15), intermediary trader test certificates are invalid. Only certificates tracing back to the primary melting installation are legally compliant. ERPs accept distributor invoices blindly.
5. **Absence of Verifier-Ready Audit Trails:** An accredited EU verifier demands line-by-line grounding: every number in the declaration must cite its verbatim source text from an attached primary document.

---

## 3. Implemented Features & Technical Architecture

### Architecture Overview
```
[Messy Plant Paperwork]
 (Photos, Scans, Bills)
          │
          ▼
┌────────────────────────────────────────────────────────┐
│ 1. Multimodal Document Extraction (Gemini Flash Vision) │
│    - Phone photos, rubber stamps, Indian lakh formats │
│    - Verbatim source quote grounding for every field   │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────┐
│ 2. Human-in-the-Loop Review Trail                      │
│    - Clerk verifies, corrects, or overrides data      │
│    - Immutable edit log with timestamp & user ID      │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────┐
│ 3. Automated Pre-Audit Agent (Google ADK & Skills)     │
│    - Mass balance breach detection (Annex V benchmarks)│
│    - Heat overdraft & double-allocation checks        │
│    - Trader invoice detection & auto-drafted RFIs     │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────┐
│ 4. Deterministic Calculation Engine (Zero AI in math)  │
│    - Direct emissions (fuel combustion by process)    │
│    - Indirect emissions (grid power emission factor)   │
│    - Embedded precursor emissions from verified MTCs  │
│    - Active Carbon Depletion Ledger                   │
└────────────────────────────────────────────────────────┘
          │
          ▼
┌────────────────────────────────────────────────────────┐
│ 5. Audit-Ready Verifier Dossier (ZIP Export)           │
│    - Audit trail JSON with source coordinates         │
│    - Official EU XML & CSV declarations               │
│    - Physical mass balance report & carbon ledger     │
│    - Attached primary evidence documents              │
└────────────────────────────────────────────────────────┘
```

---

### Detailed Feature Breakdown

#### A. Multimodal Document Ingestion (`extract.py`)
- **Real-World Paperwork Handling:** Reads scanned PDFs, low-light phone photos from factory floors, and stamps.
- **Indian Numbering Support:** Parses Indian numbering conventions (e.g., `1,00,000 kg` vs MT).
- **Verbatim Text Grounding:** For every extracted field (heat number, emissions intensity, tensile strength, weight), the engine captures the exact verbatim text snippet from the document.
- **Local Fallback Mode:** Operates with a deterministic mock parser when offline or in sandboxed demo environments.

#### B. Human-in-the-Loop Review & Audit Trail (`server.py`, `app.js`)
- **Review Queue:** Flags unverified fields with side-by-side document evidence.
- **Correction Logging:** When a clerk edits a value, the platform records:
  - Original AI-extracted value
  - User-corrected value
  - User ID and ISO timestamp
  - Verbatim source quote
- **Tamper-Evident History:** The review history is permanently bound to the export dossier for verifier inspection.

#### C. Metallurgical Mass Balance Engine (`calc.py`)
Compliant with EU CBAM Annex V:
- **Finished Yield Ratio:** Calculates `yield_ratio = finished_goods_t / precursor_consumed_t`.
- **Scrap Loss & Specific Consumption:** Computes process scrap rate and specific precursor consumption (`t precursor / t product`).
- **EU Benchmark Validation:** Automatically validates against EU operational benchmarks:
  - **Bolts & Screws:** 88% yield (12% scrap loss)
  - **Nuts:** 82% yield (18% scrap loss)
  - **Washers:** 90% yield (10% scrap loss)
- **Breach Alerting:** Automatically raises verifier warnings if reported production deviates by more than ±5% from metallurgical physical limits.

#### D. Sub-Installation Process Route Allocation (`calc.py`)
- **Disaggregated Processes:** Separates total plant energy into individual sub-installation routes.
- **Route Specificity:**
  - `heat_treated`: Continuous hardening/tempering furnace fuel is allocated directly to heat-treated products.
  - `cold_formed`: Assigned `0.0 tCO₂/t` direct fuel consumption for cold-headed washers, screws, and standard nuts.
- **Financial Defense:** Prevents cold-formed goods from being unfairly penalized with overall plant furnace emissions.

#### E. Active Carbon Depletion Ledger (`server.py`)
- **Double-Allocation Prevention:** Tracks physical heat batches across partial quarterly shipments.
- **Ledger Balances:** Maintains live records of:
  - `purchased_t`: Total verified tonnage on the primary MTC.
  - `allocated_t`: Cumulative tonnage exported across shipments.
  - `remaining_t`: Legally available verified steel balance.
- **Overdraft Guardrails:** If an exporter attempts to claim 30 MT from a heat with only 20 MT remaining, the ledger automatically caps the verified allocation at 20 MT and prices the excess 10 MT at default penalty rates.

#### F. Trader vs. Melting Mill Legal Verification (`server.py`, `agent/`)
- **Legal Mandate:** Implements EU CBAM Article 3(15) rules on legal origin.
- **Intermediary Detection:** Detects distributor/stockist invoices that lack steelmaking installation details.
- **Automated Mill RFI:** When a trader certificate is detected, the Review Agent automatically generates a formal RFI (Request for Information) letter addressed to the trader to obtain the original melting furnace heat certificate.

#### G. AI Pre-Audit Review Agent (`agent/agent.py`, `agent/tools.py`)
Built using the **Google Agent Development Kit (ADK)** and Google Agent Skills:
- **Autonomous Audit Checklist:** Simulates the workflow of an accredited third-party verification body (e.g., TÜV, DNV, SGS).
- **Multi-Skill Suite:**
  - `skill_mass_balance`: Checks physical material conservation.
  - `skill_heat_overdraft`: Audits remaining mill certificate balances.
  - `skill_fuel_attribution`: Flags missing monthly energy receipts.
  - `skill_trader_audit`: Identifies intermediate distributors.
- **Verifier Readiness Score:** Computes a 0–100% audit-readiness index surfaced directly on the exporter dashboard.

#### H. 1-Click Verifier Dossier ZIP Export (`server.py`)
Generates an export package containing:
1. `audit_trail.json`: Complete cryptographic provenance and edit log.
2. `cbam_declaration.csv`: Official EU quarterly format.
3. `mass_balance_report.json`: Annex V yield and scrap breakdown.
4. `carbon_ledger.json`: Heat-level batch allocation ledger.
5. Primary evidence files (raw MTCs, invoices, fuel receipts).

---

## 4. Financial & Environmental Impact

| Metric | Without Verified Data (EU Default) | With CImEx Verified Mill Data | Net Benefit |
| :--- | :--- | :--- | :--- |
| **Embedded Carbon (CN 7318 15)** | **4.38 – 5.72 tCO₂/t** | **2.25 tCO₂/t** | **-54% emissions intensity** |
| **Direct Process Fuel** | Full plant average | **0.0 tCO₂/t** (cold-formed line) | Zero direct fuel penalty |
| **Mass Balance Scrap Allowance** | Unaccounted | Full Annex V credit | Accurate product footprint |
| **Carbon Cost Impact (2026–2034)** | €1,821,450 | €972,375 | **€849,075 saved** (per 1,000 t/yr) |

---

## 5. Verification & Test Suite

The entire codebase is verified by 11 automated test suites:
- `test_calc.py`: Annex V yield ratios, scrap rates, sub-installation route allocation.
- `test_app.py`: Web dashboard routing, missing document detection, quarterly aggregation.
- `test_verifier.py`: Source text grounding validation.
- `test_review.py`: Human-in-the-loop audit logs and correction overrides.
- `test_match.py`: Heat number resolution across invoices and production logs.
- `test_capture.py`: Fuel receipt aggregation and unit conversion.
- `test_extract.py`: Multimodal vision prompt resilience.
- `test_real_certs.py`: Extraction against authentic Indian steel certificates (Jindal, Tata).
- `test_agent_tools.py`: Google ADK Review Agent tool execution.
- `test_pages.py`: Client-side Pyodide engine parity.
- `test_simulation.py`: Simulation engine alignment.

**Result:** 100% passing test suites across all deterministic and agent pipelines.

---

## 6. Deployment & Access Links
- **GitHub Repository:** [https://github.com/HemanthDadi1/CImEx-Compliance](https://github.com/HemanthDadi1/CImEx-Compliance)
- **Live Web Prototype:** [https://cimex-compliance.vercel.app](https://cimex-compliance.vercel.app)
- **Interactive App:** [https://cimex-compliance.vercel.app/app.html](https://cimex-compliance.vercel.app/app.html)
- **Pitch Deck (PDF):** `pitch/CImEx-Buildathon-Idea-Submission.pdf` (8 slides, 980 KB)
