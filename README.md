# Data-Sources Catalog — Pan-India Land & Agricultural Intelligence

> **Purpose.** The authoritative reference for *where every input layer comes from*, its
> access model, its trust class, and its refresh cadence. Every `/spec-design` for an overlay,
> records, crop, soil, water, weather, or valuation feature must pull from this catalog.
> Faithfully ported from *Pan-India Agricultural Land Intelligence: A Data-Sourcing Roadmap*
> and cross-referenced to `docs/pm/land-intelligence-epic.md` and `LANDEED_STRATEGY.md`.
>
> **Compiled from source docs; figures carry the dates the sources gave (2023–2025) — refresh
> against dilrmp.gov.in / agristack.gov.in before publication.**

## The core thesis
The authoritative core of a pan-India stack rests on **three government "digital public
infrastructure" systems** — supplemented by open global datasets and, critically, Google
DeepMind's free ALU/AMED APIs:
1. **DILRMP / Bhu-Naksha** — cadastre + ownership.
2. **AgriStack** — farmer/crop registries.
3. **ISRO Bhuvan / Bhoonidhi / Krishi-DSS** — earth observation.
Supplements: Sentinel-2, ERA5, SoilGrids, VIIRS; and **Google DeepMind ALU/AMED**.
**No single source is complete or fully accurate — the differentiation is fusion, validation,
and ground-truthing.**

## Legend
- **Access:** 🆓 free/open · 🤝 partnership/MoU-gated · 💰 priced/commercial · 🔌 aggregator API (buy) · 🚫 no bulk API (scrape-risk)
- **Class (provenance):** 🟩 authoritative · 🟨 derived/modeled (label as estimate, carry confidence) · 🟦 authoritative-but-stale
- **Cadence:** how often the source refreshes (drives the `as_of` + refresh field in `provenance-confidence`).

---

## LAYER 1 — Land Records / Ownership / Cadastral Polygons

| Source | What it provides | Access | Class | Cadence / Caveat |
|---|---|---|---|---|
| **DILRMP** (Dept. of Land Resources, dilrmp.gov.in) | Central scheme; RoR computerised 6.25M/6.57M villages (95.09%, 31 Dec 2023; PIB 2 Aug 2025 says 99.79% textual). Cadastral maps digitised >68% in 28 states/UTs. **Geo-referencing complete for only 3,26,776/6,57,397 villages = 49.10%** | 🤝 | 🟦 | Heterogeneous; the textual-vs-spatial integration gap is **the core accuracy problem** |
| **ULPIN / Bhu-Aadhaar** | 14-digit parcel ID from parcel-vertex geo-coords; ECCMA+OGC compliant; NIC BhuNaksha | 🤝 | 🟩 | Rolled out 29 states, ~50% parcels assigned; UP/Bihar/Assam lag. **Adopt as primary join key, survey-no fallback** |
| **Bhu-Naksha** (bhunaksha.nic.in) | NIC per-state cadastral mapping; parcel polygons + survey numbers | 🚫 (per-state, per-plot) | 🟩 | Tracks the 49.10% georeferencing figure; no clean national bulk geometry API — fragmentation is the moat |
| **State Bhulekh / RoR portals** | Textual RoR (7/12, 8A, Patta, Khatauni, RTC, etc.) — see per-state list below | 🚫 captcha/rate-limited, one record at a time | 🟦 | **Online RoRs are not proof of title** (Supreme Court). Bulk access = state MoU |
| **NGDRS** | Registration / deed data | 🤝 | 🟦 | Registration record integration >75% |
| **SVAMITVA** | Survey of India drone survey of rural *abadi* (inhabited) at 1:500; property cards (e.g. all 6,250 Lal Dora villages in Haryana, 25.16M cards by Apr 2025) | 🤝 | 🟩 | Covers **habitation, not agricultural land** — useful for village-core built-up only |
| **FMB (Field Measurement Book)** | Cadastral field maps | 🤝 | 🟩 | Being digitised under DILRMP |

