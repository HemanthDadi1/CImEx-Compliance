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

## 3. End-to-End Backend Processing Lifecycle: Ingestion to Report Creation

The backend executes a 7-stage deterministic and agentic pipeline that transforms messy, unstructured factory paperwork into an accredited, audit-ready compliance package:

```
[Raw Files: PDFs, Phone Photos, Bills]
                   │
                   ▼  Step 1: Document Upload & Ingestion (/api/upload)
┌────────────────────────────────────────────────────────┐
│ extract.py: Gemini Multimodal Vision & Grounding       │
│  - Extracts structured fields                          │
│  - Captures verbatim source quotes (source_quote)      │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 2: Normalization & Categorization
┌────────────────────────────────────────────────────────┐
│ server.py: Document Normalizer & Schema Storage        │
│  - Identifies: mtc, invoice, fuel, production          │
│  - Converts units: Lakhs -> MT, Litres -> Tonnes/GJ    │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 3: Human-in-the-Loop Review Trail
┌────────────────────────────────────────────────────────┐
│ server.py: Review & Tamper-Evident Edit Logging        │
│  - Side-by-side inspection                             │
│  - Records: old_val, new_val, editor_id, timestamp     │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 4: AI Pre-Audit Review Agent
┌────────────────────────────────────────────────────────┐
│ agent/agent.py: Google ADK Pre-Audit Agent             │
│  - Checks physical mass balance, heat overdrafts       │
│  - Flags intermediary trader invoices                  │
│  - Computes Verifier Readiness Score (0–100%)          │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 5: Deterministic Calculation Engine
┌────────────────────────────────────────────────────────┐
│ calc.py: Strict Mathematical Rules (ZERO AI)           │
│  - Annex V Mass Balance (Yield, Scrap, Benchmarks)     │
│  - Sub-Installation Routing (0.0 direct fuel for cold) │
│  - Specific Embedded Emissions: SEE_dir + SEE_prec     │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 6: Active Carbon Depletion Ledger
┌────────────────────────────────────────────────────────┐
│ server.py: FIFO Heat Balance Sheet                     │
│  - Tracks purchased_t, allocated_t, remaining_t        │
│  - Overdraft protection: falls back excess to default  │
└────────────────────────────────────────────────────────┘
                   │
                   ▼  Step 7: Verifier Dossier Generation (/api/dossier)
┌────────────────────────────────────────────────────────┐
│ 1-Click ZIP Export:                                    │
│  - cbam_declaration.csv (Official EU format)           │
│  - audit_trail.json (Full cryptographic provenance)    │
│  - mass_balance_report.json & carbon_ledger.json       │
│  - Attached primary evidence files                     │
└────────────────────────────────────────────────────────┘
```

---

### Step 1: Document Ingestion & Multimodal Vision (`extract.py`)
When an exporter uploads a document (scanned PDF, phone photo from the plant floor, or fuel bill) via `POST /api/upload`:
1. **File Type Detection:** File bytes and MIME type are received in `server.py`.
2. **Gemini Vision Extraction (`extract.py`):** The image or PDF pages are dispatched to **Gemini 3.8 / 2.5 Flash** with a strict JSON schema prompt instructing the model to extract:
   - Installation details (Mill name, location, country).
   - Precursor details (Heat number, steel grade, weight in tonnes, emissions intensity `tCO₂/t`).
   - Production / Fuel quantities (Diesel litres, LPG kg, finished fasteners MT).
3. **Verbatim Text Grounding:** For *every single extracted field*, the prompt forces Gemini to extract the verbatim text snippet where the value appears:
   ```json
   {
     "heat_number": "KIS-H88213",
     "heat_number_quote": "Heat No: KIS-H88213",
     "specific_emissions": 2.10,
     "specific_emissions_quote": "Specific Carbon Footprint: 2.10 tCO2/t steel",
     "quantity_t": 200.0,
     "quantity_t_quote": "Net Weight: 200.00 MT"
   }
   ```
   *If running in an offline or test environment without an API key, the system automatically falls back to regex-based mock parsing.*

---

### Step 2: Normalization & Categorization (`server.py`)
In `server.py`, the extracted JSON is classified and normalized:
1. **Document Typing:** Assigned one of four CBAM data categories:
   - `mtc`: Primary Mill Test Certificate (steel precursor emission factor).
   - `invoice`: Commercial purchase invoice (proves legal acquisition & quantity).
   - `fuel`: Energy receipts (diesel, LPG, natural gas for heating/forklifts).
   - `production`: Monthly manufacturing logs (goods finished, scrap generated).
2. **Unit Conversion & Indian Numbering Handling:**
   - Converts Indian numbering (e.g. `1,00,000 kg` $\rightarrow$ `100.0 MT`).
   - Converts liquid fuel volumes to mass and net calorific values ($NCV$) using EU standard factors (e.g., diesel density $0.84\text{ kg/L}$, $NCV = 43.0\text{ TJ/Gg}$).
3. **Registry Storage:** Stored in the in-memory document store `DOCUMENTS[doc_id]` with verification status `status: "to_confirm"`.

---

