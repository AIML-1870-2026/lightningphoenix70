# Drug Safety Explorer — Product Specification

**Version:** 1.0  
**Status:** Draft  
**Target Audience:** General public / curious patients  
**Format:** Single-page web application (HTML + CSS + JS, no build step required)

---

## 1. Overview

Drug Safety Explorer is an interactive, single-page web tool that queries the OpenFDA API to help everyday users explore and compare drug safety information. Users can search up to four drugs simultaneously and view side-by-side safety data drawn from three official FDA data sources. The tool is designed to be approachable and clearly contextualized — not a clinical decision-making tool, but a transparent window into publicly available FDA data.

---

## 2. Goals

- Allow any user, regardless of medical background, to look up safety signals for 1–4 drugs at once
- Surface meaningful patterns from real-world adverse event reports, official labeling, and approval history
- Clearly communicate the limitations of the underlying data at every relevant touchpoint
- Meet all OpenFDA attribution and terms-of-service requirements
- Be fully self-contained (no backend, no login, no data storage)

---

## 3. Data Sources (OpenFDA Endpoints)

All data is fetched client-side directly from `https://api.fda.gov`. No API key is required for public use under the rate limit (40 requests/minute per IP). An API key may be added via a config constant for higher limits.

### 3.1 Adverse Events — `/drug/event.json`
Source: FDA Adverse Event Reporting System (FAERS), 2004–present, updated quarterly.

**Fields used:**
| Field | Purpose |
|---|---|
| `patient.reaction.reactionmeddrapt.exact` | Top reported reactions (count query) |
| `patient.patientagegroup` | Age group distribution |
| `patient.patientsex` | Sex breakdown |
| `serious` | Proportion of serious vs. non-serious reports |
| `patient.reaction.reactionoutcome` | Outcome codes (hospitalization, death, disability, etc.) |
| `receivedate` | Report receive date (for timeline use if added later) |
| `primarysourcecountry` | Country of reporter origin |

**Key queries:**
- Count of top 10 reactions: `count=patient.reaction.reactionmeddrapt.exact`
- Outcome breakdown: `count=patient.reaction.reactionoutcome`
- Sex breakdown: `count=patient.patientsex`
- Age group: `count=patient.patientagegroup`
- Total report count: `limit=1` to read `meta.results.total`

### 3.2 Product Labeling — `/drug/label.json`
Source: Structured Product Labeling (SPL) submitted by manufacturers.

**Fields used:**
| Field | Purpose |
|---|---|
| `openfda.brand_name` | Display name |
| `openfda.generic_name` | Generic name |
| `openfda.manufacturer_name` | Manufacturer |
| `warnings_and_cautions` | Official warnings text |
| `boxed_warning` | Black box warning (most severe) |
| `contraindications` | Who should not take the drug |
| `drug_interactions` | Known interactions |
| `adverse_reactions` | Manufacturer-listed adverse reactions |
| `indications_and_usage` | What the drug is approved to treat |

### 3.3 Drugs@FDA — `/drug/drugsfda.json`
Source: FDA approval records, most complete from 1998–present.

**Fields used:**
| Field | Purpose |
|---|---|
| `openfda.brand_name` | Match drug name |
| `submissions.submission_type` | NDA, ANDA, BLA |
| `submissions.submission_status_date` | Approval date |
| `sponsor_name` | Manufacturer/sponsor |
| `products.dosage_form` | Available forms |
| `products.marketing_status` | Prescription, OTC, discontinued |

---

## 4. Application Structure

### 4.1 Page Layout (top to bottom)