**Per-state RoR portals (all NIC-built, textual; most link to Bhu-Naksha for maps):**
Maharashtra — Mahabhulekh (bhulekh.mahabhumi.gov.in; 7/12, 8A, Property Card) · Karnataka — Bhoomi (RTC) ·
UP — upbhulekh.gov.in (Khatauni) · Rajasthan — Apna Khata / e-Dharti · MP — MP Bhulekh · Telangana — Dharani (now Bhu Bharati) ·
AP — Meebhoomi/Webland · TN — Patta Chitta / eServices TN · Gujarat — AnyRoR / e-Dharti (VF6/7/8A) · Bihar — Jamabandi ·
Haryana & Punjab — Jamabandi (jamabandi.nic.in / PLRS) · WB — Banglarbhumi · Odisha — Bhulekh Odisha · Jharkhand — Jharbhoomi ·
Kerala — e-Rekha / ReLIS / Ente Bhoomi · Uttarakhand — Devbhoomi · Chhattisgarh — Bhuiyan · Assam — Dharitree / ILRMS ·
HP — HP Bhulekh; plus Sikkim, Tripura (Jami), Manipur, Meghalaya, Nagaland, Arunachal, Goa, J&K, Delhi (via DDA/revenue).

**Georeferenced/digitised cadastre (better):** Maharashtra, MP, Gujarat, Odisha, Telangana, AP, Karnataka, Rajasthan, Haryana.
**Textual-mostly / lagging spatial:** UP, Bihar, Assam, parts of NE.

---

## LAYER 2 — Crop Identification & Yield

**Government / statistical (authoritative):**
| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **DES — APY** (Area–Production–Yield) | District/state, historical from 1966 | 🆓 | 🟩 | Coherence issues between district & state aggregations |
| **CCE / GCES** (Crop Cutting Experiments) | Ground-truth yield via sample harvests; basis of PMFBY | 🤝 | 🟩 | The yield ground truth; systematic CCE-vs-APY deviations documented |
| **E-Peek Pahani / Digital Crop Survey (DCS)** under AgriStack | Geo-fenced, smartphone plot-level crop capture | 🤝 | 🟩 | |
| **AgriStack / IDEA** | 3 registries (Farmer, Geo-referenced Village Maps, Crop Sown) via **UFSI open API gateway** + Consent Manager + Sandbox | 🤝 (consent-brokered) | 🟩 | 7.63cr Farmer IDs (27 Nov 2025) vs 11cr target by 2026-27; 22 states have land-record verification APIs. **Highest-value gov crop dataset but partnership-gated** |

**Satellite / remote sensing:**
| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **ISRO Bhuvan / NRSC** | FASAL (forecasting), CHAMAN (horticulture, 40 districts), KISAN; Bhuvan API (bhuvan-app1.nrsc.gov.in/api) with token | 🆓 (token) | 🟨 | **View-only licence; redistribution prohibited without NRSC authorisation** |
| **Sentinel-2 (Copernicus)** | Free, 10m, 5-day; workhorse for crop classification | 🆓 | 🟨 | 85–95% accuracy fused with Sentinel-1 SAR; drops for <600 m² plots & cloudy Kharif |
| **Landsat 8/9** (30m) · **PlanetScope** (3m, best for tiny fields) · **Google Dynamic World** (10m NRT LULC) · **ESA WorldCereal** | Optical / LULC | 🆓 / 💰 (Planet) | 🟨 | |
| **Google DeepMind ALU/AMED** | Standout free national crop-type / field-boundary source | 🤝 (allowlist) | 🟨 | See cross-cutting section |

**Accuracy note:** all satellite crop-type/yield outputs are **modeled** and season-dependent (Rabi far more
accurate than Kharif). Biomass methodology (IPCC linear model, harvest index, RPR, CCE→APY→national fallback)
is sound but must carry confidence flags.

---

## LAYER 3 — Soil, Groundwater, Water Resources

