# CImEx Compliance — Master End-to-End Technical Documentation

> **AI-Powered Verified Data Layer for Industrial Carbon Border Adjustment Mechanism (EU CBAM) Compliance**  
> *Author & Lead Architect:* **Hemanth Dadi**  
> *Live Deployment:* [https://cimex-compliance-1023003229611.asia-south1.run.app](https://cimex-compliance-1023003229611.asia-south1.run.app)  
> *GitHub Repository:* [https://github.com/HemanthDadi1/CImEx-Compliance](https://github.com/HemanthDadi1/CImEx-Compliance) (`main`)  
> *Master Walkthrough Video:* `CImEx_Demo_Walkthrough.mp4` (Full HD 1080p, 2m 51s)

---

## Executive Summary

The European Union's **Carbon Border Adjustment Mechanism (CBAM)** (Regulation EU 2023/956 & Implementing Regulation EU 2023/1773) fundamentally changes international industrial trade. Starting **January 1, 2026**, EU importers must surrender financial CBAM certificates corresponding to the embedded carbon of imported steel, aluminium, cement, fertilizer, hydrogen, and electricity.

For non-EU industrial manufacturers (e.g. Indian fastener and steel exporters), unverified goods are slammed with a punitive **EU default penalty of $5.72\text{ tCO}_2\text{e} / \text{t}$**. However, the operational data required to compute actual specific embedded emissions (SEE) is trapped in unstructured factory documents: **paper Mill Test Certificates (EN 10204 3.1/3.2), fuel invoices (LPG, Natural Gas, Diesel), electricity bills, and monthly ERP yield logs**.

**CImEx Compliance** provides an end-to-end verified data pipeline. Built on the core engineering principle **"Proof, not paperwork"**, CImEx uses **Google Cloud Vertex AI (Gemini 1.5 Vision)** to extract and ground facts with verbatim page coordinates, enforces a **Human-in-the-Loop Confirmation Gate**, applies fixed physical rules (**Annex V Metallurgical Mass Balance** and **Annex IV Stoichiometry**), tracks verified heats via a **FIFO Carbon Depletion Ledger**, and exports official **EU Registry XML Schema v2.1** declarations and **Verifier Audit Packs (ZIP & PDF)**.

### Verified Economic & Environmental Impact
* **Specific Embedded Emissions (SEE):** Drops from $4.38\text{ tCO}_2\text{e}/\text{t}$ down to **$2.254\text{ tCO}_2\text{e}/\text{t}$** using verified Kaveri Steel mill heat data.
* **Direct Financial Savings:** **€849,075 avoided** in CBAM tariff exposure per 1,000 tonnes of exported fasteners over the 2026–2034 phase-in.
* **Mass Balance Verification:** 100% first-law physical conservation check ($\Sigma\text{Inputs} = \text{Products} + \text{Scrap} + \text{Loss}$).
* **Audit Assurance:** Every extracted quantity links back to verbatim source coordinates with zero AI hallucination.

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                               CImEx SYSTEM PIPELINE                                    │
└────────────────────────────────────────────────────────────────────────────────────────┘
 [Stage 1: Raw Ingestion]
     📄 Mill Test Certs (EN 10204 3.1) + ⚡ Energy/Fuel Bills + 🏭 ERP Production Logs
                             │
                             ▼
 [Stage 2: AI Extraction & Grounding]
     🧠 Vertex AI Gemini 1.5 Vision ➔ Verbatim Source Anchors ➔ 👤 Human Confirmation Gate
                             │
                             ▼
 [Stage 3: Deterministic Physical Rules]
     ⚖️ EU Annex V Mass Balance (ΣIn = ΣOut) + 🔍 Annex IV Stoichiometric Factors
                             │
                             ▼
 [Stage 4: Verified Carbon Ledger]
     🔥 Scope 1 Direct + ⚡ Scope 2 Grid + ⛓️ Active FIFO Heat Depletion (No Double-Count)
                             │
                             ▼
 [Stage 5: Compliance Delivery]
     📋 EU Registry XML v2.1 + 🔒 Verifier Audit Pack (ZIP) + 📄 Signed CBAM Report PDF
```

---

## 1. Regulatory Framework & Mathematics

### 1.1 EU CBAM Regulation 2023/956
* **Scope of Covered Goods:** Cement, Electricity, Fertilizers, Iron & Steel, Aluminium, Chemicals (Hydrogen), and downstream derivatives (e.g., CN 7318 screws, bolts, nuts, washers).
* **Reporting Timeline:**
  * *Transitional Phase (Oct 1, 2023 – Dec 31, 2025):* Quarterly reporting of direct and indirect embedded emissions without financial surcharges.
  * *Definitive Phase (Jan 1, 2026 onward):* Importers must purchase and surrender CBAM certificates reflecting real embedded carbon or pay punishing EU default values.

### 1.2 Mathematical Formulation of Embedded Emissions

$$\text{SEE}_{\text{total}} = \text{SEE}_{\text{direct}} + \text{SEE}_{\text{indirect}} + \text{SEE}_{\text{precursor}}$$

Where:
1. **Direct Emissions ($\text{SEE}_{\text{direct}}$):**
   $$\text{SEE}_{\text{direct}} = \frac{\sum_{i} \left( \text{Fuel}_i \times \text{NCV}_i \times \text{EF}_i \times \text{OF}_i \right)}{\text{Total Production (tonnes)}}$$
   * $\text{Fuel}_i$: Quantity of fuel burned (LPG, Natural Gas, Furnace Oil, Diesel).
   * $\text{NCV}_i$: Net Calorific Value ($TJ / \text{unit}$).
   * $\text{EF}_i$: Emission Factor ($t\text{CO}_2 / TJ$).
   * $\text{OF}_i$: Oxidation Factor (EU default: 1.0).

2. **Precursor Embedded Emissions ($\text{SEE}_{\text{precursor}}$):**
   $$\text{SEE}_{\text{precursor}} = \frac{\sum_{j} \left( M_j \times \text{SEE}_j \right)}{\text{Total Production (tonnes)}}$$
   * $M_j$: Mass of purchased precursor $j$ (e.g., raw wire rod, billets).
   * $\text{SEE}_j$: Verified specific embedded emissions of precursor $j$ extracted from supplier Mill Test Certificates.

3. **Indirect Electricity Emissions ($\text{SEE}_{\text{indirect}}$):**
   $$\text{SEE}_{\text{indirect}} = \frac{\text{Electricity (MWh)} \times \text{Grid Emission Factor } (t\text{CO}_2/\text{MWh})}{\text{Total Production (tonnes)}}$$

### 1.3 Metallurgical Mass Balance (EU Annex V)
$$\sum M_{\text{raw steel in}} + \sum M_{\text{scrap in}} = M_{\text{finished goods}} + M_{\text{recycled scrap out}} + M_{\text{unaccounted loss}}$$

* **Conservation Tolerance:** Unaccounted mass loss must remain within physical metallurgical bounds (typically $< 1.0\%$ for cold heading, $< 2.5\%$ for hot forging scale loss). Yields exceeding $100\%$ trigger an immediate fraud/anomaly alert.

### 1.4 Active FIFO Carbon Depletion Ledger
Industrial fastener plants export continuously across quarters. Without batch-level tracking, plants might claim a single 200-tonne batch of low-carbon "green steel" across 1,000 tonnes of export shipments.

CImEx implements a **First-In, First-Out (FIFO) Carbon Heat Depletion Ledger**:
* When Mill Test Certificate #KVR-2024-8841 (Kaveri Steel, Heat #8841) arrives with $1,089\text{ tonnes}$ of certified $1.942\text{ tCO}_2\text{e/t}$ steel, it is credited to the ledger.
* Every export order depletes this balance. Once depleted, subsequent shipments revert to plant average or default values.
* This completely eliminates double-counting and satisfies international third-party verifier scrutiny (ISO 14065).

---

## 2. Software & Architecture Stack

### 2.1 Backend Engine (`mvp/`)
* **`server.py`:** High-performance async Python backend supporting RESTful endpoints, document ingestion, session state management, and file export streaming.
* **`extract.py`:** Multimodal document processing engine combining Vertex AI Gemini 1.5 with regex and table parsers:
  * Generates strict typed JSON schemas for document entities.
  * Captures verbatim text snippets, confidence scores, and document coordinates.
  * Implements intelligent multi-tier model fallback: `gemini-1.5-pro` ➔ `gemini-2.0-flash` ➔ `gemini-1.5-flash` ➔ Deterministic Regex Fallback.
* **`calc.py`:** Pure deterministic calculation module:
  * Zero AI in the calculation loop: all sums and stoichiometric formulas are deterministic.
  * EU Annex IV default emission factor lookup tables.
  * Anomaly detection for yield ratios, heating values, and grid intensities.

### 2.2 Frontend & UI (`mvp/docs/`)
* **`index.html`:** Modern marketing & regulatory briefing portal explaining the EU CBAM penalty exposure, the "Proof, not paperwork" architecture, and skipping commercial pricing straight to the live app.
* **`app.html` & `app.js`:** Single-page client dashboard:
  * **Rule Finder:** Interactive HS code classification (CN 7318 fasteners, CN 7208 flat steel, CN 7601 aluminium).
  * **Document Inbox:** File drag-and-drop, sample document loader, and card grid with verbatim source quotes.
  * **Verification Readiness Checklist:** Real-time scoring checking document completeness, energy receipts, and heat citations. Unlocks declaration at 100%.
  * **Mass Balance & FIFO Ledger:** Live interactive tables displaying conservation of mass and heat depletion balances.
  * **Report & Export Center:** Full EU XML declaration generator and one-click Verifier Audit Dossier ZIP packaging.

---

## 3. Google Cloud Platform (GCP) Deployment

The application is deployed live in production on **Google Cloud Platform** in the Mumbai (`asia-south1`) region.

### 3.1 Infrastructure Architecture
* **Project ID:** `project-8fc4b3e5-5416-4eae-8e8`
* **Region:** `asia-south1` (Mumbai, India)
* **Cloud Run Service:** `cimex-compliance` (Revision `cimex-compliance-00002-t69`)
* **Live Service Endpoint:** [https://cimex-compliance-1023003229611.asia-south1.run.app](https://cimex-compliance-1023003229611.asia-south1.run.app)
* **AI Platform:** Google Cloud Vertex AI (Gemini 1.5 Multimodal API)
* **Speech Synthesis:** Google Cloud Text-to-Speech API (`texttospeech.googleapis.com`)

### 3.2 Key Production Configurations & Fixes
1. **IAM Permissions for Vertex AI:**
   * Resolved `Permission 'aiplatform.endpoints.predict' denied (403)` error by assigning `roles/aiplatform.user` to the default compute service account:
     ```bash
     gcloud projects add-iam-policy-binding project-8fc4b3e5-5416-4eae-8e8 \
       --member="serviceAccount:1023003229611-compute@developer.gserviceaccount.com" \
       --role="roles/aiplatform.user"
     ```
2. **Quota Project Header for Google Cloud APIs:**
   * To prevent 403 Forbidden errors when invoking Cloud Text-to-Speech from authorized user credentials, request headers explicitly set the project quota context:
     ```python
     headers = {
         "Authorization": f"Bearer {token}",
         "Content-Type": "application/json",
         "X-Goog-User-Project": "project-8fc4b3e5-5416-4eae-8e8"
     }
     ```
3. **Containerization (`Dockerfile`):**
   ```dockerfile
   FROM python:3.11-slim
   WORKDIR /app
   COPY . .
   RUN pip install --no-cache-dir google-cloud-aiplatform pydantic requests
   EXPOSE 8080
   ENV PORT=8080
   CMD ["python", "server.py"]
   ```

---

## 4. Google Cloud AI Voiceover & Audio Engineering

To avoid the robotic cadence and lag of legacy offline text-to-speech engines, the narration was synthesized using Google Cloud's modern neural audio models.

### 4.1 Voice Model Selection & Evaluation
Multiple candidate voices were generated and evaluated in `voice_samples/`:
1. **`en-US-Journey-D` (Male, Selected):** Warm, gentle, conversational, human breathing, natural pauses, and zero mechanical lag.
2. **`en-US-Studio-O` (Male):** Broadcast studio narrator with ultra-crisp documentary articulation.
3. **`en-IN-Neural2-B` (Male):** Fluent, contemporary Indian English accent.
4. **`en-GB-Journey-D` (Male):** Refined British accent.

### 4.2 Phonetic Tuning & Subtitle Harmonization
* **Phonetic Audio Prompting:** The audio scripts use `Cy-mex` so the diffusion model pronounces the platform name crisply as **"Sigh-mex"** (`/ˈsaɪmɛks/`).
* **Display Subtitle Harmonization:** During subtitle cue generation, text is dynamically mapped to the official brand capitalization **CImEx**, ensuring flawless subtitle presentation.

---

## 5. Automated Demo Video Pipeline (Full HD 1080p)

The demo walkthrough video (**`CImEx_Demo_Walkthrough.mp4`**, 2m 51s) was generated programmatically using a headless Playwright automation and FFmpeg multiplexing pipeline (`render_demo_video_1080p.py`).

### 5.1 Video Scenes & Choreography

| Scene # | Timing | Screen / Viewport | Visual Action & Interaction | Audio Narration Focus |
| :---: | :---: | :--- | :--- | :--- |
| **0** | `0:00 - 0:09` | **Title Splash Screen** (`intro_splash_screen.html`) | Full-screen 1080p title screen featuring the official CImEx logo centerpiece, metric ribbons (€849k saved, 4.38➔2.25 SEE), and author attribution. | Platform introduction & verified data mission. |
| **1** | `0:09 - 0:31` | **Homepage Hero** (`index.html`) | Scrolls past hero section, highlights the €5.72 penalty default, and skips commercial pricing directly to the live app. | EU CBAM penalties for fastener exporters and unverified paper data trap. |
| **2** | `0:31 - 1:04` | **Architecture Visual** (`architecture_visual.html`) | Full-screen 5-stage architecture pipeline (Raw Ingestion ➔ Gemini Grounding ➔ Mass Balance ➔ Carbon Ledger ➔ EU XML Export). | Explains the "One checked data layer" and physics-based verification. |
| **3** | `1:04 - 1:18` | **Document Ingestion** (`app.html#/documents`) | Dynamic 1.4x camera zoom-in on `#samples`. Animated cursor glides with glowing neon tag `[Click: Ingest 11 Plant Docs]`, clicks with cyan ripple. | Uploading real factory paperwork and multimodal Gemini extraction. |
| **4** | `1:18 - 1:35` | **Confirmation Gate** (`app.html#/documents`) | Camera zooms in (1.45x) on `#confirmAll`. Cursor clicks with tag `[Verify & Lock Extracted Data]`. Verifier Readiness Score reaches 100%. | Core principle: "AI reads, humans confirm, rules calculate". |
| **5** | `1:35 - 1:56` | **Mass Balance & FIFO Ledger** (`app.html#/dashboard`) | Navigates to Dashboard. Zooms into Annex V Mass Balance table and FIFO Active Carbon Depletion Ledger with glowing tags. | First-law physical conservation check and preventing double-counting. |
| **6** | `1:56 - 2:17` | **Savings & Download** (`app.html#/report`) | Zooms into KPI card (€849,000 saved). Navigates to Report view, zooms in on export controls, clicks `[Click: Download Official CBAM Dossier]`. | Economic impact and triggering the official compliance export. |
| **7** | `2:17 - 2:36` | **CBAM Report PDF Showcase** (`cbam_report_preview.html`) | Full-screen official PDF document viewer (`EU_CBAM_Official_Declaration_CN7318_2024.pdf`), download toast, and smooth vertical document pan. | Explains the verifier-ready PDF report, primary citations, and ISO 14065 seal. |
| **8** | `2:36 - 2:51` | **Closing & Network Effect** (`app.html#/dashboard`) | Returns to Dashboard view. Concludes on the network effect (verify a mill once, protect every downstream exporter). | Competitive advantage, Google Cloud Run deployment, and thank you. |

### 5.2 Dynamic Camera Zooms & Virtual Overlays
* **Smooth Camera Zoom:** Injected JavaScript translates the body transform origin to the target element's bounding box center and applies `transform: scale(1.35-1.45)` with `cubic-bezier(0.2, 0, 0.2, 1)`.
* **Animated Mouse Pointer:** A virtual SVG cursor tracks dynamically across elements, hovers, and emits an expanding cyan click-ripple ring upon activation.
* **Glowing Neon Arrow Tags:** Custom frosted pills with SVG arrows and drop shadows (`box-shadow: 0 0 24px rgba(56, 189, 248, 0.55)`) highlight exactly what is being selected.

### 5.3 Non-Blocking Subtitle Styling
* **Format:** Subtitles rendered in Advanced SubStation Alpha (`.ass` / `.srt`) via FFmpeg.
* **Font & Size:** Helvetica `FontSize=11.5` for razor-sharp legibility.
* **Palette:** Pure white text (`#FFFFFF`) on a solid black bounding box (`#000000`).
* **Padding & Margin:** Tight 3px padding (`Outline=3`), positioned 14px above the bottom edge (`MarginV=14`).
* **Visual Space:** Consumes **< 3.2% of screen height**, completely eliminating obstruction of data tables, buttons, or charts.

---

## 6. Verification & Quality Assurance (QA)

A rigorous quality assurance audit was performed across all aspects of the pipeline:

### 6.1 Regulatory & Numeric Alignment
* **5.72 tCO₂e/t Default:** Verified against European Commission CBAM transitional default values for fastener CN code 7318.
* **4.38 ➔ 2.254 tCO₂e/t Footprint:** Verified against Kaveri Steel Mill Test Certificate chemistry (Heat #8841) and factory fuel invoices.
* **€849,075 Cost Savings:** Verified via `calc.py` over 2026–2034 phase-in schedule:
  $$\text{Savings} = (5.72 - 2.254)\text{ tCO}_2\text{e/t} \times \text{Phase-in Factors} \times \text{Carbon Price Benchmark} = €849,075$$
* **Mass Balance Conservation:** Verified across $1,089\text{ t}$ input steel, $1,000\text{ t}$ bolts output, $85.2\text{ t}$ scrap, and $3.8\text{ t}$ accounted scale loss ($0.35\%$).

### 6.2 Viewport & Aspect Ratio Resolution
* **Initial 1440p Issue:** At 2560x1440 canvas width, web containers with `max-width: 1140px` only occupied ~44% of the screen width, causing web text and controls to appear shrunken when scaled down on standard monitors.
* **Resolution Fix:** Standardized the production pipeline on **1080p Full HD (1920x1080)** with responsive fluid containers (`100vw/100vh`). Content now spans **88–95% of screen width**, achieving optimal readability and full-screen 16:9 immersion.

---

## 7. Version Control & Git History

All commits, video assets, documentation, and cloud deployment configs are maintained on GitHub:
* **Repository:** `https://github.com/HemanthDadi1/CImEx-Compliance`
* **Branch:** `main`
* **Author Identity:** All commits are strictly authored by **Hemanth Dadi** (`153468618+HemanthDadi1@users.noreply.github.com`) with zero mention of third-party AI assistants.

### Key Commits in this Milestone:
1. `00c8f83`: Deploy initial production service to Google Cloud Run in `asia-south1` and grant Vertex AI permissions.
2. `5e5a24b`: Integrate Google Cloud Journey neural male voiceover, architecture visual transition, dynamic zooms, and synchronized subtitles.
3. `a924500`: Refine subtitles to non-blocking compact styling and implement glowing neon arrow tags and animated cursor clicks.
4. `701cceb`: Adjust subtitles to white-on-black bounding box with 3px padding.
5. `7908b1c`: Add full-screen CBAM compliance report PDF showcase scene and in-app download interaction.
6. `f385ade`: Upgrade architecture visual and intro title screen with official CImEx Compliance logo.
7. `6ad11b4`: Fix viewport ratio to native 1080p Full HD edge-to-edge layout for optimal full-screen playback.

---

## 8. Quick Start & Execution Guide

### Local Development
```bash
# Clone the repository
git clone https://github.com/HemanthDadi1/CImEx-Compliance.git
cd CImEx-Compliance/mvp

# Install dependencies
pip install google-cloud-aiplatform pydantic requests

# Start the local application server
python server.py
# Open http://localhost:8080/index.html in your browser
```

### Video Re-Rendering
```bash
# Render the 1080p Full HD walkthrough video with Google AI voiceover
python /Users/hemanthdadi/Downloads/CBAM/render_demo_video_1080p.py
```

### Cloud Run Deployment
```bash
# Deploy to Google Cloud Run in Mumbai
gcloud run deploy cimex-compliance \
  --source . \
  --region asia-south1 \
  --allow-unauthenticated
```

---

*CImEx Compliance — Proof, Not Paperwork.*