```
┌──────────────────────────────────────────────────────┐
│  HEADER: Logo + Title + Educational Disclaimer Banner│
├──────────────────────────────────────────────────────┤
│  DRUG INPUT ZONE                                     │
│  [ Search slot 1 ] [ Search slot 2 ] [ + Add Drug ]  │
│  Preset comparisons: [Ibuprofen vs Acetaminophen]    │
│                      [Ozempic vs Wegovy]             │
├──────────────────────────────────────────────────────┤
│  RESULTS DASHBOARD (renders after search)            │
│                                                      │
│  ┌─────────────────────┐  ┌─────────────────────┐   │
│  │ Drug Summary Cards  │  │ Approval History     │   │
│  │ (one per drug)      │  │ (Drugs@FDA)          │   │
│  └─────────────────────┘  └─────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ Adverse Event Charts (side-by-side)          │   │
│  │  - Severity / Outcome Breakdown (donut)      │   │
│  │  - Demographic Breakdown (age + sex bars)    │   │
│  │  - Top Reactions (horizontal bar, per drug)  │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ Labeling Panel                               │   │
│  │  - Black Box Warning (if present)            │   │
│  │  - Warnings & Contraindications              │   │
│  │  - Drug Interactions                         │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │ [▼] About This Data (collapsible)            │   │
│  └──────────────────────────────────────────────┘   │
├──────────────────────────────────────────────────────┤
│  FOOTER: OpenFDA attribution + links                 │
└──────────────────────────────────────────────────────┘
```

### 4.2 Component Inventory

| Component | Description |
|---|---|
| `DisclaimerBanner` | Persistent top banner; non-dismissible |
| `DrugInputZone` | Up to 4 search slots + preset buttons |
| `DrugSearchInput` | Autocomplete input using NDC/label name search |
| `PresetButton` | One-click loads preset drug pair |
| `DrugSummaryCard` | Name, generic, manufacturer, approval year, marketing status |
| `ApprovalHistoryCard` | Submission type, approval date, sponsor, dosage forms |
| `OutcomeDonutChart` | Severity breakdown per drug (SVG/Canvas) |
| `DemographicsBarChart` | Age groups + sex split, one bar group per drug |
| `TopReactionsChart` | Horizontal bars of top 10 reactions, overlaid or side-by-side |
| `LabelingPanel` | Tabbed or accordion: Black Box / Warnings / Interactions |
| `BlackBoxCallout` | Visually distinct red/amber callout if boxed warning exists |
| `HelpButton` | `?` icon — opens an informational popup modal |
| `InfoModal` | Popup with plain-English explanation of a term or concept |
| `AboutDataSection` | Collapsible; covers FAERS limitations, reporting bias, data lag |
| `LoadingState` | Per-drug skeleton loaders during API fetch |
| `ErrorState` | Friendly message if drug not found or API fails |
| `Footer` | OpenFDA attribution, disclaimer link, source link |

---

## 5. Drug Input & Search

### 5.1 Search Behavior
- Each slot is a text input with live autocomplete
- Autocomplete queries `/drug/label.json?search=openfda.brand_name:TERM&limit=5` and `/drug/label.json?search=openfda.generic_name:TERM&limit=5`
- Results show both brand and generic name (e.g. "Advil (ibuprofen)")
- Selecting a result stores both the brand name and the generic name for use in subsequent queries
- Minimum 2 characters before autocomplete fires; debounced at 300ms

### 5.2 Slot Management
- Default state: 2 slots visible
- "+ Add Drug" button adds a 3rd, then 4th slot
- Each slot has an "×" remove button (minimum 1 slot must remain)
- Slots animate in smoothly on add

### 5.3 Preset Comparisons
Two preset buttons load drug pairs instantly:
- **Ibuprofen vs Acetaminophen** — common OTC pain relievers, familiar to most users
- **Ozempic vs Wegovy** — high public interest, same active ingredient (semaglutide) but different approvals

Preset buttons are labeled with plain language: *"Compare Ibuprofen vs Acetaminophen →"*

### 5.4 Search Submit
- "Explore" button triggers all API calls in parallel (one `Promise.all` per data section)
- Loading skeletons appear per drug card while fetching
- Results render progressively as each data section resolves

---

## 6. Results Dashboard

### 6.1 Drug Summary Cards
One card per drug, rendered in a responsive grid.

**Displays:**
- Brand name (large) + generic name (smaller, muted)
- Manufacturer name
- First approval year (from Drugs@FDA)
- Marketing status badge: `Prescription` / `OTC` / `Discontinued`
- Total FAERS adverse event report count with a `?` help button

**Help button copy (InfoModal):** *"This number is the total count of adverse event reports submitted to the FDA for this drug since 2004. A higher number does not mean a drug is more dangerous — widely used drugs naturally have more reports."*