**Soil:**
| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **SoilGrids 2.0 (ISRIC)** | Global 250m, ML-modeled, 6 depth intervals, CC-BY 4.0; WCS/WebDAV/GEE | 🆓 | 🟨 | ISRIC REST API intermittently paused — use WCS or GEE. **Modeled, coarse for parcel-level** |
| **NBSS&LUP BHOOMI Geoportal** (bhoomigeoportal-nbsslup.in) | ICAR national soil maps; nodal for "Soil" theme; building UNSIS | 🆓 | 🟩 | **Most authoritative Indian soil source** |
| **Soil Health Card scheme** | Plot-level soil test data since 2015 | 🆓 | 🟨 | Uneven data quality & adoption |
| **FAO soil data** | Supplementary | 🆓 | 🟨 | |

**Groundwater:**
| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **CGWB / India-WRIS** (nwic.gov.in, indiawris.gov.in) | Station-wise levels (~23,000+ wells → piezometers w/ telemetry), quality (85 params), In-GRES (extraction/recharge/categorisation), NAQUIM aquifer mapping (14 principal/42 major) | 🆓 (viewer/download; community GitHub scrapers exist) | 🟦 | **Authoritative but point-based & quarterly — interpolation required for parcel-level** |

**Surface water / water bodies:** NRSC water bodies on Bhuvan · JRC Global Surface Water (🆓) ·
1st Census of Water Bodies (~24,24,540 water bodies, via Jal Dharohar on India-WRIS).

**Rainfall / soil moisture:** NRSC soil moisture (ScatSat) · ISRO GSMaP_ISRO rain (MOSDAC) · Sentinel-1-derived
soil moisture (also a SatSure commercial product).

---

## LAYER 4 — Weather / Climate / Early-Warning

| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **IMD** | Gridded daily rainfall (0.25°, 6,955 stations) + temp (1°); via open-source **IMDLIB** Python lib (binary→NetCDF/GeoTIFF/CSV) | 🆓 | 🟩 | **Authoritative Indian ground-station product** |
| **NASA POWER** | Free global agroclimatology API (solar, temp, precip, ET) | 🆓 | 🟨 | Already wired (solar) |
| **ERA5 / ERA5-Land (Copernicus)** | Reanalysis 0.25°/0.1°, 1940/1950–present, CC-BY 4.0 | 🆓 | 🟨 | Best for climate baselines & ML training, not day-to-day accuracy |
| **Open-Meteo** | Free JSON API wrapping ERA5; historical + 16-day forecast + CMIP6 (1950–2050, 10km); no auth, self-hostable | 🆓 | 🟨 | Excellent for reference ET, dry-spell, heatwave derivations |
| **State agromet** | Mahavedha (Maharashtra AWS), other state AWS | 🆓/🤝 | 🟩 | |
| **Early warning** | IMD; NRSC/Bhuvan disaster services (NADAMS drought, flood inundation, forest fire); MOSDAC; FASAL; **Krishi-DSS** (16 Aug 2024 — integrates RISAT-1A, MOSDAC, Bhuvan, VEDAS, soil, weather, reservoir, groundwater) | 🤝 | 🟨 | Krishi-DSS external developer/API access currently limited (gov-channel dashboards) |

---

## LAYER 5 — Valuation / Comparable Listings / Market Prices

| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **Circle rates / guideline values** (state IGR depts) | Karnataka Guidance Value (Kaveri), TN Guideline (TNREGINET), Maharashtra Ready Reckoner (revised ~annually, eff. 1 Apr), Delhi/UP/Haryana/Punjab Circle/Collector Rate, Telangana & AP "Market Value", Kerala Fair Value, WB Market Value | 🚫/🆓 viewer | 🟦 | Revisions irregular & politically timed. **Floor prices for stamp duty, NOT market prices — treat as lower bound** |
| **Agmarknet** (agmarknet.gov.in) | Daily min/max/modal wholesale prices + arrivals, 3,000+ mandis, 200+ commodities; via **data.gov.in API** (NDSAP); Ashoka CEDA cleaned Census-2011-mapped version | 🆓 | 🟩 | |
| **eNAM** | National Agriculture Market integrated mandi platform | 🆓 | 🟩 | |
| **FCI/CACP MSP** | Support prices | 🆓 | 🟩 | |
| **Listings / comparables** — SFarms.in (agri-land), 99acres, Magicbricks, OLX | Asking prices, self-reported | 💰/🚫 | 🟨 | **Noisy — directional only, heavy outlier filtering** |
| **Registration/stamp-duty transaction data** | State IGR portals + encumbrance certificates — closest to actual transacted prices | 🚫 (aggregators scrape) | 🟦 | Not exposed via clean APIs |

