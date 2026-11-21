# CImEx Compliance — 3–5 Minute Presentation Pitch Script

> **Event:** Startup Buildathon India 2026 (Google DeepMind | Google Cloud | Nasscom | Headstart)  
> **Presenter:** Hemanth Dadi (Lead Architect & Founder, CImEx Compliance)  
> **Target Delivery Time:** ~3.5 to 4.5 minutes (approx. 560 words spoken at 130–140 WPM)  
> **Key Mantra:** *"AI reads. People confirm. Rules decide."*

---

## Pitch Structure & Timing Overview

| Beat | Slide / Visual | Timecode | Key Message |
| :--- | :--- | :---: | :--- |
| **1. The Hook & The Cliff** | Slide 1–2 (Problem) | `0:00 – 0:50` | EU CBAM is live; Indian MSMEs face punitive 5.72 defaults and price cuts. |
| **2. The Core Solution & USP** | Slide 3–5 (Architecture) | `0:50 – 1:45` | Not another ERP: AI reads, people confirm, deterministic rules decide. |
| **3. The Deep Tech Innovations** | Slide 4, 6, 9 (Engine & Ledger) | `1:45 – 2:50` | Annex V Mass Balance + FIFO Carbon Depletion Ledger (zero double-counting). |
| **4. The Economic & Live Proof** | Slide 10–11, 13 (Results & Demo) | `2:50 – 3:45` | Footprint cuts 59% (4.38 ➔ 2.25 tCO₂e/t), €26M saved by 2030, live on Cloud Run. |
| **5. Roadmap & The Ask** | Slide 12–14 (Future & Close) | `3:45 – 4:20` | Network effect: verify a mill once, protect every buyer. The closing call. |

---

## Spoken Presentation Script

### [0:00 – 0:50] 1. The Hook & The Crisis (Slides 1 & 2)

*(Tone: Urgent, authoritative, grounded)*

Good morning, judges and fellow builders. 

On January 1st, 2026, the European Union activated the Carbon Border Adjustment Mechanism—**CBAM**. 

For India's €3.2 billion iron and steel export sector, the rules of global trade just changed overnight. If an Indian fastener or steel exporter cannot prove its actual carbon footprint, European customs automatically slams them with a punitive EU default penalty: **5.72 tonnes of CO₂ per tonne of steel**, plus an escalating markup of up to 30%. 

According to GTRI, this triggers immediate price cuts of 15 to 22% or lost European buyers. 

Here is the tragedy: our Indian mills—using efficient Electric Arc Furnaces and sponge iron—actually produce steel at **2.1 to 2.3 tonnes of CO₂**. The low-carbon data exists! But it is trapped in messy, unstructured factory paperwork: paper Mill Test Certificates, fuel invoices, and meter logs. And traditional ESG consultants charge ₹15 to 20 lakh per plant just to manually fill out spreadsheets. 

We built **CImEx Compliance** to solve this.

---

### [0:50 – 1:45] 2. The Core Solution & USP (Slides 3, 4 & 5)

*(Tone: Clear, deliberate, high-conviction)*

CImEx is not another ERP. Traditional ERPs track money; CBAM tracks **carbon heats and physical mass**. 

We built CImEx around one strict regulatory principle:  
**"AI reads. People confirm. Rules decide."**

* **First, AI reads:** Using **Google Cloud Vertex AI and Gemini 1.5**, we ingest scans, mobile camera photos under factory lighting, rubber stamps, and Indian digit formats like 1,00,000 kg. Crucially, Gemini grounds every single number with its verbatim source line and coordinates.
* **Second, People confirm:** We enforce a Human-in-the-Loop gate. A plant clerk reviews highlighted values side-by-side with the original paper cert. Every edit is logged in an immutable audit trail.
* **Third, Rules decide:** **Zero AI in the maths.** Under ISO 14065, an EU verifier will immediately reject a carbon declaration if calculations are done by a black-box LLM. In CImEx, 100% of the emissions stoichiometry is computed by frozen, deterministic Python rules.

