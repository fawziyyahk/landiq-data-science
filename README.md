# LandIQ — Data Science

> Soil health intelligence for Nigerian agricultural land  
> Women Techsters Fellowship 2026 · Group 32 · Paragon Squad · SDG 15: Life on Land

---

## What This Repo Contains

This is the complete data science work behind LandIQ — a Flutter mobile app that gives farmers and land buyers an instant soil health score for any GPS location in Nigeria. This repo covers everything from raw dataset exploration to the scoring algorithm, geospatial lookup system, and automated explanation engine that powers the app.

LandIQ was built as a capstone project under the Women Techsters Fellowship 2026 (Data Science and Engineering track).

---

## The Problem We Solved

Nigerian farmers and land buyers have no easy way to assess soil quality before investing in land. The Nigerian Soil Survey (2017) contains detailed soil data for 658 mapping units across the entire country — but it is buried in shapefiles, written in agronomist language, and completely inaccessible to the average person.

We turned that dataset into a system that returns a score, a badge (Gold / Silver / Bronze), a degradation risk level, and a plain-English explanation — in response to any GPS coordinate inside Nigeria.

---

## The Dataset

| Property | Value |
|---|---|
| Source | Nigerian Soil Survey (2017) |
| Total Records | 658 soil mapping units |
| Coverage | All of Nigeria (national) |
| Geographic Bounds | Longitude: 2.45 – 14.77 · Latitude: 4.39 – 13.93 |
| File Format | ESRI Shapefile (.shp + companion files) |
| Original State | Unlabeled — no scores, no badges, no risk levels |

Key fields used: `SUITABILIT`, `SOIL_PH`, `DRAINAGE`, `SLOPE`, `Depth`, `ECOLOGICAL`, `MAJOR_CROP`

---

## What We Built

### 1. Data Exploration → `data_summary.json`
Full statistical and geographic profile of the dataset. Confirmed national coverage, identified that all key fields are stored as **text ranges** (e.g. `5.5 - 7.0`, `0 - 6%`) requiring custom parsing logic, and extracted the geographic bounds used by backend for coordinate validation.

### 2. Scoring Algorithm → `soil_health_scores.csv` + `scoring_summary.json`
The core intelligence of LandIQ. Converts qualitative soil data into a score out of 100.

| Indicator | Max Points | Weight | Scientific Basis |
|---|---|---|---|
| Suitability | 40 pts | 40% | FAO primary land evaluation criterion |
| pH Level | 20 pts | 20% | IITA: primary yield-limiting factor in Nigeria |
| Drainage | 20 pts | 20% | IITA: primary yield-limiting factor in Nigeria |
| Slope | 10 pts | 10% | Long-term sustainability — gradual erosion impact |
| Soil Depth | 10 pts | 10% | Root development — partially manageable |

**Calibration:** The initial algorithm produced only 11 Gold zones (1.7%) — below the FAO-expected 10–20% for S1-class land. Root cause was pH thresholds calibrated for temperate crops (wheat, potatoes). We recalibrated to IITA guidelines for tropical Nigerian crops (maize, yam, cassava, sorghum), yielding 87 Gold zones (13.2%) — correctly within FAO benchmarks.

### 3. Badge System
Scores map to the internationally recognized FAO Land Suitability Classes:

| Badge | Score Range | FAO Class | Count |
|---|---|---|---|
| 🥇 Gold | 70 – 100 | S1: Highly Suitable | 305 zones |
| 🥈 Silver | 45 – 69 | S2: Moderately Suitable | 323 zones |
| 🥉 Bronze | 0 – 44 | S3/N: Marginally/Not Suitable | 30 zones |

### 4. Degradation Risk Assessment
A second analytical layer that answers: *is this land at risk of getting worse over time?*

Risk is calculated by counting how many of the following factors are present:
- Steep slope (slope score < 7 pts)
- Poor drainage (drainage score < 12 pts)
- pH stress (pH score < 15 pts)
- Shallow depth (depth score < 6 pts)

| Risk Factors Present | Risk Level |
|---|---|
| 0 – 1 | LOW |
| 2 | MEDIUM |
| 3 or more | HIGH |

A zone can be Silver badge (good soil) but HIGH risk (degrading). That distinction is critical intelligence for long-term land investment.

### 5. Geospatial Lookup System → `test_results.json`
Point-in-polygon operation: given any GPS coordinate inside Nigeria, identifies which of the 658 soil polygons contains that point and returns the full soil assessment. This is the function the mobile app calls when a user taps *Check This Land*.