**Valuation triangulation:** circle rate (floor) + registered-transaction comparables + listing asking-prices
(ceiling), with outlier removal.

---

## LAYER 6 — Infrastructure / Demographics / Commercial Activity

| Source | What | Access | Class | Caveat |
|---|---|---|---|---|
| **Census of India 2011** | Village-level demographics, literacy, housing, amenities (village = 16-digit census code, joinable to SoI village polygons) | 🆓 | 🟦 | **15 years stale; next census pending** |
| **Nighttime lights** — NOAA VIIRS DNB (500m, monthly/annual, 2012–present) · DMSP-OLS (1992–2013) · NASA Black Marble | Economic-activity / electrification proxy | 🆓 | 🟨 | Already ingest district-level VIIRS 2012–2024 |
| **LULC / built-up** — ESA WorldCover (10m) · Bhuvan LULC (NRSC, 23.5m, 5-yearly) · Google Dynamic World (10m) · GHSL | Land cover | 🆓 | 🟨 | |
| **Population** — WorldPop (100m/1km gridded, modeled from Census 2011) | Population grid | 🆓 | 🟨 | Uses 1km 2020 layer |
| **Roads / POI** — OpenStreetMap (ODbL, free — schools, roads, POIs) · Google Maps/Places (paid, richest commercial POI) · NHAI, PMGSY rural roads | Roads/frontage/access | 🆓/💰 | 🟨 | OSM enables road-frontage compute |
| **Commercial / economic** — Udyam MSME registration, GST (aggregate), Livestock Census, data.gov.in socioeconomic | Economic activity | 🆓 | 🟨 | |

---

## Cross-cutting — Google DeepMind ALU & AMED (Terrastack's key asset; free, national, partner-allowlisted)

| API | What | Access | Class | Notes |
|---|---|---|---|---|
| **ALU** (Agricultural Landscape Understanding) | Field/water/vegetation boundary segmentation from 30–50cm imagery; features at 1m; refreshed ~6 months | 🤝 allowlist (GCP project + billing + Workspace ID); output GeoJSON at S2 cell 13 (~1km) | 🟨 | 0.68 mean IoU for fields; 82% accuracy in a 19-village Telangana pilot (under/over-segmentation noted). Free, **no rate limit** |
| **AMED** (Agricultural Monitoring & Event Detection) | Field-level crop type, season, size, sowing/harvest dates + 3-yr history; Sentinel-1/2 + Landsat (10–30m); ~15-day refresh; 12 crops | 🤝 allowlist | 🟨 | 94% agreement Rabi, **75% Kharif** — season-flag Kharif as lower-confidence |

Used to detect **farm boundaries, encroachment and ownership changes**; being integrated into Krishi-DSS.

**Commercial geospatial / agri vendors (buy-vs-build reference):**
- **SatSure** (Bengaluru/St. Gallen) — Sparta (crop lifecycle), Sage (fintech/agri-loan underwriting & portfolio), India Cropland Data Layer, farm-level soil moisture. API + AWS Marketplace; works with banks/insurers/state agri since 2017. **Strongest India lending-focused EO vendor; likely closest competitor / buy-vs-build reference.**
- **Cropin** — full-stack agtech SaaS (crop health, disease, advisory, supply chain).
- **Farmonaut** — affordable SaaS (₹200–₹5,578/mo), API/app/WhatsApp; also an Agmarknet API wrapper.
- **Satyukt** (Bengaluru) — satellite crop/soil analytics. **RMSI** — large geospatial services firm.
- **Digital Green, DeHaat, Ninjacart, Stellapps, AgroStar** — agtech, more advisory/supply-chain than land intelligence.