### 6.2 Approval History Card
Compact card showing:
- Application type (NDA = new drug, ANDA = generic, BLA = biologic) with `?` help
- Approval date
- Sponsor name
- Available dosage forms

**Help button copy for "NDA/ANDA/BLA":** *"NDA (New Drug Application) is submitted for brand-name drugs. ANDA (Abbreviated New Drug Application) covers generics. BLA (Biologics License Application) is for biologic drugs like insulin or vaccines."*

### 6.3 Adverse Event Charts

All three chart types render side-by-side per drug using a consistent color palette (one distinct warm color per drug slot).

#### 6.3.1 Severity / Outcome Breakdown — Donut Chart
- One donut per drug
- Segments: Recovered, Recovering, Not Recovered, Fatal, Unknown
- Outcome codes mapped to plain English labels (see mapping table below)
- Center label shows total serious reports
- `?` help button: *"These outcomes are self-reported and not medically verified. 'Fatal' means the reporter indicated the patient died — it does not confirm the drug caused the death."*

**Outcome code mapping:**

| API Code | Display Label |
|---|---|
| 1 | Recovered / Resolved |
| 2 | Recovering |
| 3 | Not Recovered |
| 4 | Recovered with lasting effects |
| 5 | Fatal |
| 6 | Unknown |

#### 6.3.2 Demographic Breakdown — Bar Charts
Two sub-charts per drug:

**Sex breakdown** — simple horizontal bar or pie:
- Female / Male / Unknown

**Age group breakdown** — grouped bar chart:
- Neonate / Infant / Child / Adolescent / Adult / Elderly / Unknown
- `?` help button: *"Age and sex data comes from the adverse event report, which is often filled in by a healthcare provider or consumer. Many reports leave demographics blank, which is why 'Unknown' may be large."*

#### 6.3.3 Top Reactions — Horizontal Bar Chart
- Top 10 reactions by report count per drug
- Displayed side-by-side (or stacked on mobile)
- Reactions use MedDRA terms from the API, displayed as-is (medical terminology)
- `?` help button: *"These are MedDRA terms — a standardized medical vocabulary. One report can list many reactions. A reaction appearing often means it was frequently mentioned alongside this drug, not that the drug definitively caused it."*

### 6.4 Labeling Panel
Accordion or tab group with three sections per drug:

**Tab 1 — ⚠️ Black Box Warning**
- If `boxed_warning` field is present: render full text in a visually distinct amber/red callout box
- If absent: show "No black box warning found in current labeling."
- `?` help: *"A Black Box Warning is the FDA's most serious warning. It means clinical studies found the drug carries a significant risk of serious or life-threatening adverse effects."*

**Tab 2 — Warnings & Contraindications**
- Render `warnings_and_cautions` and `contraindications` text
- Long text truncated at ~400 characters with "Read more" expansion
- If absent: "Not found in current labeling."

**Tab 3 — Drug Interactions**
- Render `drug_interactions` text
- Truncated with "Read more"
- `?` help: *"This text comes from the official prescribing information submitted by the manufacturer. It may not reflect newly discovered interactions."*

---

## 7. Data Limitations Communication

### 7.1 Collapsible "About This Data" Section
Located below the results dashboard, above the footer. Collapsed by default. Clicking the header expands it.

**Content:**

**What is FAERS?**
The FDA Adverse Event Reporting System (FAERS) collects voluntary reports of adverse events from patients, healthcare providers, and manufacturers. Reports are submitted — not verified — meaning the FDA does not confirm a drug caused the event described.

**Why report counts can be misleading**
Popular drugs have more reports simply because more people take them. A drug with 100,000 reports is not necessarily more dangerous than one with 1,000 — it may just be more widely used.

**Data lag**
FAERS data is released quarterly and may lag by 3 or more months. Recent safety signals may not yet be reflected.

**Underreporting**
Adverse events are significantly underreported. Studies estimate only a fraction of real-world adverse events are ever submitted to the FDA.

**Correlation ≠ Causation**
A report listing a drug alongside a reaction does not mean the drug caused the reaction. The patient may have been taking many drugs, or the event may be unrelated.