**Sample test results:**

| Location | Latitude | Longitude | Badge | Score | Risk |
|---|---|---|---|---|---|
| Abuja (FCT) | 9.0820 | 7.5324 | Silver | 48 | HIGH |
| Kano | 11.9964 | 8.5211 | Silver | 68 | MEDIUM |

### 6. Comparison Feature → `test_results_mvp.json`
Ranks 2–3 farm locations and recommends the best one. Backend designed their comparison API endpoint directly from this output.

### 7. Explanation Generation System
Automatically generates plain-English explanations for every soil assessment — no soil science degree required. Rule-based, three-layer system:

- **Layer 1 — Badge Introduction:** Sets tone from score and badge
- **Layer 2 — Factor Insights:** Adds detail based on actual indicator values (e.g. drainage, slope, depth findings)
- **Layer 3 — Risk Summary:** Lists degradation risk factors in plain language

This system has **zero cost, zero latency, and zero external dependency**. No AI API needed for explanations at MVP.

> *Example output: "Good soil health (Score: 67/100). This land can support reliable crop production with proper management. Poor drainage may require water management systems. Deep soil allows good root development. Gentle slope means low erosion risk."*

---

## Output Files Delivered to Backend

Every file below was created by the data science team. Backend built their entire system using these as the foundation.

| File | Contents | Backend Use |
|---|---|---|
| `soil_types_labeled.shp` | All 658 zones with badge, score, and risk as new columns | GeoJSON conversion and database |
| `soil_health_scores.csv` | Every zone mapped to badge, score, risk, and all soil properties | DB schema design |
| `data_summary.json` | Geographic bounds, record count, distribution statistics | Coordinate validation logic |
| `scoring_summary.json` | Score statistics (mean: 70.5, median: 69, range: 37–100), badge distribution: 305 Gold / 323 Silver / 30 Bronze | TINYINT data type choice |
| `test_results.json` | Location lookup results for Abuja, Lagos, Kano | API response structure |
| `test_results_mvp.json` | MVP tests including comparison feature | Comparison endpoint design |

---

## Why Rule-Based, Not Machine Learning

| Reason | Detail |
|---|---|
| Insufficient data | ML requires 5,000–10,000 labeled examples minimum. We have 658 zones. |
| No ground truth labels | We had no pre-validated soil quality scores to train on — ML would have been circular. |
| Interpretability | A farmer asks *"why did my land score 42?"* — a neural network cannot answer that. Our system explains every point. |
| Timeline | Properly building, training, and validating an ML model takes months. We had weeks. |

**Rule-based is not the fallback — it is the correct choice for this problem.**

Our system is also **ML-ready**: when larger labeled datasets become available, the architecture supports adding an ML layer without rebuilding from scratch.

Rule-based expert systems are classical AI (symbolic AI). This system captures FAO agronomist knowledge, applies weighted rules to evaluate soil properties, makes decisions, generates interpretable explanations, and handles 658 scenarios — that is an expert system by definition.

---

## Repo Structure

```
landiq-data-science/
├── data/
│   ├── raw/                  # Original Nigerian Soil Survey shapefile
│   ├── processed/            # Cleaned and parsed dataset
│   └── outputs/              # All files delivered to backend
│       ├── soil_types_labeled.shp
│       ├── soil_health_scores.csv
│       ├── data_summary.json
│       ├── scoring_summary.json
│       ├── test_results.json
│       └── test_results_mvp.json
├── notebooks/                # Exploration and analysis notebooks
├── src/                      # Core source code
│   ├── scoring_algorithm.py  # Weighted scoring logic
│   ├── geospatial_lookup.py  # Point-in-polygon lookup
│   ├── explanation_engine.py # Plain-English explanation generator
│   └── risk_assessment.py    # Degradation risk calculator
├── tests/                    # Test cases and validation
├── docs/
│   └── technical_documentation.pdf
└── README.md
```

---

## Project Context

| Detail | Value |
|---|---|
| Programme | Women Techsters Fellowship 2026 |
| Track | Data Science and Engineering |
| Group | 32 — Paragon Squad |
| SDG Alignment | SDG 15: Life on Land |
| App | LandIQ (Flutter mobile) |

---

## Related Repos

- [`landiq-app`](https://github.com/Eso3011) — Main project repo (frontend, backend, all tracks)