**Land-record aggregator APIs (lending/KYC):**
- **Surepass** — Land Record Check API (state-specific: Karnataka, Telangana, Gujarat, MP…) + **Land Record Monitoring API**. *(Primary records path — see `docs/pm/surepass-records.md`.)*
- **Signzy, Karza (Perfios), Gridlines, Zoop** — KYC/verification marketplaces; some expose land-record checks.
- **Landeed, mypatta** — consumer/B2B document-search (EC, RTC, AnyROR, 7/12, Patta Chitta, FMB, 8A).
- **Setu** — fintech APIs (Aadhaar/PAN/bank/DigiLocker KYC), not land records per se.
- **API Setu / apisetu.gov.in** (MeitY) — official gov API gateway (DigiLocker-backed docs); directory has a "Land" tag.
- **Reliability caveat:** aggregators largely scrape/screen-scrape state portals; freshness, uptime & legal standing vary. Convenient but **derivative — validate against the source portal for high-value decisions.**

---

## Access models — free vs partnership-gated vs priced
- **🆓 Freely available:** Bhuvan (view-only), Bhoonidhi open EO (5m+ IRS, Sentinel, Landsat — free under Indian Space Policy 2023; API released, contact bhoonidhi@nrsc.gov.in), Agmarknet/data.gov.in, Census 2011, India-WRIS/CGWB viewers, SoilGrids, IMD gridded (via IMDLIB), state Bhulekh single-record views.
- **💰 Priced:** High-resolution IRS/commercial satellite via Bhoonidhi/NSIL (per NRSC schedule).
- **🤝 Partnership/MoU-gated:** AgriStack UFSI API, bulk state land-record access (Maharashtra DoLR route), Krishi-DSS, ISRO/NRSC bulk data-sharing.

---

## Accuracy validation & ground-truthing — best practices (mandatory discipline)
1. **Label every layer authoritative vs derived/modeled** in UI + API payload; carry per-field confidence scores. *(→ `provenance-confidence`.)*
2. **Cross-validate ownership** by aligning cadastral polygons (Bhu-Naksha/ALU) against satellite imagery + RoR text; **flag mismatches rather than assert**. *(→ `area-mismatch-flag`.)*
3. **Anchor crop/yield** to CCE/APY & AgriStack DCS; use satellite (AMED/Sentinel) as the spatial disaggregator, not sole source; season-flag Kharif as lower-confidence.
4. **Triangulate valuation:** circle rate (floor) + registered-transaction comparables + listing asking-prices (ceiling), outlier-removed.
5. **Field ground-truth loop:** surveyor mobile app + geo-fenced photos (mirroring AgriStack DCS) → proprietary validation set; report IoU / overall accuracy / MAE per state per season.
6. **Version & date-stamp every source** (cadence ranges: daily [Agmarknet, weather] → 15-day [AMED] → 6-monthly [ALU] → 5-yearly [Bhuvan LULC] → 15-yearly [Census]).

---

## Staged acquisition plan (from the PDF "Recommendations")
- **Stage 1 (now — deepen the moat where data is best):** consolidate free/partner assets already touched —
  ALU/AMED (field boundaries + 12 crops), Bhoonidhi/Sentinel (imagery), IMD/Open-Meteo/ERA5 (climate),
  Agmarknet (prices), SoilGrids/NBSS&LUP (soil), CGWB/India-WRIS (groundwater), VIIRS + Census 2011 + OSM
  (infra/demographics). Use the Maharashtra DoLR MoU as the template for state land-record bulk access.
  **Prioritise states with georeferenced cadastre AND AgriStack coverage: Maharashtra, MP, Gujarat, Telangana, Karnataka, Rajasthan.**
- **Stage 2 (partnership push):** apply for **AgriStack UFSI API** access (Farmer/Crop Sown/Village Map) and
  **Bhoonidhi API** credentials; pursue Krishi-DSS integration once external access opens. Converts modeled
  layers into authoritative ones for lending use cases.