**Labeling limitations**
Product labeling data reflects what the manufacturer submitted. It may be outdated, vary by formulation, or show labeling for a different dosage form than the user intended.

**Drugs@FDA coverage**
Approval data is most complete for drugs approved after 1998. Older approvals may have incomplete records.

---

## 8. Contextual Help Buttons

Every section that uses technical language or presents data that is easy to misinterpret includes a `?` button that opens a plain-English `InfoModal`.

**Full list of help buttons and their copy:**

| Location | Trigger Label | Modal Title | Plain-English Explanation |
|---|---|---|---|
| Total FAERS count | `?` | "What does this number mean?" | More reports ≠ more dangerous. Popular drugs have more reports. |
| Outcome donut | `?` | "About these outcomes" | Self-reported, not medically confirmed. Fatal ≠ drug caused death. |
| Demographics chart | `?` | "About demographics" | Often incomplete — many reports leave age/sex blank. |
| Top reactions chart | `?` | "What are these reactions?" | MedDRA terms. Listed alongside the drug, not proven to be caused by it. |
| Black box warning | `?` | "What is a Black Box Warning?" | FDA's most serious warning level. Indicates clinically significant risk. |
| Drug interactions tab | `?` | "About this interactions data" | From manufacturer's label; may not include newly discovered interactions. |
| NDA/ANDA/BLA badge | `?` | "What do these abbreviations mean?" | NDA = brand new drug. ANDA = generic. BLA = biologic. |
| Serious reports count | `?` | "What counts as 'serious'?" | FDA defines serious as: hospitalization, disability, life-threatening, or death. |

**InfoModal behavior:**
- Triggered by clicking `?` icon
- Appears as a small centered overlay modal with a soft drop shadow
- Dismissed by clicking outside, pressing Escape, or clicking "Got it"
- Accessible: modal traps focus, has `role="dialog"` and `aria-modal="true"`

---

## 9. Required Legal & Attribution Elements

### 9.1 Educational Disclaimer Banner
A non-dismissible banner fixed at the top of the page (or immediately below the header).

**Text:**
> ⚕️ **Educational Use Only.** This tool is for informational and educational purposes only. It is not a substitute for professional medical advice, diagnosis, or treatment. Always consult a qualified healthcare provider before making decisions about medications.

**Style:** Warm amber/yellow background, body-weight text, small but clearly visible. Does not scroll away.

### 9.2 OpenFDA Attribution
Required per OpenFDA terms of service. Displayed in the page footer.