### Step 3: Human-in-the-Loop Review & Audit Logging (`server.py`)
Before any calculation can be finalized, the human review trail comes into play:
1. **Side-by-Side Review:** The UI displays the extracted value directly adjacent to its source document quote.
2. **Verification & Overrides:**
   - When a clerk confirms a field, `POST /api/documents/confirm` sets `status: "confirmed"`.
   - If the clerk fixes an OCR mistake, `POST /api/documents/edit` records the change.
3. **Tamper-Evident Audit Trail:** Every single edit is logged with an immutable entry:
   ```json
   {
     "doc_id": "mtc_1",
     "field": "quantity_t",
     "old_value": 20.0,
     "new_value": 200.0,
     "changed_by": "clerk_chennai_plant",
     "timestamp": "2026-11-20T14:30:00Z",
     "source_quote": "Net Weight: 200.00 MT"
   }
   ```
   *Unconfirmed documents trigger verifier warnings and cap the Verifier Readiness Score.*

---

### Step 4: AI Pre-Audit Review Agent (`agent/agent.py` & `agent/tools.py`)
Using the **Google Agent Development Kit (ADK)** and Google Agent Skills, the Review Agent acts as a preliminary third-party auditor before an accredited verifier sees the file:
1. **Material Conservation Audit:** Checks whether the claimed output vs. steel input violates physical metallurgical limits.
2. **Intermediary Reseller Detection (EU CBAM Art. 3(15)):** Scans invoices for trader/distributor names that are not primary steelmakers. If flagged, it automatically drafts a formal RFI to request the original melting mill test certificate.
3. **Missing Period Checks:** Scans whether quarterly production months have matching fuel receipts.
4. **Readiness Scoring:** Computes the **Verifier Readiness Score (0–100%)** based on required documentation completeness.

---

### Step 5: Deterministic Calculation Engine (`calc.py`)
> **Crucial Rule:** Zero AI is used in the math. All calculations are executed by deterministic Python functions in `calc.py`.

#### A. Metallurgical Mass Balance (Annex V)
Reconciles physical steel inputs, outputs, and scrap:
$$\text{Yield Ratio} = \frac{\text{Finished Goods (t)}}{\text{Precursor Steel Consumed (t)}} = \frac{1,000}{1,040} = 96.15\%$$
$$\text{Scrap Rate} = \frac{\text{Physical Scrap (t)}}{\text{Precursor Steel Consumed (t)}} = \frac{40}{1,040} = 3.85\%$$
*The engine validates this against EU sector benchmarks (88% for bolts, 82% for nuts). If a plant claims 100% yield with 0% scrap, the engine flags a physical impossibility breach.*

#### B. Sub-Installation Process Route Allocation
Instead of dumping all plant fuel into every bolt, the factory is split into routes:
- `cold_formed`: Cold-heading and thread-rolling lines use only electricity. Direct fuel emissions are allocated strictly at **$0.0\text{ tCO}_2/\text{t}$**.
- `heat_treated`: Continuous tempering/hardening furnaces are allocated strictly to heat-treated product lines.

#### C. Specific Embedded Emissions Math
$$\text{SEE}_{\text{direct}} = \frac{\text{Direct Fuel Consumption} \times \text{Fuel Emission Factor}}{\text{Total Production}} = \frac{200\text{ tCO}_2}{1,000\text{ t}} = 0.20\text{ tCO}_2/\text{t}$$
$$\text{SEE}_{\text{precursor}} = \frac{\sum (\text{Allocated Heat Tonnes} \times \text{Specific Heat Footprint})}{\text{Total Production}} = 4.18\text{ tCO}_2/\text{t}$$
$$\text{Total Embedded Emissions} = \text{SEE}_{\text{direct}} + \text{SEE}_{\text{precursor}} = 0.20 + 4.18 = \mathbf{4.38\text{ tCO}_2/\text{t}}$$

---

### Step 6: Active Carbon Depletion Ledger (`server.py`)
When an exporter ships goods across multiple quarters, standard ERPs risk double-counting clean steel. The backend enforces an active FIFO depletion ledger:
1. **Heat Registration:** Each verified MTC registers its total purchased tonnage:
   - `KIS-H88213`: 200 MT purchased @ 2.10 tCO₂/t.
2. **Shipment Allocation:** For each export declaration, the engine deducts the steel consumed:
   $$\text{Remaining Balance} = \text{Purchased Tonnage} - \text{Cumulative Allocated Tonnage}$$
3. **Overdraft Fallback:** If an exporter attempts to claim 250 MT from a 200 MT certificate:
   - The first 200 MT is priced at the clean **2.10 tCO₂/t**.
   - The excess 50 MT is capped and automatically falls back to the **EU default penalty rate (5.72 tCO₂/t + 20% mark-up)**.

---

### Step 7: Verifier Dossier & Report Export (`/api/dossier`)
When the clerk clicks **"Download Verifier Dossier (ZIP)"**, `server.py` bundles the complete proof chain into an archive containing:
1. **`cbam_declaration.csv`**: The exact format required by EU customs authorities and EU importers.
2. **`audit_trail.json`**: Chronological log linking every single number in the declaration to its verbatim document quote, timestamp, and confirming clerk.
3. **`mass_balance_report.json`**: Complete Annex V breakdown of input wire rod, scrap recovery, and yield benchmarks.
4. **`carbon_ledger.json`**: Real-time batch-level inventory showing initial, allocated, and remaining certified balances.
5. **Primary Document Evidence**: Raw PDFs and photos uploaded by the plant.

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