- **Stage 3 (fill gaps commercially):** where gov cadastre/crop data is weak, license **SatSure** (India Cropland
  Data Layer, soil moisture, Sage) or **PlanetScope** (3m, smallholder plots) rather than build. Use land-record
  aggregators (Surepass/Landeed) only as a stop-gap where no state MoU exists, clearly flagged as derivative.

**Thresholds that change the plan:**
- AgriStack UFSI access granted → make Farmer ID/ULPIN the primary key, demote scraped RoR.
- Kharif satellite crop accuracy stays <80% for a state → require CCE/AgriStack corroboration before publishing crop labels there.
- A state's cadastre georeferencing crosses ~90% → promote it from "textual/estimated" to "polygon-authoritative" tier.
- Census 2026 releases → replace WorldPop/Census-2011 demographic layers.

---

## Caveats (carry into every spec)
- Terrastack's "200+ data sources" claim is marketing — treat as positioning, not verified count.
- **Online RoRs are not legal title** (Supreme Court) — never present ownership data as conclusive.
- **State portals have no sanctioned bulk API** — scraping carries legal/ToS + reliability risk; MoUs are the durable path.
- All crop-type/yield/soil/valuation/climate-risk layers are **modeled/derived** and season/resolution-sensitive; the **49.10% cadastre-georeferencing gap** and **2011 census staleness** are the two largest accuracy limiters.
- Some figures (DILRMP %, AgriStack coverage) are 2023–2025 gov/press — refresh against dilrmp.gov.in / agristack.gov.in before publication (note 31 Dec 2023 RoR 95.09% vs PIB 2 Aug 2025 99.79%).
- **No published India-wide ALU field-boundary total** — do NOT attribute the "3.17 billion polygons" figure (a Microsoft global dataset) to Google/ALU.

---

## AGTMap status map (what we have vs buy vs compute vs acquire — from LANDEED_STRATEGY §8)
Legend: ✅ have (wired) · 🔌 buy (records API) · 🧮 compute (derive) · 🟡 acquire (ingest new).

| Layer | Source | Status |
|---|---|---|
| Parcel geometry + area + centroid + district/taluka/village/survey | cadastral tables, PostGIS | ✅ have (Gujarat) |
| Owner, tenure, EC, mutation | SurePass Gujarat API | 🔌 buy |
| Area mismatch (record vs polygon) | SurePass area ↔ ST_Area | 🧮 *moat* |
| Encroachment | buildings ∩ parcel | ✅ have *(moat)* |
| Solar / agri suitability | NASA POWER + PVGIS | ✅ have |
| Soil | punjab_soil_health (SLUSI) | ✅ Punjab · 🟡 else (SoilGrids/NBSS&LUP) |
| Slope / topography | punjab_contours / DEM | ✅ Punjab · 🟡 else |
| Forest type | punjab_forest_type | ✅ Punjab · 🟡 else |
| Zoning / land-use / master plan | — | 🟡 acquire (TCPO/state; some Bhuvan) |
| CRZ / defense / heritage | — | 🟡 acquire (Parivesh/Bhuvan) |
| Flood hazard | — | 🟡 acquire (Bhuvan/NRSC NADAMS) |
| Roads / frontage | — (have buildings) | 🟡 acquire (OSM) |
| Groundwater | — | 🟡 acquire (CGWB/India-WRIS) |
| Crop type / yield | — | 🟡 acquire (Google ALU/AMED, Sentinel, CCE/APY) |
| Litigation nearby | eCourts / Tavily | 🟡 / 🧮 (partially wired) |
| Comparable prices | — | 🟡 acquire (IGR registry / circle rate / Agmarknet) |

**Priority datasets that unlock the full national dossier:** 1) land-use/master-plan zoning · 2) flood hazard ·
3) roads (OSM) · 4) CRZ + national forest/defense · 5) DEM (SRTM/Bhuvan → slope outside Punjab).

---
*See also:* `docs/pm/land-intelligence-epic.md` (roadmap) · `LANDEED_STRATEGY.md` (product/GTM) ·
per-feature briefs in `docs/pm/` consume specific rows of this catalog in their `/spec-design` step.