**Text:**
> This tool uses the openFDA API but is not endorsed by or affiliated with the U.S. Food and Drug Administration. Data sourced from [openFDA](https://open.fda.gov). OpenFDA data is provided for informational purposes and has not been validated for clinical use.

**Footer also includes:**
- Link to openFDA terms of service: `https://open.fda.gov/terms/`
- Link to openFDA API documentation: `https://open.fda.gov/apis/`
- Statement: "Drug safety data from FDA Adverse Event Reporting System (FAERS), FDA Structured Product Labeling, and Drugs@FDA."

---

## 10. Visual Design

### 10.1 Design Principles
- **Warm & approachable** — the tool is for curious patients, not clinicians
- **Clarity over density** — limit information per screen; use progressive disclosure
- **Trust through transparency** — data limitations are surfaced, not hidden
- **Accessibility** — WCAG 2.1 AA target; sufficient contrast, keyboard navigable, screen-reader friendly

### 10.2 Color Palette

| Role | Color | Notes |
|---|---|---|
| Background | `#FDF6EE` | Warm off-white |
| Surface (cards) | `#FFFFFF` | Pure white for contrast |
| Primary accent | `#D97B4F` | Warm terracotta — CTAs, active states |
| Secondary accent | `#5B8E8C` | Muted teal — secondary actions |
| Warning / Black Box | `#B45309` | Amber-brown — serious but not alarming red |
| Danger / Fatal | `#C0392B` | Reserved for fatal outcomes only |
| Text primary | `#2D2D2D` | Near-black |
| Text muted | `#6B6B6B` | Labels, secondary info |
| Border | `#E8DDD0` | Warm light border |

### 10.3 Drug Slot Colors (for chart differentiation)
Up to 4 drugs, each gets a distinct color used consistently across all charts:

| Slot | Color |
|---|---|
| Drug 1 | `#D97B4F` (terracotta) |
| Drug 2 | `#5B8E8C` (teal) |
| Drug 3 | `#8B6DB5` (soft purple) |
| Drug 4 | `#D4A017` (warm gold) |

### 10.4 Typography
- Font: **Inter** (Google Fonts, free) — clean and highly legible
- Heading scale: 2rem / 1.5rem / 1.25rem / 1rem
- Body: 1rem / line-height 1.6
- Small/label: 0.85rem
- No font below 0.8rem

### 10.5 Layout
- Max content width: 1200px, centered
- Responsive breakpoints: 1200px (desktop) / 768px (tablet) / 480px (mobile)
- Drug cards: CSS Grid, `repeat(auto-fit, minmax(260px, 1fr))`
- Charts: flex row on desktop, stack on mobile

---

## 11. Technical Specification

### 11.1 Stack
- **Single HTML file** — all CSS and JS inline or in `<style>` / `<script>` tags
- No build tools, no npm, no backend
- Charts rendered with **Chart.js** (CDN) — lightweight, well-documented
- No other external dependencies except Google Fonts (Inter)

### 11.2 API Call Strategy
All calls are `fetch()` to `https://api.fda.gov`. Calls are made in parallel per drug using `Promise.all`. Results are cached in a simple in-memory object keyed by drug name to avoid redundant calls within a session.

**Per drug, on search submit:**
```
Promise.all([
  fetchAdverseEventCounts(drugName),   // 4–5 count queries
  fetchLabelData(drugName),            // 1 query
  fetchDrugsAtFDA(drugName),           // 1 query
])
```

### 11.3 Error Handling
| Scenario | Behavior |
|---|---|
| Drug not found in any endpoint | Show "No data found for '[drug]'" card with suggestion to try generic/brand name |
| API rate limit hit (429) | Show "Too many requests — please wait a moment and try again" |
| Network error | Show "Could not reach the FDA database. Check your connection." |
| Partial data (some endpoints succeed, some fail) | Show available data; display "Unavailable" placeholder for missing sections |

### 11.4 Accessibility
- All `?` help buttons have `aria-label="Help: [section name]"`
- All charts have `aria-label` describing the data shown
- Color is never the sole indicator of meaning (always paired with text labels)
- Modal focus trap on open; focus returns to trigger button on close
- Keyboard nav: Tab through all interactive elements; Enter/Space activate buttons

### 11.5 Performance
- All API calls debounced and parallelized
- Charts render only when their section scrolls into view (IntersectionObserver)
- Autocomplete debounced at 300ms
- No images; all visuals are SVG or Canvas

---

## 12. Out of Scope (v1.0)

The following are intentionally excluded from this version:

- User accounts or saved searches
- PDF/export of results
- Timeline / trend charts (report volume over years) — data-heavy, deferred to v2
- NDC-level drug disambiguation (e.g. distinguishing 200mg vs 400mg ibuprofen)
- Drug shortage data
- Recall enforcement data
- Mobile app / PWA packaging
- Any backend or server component

---

## 13. Open Questions for Review

Before build begins, confirm the following decisions:

1. **API key**: Will this tool use an OpenFDA API key (for higher rate limits), or rely on the public unauthenticated limit (40 req/min)? If yes, where is the key stored (hardcoded constant, env var prompt)?
2. **Drug name disambiguation**: If a user searches "aspirin," the API may return many brand products. Should the app always use the first result, or show a disambiguation step?
3. **Labeling data currency**: The label endpoint may return multiple labeling documents for one drug. Should the app always use the most recently updated label?
4. **MedDRA reaction terms**: These are medical Latin terms (e.g. "nausea," "dyspnoea"). Should the app attempt to translate any to plain English, or leave them as-is?
5. **Black box warning absent state**: Should "no black box warning found" be shown as a positive reassurance, or is that misleading given labeling data gaps?