---

### [1:45 – 2:50] 3. The Technical Innovations (Slides 4, 6 & 9)

*(Tone: Technical depth, confident domain expertise)*

We solved the two hardest engineering challenges that break every other compliance tool:

**Number One: The Metallurgical Mass Balance under EU Annex V.**  
You cannot manufacture 1,000 tonnes of bolts from 900 tonnes of steel. Our engine reconciles raw steel inputs against finished output and scrap:
$$\text{Yield} = \frac{1,000\text{ t}}{1,040\text{ t}} = 96.15\%, \quad \text{Scrap} = \frac{40\text{ t}}{1,040\text{ t}} = 3.85\%$$
If a plant claims 100% yield with zero scrap, our engine flags a physical impossibility breach before a verifier catches it. Furthermore, our **Sub-Installation Route Model** isolates cold-heading from continuous heat-treatment furnaces—allocating **0.0 tCO₂/t** of direct fuel to cold-formed lines.

**Number Two: The Active FIFO Carbon Depletion Ledger.**  
The biggest fraud vector in CBAM is "double-claiming"—an exporter buying a 200-tonne coil of green steel and reusing the same certificate across 500 tonnes of quarterly shipments.  
Our ledger works like a strict bank balance sheet. Heat `KIS-H88213` deposits 200 tonnes at 2.10 tCO₂/t. As exports ship, the balance depletes. If an exporter tries to claim 250 tonnes, the excess 50 tonnes automatically triggers an overdraft alert and reverts to the EU default penalty. **Zero double-counting.**

---

### [2:50 – 3:45] 4. Economic Impact & Live Deployment (Slides 7, 10, 11 & 13)

*(Tone: Energetic, metric-driven, proud)*

The impact is transformative.

On our benchmark fastener plant, linking verified mill heat data slashes specific embedded emissions from **4.38 down to 2.25 tonnes of CO₂ per tonne—a 59% reduction** against the EU default.

In cold cash, on fastener exports to Germany and the Netherlands alone, verified data avoids **€26 million in CBAM penalties by 2030**, and **€121 million by 2034**. For an individual mid-size plant exporting 1,000 tonnes, that is **€849,075 saved directly**.

And this is not a mock-up. 
* CImEx is **live in production right now on Google Cloud Run in Mumbai (`asia-south1`)**, ensuring sensitive Indian industrial trade data never leaves sovereign borders.
* We have **11 core test suites and 21 automated validation suites**, all passing with 100% accuracy.
* With one click, the exporter downloads the official DG TAXUD **EU XML declaration** and an **ISO 14065-ready Verifier Audit ZIP Dossier**.

---

### [3:45 – 4:30] 5. The Network Effect & The Vision (Slides 12 & 14)

*(Tone: Visionary, inspiring, decisive)*

Our go-to-market begins with fastener clusters in Chennai, Pune, and Ludhiana, before expanding to wire, pipes, and aluminium.

Our real defensibility is the **Network Effect**:  
When we verify one primary melting mill—like Kaveri Steel or Tata Steel—**every single downstream fastener exporter that buys from that heat moves off the penalty default instantly.**

CImEx transforms CBAM from an existential export penalty into India's greatest competitive manufacturing advantage. 

Proof, not paperwork.

Thank you. We are ready for your questions.

---

## Speaker Delivery Cheatsheet

| Cue | What to Do / Emphasize |
| :--- | :--- |
| **"€3.2 Billion"** | Plant feet, make eye contact across all judges. |
| **"Zero AI in the maths"** | Slow down pace slightly. Stress this word-for-word. Judges love auditable reliability over LLM hype. |
| **"1,000 tonnes from 900 tonnes"** | Use hands to illustrate the physical absurdity of impossible mass balances. |
| **"Overdraft Fallback"** | Point to the FIFO ledger on screen; explain how it eliminates greenwashing audits. |
| **"Live on Cloud Run in Mumbai"** | Show the live link; emphasize data sovereignty and production readiness. |
